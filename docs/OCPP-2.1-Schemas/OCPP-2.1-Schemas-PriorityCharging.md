# OCPP 2.1 Schemas — PriorityCharging

> **Functional Block:** K
>
> **Types Reference:** Shared types referenced below are defined in [OCPP-2.1-DataTypes.md](../OCPP-2.1-DataTypes.md).
> Types used only within this block are documented [inline below](#local-types).

## Messages

- [NotifyPriorityCharging](#notifyprioritycharging) (CS → CSMS)
- [UsePriorityCharging](#useprioritycharging) (CSMS → CS)

---

## NotifyPriorityCharging

**Direction:** CS → CSMS

### NotifyPriorityChargingRequest

| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| `activated` | boolean | **Yes** |  | True if priority charging was activated. False if it has stopped using the priority charging profile. |
| `transactionId` | string | **Yes** | maxLength: 36 | The transaction for which priority charging is requested. |
| `customData` | [CustomDataType](../OCPP-2.1-DataTypes.md#customdatatype) | No |  |  |


<details>
<summary>Example NotifyPriorityChargingRequest</summary>

```json
{
  "activated": false,
  "transactionId": "string"
}
```

</details>

### NotifyPriorityChargingResponse

*No required fields. An empty `{}` is a valid response.*

---

## UsePriorityCharging

**Direction:** CSMS → CS

### UsePriorityChargingRequest

| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| `activate` | boolean | **Yes** |  | True to request priority charging. False to request stopping priority charging. |
| `transactionId` | string | **Yes** | maxLength: 36 | The transaction for which priority charging is requested. |
| `customData` | [CustomDataType](../OCPP-2.1-DataTypes.md#customdatatype) | No |  |  |


<details>
<summary>Example UsePriorityChargingRequest</summary>

```json
{
  "activate": false,
  "transactionId": "string"
}
```

</details>

### UsePriorityChargingResponse

| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| `status` | [PriorityChargingStatusEnumType](#prioritychargingstatusenumtype) | **Yes** |  |  |
| `statusInfo` | [StatusInfoType](../OCPP-2.1-DataTypes.md#statusinfotype) | No |  |  |
| `customData` | [CustomDataType](../OCPP-2.1-DataTypes.md#customdatatype) | No |  |  |


---

## Local Types

*Types used only within this block's messages.*

### PriorityChargingStatusEnumType

Result of the request.

| Value |
|-------|
| `Accepted` |
| `Rejected` |
| `NoProfile` |

**Used in:** UsePriorityCharging

---
