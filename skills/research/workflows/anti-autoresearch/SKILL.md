---
name: anti-autoresearch
description: >-
  Anti-Autoresearch 事后诚信取证流水线，与 autoresearch-loop 组合使用：把 autoresearch 的 results.tsv/run.log/分支产物整理成可锚定论文目录，再执行 span 锚定证据账本、跨模型审计、确定性裁决，输出 REPORT.md/report.json。当用户说 anti-autoresearch、审一篇投稿的诚信、forensic review、integrity audit this paper、审计 autoresearch 结果、检查 val_bpb 数字是否自洽时使用。detect-only，不改被审论文，不做 AI 文本作者判定。
allowed-tools: Bash(*), Read, Write, Edit, Skill, WebSearch, WebFetch, mcp__codex__codex, mcp__mcp-dblp__search, mcp__mcp-dblp__fuzzy_title_search, mcp__mcp-dblp__get_venue_info
---

# Anti-Autoresearch（与 autoresearch-loop 组合）

把 `guts-yang/Anti-Autoresearch` 的完整取证流水线封装为技能，并与 `autoresearch-loop` 形成闭环：

```
autoresearch-loop                         anti-autoresearch
----------------                          ------------------
改 train.py / commit / 跑实验       →     把实验产物整理为可锚定论文目录
results.tsv / run.log / 分支         →     evidence ledger → 审计 → 确定性裁决
AUTORESEARCH_SUMMARY.md             →     REPORT.md / report.json
                                   ←     AUDIT_FEEDBACK.md（下一轮实验线索）
```

职责边界：**autoresearch-loop 是执行者，会改 `train.py` 并 reset；anti-autoresearch 是只读取证者，绝不修改被审论文或代码，也不参与训练循环。**

## 何时使用

- 用户要求审一篇论文 / 投稿 / arXiv 论文的诚信与自洽性。
- 用户要求检查 autoresearch 跑出的数字、表格、baseline、引用是否经得起核实。
- `autoresearch-loop` 到达预算或人工中断后，需要投稿前门控。
- 用户提到 `REPORT.md`、`claims.json`、`span 锚定`、`HARD_FLAGS`、`REVIEW_UNAVAILABLE`。

不要在以下情况使用：用户只是要继续跑实验循环（用 `autoresearch-loop`）；用户要写论文（用 `paper-writing`）；用户要公开指控某作者造假（本技能明确拒绝）。

## 包内资源与运行根目录

```
anti-autoresearch/
├── SKILL.md                       # 本入口
├── references/
│   └── autoresearch-bridge.md     # results.tsv/run.log → 可锚定论文目录的详细映射
└── bundle/                        # Anti-Autoresearch 原仓库完整快照（不含 .git）
    ├── workflows/anti-autoresearch/SKILL.md   # 主流程，Step 0-5 的唯一权威
    ├── skills/*/SKILL.md                      # 11 个子技能与逐字 reviewer prompt
    ├── tools/                                 # 确定性工具（含 adjudicate_findings.py）
    ├── references/                            # taxonomy、可观测性、独立性契约
    ├── schemas/
    ├── eval/
    └── tests/
```

**关键路径约定**：bundle 内所有 `SKILL.md` 用 `ROOT="${AAR_ROOT:-$(git rev-parse --show-toplevel 2>/dev/null || pwd)}"` 定位工具。为避免被外层 git 仓库（如 `Agent-utils`）抢走 ROOT，执行前必须显式导出 `AAR_ROOT`：

```bash
AAR_SKILL="${AAR_SKILL:-/usr/local/workspace/Agent-utils/skills/research/workflows/anti-autoresearch}"
export AAR_ROOT="$AAR_SKILL/bundle"
ROOT="$AAR_ROOT"
cd "$ROOT" || exit 1
test -f "$ROOT/tools/adjudicate_findings.py" || { echo "FATAL: ROOT 未指向 Anti bundle"; exit 1; }
```

`PAPER_DIR` 必须放在 `bundle/` **之外**的绝对路径，且路径不含空格。不要 `cd` 进 `PAPER_DIR` 后再运行工具。

## 与 autoresearch-loop 的组合模式

### 模式 A：自审自己的 autoresearch 产物（推荐闭环）

1. 先让 `autoresearch-loop` 跑完当前预算或到达人工中断点，保留 `results.tsv`、`run.log`、`AUTORESEARCH_SUMMARY.md`、分支 `autoresearch/<tag>` 和 `train.py` 最终版本。
2. 按 `references/autoresearch-bridge.md` 生成 `PAPER_DIR`：至少包含 `submission.tex`（可锚定句子）、`results/experiments.csv`、`code/train.py`、`code/prepare.py`。
3. 执行本技能的 Step 0-5，产出 `REPORT.md` / `report.json`。
4. 若 verdict 为 `HARD_FLAGS` 或 `SOFT_FLAGS`，把 evidence 表里的 `pattern_id + span + 建议核实点` 写成 `AUDIT_FEEDBACK.md`，放回 autoresearch 项目根目录，作为下一轮 `train.py` 实验的候选线索；**不要把审计 finding 当成已证实的造假**。
5. 在 `AUTORESEARCH_SUMMARY.md` 追加审计 verdict 与 `PAPER_DIR` 路径。

### 模式 B：直接审第三方论文

输入为目录 / PDF / arXiv id，直接进入主流程 Step 0。无需 bridge。

### 组合纪律

- 不要在训练循环内调用本技能；它是投稿前 / 阶段末门控，不是每次实验的监控器。
- 禁止用 `/loop`、`/schedule`、`CronCreate` 轮询裁决。裁决只在论文 / 代码 / 结果文件变化时重跑。
- 无跨模型 reviewer 时只能确定性-only，此时**永远不能得出 CLEAN**；`REVIEW_UNAVAILABLE` 不等于通过。
- `CLEAN_GIVEN_EVIDENCE` 只表示“在本次观测等级下没查到可查问题”，不是“诚实”证明。

## 执行方式

**完整模式（有 codex MCP）**：读取并严格遵循 `bundle/workflows/anti-autoresearch/SKILL.md` 的 Step 0-5。该文件是唯一权威；本入口只覆盖运行根目录和 bridge 约定。按 ledger 的 claim 类型决定是否跑 citation / baseline / proof / eval-design 维度；每个维度开**新的** codex 线程，模型 `gpt-5.6-sol`、`xhigh`、`read-only`、串行，禁止 `codex-reply` 串味。LLM 只提出 finding，只有 `tools/adjudicate_findings.py` 决定 verdict。

**确定性-only 回退（无 codex / 离线）**：在 `bundle/` 下执行以下完整命令块；它会跑全部四个确定性检查器，但报告必须声明语义与代码级维度未运行。

```bash
AAR_SKILL="${AAR_SKILL:-/usr/local/workspace/Agent-utils/skills/research/workflows/anti-autoresearch}"
export AAR_ROOT="$AAR_SKILL/bundle"; ROOT="$AAR_ROOT"; cd "$ROOT" || exit 1
PAPER_DIR="<absolute paper dir, no spaces>"

SLUG=$(basename "$PAPER_DIR" | tr -cs 'A-Za-z0-9.' '-')
TXT=(); [ -f "$PAPER_DIR/paper.txt" ] && TXT=(--pdf-text "$PAPER_DIR/paper.txt")

python3 "$ROOT/tools/build_manifest.py" --paper-id "$SLUG" --dir "$PAPER_DIR" "${TXT[@]}" \
  --out "$PAPER_DIR/artifact_manifest.json"
L=$(python3 -c 'import json,sys;print(json.load(open(sys.argv[1]))["observability_level"])' "$PAPER_DIR/artifact_manifest.json")

if ls "$PAPER_DIR"/*.tex >/dev/null 2>&1; then
  python3 "$ROOT/tools/build_claim_ledger.py" --paper-id "$SLUG" --latex "$PAPER_DIR"/*.tex \
    --observability-level "$L" --out "$PAPER_DIR/claims.json"
else
  python3 "$ROOT/tools/build_claim_ledger.py" --paper-id "$SLUG" --pdf-text "$PAPER_DIR/paper.txt" \
    --observability-level "$L" --out "$PAPER_DIR/claims.json"
fi

python3 "$ROOT/tools/check_numeric_consistency.py" --ledger "$PAPER_DIR/claims.json" \
  --out "$PAPER_DIR/consistency-audit.deterministic.findings.json"
python3 "$ROOT/tools/check_presentation.py" --ledger "$PAPER_DIR/claims.json" \
  --out "$PAPER_DIR/presentation-signals.deterministic.findings.json"
python3 "$ROOT/tools/check_stat_consistency.py" --ledger "$PAPER_DIR/claims.json" \
  --out "$PAPER_DIR/consistency-audit.stat.findings.json"
python3 "$ROOT/tools/check_ai_style.py" --ledger "$PAPER_DIR/claims.json" \
  --out "$PAPER_DIR/ai-style-impressions.findings.json"

python3 - "$PAPER_DIR" <<'PY'
import json, os, sys
cov = {k: "review_unavailable" for k in [
    "consistency-audit", "experiment-forensics", "baseline-comparison-audit",
    "citation-forensics", "presentation-signals", "proof-derivation-forensics",
    "eval-design-forensics", "adversarial-case-builder",
    "novelty-duplication-advisory", "ai-style-impressions"]}
json.dump(cov, open(os.path.join(sys.argv[1], "coverage.json"), "w"), indent=2)
PY

python3 "$ROOT/tools/adjudicate_findings.py" \
  --findings "$PAPER_DIR"/*.findings.json \
  --ledger "$PAPER_DIR/claims.json" --paper-id "$SLUG" --observability-level "$L" \
  --taxonomy-version 0.5 --coverage "$PAPER_DIR/coverage.json" \
  --limitation "Deterministic-only run (no cross-model reviewer): semantic + code-level dimensions were NOT run." \
  --out "$PAPER_DIR/report.json" --md "$PAPER_DIR/REPORT.md"
```

## 观测等级与 verdict

| 等级 | 条件 | 能判定什么 |
|---|---|---|
| L0 | 只有 PDF / `paper.txt` | 文本自洽、数值算术、表面信号；不能断言代码/结果造假 |
| L1 | 有 `.tex` 源码 | 精确 span 锚定、证明/推导；仍不能断言代码/结果造假 |
| L2 | 有代码目录 + `results/`、`outputs/` 或 `logs/` 下的 JSON/CSV | 可检查代码与结果文件对应关系；**仍不做 L3 复现** |

verdict 由规则计算：`critical` 存活 → `HARD_FLAGS`；否则 major/minor → `SOFT_FLAGS`；否则 `CLEAN_GIVEN_EVIDENCE`。任一 verdict-bearing 维度 `review_unavailable` → `REVIEW_UNAVAILABLE`。校验必须满足：

```bash
python3 - "$PAPER_DIR/report.json" <<'PY'
import json, sys
r = json.load(open(sys.argv[1], encoding="utf-8"))
assert r["overall_verdict"] in {"CLEAN_GIVEN_EVIDENCE", "SOFT_FLAGS", "HARD_FLAGS", "REVIEW_UNAVAILABLE"}
assert r["adjudicator"] == "deterministic-rules-v2" and r["human_review_required"] is True
assert r["anchoring_verified"] is True
assert r["limitations"]
print(r["overall_verdict"], "L" + str(r["observability_level"]))
PY
```

## 输出契约

写在 `PAPER_DIR`，绝不改被审文件：`artifact_manifest.json`、`claims.json`、各 `*.findings.json`、`coverage.json`、`adversarial-case-builder.memo.md`（完整模式）、`novelty-duplication-advisory.memo.md`（完整模式）、`report.json`、`REPORT.md`、`.aris/traces/...`。

呈现顺序：verdict + L + taxonomy → 证据表 → 详情 → advisory memo → limitations。AIS 文风印象与 advisory 是**零裁决权重**独立小节，不能用来判定作者身份或学术不端。

## 边界

- detect-only，决策支持；`human_review_required` 永远为 true。
- 不是 AI 文本检测器，不输出“作者概率”，不下学术不端结论。
- 不验证 `run.log` 是否真的由某个 commit 产生；不做 L3 复现。
- 不得用于公开指控、自动拒稿或无人参与的判定。
- 工具依赖：Python 3（stdlib 为主，scipy 可选）、`pdftotext`/`mutool`/PyMuPDF（PDF 输入时）、git；完整模式依赖 codex MCP。无 GPU 需求。
