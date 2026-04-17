# OCPP 2.1 Schemas — FrequencyContainment

> **Functional Block:** K/R
>
> **Types Reference:** Shared types referenced below are defined in [OCPP-2.1-DataTypes.md](../OCPP-2.1-DataTypes.md).
> Types used only within this block are documented [inline below](#local-types).

## Messages

- [AFRRSignal](#afrrsignal) (CSMS → CS)

---

## AFRRSignal

**Direction:** CSMS → CS

### AFRRSignalRequest

| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| `signal` | integer | **Yes** |  | Value of signal in _v2xSignalWattCurve_. |
| `timestamp` | string (date-time) | **Yes** |  | Time when signal becomes active. |
| `customData` | [CustomDataType](../OCPP-2.1-DataTypes.md#customdatatype) | No |  |  |


<details>
<summary>Example AFRRSignalRequest</summary>

```json
{
  "signal": 0,
  "timestamp": "2024-01-15T10:30:00Z"
}
```

</details>

### AFRRSignalResponse

| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| `status` | [GenericStatusEnumType](../OCPP-2.1-DataTypes.md#genericstatusenumtype) | **Yes** |  |  |
| `statusInfo` | [StatusInfoType](../OCPP-2.1-DataTypes.md#statusinfotype) | No |  |  |
| `customData` | [CustomDataType](../OCPP-2.1-DataTypes.md#customdatatype) | No |  |  |


---
