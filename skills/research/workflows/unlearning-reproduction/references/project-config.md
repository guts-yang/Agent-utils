# Unlearning Reproduction Project Configuration

This file is the default configuration for the project-specific
`unlearning-reproduction` orchestrator. Replace repository paths, branch names,
baseline references, and metric definitions for another project instead of
editing the orchestration rules.

## Default layout

```yaml
project_root: /usr/local/workspace
archive_root: archives/unlearning-reproduction
state_root: .aris/runs
heavy_data_root: /jizhicfs/chenyliao
default_track: both
default_mode: full
default_auto_proceed: false
default_pilot: skip
```

## Tracks

```yaml
tracks:
  llm-unlearning:
    workspace: /usr/local/workspace/tracks/llm-unlearning
    heavy_data: /jizhicfs/chenyliao/llm-unlearning
    repositories:
      - name: ASU-unlearning
        branch: Auto/20261122
      - name: L7_ASU-Novel-Methods
        branch: new_method
    reference_only: []
    baseline:
      name: "<replace with LLM baseline>"
      config_path: "<replace with frozen baseline config>"
      model: "<replace with model>"
      dataset_split: "<replace with split>"
      metrics: "<replace with official metric definitions>"

  lrm-unlearning:
    workspace: /usr/local/workspace/tracks/lrm-unlearning
    heavy_data: /jizhicfs/chenyliao/lrm-unlearning
    repositories:
      - name: R-TOFU
        branch: Auto
      - name: redact
        branch: fuxian
    reference_only:
      - name: awesome-lrm-unlearning
        branch: main
    baseline:
      name: "<replace with LRM baseline>"
      config_path: "<replace with frozen baseline config>"
      model: "<replace with model>"
      dataset_split: "<replace with split>"
      metrics: "<replace with official metric definitions>"
```

## Environment contract

Only variable names may appear in logs or reports. Values must remain in the
shell environment:

```text
GITHUB_TOKEN
HF_TOKEN
Hy4_TOKEN
DEEPSEEK_API_KEY
GITHUB_NAME
GITHUB_EMAIL
```

The orchestrator checks whether required names are present when a phase needs
them. It must never print, interpolate into a prompt artifact, or commit their
values.

## Immutable protocol fields

Before Phase 1, each track must freeze:

- repository commit/branch;
- baseline configuration path and commit;
- model and tokenizer;
- dataset and split proportions;
- training budget and seeds;
- evaluation scripts and metric definitions;
- heavy artifact root.

Changing one of these fields creates a new run or requires an explicit
baseline re-alignment record. It is not a silent edit to an existing run.
