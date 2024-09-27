---
title: ELMO OSCP API Specification

language_tabs: # must be one of https://github.com/rouge-ruby/rouge/wiki/List-of-supported-languages-and-lexers
  - json

toc_footers:
  - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

search: true

code_clipboard: true

meta:
  - name: description
    content: ELMO OSCP API Specification
---

# ELMO OSCP API Specification

此文件描述 CSMS 如何透過 OSCP API 與 ELMO 進行各種溝通流程，包含 Register、Handshake、日前型協商及緊急通知等流程。

# ELMO OSCP 測試環境

|                              |                                          |
|:-----------------------------|:-----------------------------------------|
| **ELMO 測試環境 Base URL**       | `https://elmo-oscp-testbed.plato-io.com` |
| **`<OSCP_PREFIX>`**          | `/oscp/2.0`                              |
| **`<CALLBACK_PREFIX>`**      | `/trigger-callback/2.0`                  |
| **ELMO Authorization Token** | `<ELMO-AUTH-TOKEN>`                      |


# 流程

## Register

Register 流程用於 CSMS 初次註冊並取得 ELMO OSCP API 的使用權限。

|   |                        |                                                          |
|---|:-----------------------|:---------------------------------------------------------|
| 1 | **取得 Initial Token**   | 向 ELMO 管理員取得 initial token。                              |
| 2 | **CSMS 向 ELMO 發送註冊請求** | CSMS 發送 `register` 請求到 ELMO，進行註冊。                        |
| 3 | **模擬 ELMO 發送註冊回應**     | (測試環境 Only)<br>手動觸發測試平台的 `register callback`，模擬 ELMO 回應。 |
| 4 | **CSMS 接收註冊回應**        | CSMS 會接收到來自 ELMO 的 `register` 回應，表示註冊成功。                 |
| 5 | **儲存 ELMO Token**      | 註冊成功後，CSMS 需儲存向 ELMO 認證用的 token。                         |


## Handshake

Handshake 流程用於建立 CSMS 與 ELMO 之間的初步連接。

|   |                                   |                                                                        |
|---|:----------------------------------|:-----------------------------------------------------------------------|
| 1 | **模擬 ELMO 發送 Handshake 請求**       | (測試環境 Only)<br>手動觸發測試平台的 `handshake callback`，模擬 ELMO 發送 Handshake 請求。 |
| 2 | **CSMS 接收 Handshake 請求**          | CSMS 接收到來自 ELMO 的 `handshake`，表示啟動初步連接流程。                              |
| 3 | **CSMS 回應 Handshake Acknowledge** | CSMS 需要向 ELMO 回覆 `handshake_acknowledge`，確認連接成功。                       |


## 日前型協商

日前型協商流程用於 ELMO 與 CSMS 進行協商與確認充電站的可用容量。

### 指定容量通知

ELMO 會在每日 10:00 前，向 CSMS 發送指定容量通知，提供隔日的充電站可用容量。

|   |                      |                                                                                                                                           |
|---|:---------------------|:------------------------------------------------------------------------------------------------------------------------------------------|
| 1 | **模擬 ELMO 發送指定容量通知** | (測試環境 Only)<br>手動觸發測試平台的 `update_group_capacity_forecast callback`，<br>`purpose` 帶入 `negotiation_assign_capacity`，<br>模擬 ELMO 發送指定容量通知請求。 |
| 2 | **CSMS 接收指定容量通知**    | CSMS 接收到來自 ELMO 的 `update_group_capacity_forecast`，日期為隔日，可取得隔日充電站可用容量。                                                                    |
| 3 | **套用指定容量**           | CSMS 根據通知套用指定容量。                                                                                                                          |


### 申請額外可用容量

當 CSMS 判斷充電站在隔日需要額外的容量時，在當日 14:00 前可以向 ELMO 申請額外可用容量。ELMO 會在當日 16:00 前，回覆 CSMS 申請結果。

|   |                      |                                                                                                                                                             |
|---|:---------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 1 | **CSMS 發送額外可用容量申請**  | CSMS 向 ELMO 發送 `adjust_group_capacity_forecast` 請求，申請隔日的額外可用容量。                                                                                             |
| 2 | **模擬 ELMO 回覆額外可用容量** | (測試環境 Only)<br>手動觸發測試平台的 `update_group_capacity_forecast callback`，<br>`purpose` 帶入 `negotiation_reply_request_additional_capacity`，<br>模擬 ELMO 回覆額外可用容量申請。 |
| 3 | **CSMS 接收額外可用容量回覆**  | CSMS 接收到來自 ELMO 的 `update_group_capacity_forecast`，日期為隔日，可取得更新的隔日充電站可用容量。                                                                                   |
| 4 | **套用指定容量**           | CSMS 根據回覆套用指定容量。                                                                                                                                            |


## 緊急通知

當 ELMO 需要調控當日接下來的可用容量時，會發送緊急通知。

|   |                    |                                                                                                                                         |
|---|:-------------------|:----------------------------------------------------------------------------------------------------------------------------------------|
| 1 | **模擬 ELMO 發送緊急通知** | (測試環境 Only)<br>手動觸發測試平台的 `update_group_capacity_forecast callback`，<br>`purpose` 帶入 `emergency_assign_capacity`，<br>模擬 ELMO 發送指定容量通知請求。 |
| 2 | **CSMS 接收緊急通知**    | CSMS 接收到來自 ELMO 的 `update_group_capacity_forecast`，日期為當日，表示緊急調控可用容量。                                                                    |
| 3 | **套用指定容量**         | CSMS 根據通知快速套用緊急調控的容量。                                                                                                                   |

## 回報累積用電量

此流程用於 CSMS 向 ELMO 回報充電站的累積用電量。

|   |                    |                                                        |
|---|:-------------------|:-------------------------------------------------------|
| 1 | **收集充電站用電量**       | CSMS 收集充電站的累積用電量 (kWh)                                 |
| 2 | **CSMS 發送累積用電量更新** | CSMS 向 ELMO 發送 `update_group_measurements`，回報充電站累積用電量。 |
| 3 | **回傳頻率**            | CSMS 需要每 1 分鐘回報充電站的累積用電量。                              |


# Messages - General

## HTTP Requests

參見 OSCP 2.0 Specification – 4.1.2. HTTP Requests

| HTTP Header        | Purpose                                                         |
|:-------------------|:----------------------------------------------------------------|
| `Authorization`    | `Token <ELMO-AUTH-TOKEN>`                                       |
| `X-Request-ID`     | (Unique request ID of this message.)                            |
| `X-Correlation-ID` | (Reference to a request ID that this message is a response to.) |


# Messages - Register

## CSMS 向 ELMO 發送註冊請求

> Example request message :

```json
{
  "token": "csms-oscp-token",
  "version_url": [
    {
      "version": "2.0",
      "base_url": "https://csms-base-url/oscp/fp/2.0"
    }
  ]
}
```

### Endpoints

|                 |                          |
|:----------------|:-------------------------|
| **Endpoint**    | `<OSCP_PREFIX>/register` |
| **HTTP Method** | `POST`                   |

### Message

參見 OSCP 2.0 Specification – 4.3.1.1. Register


## 模擬 ELMO 發送註冊回應

> Example request message :

```json
{
  "callback_url": "https://csms-base-url/oscp/fp/2.0/register",
  "header": {
    "authorization": "<CSMS-AUTH-TOKEN>"
  }
}
```

### Endpoints

|                 |                              |
|:----------------|:-----------------------------|
| **Endpoint**    | `<CALLBACK_PREFIX>/register` |
| **HTTP Method** | `POST`                       |

### Message

| Field Name             | Field Type | Description                 |
|:-----------------------|:-----------|:----------------------------|
| `callback_url`         | string     | CSMS 接收 OSCP Register 的 URL |
| `header.authorization` | string     | CSMS 接收 OSCP 的 token        |



# Messages - Handshake

## 模擬 ELMO 發送 Handshake 請求

> Example request message :

```json
{
  "callback_url": "https://csms-base-url/oscp/fp/2.0/handshake",
  "header": {
    "authorization": "<CSMS-AUTH-TOKEN>"
  }
}
```

### Endpoints

|                 |                               |
|:----------------|:------------------------------|
| **Endpoint**    | `<CALLBACK_PREFIX>/handshake` |
| **HTTP Method** | `POST`                        |

### Message

| Field Name             | Field Type | Description                 |
|:-----------------------|:-----------|:----------------------------|
| `callback_url`         | string     | CSMS 接收 OSCP Handshake 的 URL |
| `header.authorization` | string     | CSMS 接收 OSCP 的 token        |


## CSMS 回應 Handshake Acknowledge

> Example request message :

```json
{}
```

### Endpoints

|                 |                                       |
|:----------------|:--------------------------------------|
| **Endpoint**    | `<OSCP_PREFIX>/handshake_acknowledge` |
| **HTTP Method** | `POST`                                |

### Message

參見 OSCP 2.0 Specification – 4.3.3. HandshakeAcknowledge


# Messages - 日前型協商

## 模擬 ELMO 發送指定容量通知

> Example request message :

```json
{
  "callback_url": "https://csms-base-url/oscp/fp/2.0/update_group_capacity_forecast",
  "header": {
    "authorization": "CSMS_TOKEN"
  },
  "purpose": "negotiation_assign_capacity",
  "group_id": "CHARGING_STATION_ID",
  "capacity": 100
}
```

### Endpoints

|                 |                                                    |
|:----------------|:---------------------------------------------------|
| **Endpoint**    | `<CALLBACK_PREFIX>/update_group_capacity_forecast` |
| **HTTP Method** | `POST`                                             |

### Message

| Field Name             | Field Type | Description                                    |
|:-----------------------|:-----------|:-----------------------------------------------|
| `callback_url`         | string     | CSMS 接收 OSCP UpdateGroupCapacityForecast 的 URL |
| `header.authorization` | string     | CSMS 接收 OSCP 的 token                           |
| `purpose`              | string     | 帶入 `negotiation_assign_capacity`               |
| `group_id`             | string     | ELMO 提供的充電站 ID                                 |
| `capacity`             | number     | 指定可用容量 (kW)                                    |


## CSMS 發送額外可用容量申請

> Example request message :

```json
{
    "group_id": "CHARGING_STATION_ID",
    "type": "CONSUMPTION",
    "forecasted_blocks": [
        {
            "capacity": 150,
            "phase": "ALL",
            "unit": "KW",
            "start_time": "2024-09-13T00:00:00.000+08:00",
            "end_time": "2024-09-13T01:00:00.000+08:00"
        },
        {
            "capacity": 150,
            "phase": "ALL",
            "unit": "KW",
            "start_time": "2024-09-13T01:00:00.000+08:00",
            "end_time": "2024-09-13T02:00:00.000+08:00"
        },
        {
            "capacity": 150,
            "phase": "ALL",
            "unit": "KW",
            "start_time": "2024-09-13T02:00:00.000+08:00",
            "end_time": "2024-09-13T03:00:00.000+08:00"
        },
        {
            "capacity": 150,
            "phase": "ALL",
            "unit": "KW",
            "start_time": "2024-09-13T03:00:00.000+08:00",
            "end_time": "2024-09-13T04:00:00.000+08:00"
        },
        {
            "capacity": 150,
            "phase": "ALL",
            "unit": "KW",
            "start_time": "2024-09-13T04:00:00.000+08:00",
            "end_time": "2024-09-13T05:00:00.000+08:00"
        },
        {
            "capacity": 150,
            "phase": "ALL",
            "unit": "KW",
            "start_time": "2024-09-13T05:00:00.000+08:00",
            "end_time": "2024-09-13T06:00:00.000+08:00"
        },
        {
            "capacity": 150,
            "phase": "ALL",
            "unit": "KW",
            "start_time": "2024-09-13T06:00:00.000+08:00",
            "end_time": "2024-09-13T07:00:00.000+08:00"
        },
        {
            "capacity": 150,
            "phase": "ALL",
            "unit": "KW",
            "start_time": "2024-09-13T07:00:00.000+08:00",
            "end_time": "2024-09-13T08:00:00.000+08:00"
        },
        {
            "capacity": 150,
            "phase": "ALL",
            "unit": "KW",
            "start_time": "2024-09-13T08:00:00.000+08:00",
            "end_time": "2024-09-13T09:00:00.000+08:00"
        },
        {
            "capacity": 150,
            "phase": "ALL",
            "unit": "KW",
            "start_time": "2024-09-13T09:00:00.000+08:00",
            "end_time": "2024-09-13T10:00:00.000+08:00"
        },
        {
            "capacity": 150,
            "phase": "ALL",
            "unit": "KW",
            "start_time": "2024-09-13T10:00:00.000+08:00",
            "end_time": "2024-09-13T11:00:00.000+08:00"
        },
        {
            "capacity": 150,
            "phase": "ALL",
            "unit": "KW",
            "start_time": "2024-09-13T11:00:00.000+08:00",
            "end_time": "2024-09-13T12:00:00.000+08:00"
        },
        {
            "capacity": 150,
            "phase": "ALL",
            "unit": "KW",
            "start_time": "2024-09-13T12:00:00.000+08:00",
            "end_time": "2024-09-13T13:00:00.000+08:00"
        },
        {
            "capacity": 150,
            "phase": "ALL",
            "unit": "KW",
            "start_time": "2024-09-13T13:00:00.000+08:00",
            "end_time": "2024-09-13T14:00:00.000+08:00"
        },
        {
            "capacity": 150,
            "phase": "ALL",
            "unit": "KW",
            "start_time": "2024-09-13T14:00:00.000+08:00",
            "end_time": "2024-09-13T15:00:00.000+08:00"
        },
        {
            "capacity": 150,
            "phase": "ALL",
            "unit": "KW",
            "start_time": "2024-09-13T15:00:00.000+08:00",
            "end_time": "2024-09-13T16:00:00.000+08:00"
        },
        {
            "capacity": 150,
            "phase": "ALL",
            "unit": "KW",
            "start_time": "2024-09-13T16:00:00.000+08:00",
            "end_time": "2024-09-13T17:00:00.000+08:00"
        },
        {
            "capacity": 150,
            "phase": "ALL",
            "unit": "KW",
            "start_time": "2024-09-13T17:00:00.000+08:00",
            "end_time": "2024-09-13T18:00:00.000+08:00"
        },
        {
            "capacity": 150,
            "phase": "ALL",
            "unit": "KW",
            "start_time": "2024-09-13T18:00:00.000+08:00",
            "end_time": "2024-09-13T19:00:00.000+08:00"
        },
        {
            "capacity": 150,
            "phase": "ALL",
            "unit": "KW",
            "start_time": "2024-09-13T19:00:00.000+08:00",
            "end_time": "2024-09-13T20:00:00.000+08:00"
        },
        {
            "capacity": 150,
            "phase": "ALL",
            "unit": "KW",
            "start_time": "2024-09-13T20:00:00.000+08:00",
            "end_time": "2024-09-13T21:00:00.000+08:00"
        },
        {
            "capacity": 150,
            "phase": "ALL",
            "unit": "KW",
            "start_time": "2024-09-13T21:00:00.000+08:00",
            "end_time": "2024-09-13T22:00:00.000+08:00"
        },
        {
            "capacity": 150,
            "phase": "ALL",
            "unit": "KW",
            "start_time": "2024-09-13T22:00:00.000+08:00",
            "end_time": "2024-09-13T23:00:00.000+08:00"
        },
        {
            "capacity": 150,
            "phase": "ALL",
            "unit": "KW",
            "start_time": "2024-09-13T23:00:00.000+08:00",
            "end_time": "2024-09-14T00:00:00.000+08:00"
        }
    ]
}
```

### Endpoints

|                 |                                                |
|:----------------|:-----------------------------------------------|
| **Endpoint**    | `<OSCP_PREFIX>/adjust_group_capacity_forecast` |
| **HTTP Method** | `POST`                                         |

### Message

參見 OSCP 2.0 Specification – 4.4.2. AdjustGroupCapacityForecast


## 模擬 ELMO 回覆額外可用容量

> Example request message :

```json
{
  "callback_url": "https://csms-base-url/oscp/fp/2.0/update_group_capacity_forecast",
  "header": {
    "authorization": "CSMS_TOKEN"
  },
  "purpose": "negotiation_reply_request_additional_capacity",
  "group_id": "CHARGING_STATION_ID",
  "capacity": 150
}
```

### Endpoints

|                 |                                                    |
|:----------------|:---------------------------------------------------|
| **Endpoint**    | `<CALLBACK_PREFIX>/update_group_capacity_forecast` |
| **HTTP Method** | `POST`                                             |

### Message

| Field Name             | Field Type | Description                                        |
|:-----------------------|:-----------|:---------------------------------------------------|
| `callback_url`         | string     | CSMS 接收 OSCP UpdateGroupCapacityForecast 的 URL     |
| `header.authorization` | string     | CSMS 接收 OSCP 的 token                               |
| `purpose`              | string     | 帶入 `negotiation_reply_request_additional_capacity` |
| `group_id`             | string     | ELMO 提供的充電站 ID                                     |
| `capacity`             | number     | 指定可用容量 (kW)                                        |


# Messages - 緊急通知

## 模擬 ELMO 發送緊急通知

> Example request message :

```json
{
  "callback_url": "https://csms-base-url/oscp/fp/2.0/update_group_capacity_forecast",
  "header": {
    "authorization": "CSMS_TOKEN"
  },
  "purpose": "emergency_assign_capacity",
  "group_id": "CHARGING_STATION_ID",
  "capacity": 50
}
```

### Endpoints

|                 |                                                    |
|:----------------|:---------------------------------------------------|
| **Endpoint**    | `<CALLBACK_PREFIX>/update_group_capacity_forecast` |
| **HTTP Method** | `POST`                                             |

### Message

| Field Name             | Field Type | Description                                    |
|:-----------------------|:-----------|:-----------------------------------------------|
| `callback_url`         | string     | CSMS 接收 OSCP UpdateGroupCapacityForecast 的 URL |
| `header.authorization` | string     | CSMS 接收 OSCP 的 token                           |
| `purpose`              | string     | 帶入 `emergency_assign_capacity`                 |
| `group_id`             | string     | ELMO 提供的充電站 ID                                 |
| `capacity`             | number     | 指定可用容量 (kW)                                    |


# Messages - 回報累積用電量

## CSMS 發送累積用電量更新

> Example request message :

```json
{
  "group_id": "CSMS-GROUP-ID",
  "measurements": [
    {
      "value": 123.456,
      "phase": "ALL",
      "unit": "KWH",
      "direction": "IMPORT",
      "measure_time": "2024-01-01T11:22:33.456+08:00"
    }
  ]
}
```

### Endpoints

|                 |                                           |
|:----------------|:------------------------------------------|
| **Endpoint**    | `<OSCP_PREFIX>/update_group_measurements` |
| **HTTP Method** | `POST`                                    |

### Message

參見 OSCP 2.0 Specification – 4.5.1. UpdateGroupMeasurements
