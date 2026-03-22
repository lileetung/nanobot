# 頻道抽象介面 — `nanobot/channels/base.py`

## 模組用途與設計意圖

定義所有聊天頻道的**統一藍圖（ABC）**，讓不同平台（Telegram、Discord、Slack）遵守同一份契約，實現可插拔的頻道架構。

## 關鍵類別與函式摘要

### `BaseChannel(ABC)`
- **`__init__(config, bus)`**：接收頻道設定與訊息匯流排，透過依賴注入解耦
- **`start()`**（抽象）：連接平台、開始監聽訊息，長時間運行的 async task
- **`stop()`**（抽象）：停止監聽、釋放資源
- **`send(OutboundMessage)`**（抽象）：將訊息發送到該平台
- **`is_allowed(sender_id)`**：白名單權限檢查，預設拒絕（空列表=全擋，`"*"`=全放）
- **`_handle_message(...)`**：核心共用流程 — 權限檢查 → 包裝成 `InboundMessage` → 發布到匯流排
- **`is_running`**：狀態查詢屬性

## 值得注意的設計模式

- **Template Method 模式**：`_handle_message()` 定義固定流程，子類別只需實作抽象方法
- **依賴注入**：`bus` 從外部傳入，頻道不需知道匯流排的具體實作
- **安全預設拒絕**：`is_allowed()` 在沒有設定允許名單時，預設拒絕所有存取

## 與其他模組的關聯

- `bus/events.py`：使用 `InboundMessage` 和 `OutboundMessage` 資料結構
- `bus/queue.py`：透過 `bus.publish_inbound()` 將訊息送進匯流排
- `channels/slack.py` 等具體頻道：繼承此類別，實作平台特定邏輯
