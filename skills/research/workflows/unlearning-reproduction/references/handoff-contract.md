# Unlearning Reproduction Handoff Contract

The orchestrator uses one global run record and one handoff per track. A
handoff is valid only when it points to concrete artifacts and states the next
allowed phase.

## `INIT_REPORT.md`

Each track writes this file at:

```text
tracks/<track>/INIT_REPORT.md
```

Required fields:

```yaml
schema_version: "1"
track: llm-unlearning|lrm-unlearning
status: ready|blocked
workspace: "<absolute path>"
repositories:
  - name: "<repository>"
    path: "<absolute path>"
    branch: "<branch>"
    commit: "<sha>"
baseline:
  name: "<frozen baseline>"
  config_path: "<path>"
  config_commit: "<sha or unknown>"
  model: "<model>"
  dataset_split: "<split>"
  metrics: "<definitions or file path>"
compute:
  gpu: "<recorded GPU>"
  cuda: "<version>"
  torch: "<version>"
dependencies: "<check summary or artifact path>"
heavy_data_root: "<absolute path>"
blockers: []
```

`status: ready` means initialization checks passed. It does not mean that the
idea, code, experiment, or claim has passed.

## `HANDOFF.md`

Each phase updates the same track-local file:

```yaml
schema_version: "1"
run_id: "<run id>"
track: llm-unlearning|lrm-unlearning
phase: init|idea-discovery|experiment-bridge|review|audit|claim|archive
status: running|done|accepted|blocked|skipped
selected_idea: "<title or null>"
baseline_ref: "<path and commit from INIT_REPORT.md>"
artifacts:
  - path: "<workspace-relative or absolute path>"
    kind: "<proposal|plan|result|tracker|review|audit|claim|postmortem>"
evidence_handle: "<review trace, verifier report, or deterministic artifact>"
claim_verdict: yes|partial|no|unavailable|null
blocking_items: []
next_action: "<one concrete action>"
updated_at: "<ISO-8601 timestamp>"
```

The `next_action` must name the next phase or the reason the run is blocked.
Do not write a generic “continue” when a gate failed.

## Global run record

`.aris/runs/<run_id>.json` tracks both lines:

```json
{
  "run_id": "unlearning-2026-11-25",
  "tracks": {
    "llm-unlearning": {
      "phase": "experiment-bridge",
      "status": "running",
      "artifact": "tracks/llm-unlearning/HANDOFF.md",
      "evidence_handle": null
    },
    "lrm-unlearning": {
      "phase": "idea-discovery",
      "status": "blocked",
      "artifact": "tracks/lrm-unlearning/HANDOFF.md",
      "evidence_handle": null
    }
  }
}
```

`accepted` requires a non-empty evidence handle. For reviewer gates, record
the actual reviewer and trace/thread identifier. For deterministic gates,
record the verifier report path and, where available, its content hash.

## Result routing

| Result | Required handoff action |
|---|---|
| `yes` | Archive the proposal, experiment evidence, audit, and claim; mark the track accepted. |
| `partial` | Record the narrowed claim or missing evidence; return concrete supplementary runs to `experiment-bridge`. |
| `no` | Record a postmortem and tried approaches; select the next candidate and return to idea/experiment planning. |
| `unavailable` | Mark blocked; do not replace a missing reviewer or audit with the executor's judgment. |

The comparison to the frozen baseline is always recorded, but it is not itself
an acceptance verdict.
