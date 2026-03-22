# Nanobot 專案導讀進度

## 閱讀順序

| # | 階段 | 模組 | 狀態 | 筆記 |
|---|------|------|------|------|
| 1 | 理解資料流 | `nanobot/bus/events.py` | ✅ 已完成 | DTO 模式定義 InboundMessage/OutboundMessage，session_key 支援對話隔離 |
| 1 | 理解資料流 | `nanobot/channels/base.py` | ✅ 已完成 | ABC 定義頻道契約，Template Method 模式，_handle_message 為核心共用流程 |
| 1 | 理解資料流 | `nanobot/channels/slack.py` | ✅ 已完成 | Socket Mode 連線、多層權限策略、thread 級 session 隔離、Markdown→mrkdwn 轉換 |
| 1 | 理解資料流 | `nanobot/bus/queue.py` | ✅ 已完成 | 雙向 asyncio.Queue 解耦頻道與 Agent，極簡設計只負責傳遞 |
| 1 | 理解資料流 | `nanobot/agent/loop.py` | ✅ 已完成 | 核心迴圈：run→dispatch→process_message→_run_agent_loop，Task 並發、記憶整合、工具逐一執行 |
| 2 | Agent 內部機制 | `nanobot/agent/context.py` | ✅ 已完成 | 分層組裝系統提示詞（Identity→Bootstrap→Memory→Skills），Runtime Context 防注入標記，多模態圖片支援 |
| 2 | Agent 內部機制 | `nanobot/agent/memory.py` | ✅ 已完成 | 雙層記憶（MEMORY.md 長期 + HISTORY.md 日誌），LLM 驅動的 consolidate 整合，滑動窗口漸進壓縮 |
| 2 | Agent 內部機制 | `nanobot/agent/tools/base.py` + `registry.py` | ⬜ 未開始 | 工具系統抽象與註冊機制 |
| 2 | Agent 內部機制 | `nanobot/agent/tools/shell.py` + `filesystem.py` | ⬜ 未開始 | 安全防護、路徑限制 |
| 2 | Agent 內部機制 | `nanobot/agent/loop.py`（深讀） | ⬜ 未開始 | has_tool_calls 判斷、Function Calling 協定、tools.get_definitions() |
| 2 | Agent 內部機制 | `nanobot/templates/SOUL.md` | ⬜ 未開始 | Agent 人格設定，prompt engineering 實踐 |
| 3 | 設定與 Provider | `nanobot/providers/base.py` | ⬜ 未開始 | LLM 抽象介面（LLMResponse 支援 tool_calls、reasoning、thinking） |
| 3 | 設定與 Provider | `nanobot/providers/litellm_provider.py` | ⬜ 未開始 | |
| 3 | 設定與 Provider | `nanobot/providers/registry.py` | ⬜ 未開始 | Provider 註冊與選擇機制 |
| 3 | 設定與 Provider | `nanobot/providers/azure_openai_provider.py` | ⬜ 未開始 | Azure OpenAI 實作 |
| 3 | 設定與 Provider | `nanobot/providers/openai_codex_provider.py` | ⬜ 未開始 | OpenAI Codex + OAuth |
| 3 | 設定與 Provider | `nanobot/providers/custom_provider.py` | ⬜ 未開始 | 自訂 OpenAI-compatible endpoint |
| 3 | 設定與 Provider | `nanobot/providers/transcription.py` | ⬜ 未開始 | 語音轉文字 |
| 3 | 設定與 Provider | `nanobot/config/schema.py` + `loader.py` | ⬜ 未開始 | |
| 3 | 設定與 Provider | `nanobot/config/paths.py` | ⬜ 未開始 | 工作區路徑定義 |
| 4 | 更多工具 | `nanobot/agent/tools/web.py` | ⬜ 未開始 | web_search / web_fetch |
| 4 | 更多工具 | `nanobot/agent/tools/message.py` | ⬜ 未開始 | 主動發送訊息到 channel |
| 4 | 更多工具 | `nanobot/agent/tools/mcp.py` | ⬜ 未開始 | MCP 伺服器整合 |
| 4 | 更多工具 | `nanobot/agent/tools/spawn.py` | ⬜ 未開始 | 子代理任務生成 |
| 4 | 更多工具 | `nanobot/agent/tools/cron.py` | ⬜ 未開始 | 排程任務工具 |
| 5 | 進階功能 | `nanobot/agent/skills.py` | ⬜ 未開始 | Skills 載入與解析邏輯 |
| 5 | 進階功能 | `nanobot/agent/subagent.py` | ⬜ 未開始 | 子代理生成與管理 |
| 5 | 進階功能 | `nanobot/heartbeat/` | ⬜ 未開始 | 定時觸發機制 |
| 5 | 進階功能 | `nanobot/cron/` + `cron/types.py` | ⬜ 未開始 | 排程服務與型別定義 |
| 5 | 進階功能 | `nanobot/skills/` — 框架機制 | ⬜ 未開始 | README.md、SKILL.md 格式 |
| 5 | 進階功能 | `nanobot/skills/memory/` | ⬜ 未開始 | 記憶存取技能 |
| 5 | 進階功能 | `nanobot/skills/summarize/` | ⬜ 未開始 | 摘要技能（URL、檔案、YouTube） |
| 5 | 進階功能 | `nanobot/skills/github/` | ⬜ 未開始 | GitHub 互動技能（gh CLI） |
| 5 | 進階功能 | `nanobot/skills/tmux/` | ⬜ 未開始 | 遠端控制 tmux session |
| 5 | 進階功能 | `nanobot/skills/weather/` | ⬜ 未開始 | 天氣查詢（wttr.in / Open-Meteo） |
| 5 | 進階功能 | `nanobot/skills/cron/` | ⬜ 未開始 | 排程任務建立技能 |
| 5 | 進階功能 | `nanobot/skills/clawhub/` | ⬜ 未開始 | 技能市集搜尋與安裝 |
| 5 | 進階功能 | `nanobot/skills/skill-creator/` | ⬜ 未開始 | 自動建立新技能 |
| 6 | 輔助模組 | `nanobot/cli/commands.py` | ⬜ 未開始 | CLI 入口（onboard/gateway/agent/status） |
| 6 | 輔助模組 | `nanobot/session/manager.py` | ⬜ 未開始 | JSONL 對話歷史管理 |
| 6 | 輔助模組 | `nanobot/channels/manager.py` | ⬜ 未開始 | Channel 初始化與路由 |
| 6 | 輔助模組 | `nanobot/utils/helpers.py` | ⬜ 未開始 | 通用工具函式 |
| 6 | 輔助模組 | `nanobot/templates/` | ⬜ 未開始 | AGENTS.md、TOOLS.md、USER.md、HEARTBEAT.md 模板 |

**狀態說明**：⬜ 未開始 ｜ 🔄 進行中 ｜ ✅ 已完成

## 導讀紀錄

<!-- 每次導讀後在此新增紀錄 -->
<!-- 格式：
### YYYY-MM-DD — 模組名稱
- **重點摘要**：...
- **關鍵發現**：...
- **待釐清問題**：...
- **下次建議**：...
-->
