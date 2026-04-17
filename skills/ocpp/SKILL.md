---
name: ocpp
description: >
  OCPP protocol reference for EV charging infrastructure development.
  Covers OCPP 2.1, OCPP 2.0.1, and OCPP 1.6J. Use when working with OCPP messages,
  charging station code, CSMS/Central System backends, smart charging,
  transaction handling, or EV charging protocols. Activates on keywords:
  OCPP, charging station, charge point, CSMS, Central System, EVSE,
  charging profile, BootNotification, TransactionEvent, StartTransaction,
  StopTransaction, SetChargingProfile, SetDERControl, BatterySwap,
  NotifyAllowedEnergyTransfer, AFRRSignal, or any OCPP message name.
user-invocable: true
allowed-tools: Read, Grep, Glob
argument-hint: "[topic: smart-charging | authorize | transactions | schemas | sequences | 1.6 | ...]"
---

# OCPP — AI Agent Reference

You are assisting a developer working on EV charging infrastructure using OCPP.
This skill covers **OCPP 2.1**, **OCPP 2.0.1**, and **OCPP 1.6J**. Use it to provide accurate
schema references, implementation guidance, and to flag areas where the spec is silent.

## Version Detection

Detect the OCPP version from the developer's code:
- **1.6J indicators:** `StartTransaction`, `StopTransaction`, `RemoteStartTransaction`, `RemoteStopTransaction`, `Charge Point`, `Central System`, `idTag` (string), `connectorId` without `evseId`, `.req` / `.conf` naming
- **2.0.1 indicators:** `TransactionEvent`, `RequestStartTransaction`, `RequestStopTransaction`, `Charging Station`, `CSMS`, `IdTokenType` (object), `evseId`, `Request` / `Response` naming, no V2X/DER messages
- **2.1 indicators:** All 2.0.1 indicators PLUS: `NotifyAllowedEnergyTransfer`, `SetDERControl`, `BatterySwap`, `AFRRSignal`, `UsePriorityCharging`, `OpenPeriodicEventStream`, `SetDefaultTariff`, `NotifyWebPaymentStarted`, `V2XCtrlr`, `DERCtrlr`, `TariffCtrlr`

If unclear, ask the developer which version they're using.

## Quick Reference

**What is OCPP:** Open Charge Point Protocol — communication between EV Charging Stations and a management backend over WebSocket + JSON. The station initiates the connection. Both sides can send messages.

### OCPP 2.1
- **Roles:** Charging Station (CS) ↔ CSMS
- **Device Model:** Charging Station → EVSE(s) → Connector(s). `evseId` and `connectorId` are 1-indexed. `evseId=0` means the whole station.
- **91 messages**, organized by 19 Functional Blocks (A–S)
- **New in 2.1:** Bidirectional charging (Q), DER control (R), Battery swapping (S), Tariff and Cost (I) as full block, Priority charging, Frequency containment, Periodic event streams, Web payment

### OCPP 2.0.1
- **Roles:** Charging Station (CS) ↔ CSMS
- **Device Model:** Charging Station → EVSE(s) → Connector(s). `evseId` and `connectorId` are 1-indexed. `evseId=0` means the whole station.
- **64 messages**, organized by Functional Block

### OCPP 1.6J
- **Roles:** Charge Point (CP) ↔ Central System (CS)
- **Device Model:** Charge Point → Connector(s). `connectorId` is 1-indexed. `connectorId=0` means the whole Charge Point. No EVSE concept.
- **28 messages**, organized by Feature Profile (Core, Smart Charging, Firmware, Local Auth List, Reservation, Remote Trigger)

**Message Frame (both versions):** JSON-RPC-like. Three types:
- `CALL` — `[2, messageId, action, payload]`
- `CALLRESULT` — `[3, messageId, payload]`
- `CALLERROR` — `[4, messageId, errorCode, errorDescription, errorDetails]`

## All 91 OCPP 2.1 Messages

Includes all 2.0.1 messages plus 27 new ones marked with ★.

### Provisioning & Lifecycle (Block B)
- `BootNotification` (CS→CSMS) — Station registers on connect
- `Heartbeat` (CS→CSMS) — Keepalive
- `StatusNotification` (CS→CSMS) — Connector/EVSE status changes
- `GetVariables` (CSMS→CS) — Read configuration
- `SetVariables` (CSMS→CS) — Write configuration
- `GetBaseReport` (CSMS→CS) — Request full variable inventory
- `GetReport` (CSMS→CS) — Request custom report
- `NotifyReport` (CS→CSMS) — Variable inventory response (paginated)
- `SetNetworkProfile` (CSMS→CS) — Configure network connection profiles
- `Reset` (CSMS→CS) — Reboot station
- `DataTransfer` (CS↔CSMS) — Bidirectional vendor extension

### Authorization (Blocks C/D)
- `Authorize` (CS→CSMS) — Validate ID token (extended in 2.1 for prepaid, ad hoc, QR)
- `ClearCache` (CSMS→CS) — Clear authorization cache
- `SendLocalList` (CSMS→CS) — Push local auth list
- `GetLocalListVersion` (CSMS→CS) — Query local auth list version

### Transactions (Block E)
- `TransactionEvent` (CS→CSMS) — Started/Updated/Ended events (extended in 2.1 for fixed costs, resume)
- `RequestStartTransaction` (CSMS→CS) — Remote start
- `RequestStopTransaction` (CSMS→CS) — Remote stop
- `GetTransactionStatus` (CSMS→CS) — Query outstanding transaction messages
- `MeterValues` (CS→CSMS) — Send meter values outside transaction context

### Remote Control / Availability (Blocks F/G)
- `TriggerMessage` (CSMS→CS) — Request CS to send a specific message
- `UnlockConnector` (CSMS→CS) — Physically unlock connector
- `ChangeAvailability` (CSMS→CS) — Set EVSE operative/inoperative

### Smart Charging (Block K)
- `SetChargingProfile` (CSMS→CS) — Install charging profile (enhanced for V2G)
- `GetChargingProfiles` (CSMS→CS) — Query installed profiles
- `ClearChargingProfile` (CSMS→CS) — Remove profiles
- `ClearedChargingLimit` (CS→CSMS) — External limit cleared
- `NotifyChargingLimit` (CS→CSMS) — External limit notification
- `ReportChargingProfiles` (CS→CSMS) — Profile query response
- `GetCompositeSchedule` (CSMS→CS) — Calculate effective schedule
- `NotifyEVChargingSchedule` (CS→CSMS) — EV-proposed schedule (ISO 15118)
- `NotifyEVChargingNeeds` (CS→CSMS) — Report EV charging needs (ISO 15118)
- ★ `PullDynamicScheduleUpdate` (CS→CSMS) — CS requests dynamic schedule update
- ★ `UpdateDynamicSchedule` (CSMS→CS) — CSMS pushes dynamic schedule update

### Bidirectional Charging (Block Q) ★
- ★ `NotifyAllowedEnergyTransfer` (CS→CSMS) — EV allows V2X energy transfer

### DER Control (Block R) ★
- ★ `SetDERControl` (CSMS→CS) — Configure DER control settings
- ★ `GetDERControl` (CSMS→CS) — Retrieve DER control configuration
- ★ `ReportDERControl` (CS→CSMS) — Report DER control configuration
- ★ `ClearDERControl` (CSMS→CS) — Remove DER control settings
- ★ `NotifyDERAlarm` (CS→CSMS) — Report DER alarm event
- ★ `NotifyDERStartStop` (CS→CSMS) — Report DER control start/stop status

### Tariff and Cost (Block I) ★
- ★ `SetDefaultTariff` (CSMS→CS) — Set the default tariff
- ★ `GetTariffs` (CSMS→CS) — Retrieve tariff information
- ★ `ClearTariffs` (CSMS→CS) — Clear tariff information
- ★ `ChangeTransactionTariff` (CSMS→CS) — Change tariff for active transaction
- ★ `NotifySettlement` (CS→CSMS) — Report settlement information
- ★ `VatNumberValidation` (CSMS→CPO) — Validate a VAT number

### Payment (within Block I) ★
- ★ `NotifyWebPaymentStarted` (CS→CSMS) — Web payment initiated

### Priority Charging (within Block K) ★
- ★ `NotifyPriorityCharging` (CS→CSMS) — Priority charging started/modified
- ★ `UsePriorityCharging` (CSMS→CS) — Request priority charging

### Frequency Containment (within Blocks K/R) ★
- ★ `AFRRSignal` (CSMS→CS) — AFRR (frequency restoration reserve) signal

### Periodic Event Streams (within Block N) ★
- ★ `OpenPeriodicEventStream` (CSMS→CS) — Open a periodic event stream
- ★ `ClosePeriodicEventStream` (CSMS→CS) — Close a periodic event stream
- ★ `GetPeriodicEventStream` (CSMS→CS) — Query stream status
- ★ `NotifyPeriodicEventStream` (CS→CSMS) — Send periodic monitoring data
- ★ `AdjustPeriodicEventStream` (CSMS→CS) — Adjust stream parameters

### Battery Swapping (Block S) ★
- ★ `BatterySwap` (CS↔CSMS) — Battery swap operation
- ★ `RequestBatterySwap` (CS→CSMS) — Request a battery swap

### Firmware (Block L)
- `UpdateFirmware` (CSMS→CS) — Trigger firmware update
- `FirmwareStatusNotification` (CS→CSMS) — Update progress
- `PublishFirmware` (CSMS→CS) — Make firmware available to local network
- `PublishFirmwareStatusNotification` (CS→CSMS) — Publish progress
- `UnpublishFirmware` (CSMS→CS) — Remove published firmware

### Security & Certificates (Blocks A/M)
- `Get15118EVCertificate` (CS→CSMS) — EV certificate request
- `GetCertificateStatus` (CS→CSMS) — OCSP status check
- ★ `GetCertificateChainStatus` (CS→CSMS) — Certificate chain status
- `SignCertificate` (CS→CSMS) — CSR for station certificate
- `CertificateSigned` (CSMS→CS) — Signed certificate delivery
- `InstallCertificate` (CSMS→CS) — Install CA certificate
- `DeleteCertificate` (CSMS→CS) — Remove certificate
- `GetInstalledCertificateIds` (CSMS→CS) — List installed certs
- `SecurityEventNotification` (CS→CSMS) — Report security-related event

### Diagnostics & Monitoring (Block N)
- `GetLog` (CSMS→CS) — Request log upload
- `LogStatusNotification` (CS→CSMS) — Log upload progress
- `NotifyEvent` (CS→CSMS) — Component/variable events
- `SetMonitoringBase` (CSMS→CS) — Set monitoring baseline
- `SetVariableMonitoring` (CSMS→CS) — Configure variable monitors
- `SetMonitoringLevel` (CSMS→CS) — Set monitoring severity level
- `GetMonitoringReport` (CSMS→CS) — Query active monitors
- `ClearVariableMonitoring` (CSMS→CS) — Remove monitors
- `NotifyMonitoringReport` (CS→CSMS) — Monitor query response
- `CustomerInformation` (CSMS→CS) — Request customer data
- `NotifyCustomerInformation` (CS→CSMS) — Customer data response

### Display Messages (Block O)
- `CostUpdated` (CSMS→CS) — Update displayed cost
- `SetDisplayMessage` (CSMS→CS) — Show message on display
- `GetDisplayMessages` (CSMS→CS) — Query displayed messages
- `ClearDisplayMessage` (CSMS→CS) — Remove displayed message
- `NotifyDisplayMessages` (CS→CSMS) — Display message query response

### Reservation (Block H)
- `ReserveNow` (CSMS→CS) — Create reservation
- `CancelReservation` (CSMS→CS) — Cancel reservation
- `ReservationStatusUpdate` (CS→CSMS) — Reservation expired/removed

## All 64 OCPP 2.0.1 Messages

### Provisioning & Lifecycle
- `BootNotification` (CS→CSMS) — Station registers on connect
- `Heartbeat` (CS→CSMS) — Keepalive
- `StatusNotification` (CS→CSMS) — Connector/EVSE status changes
- `GetVariables` (CSMS→CS) — Read configuration
- `SetVariables` (CSMS→CS) — Write configuration
- `GetBaseReport` (CSMS→CS) — Request full variable inventory
- `NotifyReport` (CS→CSMS) — Variable inventory response (paginated)
- `SetNetworkProfile` (CSMS→CS) — Configure network connection profiles
- `Reset` (CSMS→CS) — Reboot station

### Authorization
- `Authorize` (CS→CSMS) — Validate ID token
- `ClearCache` (CSMS→CS) — Clear authorization cache
- `SendLocalList` (CSMS→CS) — Push local auth list
- `GetLocalListVersion` (CSMS→CS) — Query local auth list version

### Transactions
- `TransactionEvent` (CS→CSMS) — Started/Updated/Ended events
- `RequestStartTransaction` (CSMS→CS) — Remote start
- `RequestStopTransaction` (CSMS→CS) — Remote stop
- `GetTransactionStatus` (CSMS→CS) — Query outstanding transaction messages
- `MeterValues` (CS→CSMS) — Send meter values outside transaction context

### Remote Control
- `TriggerMessage` (CSMS→CS) — Request CS to send a specific message
- `UnlockConnector` (CSMS→CS) — Physically unlock connector
- `ChangeAvailability` (CSMS→CS) — Set EVSE operative/inoperative

### Smart Charging
- `SetChargingProfile` (CSMS→CS) — Install charging profile
- `GetChargingProfiles` (CSMS→CS) — Query installed profiles
- `ClearChargingProfile` (CSMS→CS) — Remove profiles
- `ClearedChargingLimit` (CS→CSMS) — External limit cleared
- `NotifyChargingLimit` (CS→CSMS) — External limit notification
- `ReportChargingProfiles` (CS→CSMS) — Profile query response
- `GetCompositeSchedule` (CSMS→CS) — Calculate effective schedule
- `NotifyEVChargingSchedule` (CS→CSMS) — EV-proposed schedule (ISO 15118)
- `NotifyEVChargingNeeds` (CS→CSMS) — Report EV charging needs (ISO 15118)

### Firmware
- `UpdateFirmware` (CSMS→CS) — Trigger firmware update
- `FirmwareStatusNotification` (CS→CSMS) — Update progress
- `PublishFirmware` (CSMS→CS) — Make firmware available to local network
- `PublishFirmwareStatusNotification` (CS→CSMS) — Publish progress
- `UnpublishFirmware` (CSMS→CS) — Remove published firmware

### Security & Certificates
- `Get15118EVCertificate` (CS→CSMS) — EV certificate request
- `GetCertificateStatus` (CS→CSMS) — OCSP status check
- `SignCertificate` (CS→CSMS) — CSR for station certificate
- `CertificateSigned` (CSMS→CS) — Signed certificate delivery
- `InstallCertificate` (CSMS→CS) — Install CA certificate
- `DeleteCertificate` (CSMS→CS) — Remove certificate
- `GetInstalledCertificateIds` (CSMS→CS) — List installed certs
- `SecurityEventNotification` (CS→CSMS) — Report security-related event

### Diagnostics & Monitoring
- `GetLog` (CSMS→CS) — Request log upload
- `LogStatusNotification` (CS→CSMS) — Log upload progress
- `NotifyEvent` (CS→CSMS) — Component/variable events
- `SetMonitoringBase` (CSMS→CS) — Set monitoring baseline
- `SetVariableMonitoring` (CSMS→CS) — Configure variable monitors
- `SetMonitoringLevel` (CSMS→CS) — Set monitoring severity level
- `GetMonitoringReport` (CSMS→CS) — Query active monitors
- `ClearVariableMonitoring` (CSMS→CS) — Remove monitors
- `NotifyMonitoringReport` (CS→CSMS) — Monitor query response
- `CustomerInformation` (CSMS→CS) — Request customer data
- `NotifyCustomerInformation` (CS→CSMS) — Customer data response

### Display Messages
- `CostUpdated` (CSMS→CS) — Update displayed cost
- `SetDisplayMessage` (CSMS→CS) — Show message on display
- `GetDisplayMessages` (CSMS→CS) — Query displayed messages
- `ClearDisplayMessage` (CSMS→CS) — Remove displayed message
- `NotifyDisplayMessages` (CS→CSMS) — Display message query response

### Reservation
- `ReserveNow` (CSMS→CS) — Create reservation
- `CancelReservation` (CSMS→CS) — Cancel reservation
- `ReservationStatusUpdate` (CS→CSMS) — Reservation expired/removed

### Data Transfer
- `DataTransfer` (CS↔CSMS) — Bidirectional vendor extension

## All 28 OCPP 1.6J Messages

### Core (required profile)
- `Authorize` (CP→CS) — Validate idTag
- `BootNotification` (CP→CS) — Charge Point registers after (re)boot
- `ChangeAvailability` (CS→CP) — Set connector operative/inoperative
- `ChangeConfiguration` (CS→CP) — Set a configuration key
- `ClearCache` (CS→CP) — Clear authorization cache
- `DataTransfer` (CP↔CS) — Vendor-specific data exchange
- `GetConfiguration` (CS→CP) — Read configuration keys
- `Heartbeat` (CP→CS) — Keepalive
- `MeterValues` (CP→CS) — Periodic meter readings
- `RemoteStartTransaction` (CS→CP) — Remote start
- `RemoteStopTransaction` (CS→CP) — Remote stop
- `Reset` (CS→CP) — Reboot Charge Point
- `StartTransaction` (CP→CS) — Transaction started
- `StatusNotification` (CP→CS) — Connector status/error change
- `StopTransaction` (CP→CS) — Transaction ended
- `UnlockConnector` (CS→CP) — Physically unlock connector

### Smart Charging
- `SetChargingProfile` (CS→CP) — Install charging profile
- `ClearChargingProfile` (CS→CP) — Remove profiles
- `GetCompositeSchedule` (CS→CP) — Get effective schedule

### Firmware Management
- `GetDiagnostics` (CS→CP) — Request diagnostic log upload
- `DiagnosticsStatusNotification` (CP→CS) — Upload progress
- `UpdateFirmware` (CS→CP) — Trigger firmware update
- `FirmwareStatusNotification` (CP→CS) — Update progress

### Local Auth List Management
- `SendLocalList` (CS→CP) — Push local authorization list
- `GetLocalListVersion` (CS→CP) — Query list version

### Reservation
- `ReserveNow` (CS→CP) — Reserve a connector
- `CancelReservation` (CS→CP) — Cancel reservation

### Remote Trigger
- `TriggerMessage` (CS→CP) — Request CP to send a specific message

## Key Data Types (OCPP 2.1 / 2.0.1)

- **IdTokenType** — User identification (eMAID, RFID, etc.) with optional groupIdToken
- **ChargingProfileType** — Charging limits: id, stackLevel, purpose, kind, chargingSchedule
- **MeterValueType** — Timestamped array of SampledValue (energy, power, current, voltage, SoC)
- **EVSEType** — EVSE identifier (id + optional connectorId)
- **StatusInfoType** — Reason code + additional info for status responses
- **TransactionType** — Transaction state: transactionId, chargingState, stoppedReason
- **ChargingScheduleType** — Time-based power/current limits with periods
- **IdTokenInfoType** — Authorization result: status, cacheExpiryDateTime, groupIdToken

**New in 2.1:**
- **DERControlType** — DER control settings: id, name, DERControlType, gridCodeParameters
- **AbsolutePriceScheduleType** — Absolute price schedule for tariff management
- **StreamDataElementType** — Data element for periodic event streams

## Escalation Model

When implementing OCPP behavior, you will encounter areas where the specification does not fully define what to do. These are categorized as:

### SPEC-SILENT
The OCPP specification does not define behavior for this case. You MUST flag this to the developer. Do NOT silently pick a default.

### VENDOR-DEPENDENT
Behavior depends on the Charging Station hardware or firmware. Ask which hardware/firmware is targeted.

### POLICY-DEPENDENT
Behavior depends on business rules, site configuration, or grid operator requirements. Ask about the business/operational context.

### Escalation Strictness

Check the developer's project `CLAUDE.md` for escalation preferences. They may write something like:

> For OCPP: use pragmatic escalation mode.

or:

> OCPP escalation: strict — always ask before assuming spec-silent behavior.

Two modes:

- **strict (default):** Stop and ask the developer before proceeding. Present specific options. Do not write code for the ambiguous area until answered.
- **pragmatic:** Flag the ambiguity but pick a reasonable default. Leave a visible annotation:
  ```
  // OCPP SPEC-SILENT: [description of assumption]. Verify this matches your requirements.
  ```

If no escalation preference is found in `CLAUDE.md`, default to **strict**.

## Documentation File Map

When you need detailed field-level schemas, sequence diagrams, or worked examples, read the relevant file from the plugin's `docs/` directory. Use `${CLAUDE_PLUGIN_ROOT}` to resolve the path.

| Topic | File to read |
|-------|-------------|
| **All shared data types (enums + composites)** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.0.1-DataTypes.md` |
| **Authorization schemas** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.0.1-Schemas/OCPP-2.0.1-Schemas-Authorization.md` |
| **Availability schemas** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.0.1-Schemas/OCPP-2.0.1-Schemas-Availability.md` |
| **Diagnostics schemas** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.0.1-Schemas/OCPP-2.0.1-Schemas-Diagnostics.md` |
| **Display schemas** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.0.1-Schemas/OCPP-2.0.1-Schemas-Display.md` |
| **Firmware schemas** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.0.1-Schemas/OCPP-2.0.1-Schemas-Firmware.md` |
| **Provisioning schemas** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.0.1-Schemas/OCPP-2.0.1-Schemas-Provisioning.md` |
| **Reservation schemas** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.0.1-Schemas/OCPP-2.0.1-Schemas-Reservation.md` |
| **Security schemas** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.0.1-Schemas/OCPP-2.0.1-Schemas-Security.md` |
| **Smart Charging schemas** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.0.1-Schemas/OCPP-2.0.1-Schemas-SmartCharging.md` |
| **Transaction schemas** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.0.1-Schemas/OCPP-2.0.1-Schemas-Transactions.md` |
| **Boot, auth, transaction sequences** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.0.1-Sequences/OCPP-2.0.1-Sequences.md` |
| **Offline, firmware, diagnostics sequences** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.0.1-Sequences/OCPP-2.0.1-Sequences-Operational.md` |
| **Smart Charging deep-dive** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.0.1-SmartCharging/OCPP-2.0.1-SmartCharging.md` |
| **Smart Charging worked examples** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.0.1-SmartCharging/OCPP-2.0.1-SmartCharging-Examples.md` |
| **ISO 15118 + Smart Charging** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.0.1-SmartCharging/OCPP-2.0.1-SmartCharging-ISO15118.md` |
| **OCPP 2.0.1 overview + migration guide** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.0.1.md` |
| **Documentation methodology + trust model** | `${CLAUDE_PLUGIN_ROOT}/docs/METHODOLOGY.md` |
| | |
| **OCPP 2.1 overview + migration guide** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.1.md` |
| **2.1 Data types (enums + composites)** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.1-DataTypes.md` |
| **2.1 Provisioning schemas** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.1-Schemas/OCPP-2.1-Schemas-Provisioning.md` |
| **2.1 Authorization schemas** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.1-Schemas/OCPP-2.1-Schemas-Authorization.md` |
| **2.1 Transaction schemas** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.1-Schemas/OCPP-2.1-Schemas-Transactions.md` |
| **2.1 Smart Charging schemas** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.1-Schemas/OCPP-2.1-Schemas-SmartCharging.md` |
| **2.1 Firmware schemas** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.1-Schemas/OCPP-2.1-Schemas-Firmware.md` |
| **2.1 Security schemas** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.1-Schemas/OCPP-2.1-Schemas-Security.md` |
| **2.1 Diagnostics schemas** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.1-Schemas/OCPP-2.1-Schemas-Diagnostics.md` |
| **2.1 Availability schemas** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.1-Schemas/OCPP-2.1-Schemas-Availability.md` |
| **2.1 Reservation schemas** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.1-Schemas/OCPP-2.1-Schemas-Reservation.md` |
| **2.1 Display schemas** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.1-Schemas/OCPP-2.1-Schemas-Display.md` |
| **2.1 Bidirectional Charging schemas** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.1-Schemas/OCPP-2.1-Schemas-BidirectionalCharging.md` |
| **2.1 DER Control schemas** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.1-Schemas/OCPP-2.1-Schemas-DERControl.md` |
| **2.1 Battery Swapping schemas** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.1-Schemas/OCPP-2.1-Schemas-BatterySwapping.md` |
| **2.1 Tariffs and Cost schemas** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.1-Schemas/OCPP-2.1-Schemas-TariffsAndCost.md` |
| **2.1 Payment schemas** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.1-Schemas/OCPP-2.1-Schemas-Payment.md` |
| **2.1 Periodic Event Streams schemas** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.1-Schemas/OCPP-2.1-Schemas-PeriodicEventStreams.md` |
| **2.1 Priority Charging schemas** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.1-Schemas/OCPP-2.1-Schemas-PriorityCharging.md` |
| **2.1 Frequency Containment schemas** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.1-Schemas/OCPP-2.1-Schemas-FrequencyContainment.md` |
| **2.1 Smart Charging deep-dive** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.1-SmartCharging/OCPP-2.1-SmartCharging.md` |
| **2.1 Message sequences** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-2.1-Sequences/OCPP-2.1-Sequences.md` |
| | |
| **OCPP 1.6J overview + config keys** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-1.6J.md` |
| **1.6J Core schemas** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-1.6J-Schemas/OCPP-1.6J-Schemas-Core.md` |
| **1.6J Smart Charging schemas** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-1.6J-Schemas/OCPP-1.6J-Schemas-SmartCharging.md` |
| **1.6J Firmware schemas** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-1.6J-Schemas/OCPP-1.6J-Schemas-Firmware.md` |
| **1.6J Local Auth List schemas** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-1.6J-Schemas/OCPP-1.6J-Schemas-LocalAuthList.md` |
| **1.6J Reservation schemas** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-1.6J-Schemas/OCPP-1.6J-Schemas-Reservation.md` |
| **1.6J Remote Trigger schemas** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-1.6J-Schemas/OCPP-1.6J-Schemas-RemoteTrigger.md` |
| **1.6J Message sequences** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-1.6J-Sequences/OCPP-1.6J-Sequences.md` |
| **1.6J Smart Charging deep-dive** | `${CLAUDE_PLUGIN_ROOT}/docs/OCPP-1.6J-SmartCharging/OCPP-1.6J-SmartCharging.md` |

### How to use the file map

1. Identify the topic from the developer's question
2. Read the relevant file(s) using the Read tool
3. Cite specific fields, constraints, and enum values from the docs
4. Flag any ESCALATE markers you encounter in the docs

### Topic argument routing

If invoked with `/ocpp <topic>`, immediately read the relevant files:

**OCPP 2.1 topics:**
- `/ocpp 2.1` → read OCPP-2.1.md overview
- `/ocpp 2.1 smart-charging` or `/ocpp 2.1 v2x` → read 2.1 SmartCharging deep-dive + schemas
- `/ocpp 2.1 schemas` → read all 2.1 Schema files
- `/ocpp 2.1 sequences` → read 2.1 Sequences file
- `/ocpp 2.1 types` or `/ocpp 2.1 data-types` → read 2.1 DataTypes
- `/ocpp 2.1 der` → read 2.1 DER Control schemas
- `/ocpp 2.1 tariff` or `/ocpp 2.1 cost` → read 2.1 TariffsAndCost schemas
- `/ocpp 2.1 battery-swap` → read 2.1 BatterySwapping schemas
- `/ocpp 2.1 bidirectional` → read 2.1 BidirectionalCharging schemas
- `/ocpp 2.1 priority-charging` → read 2.1 PriorityCharging schemas

**OCPP 1.6J topics:**
- `/ocpp 1.6` or `/ocpp 1.6j` → read OCPP-1.6J.md overview
- `/ocpp 1.6 smart-charging` → read 1.6J SmartCharging schemas + deep-dive
- `/ocpp 1.6 schemas` → read all 1.6J Schema files
- `/ocpp 1.6 sequences` → read 1.6J Sequences file
- `/ocpp 1.6 core` → read 1.6J Core schemas
- `/ocpp 1.6 firmware` → read 1.6J Firmware schemas
- `/ocpp 1.6 auth-list` → read 1.6J LocalAuthList schemas
- `/ocpp 1.6 reservation` → read 1.6J Reservation schemas

**OCPP 2.0.1 topics (default):**
- `/ocpp smart-charging` → read all 3 SmartCharging files
- `/ocpp authorize` or `/ocpp authorization` → read Authorization schemas
- `/ocpp transactions` → read Transaction schemas + Sequences
- `/ocpp provisioning` or `/ocpp boot` → read Provisioning schemas + Sequences
- `/ocpp schemas` → read all Schema files
- `/ocpp sequences` → read both Sequence files
- `/ocpp types` or `/ocpp data-types` → read DataTypes
- `/ocpp firmware` → read Firmware schemas + Operational sequences
- `/ocpp diagnostics` → read Diagnostics schemas + Operational sequences
- `/ocpp reservation` → read Reservation schemas
- `/ocpp display` → read Display schemas
- `/ocpp security` or `/ocpp certificates` → read Security schemas
- `/ocpp availability` → read Availability schemas
- Any other topic → search across all docs using grep

## Behavioral Guidelines

1. **Always cite the source.** When referencing a field, type, or constraint, mention which doc it comes from. Distinguish schema-derived facts (high confidence) from AI interpretation (lower confidence).

2. **Respect the escalation model.** When you encounter an `> **ESCALATE:**` marker in the docs, follow the escalation strictness rules above.

3. **Detect version from context.** Use the version detection rules above. If the code uses `StartTransaction`/`StopTransaction`, it's 1.6J — read 1.6J docs. If it uses `TransactionEvent` without 2.1-specific messages, it's 2.0.1. If it uses V2X/DER/battery swap messages, it's 2.1. If no version indicators are present, assume 2.1 and mention the assumption.

4. **Don't invent protocol behavior.** If you're unsure whether something is spec-defined, check the docs first. If the docs don't cover it, say so explicitly rather than guessing.

5. **Use the schemas for validation.** When the developer writes OCPP message payloads, validate field names, types, required/optional status, and constraints against the schema docs.
