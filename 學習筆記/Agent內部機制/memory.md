# 記憶系統 — `nanobot/agent/memory.py`

## 模組用途與設計意圖

用兩層檔案模擬持久記憶，解決 LLM 無狀態的問題。MEMORY.md 存長期事實（覆寫更新），HISTORY.md 存時間線日誌（只追加）。對話過長時，透過 LLM 呼叫自動整合舊訊息為摘要。

## 關鍵類別與函式摘要

### `MemoryStore`
- **`__init__(workspace)`**：建立 memory 目錄，定義 MEMORY.md 和 HISTORY.md 路徑
- **`read_long_term()`**：讀取 MEMORY.md 全文
- **`write_long_term(content)`**：覆寫 MEMORY.md
- **`append_history(entry)`**：追加一條到 HISTORY.md
- **`get_memory_context()`**：包裝 MEMORY.md 內容供 ContextBuilder 注入系統提示詞
- **`consolidate(session, provider, model)`**：核心 — 取出舊訊息、格式化、呼叫 LLM 以 save_memory 工具回傳摘要，寫入兩個檔案

### `_SAVE_MEMORY_TOOL`（模組級常數）
- 定義給整合用 LLM 的工具 schema，強制回傳 `history_entry`（日誌摘要）和 `memory_update`（更新後完整記憶）

## 值得注意的設計模式

- **LLM-as-a-Tool**：用 LLM 做摘要整合，而非規則式截斷
- **滑動窗口**：memory_window=50，保留最近一半，其餘整合。漸進壓縮而非一次截斷
- **Append-only History**：HISTORY.md 永遠可 grep 搜尋，不怕遺失
- **Tool-forced Output**：強制 LLM 用結構化工具回傳，確保可靠解析
- **防禦性解析**：處理 provider 回傳格式不一致（dict / JSON string / list）

## 與其他模組的關聯

- `agent/context.py`：呼叫 get_memory_context() 將記憶注入系統提示詞
- `agent/loop.py`：在 _consolidate_memory() 中背景觸發 consolidate()
- `providers/base.py`：LLMProvider.chat() 呼叫 LLM 做整合
- `session/manager.py`：Session 物件提供對話歷史和 last_consolidated 指標
