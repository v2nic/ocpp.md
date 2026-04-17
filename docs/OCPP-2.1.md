# OCPP.md — Open Charge Point Protocol Reference for AI Agents

> **Purpose:** This file provides structured context about the OCPP 2.1 protocol for AI agents working on EV charging infrastructure. It is not a replacement for the official specification but a practical reference to ground AI-assisted development, integration, and troubleshooting.

> **Last updated:** 2026-04-16
> **Primary spec covered:** OCPP 2.1 Edition 2 (Part 1: Architecture & Topology, Part 2: Specification)
>
> Source on [Github](https://github.com/alexeimoisseev/ocpp.md).

---

## 1. What is OCPP 2.1?

OCPP 2.1 is the latest version of the Open Charge Point Protocol, released by the **Open Charge Alliance (OCA)** in January 2025. It extends OCPP 2.0.1 with significant new capabilities while maintaining backward compatibility for application logic developed for 2.0.1.

The **"2.1" suffix** denotes a minor version increment over 2.0.1, but the scope of new functionality is substantial — adding 7 new functional blocks and 26 new messages.

**Edition 2** (published 2025-12-03) is the current revision. It includes errata corrections through February 2026.

### Key New Features in OCPP 2.1

- **ISO 15118-20 Support**: Full support for bidirectional power transfer (V2X/V2G)
- **Bidirectional Charging** (Block Q): New functional block enabling EVs to act as energy sources
- **DER Control** (Block R): New functional block for Distributed Energy Resources integration
- **Improved Smart Charging** (Block K): Advanced tools for optimized energy distribution including priority charging and frequency containment
- **Extended Transaction Options**: Fixed costs, energy/time-based pricing, transaction resume after reboot
- **Battery Swapping** (Block S): Support for battery swap stations
- **Local Cost Calculation** (Block I): On-station pricing computation with tariff management
- **New Authorization Options**: Prepaid cards, ad hoc payment, secure dynamic QR codes
- **Periodic Event Streams** (Block N): Streaming monitoring data at regular intervals

### Version History

| Version | Status | Transport | Notes |
|---------|--------|-----------|-------|
| OCPP 1.2 | Legacy | SOAP/HTTP | First widely adopted version |
| OCPP 1.5 | Legacy | SOAP/HTTP | Added smart charging basics |
| OCPP 1.6 | Widely deployed | SOAP or WebSocket + JSON | Most common version in the field today |
| OCPP 2.0 | Superseded | WebSocket + JSON | Major rewrite; never widely deployed |
| OCPP 2.0.1 | Stable | WebSocket + JSON | Bugfix/clarification release of 2.0; recommended for current deployments |
| OCPP 2.1 | **Latest** | WebSocket + JSON | Adds V2G, DER control, battery swapping, payment features |

### Companion Documents

This reference is part of a larger documentation set:

- **[Smart Charging Deep-Dive](./OCPP-2.1-SmartCharging/OCPP-2.1-SmartCharging.md)** — Profile model, composite schedule calculation, V2G integration, DER control, common pitfalls.
- **[Message Sequences](./OCPP-2.1-Sequences/OCPP-2.1-Sequences.md)** — Boot sequence, authorization flows, transaction lifecycle.
- **[Data Types Reference](./OCPP-2.1-DataTypes.md)** — All reusable composite types and enumerations, plus field-level schemas for all 91 messages.
- **[Methodology](./METHODOLOGY.md)** — How these documents were produced, provenance tiers, and trust model.
- **[AI Agent Setup](./AI-AGENT-SETUP.md)** — Full configuration guide for using this reference as a Claude Code plugin.

**Detailed Schema Reference:**

- **[Provisioning](./OCPP-2.1-Schemas/OCPP-2.1-Schemas-Provisioning.md)** — BootNotification, Heartbeat, GetVariables, SetVariables, Reset, DataTransfer, etc.
- **[Authorization](./OCPP-2.1-Schemas/OCPP-2.1-Schemas-Authorization.md)** — Authorize, SendLocalList, GetLocalListVersion, ClearCache
- **[Transactions](./OCPP-2.1-Schemas/OCPP-2.1-Schemas-Transactions.md)** — TransactionEvent, RequestStartTransaction, RequestStopTransaction, MeterValues
- **[Smart Charging](./OCPP-2.1-Schemas/OCPP-2.1-Schemas-SmartCharging.md)** — SetChargingProfile, GetChargingProfiles, GetCompositeSchedule, etc.
- **[Firmware](./OCPP-2.1-Schemas/OCPP-2.1-Schemas-Firmware.md)** — UpdateFirmware, PublishFirmware, FirmwareStatusNotification
- **[Security](./OCPP-2.1-Schemas/OCPP-2.1-Schemas-Security.md)** — Certificates, CSR signing, ISO 15118, GetCertificateChainStatus
- **[Diagnostics](./OCPP-2.1-Schemas/OCPP-2.1-Schemas-Diagnostics.md)** — GetLog, NotifyEvent, variable monitoring
- **[Availability](./OCPP-2.1-Schemas/OCPP-2.1-Schemas-Availability.md)** — ChangeAvailability, UnlockConnector, TriggerMessage
- **[Reservation](./OCPP-2.1-Schemas/OCPP-2.1-Schemas-Reservation.md)** — ReserveNow, CancelReservation
- **[Display](./OCPP-2.1-Schemas/OCPP-2.1-Schemas-Display.md)** — SetDisplayMessage, CostUpdated, etc.
- **[Bidirectional Charging](./OCPP-2.1-Schemas/OCPP-2.1-Schemas-BidirectionalCharging.md)** — NotifyAllowedEnergyTransfer
- **[DER Control](./OCPP-2.1-Schemas/OCPP-2.1-Schemas-DERControl.md)** — SetDERControl, GetDERControl, ReportDERControl, etc.
- **[Battery Swapping](./OCPP-2.1-Schemas/OCPP-2.1-Schemas-BatterySwapping.md)** — BatterySwap, RequestBatterySwap
- **[Tariffs and Cost](./OCPP-2.1-Schemas/OCPP-2.1-Schemas-TariffsAndCost.md)** — SetDefaultTariff, GetTariffs, NotifySettlement, VatNumberValidation
- **[Payment](./OCPP-2.1-Schemas/OCPP-2.1-Schemas-Payment.md)** — NotifyWebPaymentStarted
- **[Periodic Event Streams](./OCPP-2.1-Schemas/OCPP-2.1-Schemas-PeriodicEventStreams.md)** — OpenPeriodicEventStream, ClosePeriodicEventStream, etc.
- **[Priority Charging](./OCPP-2.1-Schemas/OCPP-2.1-Schemas-PriorityCharging.md)** — NotifyPriorityCharging, UsePriorityCharging
- **[Frequency Containment](./OCPP-2.1-Schemas/OCPP-2.1-Schemas-FrequencyContainment.md)** — AFRRSignal

---

## 2. Architecture Overview (OCPP 2.1)

### 2.1 Roles

- **Charging Station (CS):** The physical device that charges EVs. Contains one or more EVSEs.
- **CSMS (Charging Station Management System):** The backend/server that manages charging stations remotely.

Communication is **initiated by the Charging Station**, which opens a persistent **WebSocket** connection to the CSMS. Both sides can then send messages over this connection.

### 2.2 Device Model

OCPP 2.1 maintains the same hierarchical device model as OCPP 2.0.1:

```
Charging Station
├── EVSE 1 (Electric Vehicle Supply Equipment)
│   ├── Connector 1 (e.g., CCS)
│   └── Connector 2 (e.g., CHAdeMO)
├── EVSE 2
│   └── Connector 1 (e.g., Type 2)
└── ...
```

- **Charging Station** is the top-level entity. It has a single connection to the CSMS.
- **EVSE** represents a separately controllable charging spot. Each EVSE can charge one EV at a time.
- **Connector** is a physical socket/plug. An EVSE may have multiple connectors, but only one can be active at a time.

**Important:** In OCPP 2.1, `evseId` and `connectorId` are 1-indexed integers. `evseId=0` refers to the Charging Station as a whole (used in certain messages like status notifications).

### 2.3 Transport

- **WebSocket** (RFC 6455) over TLS (wss://)
- Sub-protocol: `ocpp2.1`
- The Charging Station connects to the CSMS at a URL like: `wss://csms.example.com/ocpp/<charging_station_id>`
- The Charging Station ID is typically included in the WebSocket URL path.

### 2.4 Security Profiles

OCPP 2.1 maintains the three security profiles from OCPP 2.0.1:

| Profile | Authentication | Encryption |
|---------|---------------|------------|
| **1** | Basic Auth (HTTP) | TLS (server cert only) |
| **2** | TLS client-side certificates | TLS (mutual) |
| **3** | TLS client-side certificates + Plug & Charge | TLS (mutual) + ISO 15118 PKI |

---

## 3. Message Structure (RPC Framework)

OCPP 2.1 uses a JSON-based RPC framework over WebSocket, identical to OCPP 2.0.1. There are three message types:

### 3.1 CALL (Request)

```json
[2, "<messageId>", "<action>", {<payload>}]
```

- `2` — MessageTypeId for CALL
- `messageId` — Unique string identifier for this request (used to correlate response)
- `action` — The name of the operation (e.g., `"BootNotification"`, `"Authorize"`)
- `payload` — JSON object with the request parameters

### 3.2 CALLRESULT (Response)

```json
[3, "<messageId>", {<payload>}]
```

- `3` — MessageTypeId for CALLRESULT
- `messageId` — Must match the CALL it responds to
- `payload` — JSON object with the response data

### 3.3 CALLERROR (Error Response)

```json
[4, "<messageId>", "<errorCode>", "<errorDescription>", {<errorDetails>}]
```

- `4` — MessageTypeId for CALLERROR
- Standard error codes include: `NotImplemented`, `NotSupported`, `InternalError`, `ProtocolError`, `SecurityError`, `FormatViolationError`, `PropertyConstraintViolation`, `OccurrenceConstraintViolation`, `TypeConstraintViolation`, `GenericError`

### 3.4 Message Flow Rules

- Only **one CALL can be outstanding** at a time per direction (CS→CSMS or CSMS→CS). A new CALL must not be sent until a CALLRESULT or CALLERROR is received for the previous one.
- Both sides can initiate CALLs independently since the connection is bidirectional.
- Messages initiated by the CS are called **CS→CSMS** messages; messages initiated by the CSMS are **CSMS→CS** messages.

---

## 4. Key Messages (OCPP 2.1)

### 4.1 Provisioning & Lifecycle

| Message | Direction | Purpose |
|---------|-----------|---------|
| `BootNotification` | CS→CSMS | Sent when CS boots up. Contains vendor, model, serial number, firmware version. CSMS responds with `Accepted`, `Pending`, or `Rejected` and a heartbeat interval. |
| `Heartbeat` | CS→CSMS | Periodic keepalive. CSMS responds with current time for clock sync. |
| `StatusNotification` | CS→CSMS | Reports the status of a Connector (Available, Occupied, Reserved, Unavailable, Faulted). |
| `GetVariables` | CSMS→CS | Read configuration variables from the CS. |
| `SetVariables` | CSMS→CS | Write configuration variables on the CS. |
| `GetBaseReport` | CSMS→CS | Request a full or summary report of all variables. CS responds with `NotifyReport` messages. |
| `GetReport` | CSMS→CS | Request a custom report with specific variables. |
| `NotifyReport` | CS→CSMS | Sends variable data in response to `GetBaseReport` / `GetReport`. May be sent in multiple parts (seq/tbc). |
| `Reset` | CSMS→CS | Restart the CS (Immediate or OnIdle). |
| `DataTransfer` | Both | Vendor-specific or custom data exchange. Can be sent by either side. Uses `vendorId` and optional `messageId`. |
| `SetNetworkProfile` | CSMS→CS | Configure a network connection profile for CSMS connection. |

### 4.2 Authorization

| Message | Direction | Purpose |
|---------|-----------|---------|
| `Authorize` | CS→CSMS | Validate an idToken (RFID, app, etc.) before starting a transaction. |
| `SendLocalList` | CSMS→CS | Push a local authorization list to the CS for offline auth. |
| `GetLocalListVersion` | CSMS→CS | Query current version of the local auth list. |
| `ClearCache` | CSMS→CS | Clear the authorization cache. |

**idToken types (2.1):** `Central`, `eMAID`, `ISO14443`, `ISO15693`, `KeyCode`, `Local`, `MacAddress`, `NoAuthorization`, plus new 2.1 types for prepaid cards, ad hoc payment, and QR codes.

### 4.3 Transactions

| Message | Direction | Purpose |
|---------|-----------|---------|
| `TransactionEvent` | CS→CSMS | **The core transaction message.** Reports transaction lifecycle events. |
| `RequestStartTransaction` | CSMS→CS | Remotely start a transaction. |
| `RequestStopTransaction` | CSMS→CS | Remotely stop a transaction. |
| `GetTransactionStatus` | CSMS→CS | Query whether transaction-related messages have been delivered. |
| `MeterValues` | CS→CSMS | Send meter values outside transaction context. |

**OCPP 2.1 extensions to TransactionEvent:**
- Fixed cost, energy-based, and time-based transaction options
- Transaction resume after forced reboot
- Prepaid card balance checking

### 4.4 Smart Charging

> **Deep-dive:** For comprehensive coverage of smart charging — profile model, composite schedule calculation, V2G, DER control, and common pitfalls — see the [Smart Charging Deep-Dive](./OCPP-2.1-SmartCharging/OCPP-2.1-SmartCharging.md).

| Message | Direction | Purpose |
|---------|-----------|---------|
| `SetChargingProfile` | CSMS→CS | Set a charging profile (schedule of limits). Enhanced in 2.1 for V2G profiles. |
| `GetChargingProfiles` | CSMS→CS | Retrieve active charging profiles. |
| `ClearChargingProfile` | CSMS→CS | Remove charging profiles. |
| `ReportChargingProfiles` | CS→CSMS | Response to `GetChargingProfiles`. |
| `GetCompositeSchedule` | CSMS→CS | Get the combined/effective charging schedule. |
| `ClearedChargingLimit` | CS→CSMS | Notify that an external limit was cleared. |
| `NotifyChargingLimit` | CS→CSMS | Report current charging limits (from external source or grid). |
| `NotifyEVChargingSchedule` | CS→CSMS | Report the EV's desired charging schedule (ISO 15118). |
| `NotifyEVChargingNeeds` | CS→CSMS | Report EV charging needs (ISO 15118). |
| `PullDynamicScheduleUpdate` | CS→CSMS | CS requests a dynamic schedule update from CSMS. **New in 2.1.** |
| `UpdateDynamicSchedule` | CSMS→CS | CSMS pushes a dynamic schedule update. **New in 2.1.** |

### 4.5 Bidirectional Charging

| Message | Direction | Purpose |
|---------|-----------|---------|
| `NotifyAllowedEnergyTransfer` | CS→CSMS | Notify CSMS of allowed energy transfer modes from the EV (V2X). **New in 2.1.** |

### 4.6 DER Control

| Message | Direction | Purpose |
|---------|-----------|---------|
| `SetDERControl` | CSMS→CS | Configure DER control settings on the CS. **New in 2.1.** |
| `GetDERControl` | CSMS→CS | Retrieve DER control configuration. **New in 2.1.** |
| `ReportDERControl` | CS→CSMS | Report DER control configuration. **New in 2.1.** |
| `ClearDERControl` | CSMS→CS | Remove DER control settings. **New in 2.1.** |
| `NotifyDERAlarm` | CS→CSMS | Report a DER alarm event. **New in 2.1.** |
| `NotifyDERStartStop` | CS→CSMS | Report DER control start/stop status. **New in 2.1.** |

### 4.7 Tariffs and Cost

| Message | Direction | Purpose |
|---------|-----------|---------|
| `SetDefaultTariff` | CSMS→CS | Set the default tariff on the CS. **New in 2.1.** |
| `GetTariffs` | CSMS→CS | Retrieve tariff information from the CS. **New in 2.1.** |
| `ClearTariffs` | CSMS→CS | Clear tariff information. **New in 2.1.** |
| `ChangeTransactionTariff` | CSMS→CS | Change the tariff for an active transaction. **New in 2.1.** |
| `NotifySettlement` | CS→CSMS | Report settlement information at end of transaction. **New in 2.1.** |
| `VatNumberValidation` | CSMS→CPO | Validate a VAT number. **New in 2.1.** |

### 4.8 Payment

| Message | Direction | Purpose |
|---------|-----------|---------|
| `NotifyWebPaymentStarted` | CS→CSMS | Notify CSMS that a web-based payment has been initiated. **New in 2.1.** |

### 4.9 Priority Charging

| Message | Direction | Purpose |
|---------|-----------|---------|
| `NotifyPriorityCharging` | CS→CSMS | Report that priority charging has been started/modified. **New in 2.1.** |
| `UsePriorityCharging` | CSMS→CS | Request the CS to use priority charging. **New in 2.1.** |

### 4.10 Frequency Containment

| Message | Direction | Purpose |
|---------|-----------|---------|
| `AFRRSignal` | CSMS→CS | Send AFRR (Automatic Frequency Restoration Reserve) signal to the CS. **New in 2.1.** |

### 4.11 Periodic Event Streams

| Message | Direction | Purpose |
|---------|-----------|---------|
| `OpenPeriodicEventStream` | CSMS→CS | Open a periodic event stream for monitoring data. **New in 2.1.** |
| `ClosePeriodicEventStream` | CSMS→CS | Close a periodic event stream. **New in 2.1.** |
| `GetPeriodicEventStream` | CSMS→CS | Query periodic event stream status. **New in 2.1.** |
| `NotifyPeriodicEventStream` | CS→CSMS | Send periodic monitoring data to CSMS. **New in 2.1.** |
| `AdjustPeriodicEventStream` | CSMS→CS | Adjust parameters of an open periodic event stream. **New in 2.1.** |

### 4.12 Battery Swapping

| Message | Direction | Purpose |
|---------|-----------|---------|
| `BatterySwap` | Both | Battery swap operation request/response. **New in 2.1.** |
| `RequestBatterySwap` | CS→CSMS | Request a battery swap operation. **New in 2.1.** |

### 4.13 Firmware Management

| Message | Direction | Purpose |
|---------|-----------|---------|
| `UpdateFirmware` | CSMS→CS | Instruct CS to download and install firmware. |
| `FirmwareStatusNotification` | CS→CSMS | Report firmware update progress. |
| `PublishFirmware` | CSMS→CS | Ask a CS to publish firmware for local distribution. |
| `PublishFirmwareStatusNotification` | CS→CSMS | Report publish firmware status. |
| `UnpublishFirmware` | CSMS→CS | Stop publishing firmware. |

### 4.14 Security & Certificates

| Message | Direction | Purpose |
|---------|-----------|---------|
| `Get15118EVCertificate` | CS→CSMS | Request an EV certificate (Plug & Charge). |
| `GetCertificateStatus` | CS→CSMS | Check OCSP status of a certificate. |
| `GetCertificateChainStatus` | CS→CSMS | Check the status of a certificate chain. **New in 2.1.** |
| `SignCertificate` | CS→CSMS | Request CSMS to sign a CSR. |
| `CertificateSigned` | CSMS→CS | Return a signed certificate. |
| `InstallCertificate` | CSMS→CS | Install a CA certificate. |
| `DeleteCertificate` | CSMS→CS | Delete a certificate. |
| `GetInstalledCertificateIds` | CSMS→CS | List installed certificates. |
| `SecurityEventNotification` | CS→CSMS | Report security-related event. |

### 4.15 Diagnostics & Monitoring

| Message | Direction | Purpose |
|---------|-----------|---------|
| `GetLog` | CSMS→CS | Request diagnostic or security logs. |
| `LogStatusNotification` | CS→CSMS | Report log upload status. |
| `NotifyEvent` | CS→CSMS | Report events/alerts from monitored variables. |
| `SetMonitoringBase` | CSMS→CS | Set monitoring level for all variables. |
| `SetVariableMonitoring` | CSMS→CS | Configure monitoring on specific variables. |
| `SetMonitoringLevel` | CSMS→CS | Set the severity threshold for reporting. |
| `GetMonitoringReport` | CSMS→CS | Request a monitoring report. |
| `ClearVariableMonitoring` | CSMS→CS | Remove variable monitors. |
| `NotifyMonitoringReport` | CS→CSMS | Report monitoring configuration. |
| `CustomerInformation` | CSMS→CS | Request or clear customer data (GDPR). |
| `NotifyCustomerInformation` | CS→CSMS | Return customer data. |

### 4.16 Availability & Remote Control

| Message | Direction | Purpose |
|---------|-----------|---------|
| `ChangeAvailability` | CSMS→CS | Set an EVSE to operative or inoperative. |
| `UnlockConnector` | CSMS→CS | Remotely unlock a connector (to free a cable). |
| `TriggerMessage` | CSMS→CS | Ask CS to send a specific message (e.g., `BootNotification`, `StatusNotification`, `Heartbeat`, `MeterValues`, `FirmwareStatusNotification`). |

### 4.17 Reservation

| Message | Direction | Purpose |
|---------|-----------|---------|
| `ReserveNow` | CSMS→CS | Reserve an EVSE for a specific idToken. |
| `CancelReservation` | CSMS→CS | Cancel a reservation. |
| `ReservationStatusUpdate` | CS→CSMS | Notify reservation expired or removed. |

### 4.18 Display Messages

| Message | Direction | Purpose |
|---------|-----------|---------|
| `CostUpdated` | CSMS→CS | Push running/final cost to CS display. |
| `SetDisplayMessage` | CSMS→CS | Set a message on the CS display. |
| `GetDisplayMessages` | CSMS→CS | Retrieve display messages. |
| `ClearDisplayMessage` | CSMS→CS | Remove a display message. |
| `NotifyDisplayMessages` | CS→CSMS | Report configured display messages. |

---

## 5. Functional Blocks

OCPP 2.1 organizes features into **Functional Blocks** as defined in the specification (Part 0, Table 3):

| Block | Letter | Key Features |
|-------|--------|--------------|
| Security | A | Security profiles, security events |
| Provisioning | B | Boot, variables, reset, base reports |
| Authorization | C | Authorize, authorization cache |
| Local Authorization List Management | D | SendLocalList, GetLocalListVersion |
| Transactions | E | TransactionEvent, remote start/stop |
| Remote Control | F | Trigger, unlock, change availability |
| Availability | G | Status notifications, heartbeat |
| Reservation | H | Reserve EVSE |
| Tariff and Cost | I | Tariff management, cost calculation, settlement |
| Metering | J | Meter values in transaction events |
| Smart Charging | K | Charging profiles, composite schedules, priority charging, frequency containment |
| Firmware Management | L | Update, publish firmware |
| Certificate Management | M | Plug & Charge certificates |
| Diagnostics | N | Logs, monitoring, events, periodic event streams |
| Display Message | O | Display and cost messages |
| Data Transfer | P | Vendor-specific extensions |
| Bidirectional Power Transfer | Q | V2X energy transfer |
| DER Control | R | Distributed Energy Resources management |
| Battery Swapping | S | Battery swap operations |

---

## 6. Key Configuration Variables

OCPP 2.1 uses a **Component/Variable** model for configuration. Some important ones:

| Component | Variable | Description |
|-----------|----------|-------------|
| `AlignedDataCtrlr` | `Interval` | Interval (s) for clock-aligned meter data |
| `AuthCtrlr` | `Enabled` | Whether authorization is required |
| `AuthCtrlr` | `OfflineTxForUnknownIdEnabled` | Allow offline transactions for unknown tokens |
| `AuthCtrlr` | `LocalAuthorizeOffline` | Use local list when offline |
| `AuthCtrlr` | `LocalPreAuthorize` | Pre-authorize from local list before checking CSMS |
| `HeartbeatCtrlr` | `Interval` | Heartbeat interval in seconds |
| `SampledDataCtrlr` | `TxUpdatedInterval` | Interval (s) for periodic meter values during tx |
| `SampledDataCtrlr` | `TxUpdatedMeasurands` | Which measurands to sample |
| `TxCtrlr` | `EVConnectionTimeOut` | Timeout (s) waiting for EV to connect after auth |
| `TxCtrlr` | `StopTxOnEVSideDisconnect` | Stop transaction when EV disconnects cable |
| `TxCtrlr` | `StopTxOnInvalidId` | Stop transaction if idToken becomes invalid |
| `OCPPCommCtrlr` | `RetryBackOffRepeatTimes` | Reconnect retry count |
| `OCPPCommCtrlr` | `RetryBackOffRandomRange` | Random reconnect backoff range (s) |
| `OCPPCommCtrlr` | `RetryBackOffWaitMinimum` | Minimum reconnect wait (s) |
| `OCPPCommCtrlr` | `WebSocketPingInterval` | WebSocket ping interval (s) |
| `SmartChargingCtrlr` | `Enabled` | Whether smart charging is supported |
| `ReservationCtrlr` | `Enabled` | Whether reservations are supported |
| `V2XCtrlr` | `Enabled` | Whether V2X/bidirectional charging is supported **(New in 2.1)** |
| `DERCtrlr` | `Enabled` | Whether DER control is supported **(New in 2.1)** |
| `TariffCtrlr` | `Enabled` | Whether local tariff calculation is supported **(New in 2.1)** |
| `PeriodicEventStreamCtrlr` | `Enabled` | Whether periodic event streaming is supported **(New in 2.1)** |

---

## 7. OCPP 2.0.1 vs 2.1 — Key Differences

### 7.1 New Functional Blocks

| Block | Letter | New in 2.1? |
|-------|--------|-------------|
| Bidirectional Power Transfer | Q | ✅ New |
| DER Control | R | ✅ New |
| Battery Swapping | S | ✅ New |

Additionally, 2.1 adds new use cases to existing blocks:
- **Tariff and Cost (I)**: Now a full block with tariff management messages (was partially covered by Display in 2.0.1)
- **Smart Charging (K)**: Extended with priority charging, dynamic schedules, frequency containment
- **Diagnostics (N)**: Extended with periodic event streams

### 7.2 Block Letter Renumbering

OCPP 2.1 renumbers the functional block letters compared to what was used informally in 2.0.1:

| Block | 2.0.1 (informal) | 2.1 (official) |
|-------|-------------------|-----------------|
| Security | B/J | A/M |
| Provisioning | B | B |
| Authorization | C | C/D |
| Transactions | D | E/J |
| Remote Control / Availability | E/F | F/G |
| Reservation | N | H |
| Tariff and Cost | — | I |
| Smart Charging | H | K |
| Firmware | I | L |
| Diagnostics | K | N |
| Display | L | O |
| Data Transfer | P | P |

### 7.3 New Messages (26 total)

**Bidirectional Charging (Q):**
- `NotifyAllowedEnergyTransfer`

**DER Control (R):**
- `SetDERControl`, `GetDERControl`, `ReportDERControl`, `ClearDERControl`, `NotifyDERAlarm`, `NotifyDERStartStop`

**Battery Swapping (S):**
- `BatterySwap`, `RequestBatterySwap`

**Tariff and Cost (I):**
- `SetDefaultTariff`, `GetTariffs`, `ClearTariffs`, `ChangeTransactionTariff`, `NotifySettlement`, `VatNumberValidation`

**Payment (within I):**
- `NotifyWebPaymentStarted`

**Periodic Event Streams (within N):**
- `OpenPeriodicEventStream`, `ClosePeriodicEventStream`, `GetPeriodicEventStream`, `AdjustPeriodicEventStream`, `NotifyPeriodicEventStream`

**Priority Charging (within K):**
- `NotifyPriorityCharging`, `UsePriorityCharging`

**Frequency Containment (within K/R):**
- `AFRRSignal`

**Security Extensions:**
- `GetCertificateChainStatus`

**Smart Charging Extensions:**
- `PullDynamicScheduleUpdate`, `UpdateDynamicSchedule`

### 7.4 Enhanced Existing Messages

- `TransactionEvent` — Extended with fixed cost, energy/time options, transaction resume after forced reboot, prepaid card balance checking
- `Authorize` — New idToken types for prepaid cards, ad hoc payment, secure dynamic QR codes
- `SetChargingProfile` — Enhanced for V2G profiles and bidirectional operation
- `MeterValues` — Additional metering values for DER monitoring
- `GetCustomReport` — New message replacing `GetReport` in some contexts

---

## 8. Common Implementation Patterns

> **Deep-dive:** For detailed sequence diagrams covering boot loops (Pending/Rejected), authorization with groupId/parent tokens, full transaction lifecycle, reservations, offline queueing, and firmware updates — see the [Message Sequences](./OCPP-2.1-Sequences/OCPP-2.1-Sequences.md).

### 8.1 Boot Sequence

```
CS                                  CSMS
 |                                    |
 |--- WebSocket Connect ------------->|
 |--- BootNotification --------------->|
 |<-- BootNotificationResponse --------|  (status: Accepted/Pending/Rejected)
 |                                    |
 |--- StatusNotification (per conn) -->|  (report connector statuses)
 |                                    |
 |--- Heartbeat ---------------------->|  (periodic, per interval from boot response)
```

If `BootNotificationResponse.status` is `Pending`, the CS should retry after the `interval` and must not send any other messages except `BootNotification` until `Accepted`. If `Rejected`, the CS should retry but at the given interval.

### 8.2 V2X Session (New in 2.1)

```
CS                                  CSMS
 |                                    |
 |--- NotifyEVChargingNeeds -------->|  (EV requests V2X)
 |<-- NotifyEVChargingNeedsResponse --|
 |                                    |
 |--- NotifyAllowedEnergyTransfer -->|  (EV allows bidirectional)
 |<-- NotifyAllowedEnergyTransfer ----|
 |                                    |
 |--- SetChargingProfile ------------>|  (CSMS sets V2G profile)
 |<-- SetChargingProfileResponse ----|
 |                                    |
 |--- TransactionEvent(Updated) ----->|  (V2X session active)
```

### 8.3 DER Control Flow (New in 2.1)

```
CSMS                                CS
 |                                    |
 |--- SetDERControl ----------------->|  (configure grid code params)
 |<-- SetDERControlResponse ----------|
 |                                    |
 |--- NotifyDERStartStop ------------>|  (CS reports DER control state)
 |                                    |
 |--- NotifyDERAlarm ---------------->|  (CS reports DER alarm if needed)
```

### 8.4 Offline Behavior

When the CS loses connection to the CSMS:
- It should queue `TransactionEvent` messages and replay them in order upon reconnection.
- It can use the **Local Authorization List** or **Authorization Cache** for offline authorization.
- Configuration variable `OfflineTxForUnknownIdEnabled` controls whether unknown tokens can start transactions offline.

### 8.5 Reconnection Strategy

The CS should implement exponential backoff when reconnecting:
- Wait at least `RetryBackOffWaitMinimum` seconds
- Add a random value between 0 and `RetryBackOffRandomRange`
- Repeat up to `RetryBackOffRepeatTimes` times
- This prevents thundering herd when a CSMS restarts and many stations reconnect simultaneously.

---

## 9. JSON Schema Validation

All OCPP 2.1 messages have **JSON Schemas** published by OCA. Implementations should validate incoming messages against these schemas. The schemas define:
- Required vs optional fields
- Data types and formats (e.g., `dateTime` in RFC 3339 format)
- String length constraints
- Enum values
- Nested object structures

Schemas are available from the OCA website and are typically named like `BootNotificationRequest.json`, `BootNotificationResponse.json`, etc.

### Detailed Schema Reference

Complete field-level documentation for all 91 messages (request + response) is available in the companion files listed in [Section 1 — Companion Documents](#companion-documents).

---

## 10. Testing & Compliance

- **OCA Compliance Testing** — The Open Charge Alliance provides a compliance testing tool and test cases.
- **OCTT (OCPP Compliance Testing Tool)** — Official tool for validating OCPP implementations.
- Common test scenarios include: boot sequence, authorization flows, transaction lifecycle, offline behavior, smart charging profile application, firmware update, V2G operation, DER control, and security profile negotiation.
- **New 2.1 test areas**: Bidirectional charging flows, DER control configuration, battery swapping, tariff management, periodic event streaming, priority charging, and frequency containment reserves.

---

## 11. Useful Resources

- **OCA Official Site:** https://openchargealliance.org
- **OCPP 2.1 Specification:** Available for download from OCA (requires free registration)
- **OCPP 2.1 JSON Schemas:** Published alongside the specification
- **OCA GitHub:** Example implementations and schemas
- **ISO 15118-20:** Standard for vehicle-to-grid communication
- **IEC 61850:** Standard for DER communication
- **IEEE 2030.5:** Standard for smart energy profile

---

## 12. Using with AI Coding Agents

This reference is available as a **Claude Code plugin**. Install it once and your AI assistant gets structured OCPP 2.1 knowledge in every project.

### Quick Setup

```
/plugin marketplace add https://github.com/alexeimoisseev/ocpp.md
/plugin install ocpp@ocpp
```

Once installed, the plugin activates automatically when working with OCPP code. You can also invoke it directly:

```
/ocpp                    # General OCPP assistance
/ocpp smart-charging     # Smart charging deep-dive
/ocpp transactions       # Transaction handling
/ocpp authorize          # Authorization flow
/ocpp 2.1                # OCPP 2.1 specific
/ocpp v2x               # Bidirectional charging
/ocpp der                # DER control
```

### What the Agent Gets

The plugin provides a hybrid reference: a compact inline summary (all 91 messages, key types, escalation model) is always available, while detailed schemas, sequence diagrams, and worked examples are loaded on demand when needed.

### Escalation Handling

The OCPP spec has areas that are silent, vendor-dependent, or policy-dependent. The plugin prevents the agent from silently guessing in these areas. By default it uses **strict** mode (stops and asks). To switch to pragmatic mode, add to your `CLAUDE.md`:

```
For OCPP: use pragmatic escalation mode.
```

- **strict** (default) — agent stops and asks before making assumptions
- **pragmatic** — agent flags ambiguity with a code comment but picks a reasonable default

See the full [AI Agent Setup guide](./AI-AGENT-SETUP.md) for manual installation and detailed configuration.

---

## 13. Glossary

| Term | Definition |
|------|-----------|
| **CSMS** | Charging Station Management System (the backend server) |
| **CS** | Charging Station (the physical charger) |
| **EVSE** | Electric Vehicle Supply Equipment — a single charging spot |
| **Connector** | Physical plug/socket on an EVSE |
| **idToken** | An identifier used for authorization (RFID UID, eMAID, etc.) |
| **eMAID** | e-Mobility Account Identifier (used in Plug & Charge) |
| **SoC** | State of Charge (battery percentage) |
| **Plug & Charge** | ISO 15118-based automatic auth via EV certificate |
| **V2X** | Vehicle-to-Everything — bidirectional energy transfer (includes V2G) |
| **V2G** | Vehicle-to-Grid — EV feeds energy back to the grid |
| **DER** | Distributed Energy Resource — grid-connected device that can produce or consume energy |
| **AFRR** | Automatic Frequency Restoration Reserve — grid frequency balancing service |
| **Measurand** | A type of meter measurement (energy, power, current, voltage, etc.) |
| **Charging Profile** | A schedule defining power/current limits over time |
| **Composite Schedule** | The effective schedule after combining all active profiles |
| **Local List** | A list of pre-authorized idTokens stored on the CS |
| **OCTT** | OCPP Compliance Testing Tool |
| **OCA** | Open Charge Alliance — the standards body for OCPP |

---

*This document is a community reference for AI agents. It is not affiliated with or endorsed by the Open Charge Alliance. For the authoritative specification, refer to the official OCPP 2.1 documents from OCA.*