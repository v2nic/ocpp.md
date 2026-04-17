# OCPP 2.1 Schemas — PeriodicEventStreams

> **Functional Block:** N
>
> **Types Reference:** Shared types referenced below are defined in [OCPP-2.1-DataTypes.md](../OCPP-2.1-DataTypes.md).
> Types used only within this block are documented [inline below](#local-types).

## Messages

- [OpenPeriodicEventStream](#openperiodiceventstream) (CSMS → CS)
- [ClosePeriodicEventStream](#closeperiodiceventstream) (CSMS → CS)
- [GetPeriodicEventStream](#getperiodiceventstream) (CSMS → CS)
- [NotifyPeriodicEventStream](#notifyperiodiceventstream) (CS → CSMS)
- [AdjustPeriodicEventStream](#adjustperiodiceventstream) (CSMS → CS)

---

## OpenPeriodicEventStream

**Direction:** CSMS → CS

### OpenPeriodicEventStreamRequest

| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| `constantStreamData` | [ConstantStreamDataType](#constantstreamdatatype) | **Yes** |  |  |
| `customData` | [CustomDataType](../OCPP-2.1-DataTypes.md#customdatatype) | No |  |  |


<details>
<summary>Example OpenPeriodicEventStreamRequest</summary>

```json
{
  "constantStreamData": {
    "id": 0,
    "variableMonitoringId": 0,
    "params": {}
  }
}
```

</details>

### OpenPeriodicEventStreamResponse

| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| `status` | [GenericStatusEnumType](../OCPP-2.1-DataTypes.md#genericstatusenumtype) | **Yes** |  |  |
| `statusInfo` | [StatusInfoType](../OCPP-2.1-DataTypes.md#statusinfotype) | No |  |  |
| `customData` | [CustomDataType](../OCPP-2.1-DataTypes.md#customdatatype) | No |  |  |


---

## ClosePeriodicEventStream

**Direction:** CSMS → CS

### ClosePeriodicEventStreamRequest

| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| `id` | integer | **Yes** | min: 0.0 | Id of stream to close. |
| `customData` | [CustomDataType](../OCPP-2.1-DataTypes.md#customdatatype) | No |  |  |


<details>
<summary>Example ClosePeriodicEventStreamRequest</summary>

```json
{
  "id": 0
}
```

</details>

### ClosePeriodicEventStreamResponse

*No required fields. An empty `{}` is a valid response.*

---

## GetPeriodicEventStream

**Direction:** CSMS → CS

### GetPeriodicEventStreamRequest

| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| `customData` | [CustomDataType](../OCPP-2.1-DataTypes.md#customdatatype) | No |  |  |


### GetPeriodicEventStreamResponse

| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| `constantStreamData` | [ConstantStreamDataType](#constantstreamdatatype)[] | No | minItems: 1 |  |
| `customData` | [CustomDataType](../OCPP-2.1-DataTypes.md#customdatatype) | No |  |  |


*No fields are required. An empty `{}` is a valid response.*

---

## NotifyPeriodicEventStream

**Direction:** CS → CSMS

### NotifyPeriodicEventStreamRequest

| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| `basetime` | string (date-time) | **Yes** |  | Base timestamp to add to time offset of values. |
| `data` | [StreamDataElementType](#streamdataelementtype)[] | **Yes** | minItems: 1 |  |
| `id` | integer | **Yes** | min: 0.0 | Id of stream. |
| `pending` | integer | **Yes** | min: 0.0 | Number of data elements still pending to be sent. |
| `customData` | [CustomDataType](../OCPP-2.1-DataTypes.md#customdatatype) | No |  |  |


<details>
<summary>Example NotifyPeriodicEventStreamRequest</summary>

```json
{
  "basetime": "2024-01-15T10:30:00Z",
  "data": [
    {
      "t": 0.0,
      "v": "string"
    }
  ],
  "id": 0,
  "pending": 0
}
```

</details>

### NotifyPeriodicEventStreamResponse

*No required fields. An empty `{}` is a valid response.*

---

## AdjustPeriodicEventStream

**Direction:** CSMS → CS

### AdjustPeriodicEventStreamRequest

| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| `id` | integer | **Yes** | min: 0.0 |  |
| `params` | [PeriodicEventStreamParamsType](../OCPP-2.1-DataTypes.md#periodiceventstreamparamstype) | **Yes** |  |  |
| `customData` | [CustomDataType](../OCPP-2.1-DataTypes.md#customdatatype) | No |  |  |


<details>
<summary>Example AdjustPeriodicEventStreamRequest</summary>

```json
{
  "id": 0,
  "params": {}
}
```

</details>

### AdjustPeriodicEventStreamResponse

| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| `status` | [GenericStatusEnumType](../OCPP-2.1-DataTypes.md#genericstatusenumtype) | **Yes** |  |  |
| `statusInfo` | [StatusInfoType](../OCPP-2.1-DataTypes.md#statusinfotype) | No |  |  |
| `customData` | [CustomDataType](../OCPP-2.1-DataTypes.md#customdatatype) | No |  |  |


---

## Local Types

*Types used only within this block's messages.*

### ConstantStreamDataType

| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| `id` | integer | **Yes** | min: 0.0 | Uniquely identifies the stream |
| `params` | [PeriodicEventStreamParamsType](../OCPP-2.1-DataTypes.md#periodiceventstreamparamstype) | **Yes** |  |  |
| `variableMonitoringId` | integer | **Yes** | min: 0.0 | Id of monitor used to report his event. It can be a preconfigured or hardwired monitor. |
| `customData` | [CustomDataType](../OCPP-2.1-DataTypes.md#customdatatype) | No |  |  |


**Used in:** GetPeriodicEventStream, OpenPeriodicEventStream

---

### StreamDataElementType

| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| `t` | number | **Yes** |  | Offset relative to _basetime_ of this message. _basetime_ + _t_ is timestamp of recorded value. |
| `v` | string | **Yes** | maxLength: 2500 |  |
| `customData` | [CustomDataType](../OCPP-2.1-DataTypes.md#customdatatype) | No |  |  |


**Used in:** NotifyPeriodicEventStream

---
