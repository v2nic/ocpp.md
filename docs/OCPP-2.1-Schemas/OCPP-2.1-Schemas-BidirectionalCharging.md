# OCPP 2.1 Schemas — BidirectionalCharging

> **Functional Block:** Q
>
> **Types Reference:** Shared types referenced below are defined in [OCPP-2.1-DataTypes.md](../OCPP-2.1-DataTypes.md).
> Types used only within this block are documented [inline below](#local-types).

## Messages

- [NotifyAllowedEnergyTransfer](#notifyallowedenergytransfer) (CS → CSMS)

---

## NotifyAllowedEnergyTransfer

**Direction:** CS → CSMS

### NotifyAllowedEnergyTransferRequest

| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| `allowedEnergyTransfer` | [EnergyTransferModeEnumType](../OCPP-2.1-DataTypes.md#energytransfermodeenumtype)[] | **Yes** | minItems: 1 | Modes of energy transfer that are accepted by CSMS. |
| `transactionId` | string | **Yes** | maxLength: 36 | The transaction for which the allowed energy transfer is allowed. |
| `customData` | [CustomDataType](../OCPP-2.1-DataTypes.md#customdatatype) | No |  |  |


<details>
<summary>Example NotifyAllowedEnergyTransferRequest</summary>

```json
{
  "allowedEnergyTransfer": [
    "AC_single_phase"
  ],
  "transactionId": "string"
}
```

</details>

### NotifyAllowedEnergyTransferResponse

| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| `status` | [NotifyAllowedEnergyTransferStatusEnumType](#notifyallowedenergytransferstatusenumtype) | **Yes** |  |  |
| `statusInfo` | [StatusInfoType](../OCPP-2.1-DataTypes.md#statusinfotype) | No |  |  |
| `customData` | [CustomDataType](../OCPP-2.1-DataTypes.md#customdatatype) | No |  |  |


---

## Local Types

*Types used only within this block's messages.*

### NotifyAllowedEnergyTransferStatusEnumType

| Value |
|-------|
| `Accepted` |
| `Rejected` |

**Used in:** NotifyAllowedEnergyTransfer

---
