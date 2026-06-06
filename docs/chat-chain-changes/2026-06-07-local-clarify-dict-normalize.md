---
date: 2026-06-07
commit: local
feature: clarify 工具 choices dict→str 边界归一化
impact: 上游 agent 误传 `[{key, value}, ...]` 时，Web UI 按钮 label 恢复正常文本，response 回传不再带 Python repr 垃圾。
---

`hermes_bridge.py` 的 `_clarify_callback`（IPC 边界）在 `clarify.requested` 事件 emit 前对 `choices` 做归一化：
dict 形态取 `.value`（fallback `.label` / `.text`，再 fallback `str(c)`）；非 dict 走 `str(c)`。
归一化触发时向 stderr 打 `[clarify-normalize]` 警告，携带 session_id 和原 key 便于溯源。

**根因**：`tools/clarify_tool.py` schema 声明 `choices.items.type = "string"`，但 agent 偶发传 dict。
下游（TS event stream / Pinia store / Vue 模板）都假设 `list[str]`，dict 进来后 Vue 把它 toString，
按钮回传也把整个 dict 当 response 字符串发回，污染 agent 上下文。

**修复范围**：单文件单 hop 改动。Vue / TS / store 不动；所有客户端（Web UI / Desktop / TUI）均受益。
