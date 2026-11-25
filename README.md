# Agent Utils

个人 agent skills 与科研任务提示词仓库。每个技能是一个独立目录，入口是 `SKILL.md`；脚本、参考资料、模板和元数据留在原包里，不合并成一个大文件。

仓库共 **122 个顶层技能包**。`skills/research/workflows/anti-autoresearch/bundle/` 里另有取证子技能，它们属于该包，不重复计入 122。

## 布局

```
skills/                          顶层技能包
  <category>/<name>/             35 个分类技能（个人技能、仓库原有技能、复现编排与 Anti-Autoresearch）
  Auto-Research-In-Sleep/<name>/ 87 个 Knot Agent 科研技能，平铺，没有二级分类目录
prompts/
  research-reproduction/         本机复现项目的起点 prompt 与四段兼容提示词
  aris-reference/                Auto-Research-In-Sleep 的用法清单与系统提示词
skills-manifest.json             机器可读目录
```

`skills-manifest.json` 的每条记录包含 `name`、`directory`、`source_directory`、`category`、`source`。来源目录名和技能 `name` 不一致时（例如 `self-improving-agent__skillhub` → `self-improvement`），两边都保留。

## 怎么用

把需要的技能目录交给支持 `SKILL.md` 的 agent 加载。显式调用用 `/技能名`；自然语言命中技能 `description` 时，先读完该包的 `SKILL.md`，再按其中的 workflow 使用 `scripts/`、`references/` 等资源。

内置、marketplace、插件和 Connector 自带的技能不在这里重复存放，由各自上游维护。

## 分类技能（35）

| 分类 | 技能 |
|---|---|
| `agent-meta` | `self-improvement` |
| `ai-information` | `aihot` |
| `career-and-jobs` | `job-search-accelerator` |
| `developer-tools` | `github-python-repo`、`supervisor-skills-repo-port` |
| `documents-and-conversion` | `markdown-to-pdf-offline` |
| `visualization` | `drawio-reconstruction`、`figure-designer` |
| `research/literature-and-mapping` | `deep-research`、`paper-quick-reader`、`research-knowledge-map`、`research-literature-scout`、`research-paper-reading`、`verified-research-report` |
| `research/ideas-and-methods` | `idea-evaluator`、`research-challenge-definition`、`research-innovation-validation` |
| `research/benchmarks-and-reproduction` | `benchmark-baseline-table`、`benchmark-paper-inventory`、`research-baseline-comparison`、`research-baseline-reproduction`、`sota-table-caliber-migration` |
| `research/writing-and-publication` | `benchmark-paper-template`、`intro-drafter`、`paper-math-note`、`paper-polish`、`paper-venue-ccf-verify`、`paper-writer`、`pre-submission-reviewer`、`tech-paper-template` |
| `research/workflows` | `research-seven-round-orchestrator`、`research-weekly-review`、`vibe-research-workflow`、`unlearning-reproduction`、`anti-autoresearch` |

`research-seven-round-orchestrator` 按阶段串起阅读、文献、知识地图、挑战定义、基线比较、复现和创新验证，并在门未通过时回退。`deep-research`、`verified-research-report`、`idea-evaluator` 是对应阶段的加深工具，不替代阶段门。`research-weekly-review` 做周度或里程碑复盘。

`anti-autoresearch` 是只读取证：把 autoresearch 的 `results.tsv` / `run.log` 整理成可锚定目录，再做证据账本和确定性裁决。入口是 [`skills/research/workflows/anti-autoresearch/SKILL.md`](skills/research/workflows/anti-autoresearch/SKILL.md)。执行前要设置 `AAR_ROOT` 指向该包，避免 bundle 内脚本被外层 git 根目录带走。

`unlearning-reproduction` 是机器遗忘复现编排 skill：按线路隔离初始化、idea discovery、experiment bridge、review/audit/claim，并通过 `INIT_REPORT.md`、`HANDOFF.md` 和 `.aris/runs/` 支持阶段 gate 与恢复。入口是 [`skills/research/workflows/unlearning-reproduction/SKILL.md`](skills/research/workflows/unlearning-reproduction/SKILL.md)，默认调用 `/unlearning-reproduction`。它只编排 ARIS，不复制各阶段 skill 的研究逻辑。

来源：32 个复制自 `~/.workbuddy/skills/`（截至 2026-10-29），`github-python-repo` 是本仓库原有技能，`unlearning-reproduction` 是本仓库维护的编排技能，`anti-autoresearch` 打包自 `guts-yang/Anti-Autoresearch`（截至 2026-11-23）。

## Auto-Research-In-Sleep（87）

来自 `knot-agent-design/knot-skills`（截至 2026-11-23），全部位于 `skills/Auto-Research-In-Sleep/`。下面的分类是用途分组，不是子目录。

| 分组 | 数量 | 内容 |
|---|---:|---|
| 文献检索与知识管理 | 14 | arXiv、AlphaXiv、OpenAlex、Semantic Scholar、DeepXiv 等检索，以及研究知识库 |
| 研究构思与流程编排 | 11 | idea 生成与查新、方法细化、端到端流水线、技能语料优化 |
| 实验管理与算力调度 | 15 | 实验设计、分析、监控、消融、设计空间探索，以及启智 / vast.ai / Modal |
| 论文写作与投稿 | 17 | 大纲、LaTeX、海报、幻灯片、讲稿、rebuttal、重投 |
| 评审与研究诚信审计 | 8 | 多轮评审、引用核验、论文数字核对、投稿前取证 |
| 数学推导与形式化证明 | 5 | 公式推导、Lean、证明撰写与核验 |
| 专利挖掘与撰写 | 10 | 现有技术、权利要求、说明书、实施例、辖区格式 |
| 图表与可视化 | 6 | SVG、Mermaid、论文插图、像素图 |
| 实用工具 | 1 | 飞书通知 |

常见串联：

- 找方向：`/idea-discovery` → `/novelty-check` → `/research-refine`
- 做实验：`/experiment-plan` → `/run-experiment` 或 `/experiment-queue` → `/analyze-results`
- 写论文：`/paper-plan` → `/paper-write` → `/paper-compile` → `/paper-claim-audit`
- 投稿：`/auto-review-loop` → `/rebuttal` → `/paper-talk` 或 `/paper-poster-html`

逐项调用说明在 [`prompts/aris-reference/AUTO_RESEARCH_IN_SLEEP_USAGE.md`](prompts/aris-reference/AUTO_RESEARCH_IN_SLEEP_USAGE.md)。可直接贴进智能体的系统提示词在 [`prompts/aris-reference/AGENT_SYSTEM_PROMPT.md`](prompts/aris-reference/AGENT_SYSTEM_PROMPT.md)。

部分流水线技能会引用上游的 `shared-references/`、`tools/` 和 reviewer 后端，这些不在本快照里。只拷贝 `skills/Auto-Research-In-Sleep/` 时，`/idea-discovery` 一类技能的证据门会停在 BLOCKED。补齐方式写在 [`prompts/research-reproduction/01-project-init.md`](prompts/research-reproduction/01-project-init.md)。

## 任务提示词

[`prompts/research-reproduction/`](prompts/research-reproduction/) 现在包含一个起点 prompt 和四段兼容提示词，按 ARIS 阶段分开。新运行优先使用 [`unlearning-reproduction`](skills/research/workflows/unlearning-reproduction/SKILL.md)；这些 prompt 保留给初始化、单阶段手动执行和迁移参考。里面写了工作区路径、数据盘和仓库分支，换项目时要改这些占位内容。两条线路各自使用 `tracks/<track>/`，不混用基线、数据和指标口径。

0. [`00-bootstrap-unlearning-reproduction.md`](prompts/research-reproduction/00-bootstrap-unlearning-reproduction.md)：先 clone 远程 Agent-utils，安装编排 Skill 和 ARIS 子 Skill，检查依赖，执行 dry-run 后再开始正式流程。
1. [`01-project-init.md`](prompts/research-reproduction/01-project-init.md)：环境、ARIS skills、仓库分支、数据盘、依赖和基线口径。6 步做完即停，并按线路写 `INIT_REPORT.md`。
2. [`02-idea-discovery.md`](prompts/research-reproduction/02-idea-discovery.md)：只调用 `/idea-discovery`，选定一个 idea 并写出实验计划。跳过 GPU pilot。
3. [`03-experiment-bridge.md`](prompts/research-reproduction/03-experiment-bridge.md)：在锁定基线上实现并跑实验，停在结果和 tracker。
4. [`04-review-and-claim.md`](prompts/research-reproduction/04-review-and-claim.md)：实验结果出现后调用一次 `/auto-review-loop`，再做诚信核对与 `/result-to-claim`。claim 不成立时把下一个候选交回 03。

## 维护

1. 新的个人技能放到最匹配的分类目录下，整包复制，保留 `SKILL.md` 和附属资源。
2. 在 `skills-manifest.json` 补上 `name`、`directory`、`source_directory`、`category`、`source`，并核对 `personal_skill_count` 与 `repository_skill_count`。
3. 改了从 `~/.workbuddy/skills/` 复制来的包之后，同步回原目录，或明确把本仓库当作该技能的 source of truth。
4. 推送前确认没有密钥、`.env`、个人数据或不该公开的资料。
