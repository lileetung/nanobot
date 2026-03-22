# 訊息匯流排 — `nanobot/bus/queue.py`

## 模組用途與設計意圖

用兩條 `asyncio.Queue` 實現頻道與 Agent 之間的**非同步解耦**。頻道不需直接呼叫 Agent，Agent 也不需知道訊息來自哪個頻道。

## 關鍵類別與函式摘要

### `MessageBus`
- **`inbound: asyncio.Queue[InboundMessage]`**：頻道 → Agent 的訊息佇列
- **`outbound: asyncio.Queue[OutboundMessage]`**：Agent → 頻道的回覆佇列
- **`publish_inbound(msg)`**：頻道放入訊息（由 `BaseChannel._handle_message()` 呼叫）
- **`consume_inbound()`**：Agent 取出訊息（阻塞式等待，無訊息時自動暫停）
- **`publish_outbound(msg)`**：Agent 放入回覆
- **`consume_outbound()`**：頻道取出回覆
- **`inbound_size` / `outbound_size`**：佇列長度查詢

## 值得注意的設計模式

- **Producer-Consumer 模式**：頻道是生產者，Agent 是消費者（反向亦然）
- **極簡單一職責**：只負責傳遞，不含路由、過濾、持久化邏輯
- **asyncio.Queue 特性**：協程安全、自動阻塞等待、預設無容量上限

## 與其他模組的關聯

- `bus/events.py`：佇列中傳遞的資料型別（InboundMessage / OutboundMessage）
- `channels/base.py`：透過 `bus.publish_inbound()` 將訊息送入
- `agent/loop.py`：透過 `bus.consume_inbound()` 取出訊息處理
- `channels/manager.py`：透過 `bus.consume_outbound()` 取出回覆並路由到對應頻道
