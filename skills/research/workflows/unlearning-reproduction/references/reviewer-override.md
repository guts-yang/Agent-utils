# Reviewer Selection — Project Override (Hy4)

This project replaces the default ARIS reviewer model. The override only changes
*which* model reviews; it does **not** relax the evidence gate. A verdict is
accepted only when it is identity-bearing, comes from a model family different
from the executor's, and is recorded with a durable trace/thread id.

## Declared identities

```yaml
executor:
  model: deepseek-v4.1-flash
  family: deepseek
reviewer:
  model: hy4
  family: hunyuan
  relation: different
```

The executor family and the reviewer family must both be known and must differ.
`transport name` never establishes a family: record the identity that the
transport actually returns.

## Transport

Preferred transport: `llm-chat` MCP as an opt-in Codex fallback.

```text
LLM_BASE_URL=https://<hy4-endpoint>/openapi/v1
LLM_MODEL=hy4
LLM_API_KEY=<read from the process environment; never write the value>
LLM_REVIEW_FALLBACK_ENABLED=true
```

- `LLM_API_KEY` must come from the process environment. Do not write it into
  `mcp.json`, `.aris/reviewer.json`, logs, artifacts, or any other file.
- `LLM_REVIEW_FALLBACK_ENABLED=true` is required. Without it the server exposes
  only `chat`, and `mcp__llm-chat__chat` is explicitly **not** verdict-bearing.
- `llm-chat` is a transport, not a backend name. `reviewer_route.py` accepts
  `codex | antigravity | grok | claude-code` plus the legacy
  `oracle-pro | agy | manual | copilot | copilot-native`.

Alternative transport: the `manual` backend (browser/file paste, no API key).
The pasted response must begin with `Reviewer-Model: hy4`.

## Family-classifier prerequisite

ARIS currently classifies only `openai`, `anthropic`, `google`, `deepseek`,
`moonshot`, and `qwen`. Both
`manual-review/server.py::model_family()` and `llm-chat/server.py::model_family()`
therefore return `unknown` for `hy4`/`hunyuan` and fail closed with
"Cannot verify ... model family".

Add a `hunyuan` mapping to both classifiers:

```python
if re.search(r"(^|[^a-z0-9])(hy4|hunyuan)([^a-z0-9]|$)", name):
    families.add("hunyuan")
```

This is a classifier *extension*, not a relaxation: the same-family rejection
and the identity requirement stay in force. Do not delete or weaken those checks.

## Resolution order

1. explicit invocation override
2. `.aris/reviewer.json` (`{"backend": "manual" | "codex", "model": "hy4"}`)
3. this project override
4. the ARIS default (`gpt-6-astra`)

## Still fail-closed

If the transport is unavailable, the returned identity is missing, unknown, or
same-family, or no trace id can be recorded, emit `REVIEW_UNAVAILABLE` and mark
the phase `BLOCKED`. Never substitute the executor's judgement, never invent a
`verdict-id`, and never write `accepted` without a non-empty evidence handle.
