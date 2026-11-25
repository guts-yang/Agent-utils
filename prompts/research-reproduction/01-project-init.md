# 01 · 项目初始化 Prompt

- 执行顺序：1 / 2（先做）—— 完成 6 步初始化即停，交给 02 环节。
- 用途：从空白工作区准备 env、ARIS skills、仓库与分支、数据盘、CUDA/依赖、基线口径；不执行 ideas 探索。
- 状态：v2（2026-11-25）：移除原第 7–8 步 idea 闭环与最终输出，交由 `02-research-loop-dual-track.md`。
- 使用：复制下方代码块到空白工作区的新会话；就绪后进入 02 环节。

```text
你是本项目的初始化助手。从空白工作区 /usr/local/workspace 开始，
只完成下面 6 步；初始化完成即停，不执行 ideas 探索、不跑复现、不输出实验结果。

1. 环境配置：初始化并校验 /usr/local/workspace/.env（GitHub/HF/Hy4 等凭据）与 git 身份；
   密钥只从环境变量读取，不打印、不写进任何文件。

2. Skills（必需）：clone guts-yang/skills-repos.git，把 skills/Auto-Research-In-Sleep/
   完整安装到 ~/.codebuddy/skills/；补齐 ARIS 运行时依赖（上游 tools/、shared-references/、
   reviewer 后端，快照里缺，不补会让 /idea-discovery 的证据门 BLOCKED）；
   用 /skills 确认 /idea-discovery、/auto-review-loop 可用。

3. 代码与分支：clone 并切到对应分支 —— ASU-unlearning(Auto/20261122)、
   L7_ASU-Novel-Methods(new_method)、R-TOFU(Auto)、awesome-lrm-unlearning(main)、
   redact(fuxian)；盘点各子仓库待复现的训练 idea 与原基线配置（只盘点，不生成新 idea）。

4. 数据盘：确认 /jizhicfs/chenyliao/ 已挂载，盘点数据集与模型权重/参数；
   所有大文件（数据、checkpoint、结果、缓存）只落该目录，工作区用软链，
   禁止把大文件复制进工作区。

5. 计算与依赖：确认 CUDA 可用、torch 为 CUDA 版并记录 GPU；逐个核对各仓库
   requirements.txt / environment.yml / pyproject.toml，缺失依赖在隔离环境补齐。

6. 基线对齐（硬性约束）：锁定「xxx 原基线」，把以下作为 02 环节的硬性约束（本环节不跑实验）：
   所有创新方法的最终复现结果必须与其实验配置基本一致 —— 同一模型、同一数据集划分比例；
   所有评估指标严格按原基线的定义与呈现方式给出，并统一按基线配置复现，不得改口径。

约束：不上传数据、不泄露密钥、不擅自改基线定义；6 步完成后给出简短就绪报告即停，
把 idea 探索、复现与最终输出交给 02 环节。
```

备注：`xxx` 为占位符，用前替换为实际基线名/配置来源；`.env` 中的密钥只引用变量名，禁止写入明文。
