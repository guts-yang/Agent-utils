# 01 · 项目初始化 Prompt

- 执行顺序：1 / 4（先做）—— 完成 6 步初始化即停，交给 `02-idea-discovery.md`。
- 用途：从空白工作区准备 env、ARIS skills、仓库与分支（含逐仓 git 身份/凭据）、数据盘、CUDA/依赖、基线口径；不执行 ideas 探索。
- 状态：v4（2026-11-25）：6 步范围不变；第 3 步在 clone 后按 .env 为每个仓库配置本地 git 身份与免交互凭据，供手动提交推送；就绪报告按线路写入 `INIT_REPORT.md`，供 02–04 只读。
- 使用：复制下方代码块到空白工作区的新会话；两份报告写完后进入 02 环节。

```text
你是本项目的初始化助手。从空白工作区 /usr/local/workspace 开始，
只完成下面 6 步；初始化完成即停，不执行 ideas 探索、不跑复现、不输出实验结果。

两条线路从这一步起分开，后续不得交叉混用 idea、基线、模型、数据划分或指标口径：
- 线路 A · LLM Unlearning：/usr/local/workspace/tracks/llm-unlearning/
  仓库：ASU-unlearning(Auto/20261122)、L7_ASU-Novel-Methods(new_method)
  大文件目录：/jizhicfs/chenyliao/llm-unlearning/
- 线路 B · LRM Unlearning：/usr/local/workspace/tracks/lrm-unlearning/
  仓库：R-TOFU(Auto)、redact(fuxian)；awesome-lrm-unlearning(main) 只作综述与基线参考
  大文件目录：/jizhicfs/chenyliao/lrm-unlearning/

1. 环境配置：初始化并校验 /usr/local/workspace/.env 中的 GITHUB_NAME、GITHUB_EMAIL、
   GITHUB_TOKEN（以及 HF/Hy4 等凭据）均存在且非空；密钥只从环境变量读取，
   只引用变量名，不打印、不写进任何文件。三项 GitHub 配置缺失即报阻塞，不进入 clone。

2. Skills（必需）：clone guts-yang/Agent-utils.git，把 skills/Auto-Research-In-Sleep/
   完整安装到 ~/.codebuddy/skills/；补齐 ARIS 运行时依赖（上游 tools/、shared-references/、
   reviewer 后端，快照里缺，不补会让 /idea-discovery 的证据门 BLOCKED）；
   用 /skills 确认 /idea-discovery、/experiment-bridge、/auto-review-loop、
   /result-to-claim、/experiment-audit 可用。

3. 代码与分支：clone 并切到上面列出的分支；盘点各子仓库待复现的训练 idea 与原基线配置
   （只盘点，不生成新 idea）。仓库放在对应线路目录下。
   共享 credential helper 沿用 00 创建的 /usr/local/workspace/.git-credential-github.sh；
   若不存在，按下方 project-config.md 的模板先在仓库外创建（权限 700、不含明文 token）。
   clone 时用 git -c credential.helper=<helper> 临时注入凭据，避免交互式输入用户名。
   clone 完成后立即给本步骤涉及的所有仓库（Agent-utils 快照、两条线路的全部子仓库，
   含只读参考仓库）配置本地提交身份与免交互推送凭据，使用户可直接手动 git commit / git push：
   - 从 .env 读 GITHUB_NAME / GITHUB_EMAIL，逐仓写 --local 的 user.name / user.email，
     不改全局 git 配置；
   - 逐仓把共享 helper 设为本地 credential.helper，token 只在 git 需要凭据时从 .env 读取；
     脚本、.git/config、remote URL、报告与日志都不得含明文 token，不写 ~/.git-credentials；
   - 完整脚本模板、逐仓配置与只读验证命令见
     skills/research/workflows/unlearning-reproduction/references/project-config.md
     的「Git identity for manual commits」一节；
   - 逐仓核对三项 --local 配置已生效（不回显值），并用只读的
     GIT_TERMINAL_PROMPT=0 git ls-remote 确认免交互凭据可用
     （dry-run 只查配置，不 clone、不联网）；
   - 任一仓库配置或验证失败即报阻塞，不进入 02。

4. 数据盘：确认 /jizhicfs/chenyliao/ 已挂载，按线路盘点数据集与模型权重/参数；
   所有大文件（数据、checkpoint、结果、缓存）只落该线路的大文件目录，工作区用软链，
   禁止把大文件复制进工作区。

5. 计算与依赖：确认 CUDA 可用、torch 为 CUDA 版并记录 GPU；逐个核对各仓库
   requirements.txt / environment.yml / pyproject.toml，缺失依赖在隔离环境补齐。

6. 基线对齐（硬性约束）：每条线路各自锁定「xxx 原基线」（本环节不跑实验）。
   后续复现必须与该锁定配置一致：同一模型、同一数据集划分比例；
   评估指标严格按原基线的定义与呈现方式给出，不得改口径。
   把锁定配置的路径写进报告，不把基线定义抄成另一套口径。

6 步完成后，为每条线路写 tracks/<track>/INIT_REPORT.md，然后停。报告只含：
仓库路径与分支、锁定基线的配置路径、数据与权重位置、GPU、依赖状态、
本线路大文件目录、各仓库 git 身份与凭据配置状态（只记 configured / blocked，不记值）。
idea 探索、复现、评审与声明交给 02、03、04。

约束：不上传数据、不泄露密钥、不擅自改基线定义；不把 token 写入 .env 以外的任何文件，
不改全局 git 配置，初始化阶段不代用户提交或推送。
```

备注：`xxx` 为占位符，两条线路各自替换为实际基线名/配置来源；`.env` 中的密钥只引用变量名，禁止写入明文。
