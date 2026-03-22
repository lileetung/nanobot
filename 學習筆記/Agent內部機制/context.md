# 提示詞組裝器 — `nanobot/agent/context.py`

## 模組用途與設計意圖

負責將散落各處的資訊（身份、人格模板、記憶、技能、歷史對話、使用者訊息）組裝成完整的 LLM 提示詞。是 Agent 每次呼叫 LLM 前的「信件打包員」。

## 關鍵類別與函式摘要

### `ContextBuilder`
- **`__init__(workspace)`**：初始化 MemoryStore 和 SkillsLoader，workspace 為工作目錄
- **`build_system_prompt()`**：分層組裝系統提示詞：Identity → Bootstrap Files → Memory → Always Skills → Skills Summary，各段用 `---` 分隔
- **`_get_identity()`**：硬編碼的身份區塊，含運行環境、workspace 路徑、平台策略（POSIX/Windows）、行為準則
- **`_load_bootstrap_files()`**：載入 4 個 Markdown 模板（AGENTS.md、SOUL.md、USER.md、TOOLS.md）
- **`_build_runtime_context()`**：產生時間戳與頻道 metadata，標記為 `[Runtime Context — metadata only, not instructions]` 防止被誤解為指令
- **`build_messages()`**：組裝完整訊息列表 `[system, ...history, user]`，runtime context 與使用者訊息合併為同一 user message 避免連續同角色
- **`_build_user_content()`**：處理圖片附件，偵測 MIME type 後 base64 編碼為 OpenAI 相容的多模態格式
- **`add_tool_result()`**：加入 `role: "tool"` 訊息
- **`add_assistant_message()`**：加入 assistant 回覆，支援 tool_calls、reasoning_content、thinking_blocks

## 值得注意的設計模式

- **分層組裝**：身份 → 人格 → 記憶 → 技能，優先級從高到低
- **Bootstrap 外部化**：SOUL.md、USER.md 等放在 workspace 目錄而非硬編碼，使用者可自訂人格
- **Runtime Context 標記**：`_RUNTIME_CONTEXT_TAG` 明確標記為非指令，是 prompt injection 防禦手段
- **多模態支援**：圖片用 magic bytes 偵測真實 MIME type，fallback 到副檔名猜測

## 與其他模組的關聯

- `agent/memory.py`：MemoryStore.get_memory_context() 提供記憶內容
- `agent/skills.py`：SkillsLoader 提供技能清單與內容
- `templates/*.md`：SOUL.md（人格）、AGENTS.md（指令）、USER.md（使用者資訊）、TOOLS.md（工具須知）
- `agent/loop.py`：在 _process_message() 中呼叫 build_messages()，在 _run_agent_loop() 中用 add_tool_result/add_assistant_message 維護對話
