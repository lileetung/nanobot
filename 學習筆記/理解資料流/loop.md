# Agent 迴圈 — `nanobot/agent/loop.py`

## 模組用途與設計意圖

整個系統的**核心處理引擎**，負責從匯流排取得訊息、組裝上下文、呼叫 LLM、執行工具、回傳回覆。是連接所有模組的樞紐。

## 關鍵類別與函式摘要

### `AgentLoop`
- **`run()`**：主迴圈，持續從 bus 消費訊息，每條訊息 dispatch 為獨立 asyncio.Task
- **`_dispatch(msg)`**：加鎖處理訊息，送出回覆或錯誤訊息
- **`_process_message(msg)`**：完整處理流程 — 斜線指令 → 記憶整合觸發 → 載入歷史 → 組裝上下文 → 呼叫 agent loop → 儲存對話
- **`_run_agent_loop(messages)`**：核心迭代迴圈 — 呼叫 LLM → 有 tool_calls 就執行工具並繼續，沒有就取得最終回覆跳出。最多 40 次迭代
- **`_handle_stop(msg)`**：取消該 session 所有 task 和子代理
- **`_register_default_tools()`**：註冊預設工具（檔案、shell、web、message、spawn、cron）
- **`_save_turn(session, messages)`**：儲存對話，截斷大型工具結果、移除 runtime context、替換 base64 圖片
- **`_consolidate_memory(session)`**：委託 MemoryStore 整合記憶（超過 memory_window 時背景觸發）
- **`_connect_mcp()`**：延遲連接 MCP 伺服器（lazy init）
- **`process_direct(content)`**：CLI/cron 直接呼叫入口

## 值得注意的設計模式

- **Task-based Concurrency**：每條訊息一個 asyncio.Task，可透過 `/stop` 取消
- **Iteration-bounded Loop**：`max_iterations=40` 防止無限工具呼叫迴圈
- **Lazy Initialization**：MCP 連線在第一次需要時才建立
- **Defensive Save**：截斷工具結果、過濾空訊息、移除 runtime context，防止 session 膨脹或毒化

## 與其他模組的關聯

- `bus/queue.py`：consume_inbound / publish_outbound
- `bus/events.py`：InboundMessage / OutboundMessage 資料結構
- `agent/context.py`：ContextBuilder 組裝提示詞
- `agent/memory.py`：MemoryStore 記憶整合
- `agent/tools/registry.py`：ToolRegistry 管理工具定義與執行
- `providers/base.py`：LLMProvider.chat() 呼叫 LLM
- `session/manager.py`：SessionManager 管理對話歷史
- `agent/subagent.py`：SubagentManager 管理子代理
