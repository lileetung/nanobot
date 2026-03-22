# Slack 頻道實作 — `nanobot/channels/slack.py`

## 模組用途與設計意圖

`BaseChannel` 的具體實作，透過 Slack Socket Mode（WebSocket）連接 Slack 平台，處理收發訊息。不需公開 URL，適合本地開發與私有部署。

## 關鍵類別與函式摘要

### `SlackChannel(BaseChannel)`
- **`start()`**：建立 WebSocket 連線、取得 bot user ID、註冊事件監聽器，以 `while` 迴圈保持存活
- **`stop()`**：關閉 socket 連線
- **`send(OutboundMessage)`**：透過 `chat_postMessage` 發送文字、`files_upload_v2` 上傳檔案，支援 thread 回覆
- **`_on_socket_request()`**：核心事件處理，依序執行 ACK → 過濾 → 忽略 bot 自身 → 去重 → 權限 → 回應策略 → 加 reaction → 送進匯流排
- **`_is_allowed()`**：分 DM / 群組兩套權限邏輯
- **`_should_respond_in_channel()`**：群組回應策略（open / mention / allowlist）
- **`_to_mrkdwn()`**：Markdown → Slack mrkdwn 格式轉換，含表格轉清單、code block 保護

## 值得注意的設計模式

- **Observer 模式**：透過 `socket_mode_request_listeners` 註冊事件監聽
- **Defensive Programming**：ACK 防重送、忽略 bot 自身防無限迴圈、mention 去重
- **策略模式**：`group_policy` 控制不同回應行為
- **Thread 級 Session 隔離**：`session_key = slack:{chat_id}:{thread_ts}`，DM 不隔離

## 與其他模組的關聯

- 繼承 `channels/base.py` 的 `BaseChannel`，實作三個抽象方法
- 呼叫 `_handle_message()` 將訊息送進匯流排（來自 base.py）
- 使用 `config/schema.py` 中的 `SlackConfig` 讀取設定
