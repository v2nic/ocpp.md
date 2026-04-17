# OCPP 2.1 — Core Message Flows

> **Purpose:** Precise step-by-step message flows for boot, authorization, transaction lifecycles, V2X, DER control, and battery swapping. Optimized for AI agent consumption — uses structured sequences and explicit rules rather than visual diagrams.

> **Last updated:** 2026-04-16

---

## How This Document Was Produced

This document covers core OCPP 2.1 message flows. Its dominant confidence tier is **spec-knowledge** — behavioral sequences from the OCPP 2.1 specification Part 2. Field names, enum values, and type references are **schema-derived** (cross-referenced against the [schema documentation](../OCPP-2.1-Schemas/) and [data types reference](../OCPP-2.1-DataTypes.md), mechanically extracted from the official OCA JSON schemas). New 2.1 flows (V2X, DER, Battery Swapping) are based on the OCPP 2.1 Edition 2 specification.

This document contains **4 escalation points** marked with `> **ESCALATE:**`. When an AI agent encounters one, it MUST stop and ask the developer to make the decision. See [METHODOLOGY.md](../METHODOLOGY.md) for the full confidence and escalation model.

**Companion documents:**
- [Provisioning Schemas](../OCPP-2.1-Schemas/OCPP-2.1-Schemas-Provisioning.md) — BootNotification, Heartbeat, StatusNotification field-level details
- [Authorization Schemas](../OCPP-2.1-Schemas/OCPP-2.1-Schemas-Authorization.md) — Authorize, SendLocalList, GetLocalListVersion field-level details
- [Transaction Schemas](../OCPP-2.1-Schemas/OCPP-2.1-Schemas-Transactions.md) — TransactionEvent, RequestStartTransaction, RequestStopTransaction field-level details
- [Bidirectional Charging Schemas](../OCPP-2.1-Schemas/OCPP-2.1-Schemas-BidirectionalCharging.md) — NotifyAllowedEnergyTransfer
- [DER Control Schemas](../OCPP-2.1-Schemas/OCPP-2.1-Schemas-DERControl.md) — SetDERControl, GetDERControl, etc.
- [Battery Swapping Schemas](../OCPP-2.1-Schemas/OCPP-2.1-Schemas-BatterySwapping.md) — BatterySwap, RequestBatterySwap
- [Data Types Reference](../OCPP-2.1-DataTypes.md) — IdTokenType, IdTokenInfoType, MeterValueType, EVSEType

---

## 1. Boot Sequence

When a Charging Station powers on or resets, it must register with the CSMS before doing anything else. This is identical to OCPP 2.0.1.

For field schemas: [BootNotification](../OCPP-2.1-Schemas/OCPP-2.1-Schemas-Provisioning.md#bootnotification), [StatusNotification](../OCPP-2.1-Schemas/OCPP-2.1-Schemas-Provisioning.md#statusnotification), [Heartbeat](../OCPP-2.1-Schemas/OCPP-2.1-Schemas-Provisioning.md#heartbeat).

### 1.1 Boot Flow Steps

1. CS opens WebSocket to `wss://csms.example.com/ocpp/{cs_id}` (sub-protocol `ocpp2.1`).
2. CS sends `BootNotification` (CS→CSMS) with `reason` and `chargingStation` (model, vendorName).
3. CSMS responds with `BootNotificationResponse` containing `status`, `interval`, and `currentTime`.
4. CS behavior depends on `status`:

**If `Accepted`:**
- CS syncs clock to `currentTime`.
- CS sends `StatusNotification` (CS→CSMS) for **every connector** — reporting `connectorStatus` (`Available`, `Occupied`, `Faulted`, etc.).
- CS starts sending `Heartbeat` (CS→CSMS) every `interval` seconds. CSMS responds with `currentTime` for ongoing clock sync.
- CS is now fully operational and can send/receive all message types.

**If `Pending`:**
- CS waits `interval` seconds, then sends `BootNotification` again. Loop until `Accepted`.
- **CRITICAL RULE:** CS **must not** send any message other than `BootNotification` while in Pending state.
- CSMS may change `interval` between retries.

**If `Rejected`:**
- Same behavior as Pending — wait `interval`, retry `BootNotification`, must not send other messages.
- CS retries indefinitely at the given interval.

### 1.2 Boot Reason Values

`BootReasonEnumType`: `PowerUp`, `ApplicationReset`, `FirmwareUpdate`, `LocalReset`, `RemoteReset`, `ScheduledReset`, `Triggered`, `Unknown`, `Watchdog`.

### 1.3 Connector Status Values

`ConnectorStatusEnumType`: `Available`, `Occupied`, `Reserved`, `Unavailable`, `Faulted`.

### 1.4 Boot-Related Configuration Variables

| Component | Variable | Relevance |
|-----------|----------|-----------|
| `HeartbeatCtrlr` | `Interval` | Overridden by `interval` from `BootNotificationResponse` when Accepted |
| `OCPPCommCtrlr` | `RetryBackOffWaitMinimum` | Minimum wait before WebSocket reconnect after disconnect |
| `OCPPCommCtrlr` | `RetryBackOffRandomRange` | Random range added to reconnect wait (prevents thundering herd) |
| `OCPPCommCtrlr` | `RetryBackOffRepeatTimes` | Number of reconnect retries |
| `OCPPCommCtrlr` | `WebSocketPingInterval` | WebSocket ping interval to detect broken connections |

---

## 2. Authorization

Authorization determines whether an idToken is allowed to start a transaction. OCPP 2.1 extends the 2.0.1 authorization with new token types.

For field schemas: [Authorize](../OCPP-2.1-Schemas/OCPP-2.1-Schemas-Authorization.md#authorize), [IdTokenInfoType](../OCPP-2.1-DataTypes.md#idtokeninfotype).

### 2.1 Online Authorization

1. User presents token at CS.
2. CS sends `Authorize` (CS→CSMS) with `idToken` ({idToken: string, type: IdTokenEnumType}).
3. CSMS responds with `AuthorizeResponse` containing `idTokenInfo`.
4. `idTokenInfo.status` determines result:
   - `Accepted` — token is valid, transaction can proceed.
   - `Blocked`, `Invalid`, `Expired`, `NoCredit`, etc. — reject.
   - `ConcurrentTx` — token is valid but already in use in another transaction within the same group.

**New in 2.1:** Additional `IdTokenEnumType` values support:
- Prepaid charge cards (balance checking via transaction options)
- Ad hoc payment (credit/debit card via terminal)
- Secure dynamic QR codes for ad hoc payment

### 2.2 Authorization Decision Logic

When a token is presented, the CS follows this logic:

**Online (WebSocket connected):**

If `AuthCtrlr.LocalPreAuthorize` = true:
1. Check local list and/or cache first.
2. If found and `Accepted`: immediately allow user to proceed.
3. Simultaneously send `Authorize` to CSMS.
4. If CSMS responds with anything other than `Accepted`: **stop the transaction**.

If `AuthCtrlr.LocalPreAuthorize` = false (default):
1. Send `Authorize` to CSMS, wait for response.
2. Act on `idTokenInfo.status`.

**Offline (WebSocket disconnected):**

```
if AuthCacheCtrlr.Enabled AND token in cache AND not expired:
    → use cached status
elif AuthCtrlr.LocalAuthorizeOffline AND token in local list:
    → use local list status
elif AuthCtrlr.OfflineTxForUnknownIdEnabled:
    → accept (allow unknown tokens offline)
else:
    → reject
```

> **ESCALATE: POLICY-DEPENDENT** — Accepting unknown tokens when offline (`OfflineTxForUnknownIdEnabled=true`) has security and revenue implications.
> An AI agent MUST NOT enable this offline authorization path without asking the developer.

### 2.3 Authorization Configuration Variables

| Component | Variable | Purpose |
|-----------|----------|---------|
| `AuthCtrlr` | `Enabled` | Whether authorization is required |
| `AuthCtrlr` | `LocalPreAuthorize` | Pre-authorize from local list before CSMS round-trip |
| `AuthCtrlr` | `LocalAuthorizeOffline` | Use local list when offline |
| `AuthCtrlr` | `OfflineTxForUnknownIdEnabled` | Allow unknown tokens offline |
| `AuthCacheCtrlr` | `Enabled` | Whether authorization cache is enabled |

---

## 3. Transaction Lifecycle

OCPP 2.1 uses the same `TransactionEvent` model as 2.0.1, with extensions for fixed costs, energy/time-based pricing, and transaction resume.

For field schemas: [TransactionEvent](../OCPP-2.1-Schemas/OCPP-2.1-Schemas-Transactions.md#transactionevent), [RequestStartTransaction](../OCPP-2.1-Schemas/OCPP-2.1-Schemas-Transactions.md#requeststarttransaction), [RequestStopTransaction](../OCPP-2.1-Schemas/OCPP-2.1-Schemas-Transactions.md#requeststoptransaction).

### 3.1 Transaction Flow Steps

1. **Authorization** (see §2 above).
2. User plugs in cable.
3. CS sends `TransactionEvent` (CS→CSMS) with `eventType: Started`, `triggerReason: Authorized`, `evse`, `idToken`.
4. CSMS responds with `TransactionEventResponse` containing `idTokenInfo` (if not already authorized) and `transactionId`.
5. Charging begins → CS sends periodic `TransactionEvent` with `eventType: Updated`, `triggerReason: MeterValuePeriodic`.
6. User stops → CS sends `TransactionEvent` with `eventType: Ended`, `triggerReason: EVDeparted` or `RemoteStop`.
7. CS sends `StatusNotification` (Available).

**`eventType` values:** `Started`, `Updated`, `Ended`.

**`triggerReason` values:** `Authorized`, `CablePluggedIn`, `ChargingRateChanged`, `ChargingStateChanged`, `Deauthorized`, `EnergyLimitReached`, `EVCommunicationLost`, `EVConnectTimeout`, `MeterValueClock`, `MeterValuePeriodic`, `TimeLimitReached`, `Trigger`, `UnlockCommand`, `StopAuthorized`, `EVDeparted`, `EVDetected`, `RemoteStart`, `RemoteStop`, `AbnormalCondition`, `SignedDataReceived`, `ResetCommand`.

### 3.2 New in 2.1: Extended Transaction Options

OCPP 2.1 adds support for:

- **Fixed cost transactions**: Transaction can have a fixed cost, independent of energy consumed. Set via tariff management messages (`SetDefaultTariff`, `ChangeTransactionTariff`).
- **Energy-based pricing**: Transaction cost based on energy consumed (Wh/kWh pricing).
- **Time-based pricing**: Transaction cost based on duration (per-minute pricing).
- **Transaction resume after forced reboot**: If CS reboots during a transaction, it can resume the transaction by sending `TransactionEvent(Updated)` with the existing `transactionId` after reconnecting to CSMS.
- **Prepaid card balance checking**: Transaction cost cannot exceed the prepaid card balance. CS enforces limits based on remaining balance.

### 3.3 Remote Start/Stop

**Remote Start:**
1. CSMS sends `RequestStartTransaction` (CSMS→CS) with `evseId`, `idToken`, and optionally `chargingProfile`.
2. CS validates the token (sends `Authorize` if needed).
3. If valid, CS starts the transaction (sends `TransactionEvent(Started)` with `triggerReason: RemoteStart`).

**Remote Stop:**
1. CSMS sends `RequestStopTransaction` (CSMS→CS) with `transactionId`.
2. CS stops the transaction (sends `TransactionEvent(Ended)` with `triggerReason: RemoteStop`).

### 3.4 Transaction Configuration Variables

| Component | Variable | Purpose |
|-----------|----------|---------|
| `TxCtrlr` | `EVConnectionTimeOut` | Timeout (s) waiting for EV after auth |
| `TxCtrlr` | `StopTxOnEVSideDisconnect` | Stop transaction when EV disconnects |
| `TxCtrlr` | `StopTxOnInvalidId` | Stop if idToken invalidated mid-transaction |
| `SampledDataCtrlr` | `TxUpdatedInterval` | Periodic meter value interval (s) |
| `SampledDataCtrlr` | `TxUpdatedMeasurands` | Which measurands to sample |
| `TariffCtrlr` | `Enabled` | Whether local tariff calculation is supported **(New in 2.1)** |

---

## 4. V2X / Bidirectional Charging Flow

OCPP 2.1 adds bidirectional power transfer (V2X/V2G) as Functional Block Q. This enables EVs to act as energy sources, feeding power back to the grid or building.

For field schemas: [NotifyAllowedEnergyTransfer](../OCPP-2.1-Schemas/OCPP-2.1-Schemas-BidirectionalCharging.md#notifyallowedenergytransfer), [SetChargingProfile](../OCPP-2.1-Schemas/OCPP-2.1-Schemas-SmartCharging.md#setchargingprofile).

### 4.1 V2X Session Flow

1. CS detects EV with V2X capability (ISO 15118-20 negotiation).
2. CS sends `NotifyEVChargingNeeds` (CS→CSMS) with `chargingNeeds` indicating V2X support.
3. CSMS evaluates grid conditions and CPO policies.
4. CS sends `NotifyAllowedEnergyTransfer` (CS→CSMS) — reports which energy transfer modes the EV and CS support.
5. CSMS responds with `NotifyAllowedEnergyTransferResponse`.
6. CSMS sends `SetChargingProfile` (CSMS→CS) with a V2G-enabled profile:
   - `chargingProfilePurpose: TxProfile` (specific to this transaction)
   - `chargingSchedule` includes periods with **negative limit values** for discharge
   - `chargingRateUnit: Power` (power-based for V2G)
7. CS acknowledges with `SetChargingProfileResponse` (Accepted/Rejected).
8. CS sends `TransactionEvent(Updated)` with `chargingState: Charging` or `SuspendedEVSE`.
9. During V2G operation, EV discharges energy per the profile schedule.
10. CS sends periodic `TransactionEvent(Updated)` with meter values (negative values indicate export).

### 4.2 V2X Configuration

| Component | Variable | Purpose |
|-----------|----------|---------|
| `V2XCtrlr` | `Enabled` | Whether V2X/bidirectional charging is supported |
| `V2XCtrlr` | `MaxDischargePower` | Maximum discharge power (W) |
| `V2XCtrlr` | `MaxDischargeCurrent` | Maximum discharge current (A) |

> **ESCALATE: VENDOR-DEPENDENT** — V2X capabilities are highly dependent on the charging station hardware (inverter, metering direction, ISO 15118-20 stack). An AI agent MUST ask which hardware/firmware is targeted before implementing V2X flows.

### 4.3 V2X Offline Behavior

If the CS loses connection during a V2X session:
- The CS should follow the local DER control settings (if configured via `SetDERControl`).
- If no local DER settings are active, the CS should gracefully stop the discharge and report via `TransactionEvent(Ended)` upon reconnection.
- The CS should NOT continue V2G operation without CSMS supervision unless explicitly configured to do so.

> **ESCALATE: POLICY-DEPENDENT** — Whether a CS should continue V2G discharge when offline depends on grid operator requirements and local regulations. An AI agent MUST ask about the operational context before implementing this behavior.

---

## 5. DER Control Flow

OCPP 2.1 adds DER (Distributed Energy Resources) control as Functional Block R. This enables grid operators to control CS and EVs as distributed energy resources.

For field schemas: [SetDERControl](../OCPP-2.1-Schemas/OCPP-2.1-Schemas-DERControl.md#setdercontrol), [GetDERControl](../OCPP-2.1-Schemas/OCPP-2.1-Schemas-DERControl.md#getdercontrol), [NotifyDERAlarm](../OCPP-2.1-Schemas/OCPP-2.1-Schemas-DERControl.md#notifyderalarm), [NotifyDERStartStop](../OCPP-2.1-Schemas/OCPP-2.1-Schemas-DERControl.md#notifyderstartstop).

### 5.1 DER Control Configuration Flow

1. CSMS sends `SetDERControl` (CSMS→CS) with DER control settings:
   - `id` — unique identifier for this DER control
   - `name` — human-readable name
   - `DERControlType` — grid code parameters (e.g., frequency-watt, volt-watt, power factor)
   - `gridCodeParameters` — IEC 61850 / IEEE 2030.5 settings
2. CS validates and stores the DER control settings.
3. CS responds with `SetDERControlResponse` (Accepted/Rejected).
4. CS sends `NotifyDERStartStop` (CS→CSMS) when DER control becomes active — reports `startStop` = `Start`.
5. During operation, CS follows the DER control settings locally (adjusting charge/discharge rates based on grid frequency, voltage, etc.).

### 5.2 DER Alarm Flow

When a DER-related event occurs:

1. CS detects a grid event (e.g., frequency deviation, voltage anomaly).
2. CS sends `NotifyDERAlarm` (CS→CSMS) with alarm details.
3. CSMS may update or clear DER control settings in response.

### 5.3 DER Control Lifecycle

1. CS can query current settings: `GetDERControl` (CSMS→CS) → `ReportDERControl` (CS→CSMS).
2. CSMS can remove settings: `ClearDERControl` (CSMS→CS).
3. When DER control is stopped, CS sends `NotifyDERStartStop` with `startStop` = `Stop`.

### 5.4 DER Control Configuration Variables

| Component | Variable | Purpose |
|-----------|----------|-----------|
| `DERCtrlr` | `Enabled` | Whether DER control is supported |
| `DERCtrlr` | `ActiveDERControlId` | ID of the currently active DER control |

---

## 6. Battery Swapping Flow

OCPP 2.1 adds battery swapping as Functional Block S, designed for battery swap stations serving two- and three-wheelers and EVs.

For field schemas: [RequestBatterySwap](../OCPP-2.1-Schemas/OCPP-2.1-Schemas-BatterySwapping.md#requestbatteryswap), [BatterySwap](../OCPP-2.1-Schemas/OCPP-2.1-Schemas-BatterySwapping.md#batteryswap).

### 6.1 Battery Swap Flow

1. CS (battery swap station) detects a request for battery swap (user action or automated).
2. CS sends `RequestBatterySwap` (CS→CSMS) with swap details (station ID, EV/battery identification).
3. CSMS validates the request (checks authorization, available batteries).
4. CSMS sends `BatterySwap` (CSMS→CS) — alternatively, CS can initiate by sending `BatterySwap` directly (bidirectional message).
5. Station performs the physical battery swap.
6. CS reports swap completion via subsequent messages.

### 6.2 Battery Swap Authorization

- The swap may be authorized via standard `Authorize` flow before `RequestBatterySwap`.
- Local authorization (local list or cache) can be used for offline swap operations.
- Prepaid card balance checking applies to swap transactions.

---

*This document is a community reference for AI agents. For the authoritative specification, refer to the official OCPP 2.1 documents from OCA.*