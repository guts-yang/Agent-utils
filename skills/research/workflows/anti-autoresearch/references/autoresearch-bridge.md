# autoresearch-loop → Anti-Autoresearch 桥接

Anti-Autoresearch 不认识 `results.tsv`、`run.log`、`AUTORESEARCH_SUMMARY.md`。要让取证流水线能锚定并检查 autoresearch 的产物，必须先把它们整理成一个独立的 `PAPER_DIR`。

## 工件映射

| autoresearch-loop 产物 | Anti 能否直接读 | 桥接动作 |
|---|---|---|
| `results.tsv` | 不能 | 转成 `PAPER_DIR/results/experiments.csv`（`.csv` 在 `results/` 下才会被 manifest 计为结果文件） |
| `run.log` | 不能当论文 | 复制到 `PAPER_DIR/logs/run.log` 作旁证；**日志本身不会被语义化为论文主张** |
| `AUTORESEARCH_SUMMARY.md` | 不能 | 改写成 `submission.tex` 中的可锚定句子（摘要/方法/表格/结论） |
| `train.py` / `prepare.py` / `program.md` | 间接 | 放进 `PAPER_DIR/code/`，触发 repo 检测；`program.md` 本身不被解析 |
| 分支 `autoresearch/<tag>` / commit 链 | 部分 | 作为人工证据写进 `submission.tex` 的表格描述列；Anti 不读 git、不做 diff 或复现 |

根本限制：Anti 没有 L3 复现能力，无法验证 `run.log` 是否真的由某个 commit 产生。不要声称“已证实代码造假”。

## 推荐目录

```
PAPER_DIR/                       # 绝对路径，无空格，位于 Anti bundle 之外
├── submission.tex               # L1 可锚定源；至少含摘要、方法、结果表、比较、限制
├── refs.bib                     # 需要引用取证时提供
├── results/
│   └── experiments.csv          # 从 results.tsv 转换
├── logs/
│   └── run.log                  # 最终一轮日志旁证
└── code/
    ├── train.py
    ├── prepare.py
    └── program.md
```

有 `.tex` + `code/` + `results/*.csv` 时 manifest 派生 **L2**。只有 `submission.txt`/PDF 文本时是 **L0**，代码级 finding 会被降级。

## 转换 results.tsv

在 autoresearch 项目根目录执行（`results.tsv` 是 tab 分隔、5 列）：

```bash
PAPER_DIR="<absolute paper dir, no spaces>"
mkdir -p "$PAPER_DIR/results" "$PAPER_DIR/logs" "$PAPER_DIR/code"

python3 - "$PAPER_DIR/results/experiments.csv" <<'PY'
import csv, sys
src = "results.tsv"
out = sys.argv[1]
with open(src, encoding="utf-8") as f, open(out, "w", newline="", encoding="utf-8") as g:
    rows = [line.rstrip("\n").split("\t") for line in f if line.strip()]
    header = ["commit", "val_bpb", "memory_gb", "status", "description"]
    if rows and rows[0] == header:
        rows = rows[1:]
    w = csv.writer(g)
    w.writerow(header)
    w.writerows(r + [""] * (5 - len(r)) for r in rows)
print(out)
PY

cp -a train.py prepare.py program.md "$PAPER_DIR/code/"
cp -a run.log "$PAPER_DIR/logs/run.log"
```

若 `results.tsv` 尚未生成或只有 baseline，明确告知：L2 结果级检查不可用，不要伪造 CSV 行。

## submission.tex 最小骨架

骨架必须包含**逐字数字**，因为 ledger 的 finding 必须引用 `claims.json` 中逐字存在的 span。把 `BEST_VAL`、commit 等替换为实际值。

```latex
\documentclass{article}
\begin{document}

\section{Abstract}
We report an autoresearch run on a single NVIDIA GPU. The best validation bits per byte is BEST\_VAL.
The run used a fixed five-minute training budget and kept only changes that lowered val\_bpb.

\section{Method}
The agent may edit only \texttt{train.py}; \texttt{prepare.py} and the \texttt{evaluate\_bpb}
metric are fixed. The run tag is TAG on branch \texttt{autoresearch/TAG}.

\section{Results}
Table~\ref{tab:main} reports all retained experiments. The baseline val\_bpb is BASE\_VAL
and the best val\_bpb is BEST\_VAL.

\begin{table}[h]
\caption{Autoresearch experiment log}
\label{tab:main}
\begin{tabular}{lrrr}
commit & val\_bpb & memory\_gb & status \\
BASE\_COMMIT & BASE\_VAL & BASE\_MEM & keep \\
BEST\_COMMIT & BEST\_VAL & BEST\_MEM & keep \\
\end{tabular}
\end{table}

\section{Limitations}
Runs are comparable only within the same GPU platform. No independent reproduction was performed.

\end{document}
```

要点：

- 数值、baseline、比较对象、`seed`/重复次数、固定预算都必须写成句子或表格中的逐字 span。
- 若声称“优于 X”，必须同时给出 X 的数字，否则 `baseline-comparison-audit` 无法锚定。
- 不要用 `AUTORESEARCH_SUMMARY.md` 直接改名成 `.tex`；它缺少 LaTeX 结构和 claim 边界。

## 运行与回写

```bash
AAR_SKILL="${AAR_SKILL:-/usr/local/workspace/skills-repos/skills/research/workflows/anti-autoresearch}"
export AAR_ROOT="$AAR_SKILL/bundle"; cd "$AAR_ROOT" || exit 1
PAPER_DIR="<absolute paper dir, no spaces>"
# 按 SKILL.md 的“完整模式”或“确定性-only 回退”执行，产物写入 PAPER_DIR
```

审计后若需回写 autoresearch 项目：

1. 从 `PAPER_DIR/REPORT.md` 的证据表提取 `pattern_id`、逐字 span、建议核实点。
2. 写入 autoresearch 项目根的 `AUDIT_FEEDBACK.md`，标注“待核实，不是已证实造假”。
3. 仅把其中可操作为 `train.py` 改动的项目交给 `autoresearch-loop` 作为下一轮实验线索，例如表文数字不一致 → 检查日志抽取与表格生成链路。
4. 在 `AUTORESEARCH_SUMMARY.md` 追加：`audit_verdict`、`L`、`PAPER_DIR` 绝对路径、审计时间。
5. 若 verdict 为 `REVIEW_UNAVAILABLE`，不要推进到投稿；先补 codex reviewer 或做人工同行评审。
