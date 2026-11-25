# 03 · 双线路 Experiment Bridge Prompt

- 执行顺序：3 / 4 —— 承接 02 的 `HANDOFF.md` 与 `refine-logs/EXPERIMENT_PLAN.md`。
- 用途：在锁定基线上实现并跑完选定 idea 的实验；不开始评审。
- 状态：v1（2026-11-25）：对应 ARIS `/experiment-bridge`。评审交给 `04-review-and-claim.md`。
- 使用：先完成 `02-idea-discovery.md`，且本线路 `HANDOFF.md` 允许进入 03。复制下方代码块到新会话。

```text
你是本项目的双线路实验执行助手。只读本线路 HANDOFF.md 和 INIT_REPORT.md。
HANDOFF.md 未允许进入 03，或证据门为 BLOCKED 时，立即停止，不部署。

两条线路必须完全分开，只在自己的工作目录里工作：
- 线路 A · LLM Unlearning：/usr/local/workspace/tracks/llm-unlearning/
- 线路 B · LRM Unlearning：/usr/local/workspace/tracks/lrm-unlearning/

每条线路调用一次：
/experiment-bridge "<HANDOFF.md 里的选定 idea>" — code review: true, sanity first: true, base repo: <本线路仓库路径>

base repo 用 INIT_REPORT.md 中该线路的仓库，不新写一套代码，也不使用另一条线路的仓库。
实现和评测沿用锁定基线：同一模型、同一数据集划分比例、同一指标定义与呈现方式。
不得改口径。数据、checkpoint、结果、缓存写入 /jizhicfs/chenyliao/<track>/，
工作区只用软链。

按 EXPERIMENT_PLAN.md 的顺序执行：先 sanity，再基线，再主实验。
异常先记录，不跳过，不跨线路复用数据、模型或配置。
停在实验结果和 refine-logs/EXPERIMENT_TRACKER.md。
不调用 /auto-review-loop、/result-to-claim 或 /experiment-audit。
不根据指标是否超过基线宣布结束或继续优化。

更新本线路 HANDOFF.md：结果路径、tracker 路径、与锁定基线的对照数字
（只记录对照，不是停止条件）、是否允许进入 04。写完即停。
若 04 把本线路交回，只执行 HANDOFF.md 里写明的修改或补实验，然后再次更新交接。
```

备注：对照数字来自本次运行日志，不改写锁定基线的定义。
