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

## Git identity for manual commits

After cloning, every repository (the `Agent-utils` snapshot, all track
repositories, and reference-only repositories) must be ready for a human to run
`git commit` and `git push` without an interactive prompt. Read `GITHUB_NAME`,
`GITHUB_EMAIL`, and `GITHUB_TOKEN` from `/usr/local/workspace/.env`.

Rules:

- Repository-local config only (`git -C <repo> config --local ...`). Never edit
  global/system config or another repository's config.
- Keep the token out of every file except `.env`: no token in `.git/config`,
  remote URLs, `~/.git-credentials`, artifacts, or logs. A shared credential
  helper reads it from `.env` only when git asks for credentials.
- Store the helper outside all repositories, for example
  `/usr/local/workspace/.git-credential-github.sh` with mode `0700`. The file
  contains code, not secrets.
- Init only makes manual git operations possible. Do not commit, create
  branches, or push during init.
- If `.env` is missing or any of the three keys is empty, mark init `BLOCKED`.

Setup template (replace the `REPO_URL` / `REPO_DIR` values per repository):

```bash
ENV_FILE=/usr/local/workspace/.env
HELPER=/usr/local/workspace/.git-credential-github.sh

cat > "$HELPER" <<'EOF'
#!/usr/bin/env bash
set -euo pipefail
[ "${1:-}" = "get" ] || exit 0
ENV_FILE="${GITHUB_ENV_FILE:-/usr/local/workspace/.env}"
[ -r "$ENV_FILE" ] || exit 1
TOKEN=$(grep -m1 -E '^[[:space:]]*(export[[:space:]]+)?GITHUB_TOKEN=' "$ENV_FILE" \
  | cut -d= -f2- | tr -d '\r' | tr -d '"' | tr -d "'")
[ -n "$TOKEN" ] || exit 1
printf 'username=x-access-token\npassword=%s\n' "$TOKEN"
EOF
chmod 700 "$HELPER"

read_env() { grep -m1 -E "^[[:space:]]*(export[[:space:]]+)?$1=" "$ENV_FILE" \
  | cut -d= -f2- | tr -d '\r' | tr -d '"' | tr -d "'"; }
GH_NAME=$(read_env GITHUB_NAME)
GH_EMAIL=$(read_env GITHUB_EMAIL)
[ -n "$GH_NAME" ] && [ -n "$GH_EMAIL" ] || exit 1

# Non-interactive clone; the temporary -c config is not written to the repo.
REPO_URL="https://github.com/<owner>/<repo>.git"
REPO_DIR="/usr/local/workspace/tracks/llm-unlearning/<repo>"
GIT_TERMINAL_PROMPT=0 git -c credential.helper= \
  -c credential.helper="$HELPER" clone "$REPO_URL" "$REPO_DIR"
git -C "$REPO_DIR" checkout "<branch>"

# Apply to every cloned repository.
for repo in /usr/local/workspace/Agent-utils /usr/local/workspace/tracks/*/*; do
  [ -d "$repo/.git" ] || continue
  git -C "$repo" config --local user.name "$GH_NAME"
  git -C "$repo" config --local user.email "$GH_EMAIL"
  git -C "$repo" config --local credential.helper "$HELPER"
done
```

Verification (read-only, no remote writes; no config values are echoed):

```bash
REPO=/usr/local/workspace/Agent-utils
git -C "$REPO" config --local --get user.name >/dev/null \
  && echo "user.name: configured"
git -C "$REPO" config --local --get user.email >/dev/null \
  && echo "user.email: configured"
git -C "$REPO" config --local --get credential.helper >/dev/null \
  && echo "credential.helper: configured"
GIT_TERMINAL_PROMPT=0 git -C "$REPO" ls-remote origin HEAD
```

The last command only proves that HTTPS credentials resolve without a prompt.

In `dry-run`, check the local config keys and helper file only: do not clone,
contact the remote, or read token values. A repository that fails verification
marks init `BLOCKED`.

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
