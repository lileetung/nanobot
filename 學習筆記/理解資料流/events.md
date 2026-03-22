# 事件資料結構 — `nanobot/bus/events.py`

## 模組用途與設計意圖

定義系統中訊息傳遞的**統一信封格式**，讓多種聊天頻道（Telegram、Discord、Slack、WhatsApp）的訊息能以相同結構在系統內流動，Agent 不需關心訊息來源。

## 關鍵類別與函式摘要

### `InboundMessage`（進站訊息）
- 欄位：`channel`、`sender_id`、`chat_id`、`content`、`timestamp`、`media`、`metadata`、`session_key_override`
- `session_key` 屬性：決定哪些訊息屬於同一段對話，預設為 `{channel}:{chat_id}`，可透過 `session_key_override` 覆寫（如 Slack thread 級別隔離）

### `OutboundMessage`（出站訊息）
- 欄位：`channel`、`chat_id`、`content`、`reply_to`、`media`、`metadata`
- 比進站訊息簡單，只需知道送去哪裡、內容是什麼

## 值得注意的設計模式

- **Data Transfer Object (DTO)**：使用 `@dataclass` 定義純資料容器，不含業務邏輯
- `session_key` 的 override 機制提供了彈性，允許不同粒度的對話隔離

## 與其他模組的關聯

- `channels/base.py`：`_handle_message()` 建構 `InboundMessage`，`send()` 接收 `OutboundMessage`
- `bus/queue.py`：匯流排傳遞這兩種訊息
- `agent/loop.py`：Agent 從匯流排取得 `InboundMessage`，處理後產生 `OutboundMessage`
