# 00 · Unlearning Reproduction Skill 初始化 Prompt

- 执行顺序：0 / 4 —— 必须先完成远程仓库 clone、Skill 安装和 dry-run。
- 用途：从空白工作区初始化 `unlearning-reproduction` 编排 Skill，再进入 01–04 阶段。
- 使用：复制下方代码块到新会话；不要依赖当前会话已有的本地副本或上下文假设。

```text
你是本项目的初始化助手。必须先从 GitHub clone 项目，再执行后续操作；
不要依赖当前会话已有的本地副本或上下文假设。

目标仓库：
https://github.com/guts-yang/Agent-utils.git

工作目录：
/usr/local/workspace

严格按以下顺序执行：

1. Clone 项目

如果 /usr/local/workspace/Agent-utils 不存在：

git clone https://github.com/guts-yang/Agent-utils.git \
  /usr/local/workspace/Agent-utils

如果目录已存在，先检查其 remote 是否指向上述仓库，并执行只读同步检查。
不要覆盖用户已有的未提交修改。

2. 读取远程仓库中的编排 Skill

cd /usr/local/workspace/Agent-utils

必须先完整读取：

- skills/research/workflows/unlearning-reproduction/SKILL.md
- skills/research/workflows/unlearning-reproduction/references/project-config.md
- skills/research/workflows/unlearning-reproduction/references/handoff-contract.md
- skills/research/workflows/unlearning-reproduction/references/prompt-migration.md

后续操作必须以该 Skill 为准，不要自行重新设计阶段顺序或停止条件。

3. 安装 Skill

将以下 Skill 安装到当前 agent runtime 的 skills 目录。
默认目录为 ~/.codebuddy/skills/；如果当前运行环境使用其他 skills 根目录，
先识别实际路径再安装。

- skills/research/workflows/unlearning-reproduction/
- skills/Auto-Research-In-Sleep/ 下所有 ARIS 子 Skill

安装后确认以下 Skill 可被调用：

- /unlearning-reproduction
- /idea-discovery
- /experiment-bridge
- /auto-review-loop
- /experiment-audit
- /result-to-claim

4. 检查 ARIS 运行依赖

检查 ARIS 所需的 tools/、shared-references/ 和 reviewer backend。
如果 Agent-utils 快照中缺少这些运行时依赖，从对应上游仓库补齐；
不要在依赖缺失时声称初始化成功。

如果需要上游仓库，使用：

https://github.com/guts-yang/Agent-utils.git

只补齐运行依赖，不修改本项目的研究配置。

5. 检查环境

只检查以下环境变量是否存在，不打印变量值，不把密钥写入任何文件：

- GITHUB_TOKEN
- HF_TOKEN
- Hy4_TOKEN
- DEEPSEEK_API_KEY
- GITHUB_NAME
- GITHUB_EMAIL

同时检查：

- /usr/local/workspace/.env 是否存在或环境变量是否已注入
- CUDA 和 CUDA 版 torch
- GPU 型号
- /jizhicfs/chenyliao/ 是否挂载
- ARIS reviewer 是否可用

6. 先执行 dry-run

在真正运行实验前，先执行：

/unlearning-reproduction \
  "机器遗忘方向的双线路复现" \
  --track both \
  --mode dry-run \
  --auto-proceed false \
  --pilot skip

dry-run 必须验证：

- LLM/LRM 两条线路的工作目录隔离
- 数据盘目录隔离
- baseline 配置路径
- INIT_REPORT.md 和 HANDOFF.md 路径
- ARIS 子 Skill 可调用
- .aris/runs/<run_id>.json 的恢复路径
- 阶段顺序为 init → idea-discovery → experiment-bridge → review/audit/claim

7. dry-run 通过后，开始正式流程

/unlearning-reproduction \
  "机器遗忘方向的双线路复现" \
  --track both \
  --mode full \
  --auto-proceed false \
  --pilot skip

两条线路必须完全分开：

- LLM Unlearning：ASU-unlearning、L7_ASU-Novel-Methods
- LRM Unlearning：R-TOFU、redact；awesome-lrm-unlearning 仅作综述和基线参考

所有模型、数据划分、baseline、指标定义和结果目录必须遵循各自的
INIT_REPORT.md，不得跨线路复用。

8. 初始化阶段只允许生成：

- 每条线路的 INIT_REPORT.md
- 全局运行状态文件
- 必要的环境和依赖检查记录

不要在初始化未通过时运行训练、生成 idea、部署 GPU 实验或输出最终结论。

最后报告：

- 远程仓库 commit
- Skill 安装路径
- ARIS 依赖状态
- 两条线路的初始化状态
- dry-run 结果
- 是否允许进入 idea-discovery
- 所有阻塞项

不得打印任何 token、密码或密钥内容。
```

备注：`"机器遗忘方向的双线路复现"` 可替换成具体研究方向。
