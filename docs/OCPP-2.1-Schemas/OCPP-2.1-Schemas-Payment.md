# OCPP 2.1 Schemas — Payment

> **Functional Block:** I
>
> **Types Reference:** Shared types referenced below are defined in [OCPP-2.1-DataTypes.md](../OCPP-2.1-DataTypes.md).
> Types used only within this block are documented [inline below](#local-types).

## Messages

- [NotifyWebPaymentStarted](#notifywebpaymentstarted) (CS → CSMS)

---

## NotifyWebPaymentStarted

**Direction:** CS → CSMS

### NotifyWebPaymentStartedRequest

| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| `evseId` | integer | **Yes** | min: 0.0 | EVSE id for which transaction is requested. |
| `timeout` | integer | **Yes** |  | Timeout value in seconds after which no result of web payment process (e.g. QR code scanning) is to be expected anymore. |
| `customData` | [CustomDataType](../OCPP-2.1-DataTypes.md#customdatatype) | No |  |  |


<details>
<summary>Example NotifyWebPaymentStartedRequest</summary>

```json
{
  "evseId": 0,
  "timeout": 0
}
```

</details>

### NotifyWebPaymentStartedResponse

*No required fields. An empty `{}` is a valid response.*

---
