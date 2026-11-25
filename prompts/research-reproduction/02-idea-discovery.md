# 02 · 双线路 Idea Discovery Prompt

- 执行顺序：2 / 4 —— 承接 01 的 `INIT_REPORT.md`，只跑 `/idea-discovery`。
- 用途：为每条线路选定一个 idea，并产出实验计划；不部署、不评审。
- 状态：v4（2026-11-25）：从「想 idea → 交叉验证 → 复现 → 评测」拆出。`/auto-review-loop` 移到 04，放在实验结果之后。
- 使用：先完成 `01-project-init.md`；复制下方代码块到新会话。计划写完后进入 `03-experiment-bridge.md`。

```text
你是本项目的双线路 idea discovery 助手。前置假设：01 已完成，每条线路的
INIT_REPORT.md 已写好。本环节只调用 /idea-discovery。
不调用 /experiment-bridge、/auto-review-loop、/result-to-claim 或 /experiment-audit，
不部署实验，不跑训练。

两条线路必须完全分开。进入一条线路的目录后，只读该目录的 INIT_REPORT.md，
不重新盘点仓库，不读取另一条线路的文件：
- 线路 A · LLM Unlearning：/usr/local/workspace/tracks/llm-unlearning/
  仓库：ASU-unlearning、L7_ASU-Novel-Methods
- 线路 B · LRM Unlearning：/usr/local/workspace/tracks/lrm-unlearning/
  仓库：R-TOFU、redact；awesome-lrm-unlearning 只作综述与基线参考

每条线路在自己的工作目录里调用一次：
/idea-discovery "<本线路方向>" — AUTO_PROCEED: true, compact: true, render html: false

调用参数同时写明本线路约束：只用 INIT_REPORT.md 锁定的仓库与「xxx 原基线」；
同一模型、同一数据集划分、同一指标定义；不得改口径。
/idea-discovery 内部顺序保持为
/research-lit → /idea-creator → /novelty-check → /research-review → /research-refine-pipeline，
不要拆开或重排。AUTO_PROCEED 只选定一个 idea 做深，不为每个候选做完整复现。

跳过 GPU pilot。技能若进入 pilot，把该条标成 needs manual pilot，继续产出计划，
不启动训练。正式复现留给 03。

本线路必须同时具备这些产物，且证据门不是 BLOCKED，才能写交接：
- idea-stage/IDEA_REPORT.md
- idea-stage/IDEA_CANDIDATES.md
- idea-stage/docs/research_contract.md
- refine-logs/FINAL_PROPOSAL.md
- refine-logs/EXPERIMENT_PLAN.md
证据门为 BLOCKED 时停在本阶段，HANDOFF.md 写明 blocked，不交给 03。

证据门通过后，用 .env 中的 Hy4_TOKEN 对选定 idea 做一次独立查新。
密钥只从环境变量读取，不打印、不落盘。这次查新不替代 /novelty-check，
也不替代 04 的 /auto-review-loop。查新发现该 idea 已被覆盖时，在候选里改选
下一个未被覆盖的 idea，并让计划产物改指它；没有可选项则停，不进入 03。

然后写本线路 HANDOFF.md：选定 idea、上面五个产物的路径、锁定基线配置路径、
Hy4 查新结论、是否允许进入 03。写完即停。
大文件仍只放 /jizhicfs/chenyliao/<track>/，工作区用软链。

约束：不上传数据、不泄露密钥、不擅自改基线定义；不跨线路复用 idea、模型、
数据划分或指标口径。
```

备注：`xxx` 为占位符，换成该线路 INIT_REPORT.md 里锁定的基线名；`Hy4_TOKEN` 只引用变量名，禁止写入明文。
