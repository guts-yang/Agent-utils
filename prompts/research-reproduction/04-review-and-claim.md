# 04 · 双线路评审与声明 Prompt

- 执行顺序：4 / 4 —— 承接 03 的实验结果。每条线路调用一次 `/auto-review-loop`，再做诚信核对与声明判断。
- 用途：决定选定 idea 的 claim 是否被证据支持；不在本环节重跑训练。
- 状态：v1（2026-11-25）：停止条件用 ARIS 评审与 `/result-to-claim`。超过基线只记对照，不作为结束条件。
- 使用：先完成 `03-experiment-bridge.md`，且本线路 `HANDOFF.md` 允许进入 04。复制下方代码块到新会话。

```text
你是本项目的双线路评审与声明助手。只读本线路 HANDOFF.md、INIT_REPORT.md 和 03 的结果。
HANDOFF.md 未允许进入 04 时立即停止。不调用 /paper-writing。
不把两条线路的结果合并、互相排名，或套用另一条线路的基线与指标口径。

两条线路各自在自己的工作目录里，按下面顺序各走一遍。
不要在这些技能外面再套 /loop、/schedule 或定时器。
reviewer 使用本项目覆盖：Hy4（hunyuan 家族，executor 为 deepseek 家族），见
skills/research/workflows/unlearning-reproduction/references/reviewer-override.md；
身份不可验证、后端不可用或无 trace id 时，仍按 REVIEW_UNAVAILABLE / BLOCKED 处理，
不得用执行者判断替代 reviewer 结论。

1. /auto-review-loop "<HANDOFF.md 里的选定 idea>" — compact: true
   停止条件只用技能自己的：score >= 6 且 verdict 为 ready 或 almost，或达到 MAX_ROUNDS。
   执行者不能用「还能优化」或「已经超过基线」宣布结束。

2. /experiment-audit
   先做诚信核对，让下一步能读到 EXPERIMENT_AUDIT.json。

3. /result-to-claim
   对照 idea-stage/docs/research_contract.md 与实验结果。
   相对锁定基线的数字只写入对照记录，不作为停止条件。

按 /result-to-claim 的结论路由，训练一律交回 03，不在本环节重跑：

- claim 被支持：归档该 idea 的 refine-logs/FINAL_PROPOSAL.md、实验记录和结论。
  该 idea 停止。归档写进本线路 HANDOFF.md。
- 部分支持：收窄声明，或把要补的实验写进 HANDOFF.md，交给 03。
  补跑完成后回到本环节，重新做一次 1–3。
- 不支持：在 HANDOFF.md 记录失败原因与已排除的做法。
  从 idea-stage/IDEA_CANDIDATES.md 取下一个候选，更新选定 idea 和计划路径，
  交给 03 重新复现。没有下一个候选时，该线路停在「无可用 idea」。
- 评审要求修改代码或实验：把具体修改写入 HANDOFF.md，交回 03。
  不在本环节里直接改训练或重跑。

两条线路都结束后，并列给出各自的三件事：与锁定基线的对照、可保留 idea 及其证据、
被淘汰 idea 及原因。不跨线路排名。

约束：不上传数据、不泄露密钥、不擅自改基线定义。
```

备注：`score >= 6` 与 verdict 词汇以 `/auto-review-loop` 的技能定义为准；本提示词不另设指标门槛。
