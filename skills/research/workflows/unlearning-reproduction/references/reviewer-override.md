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
LLM_BASE_URL=https://<hy4-endpoint>/v1  # real endpoint comes from the local mcp.json; never committed
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

## Family-classifier status

The runtime MCP servers now classify `hy4` / `hunyuan` as family `hunyuan`:

- `mcp-servers/llm-chat/server.py::_FAMILY` includes
  `("hunyuan", ("hy4", "hunyuan"))`.
- `mcp-servers/manual-review/server.py::model_family()` maps `hy4|hunyuan`
  to `hunyuan` with the regex below.

The runtime source lives outside this repository, at
`/usr/local/workspace/.aris-runtime-src/mcp-servers/`. Verify the deployed copy
includes both mappings and has been restarted after the change; otherwise both
servers still return `unknown` and the phase must be `REVIEW_UNAVAILABLE` /
`BLOCKED`.

These are classifier *extensions*, not relaxations: the same-family rejection,
the identity requirement, and fail-closed behaviour for `unknown` stay in force.
Do not delete or weaken those checks. Reference mapping, kept for audit or for
re-applying to another ARIS checkout:

```python
if re.search(r"(^|[^a-z0-9])(hy4|hunyuan)([^a-z0-9]|$)", name):
    families.add("hunyuan")
```

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
