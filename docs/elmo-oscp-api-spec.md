# ELMO OSCP API Specification

此文件描述如何透過 OSCP API 與 ELMO 進行各種溝通流程，包含 Register、Handshake、日前型協商及緊急通知等流程。

## ELMO OSCP 測試環境

* ELMO Testbed URL: `https://example.com`
* `<OSCP_PREFIX>`: `/oscp/2.0`
* `<CALLBACK_PREFIX>`: `/trigger-callback/2.0`
* 認證 Token `<AUTH_TOKEN>`: `ELMO-TESTBED-TOKEN`


## 流程說明

### Register 流程

Register 流程用於 CSMS 初次註冊並取得 ELMO OSCP API 的使用權限。

1. **取得 Initial Token**: 向 ELMO 管理員取得 initial token。
2. **CSMS 向 ELMO 發送註冊請求**: CSMS 發送 `register` 請求到 ELMO，進行註冊。
3. **(測試環境 Only) 手動觸發註冊 Callback**: 手動觸發測試平台的 `register callback`，模擬 ELMO 回應。
4. **CSMS 接收註冊回應** CSMS 會接收到來自 ELMO 的 `register` 回應，表示註冊成功。
5. **儲存 ELMO Token**: 註冊成功後，CSMS 需儲存向 ELMO 認證用的 token。

### Handshake 流程

Handshake 流程用於建立 CSMS 與 ELMO 之間的初步連接。

1. **(測試環境 Only) 手動觸發 Handshake Callback**: 手動觸發測試平台的 `handshake callback`，模擬 ELMO 發送 Handshake 請求。
2. **CSMS 接收 Handshake 請求**: CSMS 接收到來自 ELMO 的 `handshake`，表示啟動初步連接流程。
3. **CSMS 回應 Handshake Acknowledge**: CSMS 需要向 ELMO 回覆 `handshake_acknowledge`，確認連接成功。

### 日前型協商流程

日前型協商流程用於 ELMO 與 CSMS 進行協商與確認充電站的可用容量。

#### 日前型協商 - 指定容量通知

ELMO 會在每日 10:00 前，向 CSMS 發送指定容量通知，提供隔日的充電站可用容量。

1. **(測試環境 Only) 手動觸發指定容量通知 Callback**: 手動觸發測試平台的 `update_group_capacity_forecast callback`，`purpose` 帶入 `negotiation_assign_capacity`，模擬 ELMO 發送指定容量通知請求。
2. **CSMS 接收指定容量通知**: CSMS 接收到來自 ELMO 的 `update_group_capacity_forecast`，日期為隔日，可取得隔日充電站可用容量。
3. **套用指定容量**: CSMS 根據通知套用指定容量。

#### 日前型協商 - 申請額外可用容量

當 CSMS 判斷充電站在隔日需要額外的容量時，在當日 14:00 前可以向 ELMO 申請額外可用容量。ELMO 會在當日 16:00 前，回覆 CSMS 申請結果。

1. **CSMS 發送額外可用容量申請** CSMS 向 ELMO 發送 `adjust_group_capacity_forecast` 請求，申請隔日的額外可用容量。
2. **(測試環境 Only) 手動觸發額外可用容量回覆 Callback**: 手動觸發測試平台的 `update_group_capacity_forecast callback`，`purpose` 帶入 `negotiation_reply_request_additional_capacity`，模擬 ELMO 回覆額外可用容量申請。
3. **CSMS 接收額外可用容量回覆**: CSMS 接收到來自 ELMO 的 `update_group_capacity_forecast`，日期為隔日，可取得更新的隔日充電站可用容量。
4. **套用指定容量**: CSMS 根據回覆套用指定容量。

### 緊急通知流程

當 ELMO 需要調控當日接下來的可用容量時，會發送緊急通知。

1. **(測試環境 Only) 手動觸發緊急通知 Callback**: 手動觸發測試平台的 `update_group_capacity_forecast callback`，`purpose` 帶入 `emergency_assign_capacity`，模擬 ELMO 發送指定容量通知請求。
2. **CSMS 接收緊急通知**: CSMS 接收到來自 ELMO 的 `update_group_capacity_forecast`，日期為當日，表示緊急調控可用容量。
3. **套用指定容量**: CSMS 根據通知快速套用緊急調控的容量。

### 回報累積用電量流程

此流程用於 CSMS 向 ELMO 回報充電站的累積用電量。

1. **收集充電站用電量**: CSMS 收集充電站的累積用電量 (kWh)
2. **CSMS 發送累積用電量更新**: CSMS 向 ELMO 發送 `update_group_measurements`，回報充電站累積用電量。
