# OCPP 2.1 — Smart Charging Deep-Dive

> **Purpose:** Practical reference for AI agents implementing or debugging OCPP 2.1 smart charging. Covers the charging profile model, composite schedule calculation, V2G/bidirectional charging, DER control, priority charging, frequency containment, tariff management, and common pitfalls.

> **Last updated:** 2026-04-16

---

## How This Document Was Produced

This document covers the OCPP 2.1 smart charging profile model, composite schedule calculation, V2G extensions, DER control, and common pitfalls. Its dominant confidence tier is **spec-knowledge** — behavioral rules from the OCPP 2.1 specification known via AI training data. Field names, enum values, and structural constraints are **schema-derived** (cross-referenced against the [schema documentation](../OCPP-2.1-Schemas/OCPP-2.1-Schemas-SmartCharging.md) and [data types reference](../OCPP-2.1-DataTypes.md), mechanically extracted from the official OCA JSON schemas).

This document contains **5 escalation points** marked with `> **ESCALATE:**`. When an AI agent encounters one, it MUST stop and ask the developer to make the decision. See [METHODOLOGY.md](../METHODOLOGY.md) for the full confidence and escalation model.

**Companion documents:**
- [Smart Charging Schemas](../OCPP-2.1-Schemas/OCPP-2.1-Schemas-SmartCharging.md) — complete field-level message schemas
- [Data Types Reference](../OCPP-2.1-DataTypes.md) — all shared types including `ChargingProfileType`, `ChargingScheduleType`
- [Bidirectional Charging Schemas](../OCPP-2.1-Schemas/OCPP-2.1-Schemas-BidirectionalCharging.md) — NotifyAllowedEnergyTransfer
- [DER Control Schemas](../OCPP-2.1-Schemas/OCPP-2.1-Schemas-DERControl.md) — SetDERControl, GetDERControl, etc.
- [Tariffs and Cost Schemas](../OCPP-2.1-Schemas/OCPP-2.1-Schemas-TariffsAndCost.md) — SetDefaultTariff, GetTariffs, etc.

---

## 1. What Smart Charging Solves

Smart charging allows a CSMS to dynamically control how much power each Charging Station or EVSE delivers (or receives in V2G mode). OCPP 2.1 extends the 2.0.1 smart charging model with:

- **Bidirectional power transfer (V2X/V2G)** — EVs can discharge energy back to the grid
- **DER control** — Grid operators can configure charging stations as distributed energy resources
- **Priority charging** — Certain transactions can be given priority over others
- **Frequency containment** — Charging stations can provide frequency restoration reserves (AFRR)
- **Dynamic schedule updates** — Real-time schedule adjustments via PullDynamicScheduleUpdate/UpdateDynamicSchedule
- **Tariff management** — Local tariff calculation and cost display on the charging station

### Messages

**Smart Charging (Block K):**

| Message | Direction | Purpose |
|---------|-----------|---------|
| `SetChargingProfile` | CSMS → CS | Install or update a charging profile (supports V2G in 2.1) |
| `GetChargingProfiles` | CSMS → CS | Retrieve active charging profiles |
| `ClearChargingProfile` | CSMS → CS | Remove charging profiles |
| `ReportChargingProfiles` | CS → CSMS | Report installed profiles (response to GetChargingProfiles) |
| `GetCompositeSchedule` | CSMS → CS | Calculate effective combined schedule |
| `ClearedChargingLimit` | CS → CSMS | Notify that an external limit was cleared |
| `NotifyChargingLimit` | CS → CSMS | Report current charging limits from external source |
| `NotifyEVChargingSchedule` | CS → CSMS | Report EV's desired charging schedule (ISO 15118) |
| `NotifyEVChargingNeeds` | CS → CSMS | Report EV charging needs (ISO 15118) |
| `PullDynamicScheduleUpdate` | CS → CSMS | CS requests a dynamic schedule update **(New in 2.1)** |
| `UpdateDynamicSchedule` | CSMS → CS | CSMS pushes a dynamic schedule update **(New in 2.1)** |

**Bidirectional Charging (Block Q):**

| Message | Direction | Purpose |
|---------|-----------|---------|
| `NotifyAllowedEnergyTransfer` | CS → CSMS | Report EV-allowed energy transfer modes (V2X) **(New in 2.1)** |

**Priority Charging (within Block K):**

| Message | Direction | Purpose |
|---------|-----------|---------|
| `NotifyPriorityCharging` | CS → CSMS | Report priority charging state **(New in 2.1)** |
| `UsePriorityCharging` | CSMS → CS | Request priority charging **(New in 2.1)** |

**Frequency Containment (within Blocks K/R):**

| Message | Direction | Purpose |
|---------|-----------|---------|
| `AFRRSignal` | CSMS → CS | Send AFRR (Automatic Frequency Restoration Reserve) signal **(New in 2.1)** |

---

## 2. Charging Profile Model

The charging profile model is the core data structure for smart charging. It is the same as in OCPP 2.0.1 with V2G extensions.

### 2.1 Profile Structure

```json
{
  "id": 1,
  "stackLevel": 0,
  "chargingProfilePurpose": "TxDefaultProfile",
  "chargingProfileKind": "Recurring",
  "recurrencyKind": "Daily",
  "chargingSchedule": [{
    "id": 1,
    "chargingRateUnit": "A",
    "chargingSchedulePeriod": [
      { "startPeriod": 0, "limit": 32.0 },
      { "startPeriod": 28800, "limit": 16.0 }
    ]
  }]
}
```

### 2.2 Profile Purposes (stack priority, highest first)

1. **`ChargingStationExternalConstraints`** — Grid operator / external limits
2. **`ChargingStationMaxProfile`** — Max power for entire station
3. **`TxDefaultProfile`** — Default for transactions on an EVSE
4. **`TxProfile`** — Specific to an active transaction

Profiles at the same purpose level use `stackLevel` to determine priority (**higher wins**).

### 2.3 New in 2.1: V2G Profile Extensions

For V2G (bidirectional) operation, charging profiles can now specify:

- **Negative limit values** in `chargingSchedulePeriod` — indicate discharge (export) rather than charge (import)
- **`chargingRateUnit: Power`** — power-based profiles (W) are preferred for V2G to handle both import and export
- The `evseId` field in `SetChargingProfile` identifies which EVSE the V2G profile applies to

Example V2G profile:
```json
{
  "id": 42,
  "stackLevel": 1,
  "chargingProfilePurpose": "TxProfile",
  "chargingProfileKind": "Absolute",
  "chargingSchedule": [{
    "id": 1,
    "chargingRateUnit": "Power",
    "chargingSchedulePeriod": [
      { "startPeriod": 0, "limit": -7000 },
      { "startPeriod": 14400, "limit": 11000 }
    ]
  }]
}
```

This profile discharges at 7 kW for the first 4 hours, then charges at 11 kW.

---

## 3. Composite Schedule

The composite schedule is the effective result of combining all active charging profiles for a given EVSE. The calculation is the same as 2.0.1.

### 3.1 Calculation Steps

1. Filter profiles by `evseId` and time window.
2. Group by `chargingProfilePurpose`.
3. Within each purpose, keep only the profile with the highest `stackLevel`.
4. Apply the purposes in priority order (ExternalConstraints > MaxProfile > TxDefaultProfile > TxProfile).
5. For each time period, the limit is the **minimum** of all applicable purposes' limits.
6. The result is the composite schedule.

> **ESCALATE: SPEC-SILENT** — When profiles use different `chargingRateUnit` values (A vs Power), the spec does not define how to convert between them. An AI agent MUST ask the developer which conversion to use (typically requires knowing the nominal voltage).

### 3.2 New in 2.1: Dynamic Schedule Updates

OCPP 2.1 adds two messages for real-time schedule adjustments:

- **`PullDynamicScheduleUpdate`** (CS→CSMS): CS requests the latest schedule from CSMS. Used when the CS detects a grid condition change or needs to refresh its schedules.
- **`UpdateDynamicSchedule`** (CSMS→CS): CSMS pushes an updated schedule without waiting for a profile install. This provides faster response than `SetChargingProfile` for time-critical adjustments (e.g., frequency containment, emergency load reduction).

---

## 4. Priority Charging

OCPP 2.1 adds priority charging support within Smart Charging (Block K).

### 4.1 Flow

1. CSMS sends `UsePriorityCharging` (CSMS→CS) with priority request parameters.
2. CS adjusts charging behavior to prioritize the specified transaction or EVSE.
3. CS sends `NotifyPriorityCharging` (CS→CSMS) when priority charging starts or changes.

### 4.2 Interaction with Charging Profiles

Priority charging interacts with the composite schedule:
- Priority transactions may override lower-priority `TxDefaultProfile` limits
- `ChargingStationMaxProfile` and `ChargingStationExternalConstraints` always take precedence over priority charging
- The `stackLevel` mechanism still applies — priority charging typically uses higher stack levels

> **ESCALATE: POLICY-DEPENDENT** — Priority charging rules (who gets priority, how much power is reserved, what happens during grid constraints) depend on business rules and CPO policies. An AI agent MUST ask about the operational context.

---

## 5. Frequency Containment (AFRR)

OCPP 2.1 adds support for Automatic Frequency Restoration Reserve (AFRR) signals, enabling charging stations to participate in grid frequency balancing.

### 5.1 Flow

1. CSMS sends `AFRRSignal` (CSMS→CS) with frequency containment reserve parameters:
   - Setpoint values (power level adjustments)
   - Direction (increase/reduce consumption or discharge)
   - Duration
2. CS adjusts its power consumption/discharge to match the AFRR signal.
3. CS sends `TransactionEvent(Updated)` with meter values reflecting the adjusted power.

### 5.2 Configuration

| Component | Variable | Purpose |
|-----------|----------|---------|
| `DERCtrlr` | `Enabled` | Whether DER control (including AFRR) is supported |
| `SmartChargingCtrlr` | `Enabled` | Whether smart charging is supported |

---

## 6. Tariff Management

OCPP 2.1 adds tariff management as part of Functional Block I (Tariff and Cost), enabling local tariff calculation on the charging station.

### 6.1 Messages

| Message | Direction | Purpose |
|---------|-----------|---------|
| `SetDefaultTariff` | CSMS → CS | Set the default tariff for the CS **(New in 2.1)** |
| `GetTariffs` | CSMS → CS | Retrieve current tariff information **(New in 2.1)** |
| `ClearTariffs` | CSMS → CS | Clear tariff information **(New in 2.1)** |
| `ChangeTransactionTariff` | CSMS → CS | Change tariff for an active transaction **(New in 2.1)** |
| `NotifySettlement` | CS → CSMS | Report settlement at end of transaction **(New in 2.1)** |
| `VatNumberValidation` | CSMS → CPO | Validate a VAT number **(New in 2.1)** |

### 6.2 Tariff Flow

1. CSMS sends `SetDefaultTariff` (CSMS→CS) with tariff structure (price per kWh, fixed fees, time-based rates).
2. CS stores the tariff locally.
3. When a transaction starts, CS uses the stored tariff for local cost calculation.
4. CSMS can update the tariff mid-transaction with `ChangeTransactionTariff`.
5. When the transaction ends, CS sends `NotifySettlement` with the final cost calculation.
6. CSMS can clear tariffs (e.g., when station is decommissioned) with `ClearTariffs`.
7. For B2B scenarios, `VatNumberValidation` allows checking VAT registration numbers.

### 6.3 Tariff and Cost Display

The tariff flow integrates with the Display Message block (O):
- `SetDisplayMessage` shows tariff info before the transaction
- `CostUpdated` pushes running cost during the transaction
- Final cost shown after transaction ends

---

## 7. Common Pitfalls

### 7.1 V2G Profile with Missing Negative Limits

**Problem:** A V2G profile is created but only has positive limit values. The CS will charge but never discharge.

**Fix:** Include negative `limit` values in `chargingSchedulePeriod` for discharge periods. Use `chargingRateUnit: Power` for clarity.

### 7.2 Conflicting Charging Profiles Across Purposes

**Problem:** A `TxDefaultProfile` with a high stack level overlaps with a `ChargingStationMaxProfile`, creating unexpected composite schedules.

**Fix:** Always verify composite schedule output with `GetCompositeSchedule` after installing profiles. If using V2G, account for both import and export directions.

### 7.3 DER Control Without V2X Capability

**Problem:** `SetDERControl` is sent to a CS that doesn't support V2X or has `V2XCtrlr.Enabled = false`.

**Fix:** Check `V2XCtrlr.Enabled` and `DERCtrlr.Enabled` before sending DER control messages. The CS will respond with `Rejected` if the feature is not supported.

### 7.4 Dynamic Schedule Update Race Condition

**Problem:** `UpdateDynamicSchedule` and `SetChargingProfile` are sent simultaneously, causing the CS to apply a stale schedule.

**Fix:** `UpdateDynamicSchedule` is designed for quick updates and should be used instead of (not alongside) `SetChargingProfile` during time-critical operations. For configuration changes, use `SetChargingProfile`. For rapid adjustments, use `UpdateDynamicSchedule`.

### 7.5 Missing Tariff for Local Cost Calculation

**Problem:** CS attempts local cost calculation but no tariff has been set via `SetDefaultTariff`.

**Fix:** Always set a default tariff before enabling `TariffCtrlr.Enabled`. The CS should fall back to displaying "Cost unknown" or using the `CostUpdated` message from CSMS when no local tariff is available.

> **ESCALATE: VENDOR-DEPENDENT** — The exact fallback behavior when no tariff is available depends on the CS firmware implementation. An AI agent MUST ask which hardware/firmware is targeted.

---

*This document is a community reference for AI agents. For the authoritative specification, refer to the official OCPP 2.1 documents from OCA.*