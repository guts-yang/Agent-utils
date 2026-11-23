# 科研工具箱（Research Toolkit）使用指南

本目录收录从 `knot-agent-design/knot-skills` 打包的 **87 个科研与工程技能**，位于 `skills/research-toolkit/`。每个技能保留原始目录结构（含脚本、参考、模板等资源），以 `SKILL.md` 作为入口。

## 调用方式

技能由智能体按需动态加载：当你的请求匹配某技能的 `description` 触发条件时，智能体自动加载对应 `SKILL.md` 执行。你也可以显式用 `/技能名` 调用，例如 `/arxiv search transformer`。调用后智能体会按技能内定义的 workflow 逐步执行。

> 提示：多数技能以科研全流程为目标，彼此可串联。典型链路：想法发现 → 实验 → 评审 → 写作 → 投稿。

## 技能分类速览

| 分类 | 数量 | 目录 |
|---|---:|---|
| 文献检索与知识管理 | 14 | [`literature-and-knowledge`](skills/research-toolkit/) |
| 研究构思与流程编排 | 11 | [`research-ideation`](skills/research-toolkit/) |
| 实验管理与算力调度 | 15 | [`experiments-and-compute`](skills/research-toolkit/) |
| 论文写作与投稿 | 17 | [`paper-writing`](skills/research-toolkit/) |
| 评审与研究诚信审计 | 8 | [`review-and-integrity`](skills/research-toolkit/) |
| 数学推导与形式化证明 | 5 | [`mathematics-and-proofs`](skills/research-toolkit/) |
| 专利挖掘与撰写 | 10 | [`patent-drafting`](skills/research-toolkit/) |
| 图表与可视化 | 6 | [`visualization`](skills/research-toolkit/) |
| 实用工具 | 1 | [`utilities`](skills/research-toolkit/) |

## 文献检索与知识管理（14）

| 技能 | 调用 | 用途 | 附带资源 |
|---|---|---|---|
| `alphaxiv` | `/alphaxiv` | 通过 AlphaXiv 的 LLM 优化摘要快速理解单篇论文，带分层来源回退。 | — |
| `arxiv` | `/arxiv` | 搜索、下载并总结 arXiv 学术论文。 | — |
| `comm-lit-review` | `/comm-lit-review` | 通信领域文献综述，采用知识库优先的检索方式。 | — |
| `deepxiv` | `/deepxiv` | 通过 DeepXiv 分层阅读开放获取论文，支持章节级阅读与趋势检索。 | — |
| `exa-search` | `/exa-search` | 基于 Exa 的 AI 搜索与内容提取，适合广泛网络检索。 | — |
| `firecrawl-search` | `/firecrawl-search` | 基于 Firecrawl 的网络搜索与页面抓取，返回全文 Markdown。 | — |
| `gemini-search` | `/gemini-search` | 通过 Gemini 检索学术论文，用于超越 arXiv 索引的广泛发现。 | — |
| `keenable-search` | `/keenable-search` | 免密钥的网络搜索与页面抓取，返回 Markdown。 | — |
| `openalex` | `/openalex` | 通过 OpenAlex API 检索论文，获取开放引用数据、机构与资助信息。 | — |
| `research-lit` | `/research-lit` | 检索与分析论文、找相关工作、总结关键思想。 | — |
| `semantic-scholar` | `/semantic-scholar` | 通过 Semantic Scholar API 检索已发表会议期刊论文，补充 arXiv 预印本。 | — |
| `web-debug-search` | `/web-debug-search` | 搜索 GitHub/Stack Exchange/中文社区/官方文档，用于软件错误排查。 | — |
| `research-wiki` | `/research-wiki` | 持久化研究知识库，跨研究生命周期累积论文、想法、实验、声明及其关系。 | — |
| `wiki-enrich` | `/wiki-enrich` | 补全 research-wiki 中各论文页的 TODO 章节。 | — |

## 研究构思与流程编排（11）

| 技能 | 调用 | 用途 | 附带资源 |
|---|---|---|---|
| `idea-creator` | `/idea-creator` | 在大致方向上生成并排序可发表的研究想法。 | — |
| `idea-discovery` | `/idea-discovery` | 从广泛研究方向到经验证想法的完整想法发现流水线。 | — |
| `idea-discovery-robot` | `/idea-discovery-robot` | 面向机器人与具身智能的想法发现流水线，强调仿真优先与 benchmark 支撑。 | — |
| `novelty-check` | `/novelty-check` | 对照近期文献验证研究想法的新颖性。 | — |
| `research-pipeline` | `/research-pipeline` | 端到端研究流水线：从广泛方向经想法发现、实验、评审到成稿 PDF。 | — |
| `research-refine` | `/research-refine` | 把模糊研究方向打磨为以问题为锚、优雅、前沿且面向实现的方法方案。 | — |
| `research-refine-pipeline` | `/research-refine-pipeline` | 串联 research-refine 与 experiment-plan，一次性产出方法方案与实验计划。 | — |
| `research-review` | `/research-review` | 从选定审稿人（Codex/Gemini/Grok/Claude Code/人工）获得深度批判性评审。 | — |
| `research-implement-feature` | `/research-implement-feature` | 从『帮我实现 X』构建可运行端到端骨架，再逐级添加功能，先写假设账本。 | — |
| `meta-optimize` | `/meta-optimize` | 分析 ARIS 使用日志，提出对 SKILL.md、评审提示词与工作流默认值的优化建议。 | — |
| `meta-apply` | `/meta-apply` | 落地用户批准的 meta-optimize / corpus-audit 补丁，唯一可修改技能语料库的技能。 | — |

## 实验管理与算力调度（15）

| 技能 | 调用 | 用途 | 附带资源 |
|---|---|---|---|
| `ablation-planner` | `/ablation-planner` | 主结果通过 result-to-claim 后，系统性设计消融实验，回答审稿人可能提出的问题。 | — |
| `analyze-results` | `/analyze-results` | 分析机器学习实验结果、计算统计量、生成对比表和洞察。 | — |
| `dse-loop` | `/dse-loop` | 面向体系结构与 EDA 的自主设计空间探索，运行程序、分析结果、调参迭代。 | — |
| `experiment-audit` | `/experiment-audit` | 在宣称结果前审计实验诚信，检测伪造真值、分数归一化造假等问题。 | — |
| `experiment-bridge` | `/experiment-bridge` | 从想法发现到自动评审的桥梁：读取实验计划、实现代码、部署到 GPU 并收集结果。 | — |
| `experiment-plan` | `/experiment-plan` | 把细化后的研究方案转化为以 claim 为驱动的详细实验路线图。 | — |
| `experiment-queue` | `/experiment-queue` | SSH 多种子/多配置实验队列，支持 OOM 重试与陈旧屏幕清理。 | scripts |
| `monitor-experiment` | `/monitor-experiment` | 监控运行中的实验、检查进度并收集结果。 | — |
| `qzcli` | `/qzcli` | 用 qzcli 管理启智平台的 GPU 计算作业，语法类似 kubectl。 | — |
| `result-to-claim` | `/result-to-claim` | 实验完成后判断结果支持哪些声明、不支持哪些、还缺什么证据。 | — |
| `run-experiment` | `/run-experiment` | 在本地/远程/Vast.ai/Modal 上部署并运行机器学习实验。 | — |
| `serverless-modal` | `/serverless-modal` | 在 Modal 上运行 GPU 工作负载，零配置、免 SSH。 | — |
| `system-profile` | `/system-profile` | 对脚本、进程、GPU、内存、互连进行性能剖析。 | — |
| `training-check` | `/training-check` | 训练期间周期性检查 WandB 指标，及早发现 NaN/loss 发散/空转 GPU。 | — |
| `vast-gpu` | `/vast-gpu` | 在 vast.ai 上租用、管理与销毁 GPU 实例。 | — |

## 论文写作与投稿（17）

| 技能 | 调用 | 用途 | 附带资源 |
|---|---|---|---|
| `auto-paper-improvement-loop` | `/auto-paper-improvement-loop` | 用 GPT-6-Astra xhigh 评审→修改→重编译的循环自动改进论文，默认两轮。 | — |
| `grant-proposal` | `/grant-proposal` | 从研究想法与文献起草结构化基金申请书，支持 NSFC/NSF/ERC 等多种格式。 | — |
| `interview-cheatsheet` | `/interview-cheatsheet` | 生成某 ML/LLM 主题的长篇中文面试速查表，含公式推导、手写代码与 25 道高频题。 | — |
| `overleaf-sync` | `/overleaf-sync` | 本地论文目录与 Overleaf 项目双向同步。 | — |
| `paper-compile` | `/paper-compile` | 把 LaTeX 论文编译成 PDF、修复错误并验证输出。 | — |
| `paper-plan` | `/paper-plan` | 从评审结论和实验结果生成结构化的论文大纲。 | — |
| `paper-write` | `/paper-write` | 依据大纲逐节起草 LaTeX 论文。 | templates |
| `paper-writing` | `/paper-writing` | 从叙述性报告到可提交 PDF 的完整论文写作流水线。 | — |
| `paper-poster` | `/paper-poster` | 已废弃，重定向到 paper-poster-html。 | — |
| `paper-poster-html` | `/paper-poster-html` | 把学术论文做成单文件 HTML/CSS 会议海报，含真实图与印刷级 PDF。 | scripts, templates |
| `paper-slides` | `/paper-slides` | 从编译好的论文生成会议演讲幻灯片（beamer→PDF + 可编辑 PPTX）与讲稿。 | — |
| `paper-talk` | `/paper-talk` | 端到端会议演讲流水线：论文→大纲→幻灯片→逐页打磨→导出。 | — |
| `rebuttal` | `/rebuttal` | Submission 回复流水线：解析外部评审、生成安全的纯文本回复并管理后续轮次。 | — |
| `render-html` | `/render-html` | 把 ARIS 的 Markdown/JSON 产物渲染成单文件 HTML 便于阅读。 | scripts |
| `resubmit-pipeline` | `/resubmit-pipeline` | 在硬约束下把润色好的论文纯文本重投到不同 venue。 | — |
| `slides-polish` | `/slides-polish` | 对幻灯片逐页 Codex 评审并做针对性排版修复。 | — |
| `writing-systems-papers` | `/writing-systems-papers` | 面向 OSDI/SOSP/ASPLOS/NSDI/EuroSys 的系统论文段落级结构蓝图。 | — |

## 评审与研究诚信审计（8）

| 技能 | 调用 | 用途 | 附带资源 |
|---|---|---|---|
| `auto-review-loop` | `/auto-review-loop` | 自主多轮研究评审循环，可对接 Codex / Gemini / 各类审稿人后端。 | — |
| `auto-review-loop-llm` | `/auto-review-loop-llm` | 用任意 OpenAI 兼容 API 驱动自主研究评审循环。 | — |
| `auto-review-loop-minimax` | `/auto-review-loop-minimax` | 用 MiniMax API 驱动的自主多轮研究评审循环。 | — |
| `citation-audit` | `/citation-audit` | 零上下文核验论文中每条参考文献真实存在、归属正确且被恰当引用。 | — |
| `citation-crosscheck` | `/citation-crosscheck` | 双盲交叉核对本地 .bib 与网络记录的字段一致性，输出逐字段差异。 | scripts |
| `integrity-forensics` | `/integrity-forensics` | 投稿前的研究诚信取证扫描，生成政策门控与义务账本。 | — |
| `kill-argument` | `/kill-argument` | 双线程对抗式评审：先构造最强拒稿备忘录，再逐点辩护并暴露未解决问题。 | — |
| `paper-claim-audit` | `/paper-claim-audit` | 零上下文核验论文中每个数字、对比与范围声明是否与原始结果文件一致。 | — |

## 数学推导与形式化证明（5）

| 技能 | 调用 | 用途 | 附带资源 |
|---|---|---|---|
| `formula-derivation` | `/formula-derivation` | 构建与推导研究公式，组织假设、梳理零散方程为连贯推导。 | — |
| `lean-formalize` | `/lean-formalize` | 在 Lean 中开发并验证数学证明，或审计已有证明是否证明原始命题。 | references |
| `proof-checker` | `/proof-checker` | 严格的数学证明验证与修复：识别缺口、补全推导、复审并生成审计报告。 | — |
| `proof-orchestrator` | `/proof-orchestrator` | 管理有状态、基于 run 目录的证明项目，支持跨轮续接与 GPT Pro 交接。 | references |
| `proof-writer` | `/proof-writer` | 为 ML/AI 理论撰写严格数学证明，补全缺失步骤或判断命题可证性。 | — |

## 专利挖掘与撰写（10）

| 技能 | 调用 | 用途 | 附带资源 |
|---|---|---|---|
| `claims-drafting` | `/claims-drafting` | 为发明撰写专利权利要求书，是专利流水线核心技能。 | — |
| `embodiment-description` | `/embodiment-description` | 为专利说明书撰写详细的实施例描述。 | — |
| `figure-description` | `/figure-description` | 处理用户提供的专利附图，生成正式的附图说明与附图标记。 | — |
| `invention-structuring` | `/invention-structuring` | 把粗糙想法结构化为正式的发明专利披露书。 | — |
| `jurisdiction-format` | `/jurisdiction-format` | 把专利申请编译为 CN/US/EP 等司法管辖区特定的提交格式。 | — |
| `patent-novelty-check` | `/patent-novelty-check` | 对照现有技术评估专利新颖性与创造性。 | — |
| `patent-pipeline` | `/patent-pipeline` | 从发明描述到司法管辖区格式提交文件的完整专利起草流水线。 | — |
| `patent-review` | `/patent-review` | 获取外部专利审查员对申请文件的评审与意见。 | — |
| `prior-art-search` | `/prior-art-search` | 检索与发明相关的专利数据库与学术文献现有技术。 | — |
| `specification-writing` | `/specification-writing` | 从权利要求与发明披露书撰写完整专利说明书。 | — |

## 图表与可视化（6）

| 技能 | 调用 | 用途 | 附带资源 |
|---|---|---|---|
| `figure-spec` | `/figure-spec` | 从结构化 JSON 生成确定性、可出版、可编辑的 SVG 架构图/流程图。 | scripts |
| `mermaid-diagram` | `/mermaid-diagram` | 根据需求生成 Mermaid 图，支持流程图、时序图、类图、ER 图等 18+ 类型。 | — |
| `paper-figure` | `/paper-figure` | 从实验结果生成可出版质量的图表。 | — |
| `paper-illustration` | `/paper-illustration` | 用 Gemini 图像生成论文插图，经迭代打磨。 | — |
| `paper-illustration-image2` | `/paper-illustration-image2` | 通过本地 Codex app-server 桥接生成可出版学术论文插图。 | scripts |
| `pixel-art` | `/pixel-art` | 生成用于 README、文档或幻灯片的像素风 SVG 插图。 | — |

## 实用工具（1）

| 技能 | 调用 | 用途 | 附带资源 |
|---|---|---|---|
| `feishu-notify` | `/feishu-notify` | 向飞书/Lark 发送通知，常作为其他技能的内部工具。 | — |

## 目录结构说明

```text
skills/research-toolkit/<skill-name>/
├── SKILL.md              # 技能入口：frontmatter + workflow
├── scripts/              # 可执行脚本（部分技能）
├── references/           # 参考文档（部分技能）
├── templates/            # 模板文件（部分技能）
└── assets/               # 资源文件（部分技能）
```

完整机器可读清单见 `skills-manifest.json`，其中每个技能条目的 `category` 为 `research-toolkit`，`source` 为 `knot-agent-design/knot-skills`（原始来源路径）。
