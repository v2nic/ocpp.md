# OCPP.md — Open Charge Point Protocol Reference

> A structured OCPP reference for AI agents and developers building EV charging infrastructure. Covers **OCPP 2.1**, **OCPP 2.0.1**, and **OCPP 1.6J** with field-level message schemas, sequence diagrams, smart charging deep-dives, and explicit markers for every place the spec leaves a decision to you.
>
> Source on [GitHub](https://github.com/alexeimoisseev/ocpp.md).

---

## Why This Exists

OCPP specifications are hundreds of pages long and full of deliberate gaps — places where behavior depends on your hardware vendor, your business rules, or your grid operator's requirements. AI coding agents that rely on training data alone tend to fill those gaps with plausible-sounding defaults. Sometimes they're right. Often they're not, and you don't find out until production.

OCPP.md gives AI agents (and human developers) a structured reference that distinguishes between what the spec defines, what it intentionally leaves open, and what depends on your specific deployment. Every ambiguous area is marked with an **escalation marker** — the agent stops and asks you instead of guessing.

---

## Using with AI Agents

### Claude Code

Install as a plugin:

```
/plugin marketplace add https://github.com/alexeimoisseev/ocpp.md
/plugin install ocpp@ocpp
```

Then use `/ocpp <topic>` to load specific reference material, or just ask OCPP-related questions — the plugin activates automatically when it detects OCPP keywords in your code or prompts.

### Other Agents (Cursor, Windsurf, Copilot, etc.)

Clone the repository and point your agent at the `docs/` directory. See the [AI Agent Setup Guide](./ai-agent-setup/) for detailed instructions.

---

## OCPP 2.1

The latest version of the specification, adding bidirectional charging (V2G/V2X), DER control, battery swapping, tariff management, periodic event streams, priority charging, and frequency containment. 91 messages organized by 19 Functional Blocks (A–S). Fully backward compatible with 2.0.1.

**Reference docs:**

- [OCPP 2.1 Overview & Architecture](./OCPP-2.1.md) — Roles, device model, transport, message frame, all 91 messages, migration guide
- [Shared Data Types](./OCPP-2.1-DataTypes.md) — 21 enums and 43 composite types used across messages

**Schemas (field-level, generated from OCA JSON schemas):**

- [Provisioning](./OCPP-2.1-Schemas/OCPP-2.1-Schemas-Provisioning.md) — BootNotification, GetVariables, SetVariables, Reset, etc.
- [Authorization](./OCPP-2.1-Schemas/OCPP-2.1-Schemas-Authorization.md) — Authorize, SendLocalList, ClearCache
- [Transactions](./OCPP-2.1-Schemas/OCPP-2.1-Schemas-Transactions.md) — TransactionEvent, RequestStartTransaction, MeterValues
- [Smart Charging](./OCPP-2.1-Schemas/OCPP-2.1-Schemas-SmartCharging.md) — SetChargingProfile, GetCompositeSchedule, dynamic schedules
- [Bidirectional Charging](./OCPP-2.1-Schemas/OCPP-2.1-Schemas-BidirectionalCharging.md) — NotifyAllowedEnergyTransfer
- [DER Control](./OCPP-2.1-Schemas/OCPP-2.1-Schemas-DERControl.md) — SetDERControl, GetDERControl, NotifyDERAlarm
- [Battery Swapping](./OCPP-2.1-Schemas/OCPP-2.1-Schemas-BatterySwapping.md) — BatterySwap, RequestBatterySwap
- [Tariffs and Cost](./OCPP-2.1-Schemas/OCPP-2.1-Schemas-TariffsAndCost.md) — SetDefaultTariff, GetTariffs, NotifySettlement
- [Payment](./OCPP-2.1-Schemas/OCPP-2.1-Schemas-Payment.md) — NotifyWebPaymentStarted
- [Periodic Event Streams](./OCPP-2.1-Schemas/OCPP-2.1-Schemas-PeriodicEventStreams.md) — OpenPeriodicEventStream, ClosePeriodicEventStream
- [Priority Charging](./OCPP-2.1-Schemas/OCPP-2.1-Schemas-PriorityCharging.md) — NotifyPriorityCharging, UsePriorityCharging
- [Frequency Containment](./OCPP-2.1-Schemas/OCPP-2.1-Schemas-FrequencyContainment.md) — AFRRSignal
- [Firmware](./OCPP-2.1-Schemas/OCPP-2.1-Schemas-Firmware.md) — UpdateFirmware, FirmwareStatusNotification
- [Security](./OCPP-2.1-Schemas/OCPP-2.1-Schemas-Security.md) — CertificateSigned, InstallCertificate, GetCertificateChainStatus
- [Diagnostics](./OCPP-2.1-Schemas/OCPP-2.1-Schemas-Diagnostics.md) — GetLog, NotifyEvent, SetVariableMonitoring
- [Availability](./OCPP-2.1-Schemas/OCPP-2.1-Schemas-Availability.md) — ChangeAvailability, StatusNotification, Heartbeat
- [Reservation](./OCPP-2.1-Schemas/OCPP-2.1-Schemas-Reservation.md) — ReserveNow, CancelReservation
- [Display](./OCPP-2.1-Schemas/OCPP-2.1-Schemas-Display.md) — SetDisplayMessage, CostUpdated

**Behavioral docs:**

- [Core Sequences](./OCPP-2.1-Sequences/OCPP-2.1-Sequences.md) — Boot, authorization, transactions, V2X, DER control, battery swapping
- [Smart Charging Deep-Dive](./OCPP-2.1-SmartCharging/OCPP-2.1-SmartCharging.md) — Profile model, composite schedules, V2G, DER, priority charging, AFRR, tariff management

---

## OCPP 2.0.1

The current specification, recommended for all new deployments. 64 messages organized by Functional Block.

**Reference docs:**

- [OCPP 2.0.1 Overview & Architecture](./ocpp-2.0.1/) — Roles, device model, transport, message frame, all 64 messages
- [Shared Data Types](./ocpp-2.0.1/data-types/) — 34 enums and composite types used across messages

**Schemas (field-level, generated from OCA JSON schemas):**

- [Provisioning](./ocpp-2.0.1/schemas/provisioning/) — BootNotification, GetVariables, SetVariables, Reset, etc.
- [Authorization](./ocpp-2.0.1/schemas/authorization/) — Authorize, SendLocalList, ClearCache
- [Transactions](./ocpp-2.0.1/schemas/transactions/) — TransactionEvent, RequestStartTransaction, MeterValues
- [Smart Charging](./ocpp-2.0.1/schemas/smart-charging/) — SetChargingProfile, GetCompositeSchedule, etc.
- [Firmware](./ocpp-2.0.1/schemas/firmware/) — UpdateFirmware, FirmwareStatusNotification
- [Security](./ocpp-2.0.1/schemas/security/) — CertificateSigned, InstallCertificate, SecurityEventNotification
- [Diagnostics](./ocpp-2.0.1/schemas/diagnostics/) — GetLog, NotifyEvent, SetVariableMonitoring
- [Availability](./ocpp-2.0.1/schemas/availability/) — ChangeAvailability, StatusNotification, Heartbeat
- [Reservation](./ocpp-2.0.1/schemas/reservation/) — ReserveNow, CancelReservation
- [Display](./ocpp-2.0.1/schemas/display/) — SetDisplayMessage, CostUpdated

**Behavioral docs:**

- [Sequences](./ocpp-2.0.1/sequences/) — Boot, authorization, transaction lifecycle, offline replay
- [Sequences (Operational)](./ocpp-2.0.1/sequences/operational/) — Firmware updates, diagnostics, reset
- [Smart Charging Deep-Dive](./ocpp-2.0.1/smart-charging/) — Profile model, composite schedules, AC/DC differences
- [Smart Charging Examples](./ocpp-2.0.1/smart-charging/examples/) — Worked examples with full JSON payloads
- [Smart Charging & ISO 15118](./ocpp-2.0.1/smart-charging/iso15118/) — EV-side schedules, Plug & Charge integration
- [Charging Profile Generator](./ocpp-2.0.1/smart-charging/generator/) — Interactive tool to build SetChargingProfileRequest payloads

---

## OCPP 1.6J

The most widely deployed version. 28 messages organized by Feature Profile. Uses JSON over WebSocket transport (the "J" suffix).

**Reference docs:**

- [OCPP 1.6J Overview & Architecture](./ocpp-1.6j/) — Roles, connector model, transport, all 28 messages, config keys, differences from 2.0.1

**Schemas (field-level, generated from OCA JSON schemas):**

- [Core](./ocpp-1.6j/schemas/core/) — BootNotification, Authorize, StartTransaction, StopTransaction, StatusNotification, etc. (16 messages)
- [Smart Charging](./ocpp-1.6j/schemas/smart-charging/) — SetChargingProfile, ClearChargingProfile, GetCompositeSchedule
- [Firmware](./ocpp-1.6j/schemas/firmware/) — UpdateFirmware, GetDiagnostics, status notifications
- [Local Auth List](./ocpp-1.6j/schemas/local-auth-list/) — SendLocalList, GetLocalListVersion
- [Reservation](./ocpp-1.6j/schemas/reservation/) — ReserveNow, CancelReservation
- [Remote Trigger](./ocpp-1.6j/schemas/remote-trigger/) — TriggerMessage

**Behavioral docs:**

- [Sequences](./ocpp-1.6j/sequences/) — Boot, authorization, transaction lifecycle, status reporting, offline behavior
- [Smart Charging Deep-Dive](./ocpp-1.6j/smart-charging/) — Profile purposes, stack levels, composite schedule, common pitfalls
- [Charging Profile Generator](./ocpp-1.6j/smart-charging/generator/) — Interactive tool to build SetChargingProfile.req payloads

---

## The Escalation Model

This is the core idea behind OCPP.md. Both OCPP specifications have areas where behavior is intentionally left to the implementer. Most AI agents treat these like any other requirement and quietly make something up. OCPP.md marks every such area so the agent knows to ask:

- **SPEC-SILENT** — The specification doesn't define this. You need to make a decision.
- **VENDOR-DEPENDENT** — Behavior varies by charging station hardware or firmware. Ask which hardware you're targeting.
- **POLICY-DEPENDENT** — Depends on business rules, site configuration, or grid operator requirements.

By default, the agent stops and asks. If you want it to pick reasonable defaults during prototyping, add this to your project configuration (e.g., `CLAUDE.md`):

```
For OCPP: use pragmatic escalation mode.
```

---

## About This Project

- **Source:** [github.com/alexeimoisseev/ocpp.md](https://github.com/alexeimoisseev/ocpp.md)
- **Methodology:** All schemas are mechanically extracted from official OCA JSON schemas. Behavioral docs are AI-authored with explicit confidence tiers. See [Methodology](./methodology/).
- **License:** Apache 2.0. Does not redistribute the official OCPP specification.
- **Disclaimer:** Non-normative documentation. Always verify against the official specification and your vendor's documentation. OCPP is a trademark of the Open Charge Alliance. This project is not affiliated with or endorsed by the OCA.
