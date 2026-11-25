---
name: unlearning-reproduction
description: This skill should be used when running or coordinating a reproducible LLM/LRM unlearning study across fixed repositories and baselines. It isolates research tracks, delegates idea discovery, experiment execution, review, audit, and claim judgment to the corresponding ARIS skills, and resumes from explicit handoff gates.
allowed-tools: Bash(*), Read, Write, Edit, Grep, Glob, Skill, WebSearch, WebFetch
agent_created: true
---

# Unlearning Reproduction Orchestrator

## Purpose

Coordinate a two-track unlearning reproduction project without duplicating the
research logic implemented by Auto-Research-In-Sleep (ARIS). The orchestrator
owns project configuration, track isolation, phase gates, handoffs, recovery,
and archival. The ARIS skills own literature work, experiment implementation,
review, integrity checks, and claim judgment.

Use this skill for requests such as:

- running the LLM/LRM unlearning reproduction workflow;
- resuming a failed or interrupted unlearning study;
- reproducing candidate ideas against a frozen baseline;
- coordinating idea discovery, experiment execution, and evidence-backed claims
  across separate unlearning tracks.

Do not use it for a single paper lookup, a standalone baseline reproduction, or
paper writing. Route those requests to the narrower ARIS skill.

## Invocation

Parse these options from the request or use the defaults:

```text
/unlearning-reproduction "<research direction>"
  --track both|llm-unlearning|lrm-unlearning
  --mode full|dry-run
  --resume <run_id>
  --auto-proceed true|false
  --pilot run|skip
```

- `track` defaults to `both`. Run tracks independently; do not combine their
  ideas, models, data splits, baselines, metrics, or rankings.
- `mode` defaults to `full`. `dry-run` validates configuration, artifacts, and
   routing without cloning, installing, querying external services, writing
   git config, or running training.
- `resume` resumes the first non-accepted phase for the selected track. Never
  infer acceptance from silence, a file name, or a metric that merely exceeds
  baseline.
- `auto-proceed` defaults to `false` for this project because moving from
  planning to GPU execution is an expensive checkpoint. Pass the resolved value
  to child skills.
- `pilot` defaults to `skip` for this reproduction project. If `run` is
  explicitly selected, obey `/idea-discovery`'s pilot limits and record the
  resulting evidence.

Read [project-config.md](references/project-config.md) and
[handoff-contract.md](references/handoff-contract.md) before acting. The four
manual prompt files remain compatibility templates under
`prompts/research-reproduction/`; use them only for project-specific wording,
not as a second state machine.

## Track configuration

The two default tracks are:

| Track | Scope | Workspace | Heavy data |
|---|---|---|---|
| `llm-unlearning` | ASU-unlearning and L7_ASU-Novel-Methods | `/usr/local/workspace/tracks/llm-unlearning/` | `/jizhicfs/chenyliao/llm-unlearning/` |
| `lrm-unlearning` | R-TOFU and redact; awesome-lrm-unlearning is reference-only | `/usr/local/workspace/tracks/lrm-unlearning/` | `/jizhicfs/chenyliao/lrm-unlearning/` |

The actual repository paths, branches, baseline configuration paths, and metric
definitions come from each track's `INIT_REPORT.md`. If those values are
missing or inconsistent with the project configuration, stop with `BLOCKED`.

## State machine

Each track follows this order:

```text
init
  -> idea-discovery
  -> experiment-bridge
  -> review
  -> audit
  -> claim
  -> archive
```

Use `.aris/runs/<run_id>.json` for the global run and
`<track>/HANDOFF.md` for the track-local state. A phase may be
`running`, `done`, `accepted`, `blocked`, or `skipped`. A phase is accepted
only when its gate is satisfied and the artifact plus evidence handle is
recorded.

When ARIS's canonical `run_state.py` helper is available, use it for global
state. If it is unavailable, do not claim resumability: write a
`BLOCKED` handoff for full mode. Dry-run may still report the missing helper.

## Phase 0: Preflight and init

1. Resolve the project root and load the track configuration.
2. Check that the required ARIS skills and their upstream `tools/`,
   `shared-references/`, and reviewer backend dependencies are available.
3. Check environment variable names without printing values. The current
   project may use `GITHUB_TOKEN`, `HF_TOKEN`, `Hy4_TOKEN`,
   `DEEPSEEK_API_KEY`, `GITHUB_NAME`, and `GITHUB_EMAIL`.
4. Clone the configured repositories at their pinned branches, then configure
   every cloned repository for manual git use as specified in
   `references/project-config.md` → "Git identity for manual commits": set
   repository-local `user.name`/`user.email` from `GITHUB_NAME`/`GITHUB_EMAIL`
   and install a shared credential helper that reads `GITHUB_TOKEN` from
   `.env` when git asks for credentials. Never write the token into a remote
   URL, `.git/config`, an artifact, or a log; never change global git config;
   do not commit or push during init. `dry-run` only checks local config keys
   and helper presence, without cloning or contacting a remote.
5. Check repository branches, CUDA/torch, dependencies, data mounts, and
   baseline configuration paths. Verify the git identity/credential setup of
   each repository read-only (no remote writes). Missing `.env` keys or a
   failed verification marks init `BLOCKED`.
6. Create or validate one `INIT_REPORT.md` per track, recording per-repository
   git identity status without values. Do not run idea discovery, training, or
   evaluation in this phase.

If an existing report is valid, reuse it after checking its paths and branch
state. If a baseline is not frozen, stop before the next phase.

## Phase 1: Idea discovery

For each selected track, call:

```text
/idea-discovery "<track research direction>"
  --AUTO_PROCEED: <resolved value>
  --compact: true
  --render html: false
```

Pass the track's frozen baseline, repositories, model, data split, metric
definition, resource limits, and output directory. Do not read the other track.
The child workflow internally performs literature survey, idea generation,
novelty verification, critical review, method refinement, and experiment
planning. Do not reimplement or reorder those sub-phases.

The phase requires these artifacts:

- `idea-stage/IDEA_REPORT.md`
- `idea-stage/IDEA_CANDIDATES.md`
- `idea-stage/docs/research_contract.md`
- `refine-logs/FINAL_PROPOSAL.md`
- `refine-logs/EXPERIMENT_PLAN.md`

When `pilot=skip`, label pilot-dependent conclusions
`needs manual pilot`; do not silently treat them as validated experiments.
Run the one configured Hy4 novelty cross-check only after the ARIS evidence
gate passes. Do not store the token or its value in an artifact.

Write the selected idea, all artifact paths, novelty result, baseline reference,
and the next action to the track `HANDOFF.md`. A `BLOCKED` evidence gate stops
the track here.

## Phase 2: Experiment bridge

After an accepted idea handoff, call:

```text
/experiment-bridge "<selected idea>"
  --code review: true
  --sanity first: true
  --base repo: <track repository>
  --compact: true
```

The child skill must use the model, dataset split, baseline configuration,
metrics, and output root recorded in `INIT_REPORT.md`. Run sanity, baseline,
main, ablation, and polish milestones in the order defined by
`EXPERIMENT_PLAN.md`. Keep heavy artifacts on the track data disk and use
workspace symlinks where needed.

Stop after results and `EXPERIMENT_TRACKER.md` are written. Do not run the
review loop or decide that a method is good because a number beats baseline.
Record failures, timeouts, partial results, and exact result paths in the
track handoff.

## Phase 3: Review, audit, and claim

Only after completed experiment results exist, run these in order:

1. `/auto-review-loop "<selected idea>" --compact: true`
2. `/experiment-audit`
3. `/result-to-claim`

Never wrap `/auto-review-loop` in an external loop or timer. Its own positive
stop is authoritative: score at least 6 and verdict `ready` or `almost`, or
the configured maximum number of rounds is reached. This review result does
not by itself support an empirical claim.

Route the claim result:

- `yes`: archive proposal, experiment records, review trace, audit, and claim
  conclusion for this idea. Mark the track accepted.
- `partial`: record the narrowed claim or missing evidence and hand concrete
  supplementary experiments back to Phase 2.
- `no`: write a postmortem, record tried approaches, select the next candidate
  from `IDEA_CANDIDATES.md`, and return to Phase 2 with an updated plan.
- review or audit unavailable: mark the track `BLOCKED`; do not substitute the
  executor's opinion for a reviewer or integrity verdict.

“Better than baseline” is a comparison datum, not an acceptance gate. All
claims must remain bounded by the frozen protocol and the evidence ledger.

## Archive and final report

For each accepted or exhausted track, write a separate archive under the
configured workspace archive root. At minimum include:

- final proposal and research contract;
- experiment plan, tracker, raw-result pointers, and baseline comparison;
- review trace and `EXPERIMENT_AUDIT.json`/report;
- `CLAIMS_FROM_RESULTS.md` and the final handoff;
- rejected ideas and their evidence-based reasons.

The final cross-track summary is parallel, not a merged leaderboard. Report
the track, baseline, selected idea, evidence status, limitations, and next
action for each line separately.

## Recovery and reporting

At every phase boundary report:

```text
track:
active_phase:
gate_status:
artifacts:
evidence_handle:
blocking_items:
next_action:
```

On resume, revalidate a `done` but not `accepted` phase before moving on.
Repeated attempts with the same candidate and unchanged blocking reason must
stop as `BLOCKED`; change the input, protocol, resource, or candidate before
retrying. A heartbeat may detect a stalled process, but may not decide that a
claim or paper is good enough.
