---
author: 小龙
created: 2026-06-23
tags: [cheat-on-content, cheat-bump, rubric, 升级, 核心]
---

# cheat-bump — ⭐Rubric 升级

## 定位

提议并执行 rubric 公式升级。工具中最高风险的动作——5 步强制 + 跨模型审核。一次操作只做一种事——要么调公式，要么调 bucket 边界。

## 两种模式

### 完整 rubric bump（--propose "..."）
改公式 / 维度 / 权重，5 步验证 + 跨模型审核（强制）

### bucket-only 重校（--bucket-only）
只重新派生 bucket 边界，数据自动派生，无审核

## 完整 Rubric Bump 工作流

```
[用户：升级 rubric --propose "ER×1.5→2.0，砍 NA，加 MS"]
→ Phase 0: 前置门槛检查
→ Phase 1: 写出新公式完整方程
→ Phase 2: 校准池全量重打分（强制 cheat-score-blind sub-agent）
→ Phase 3: 计算排序一致性
→ Phase 4: 跨模型独立审核（强制）
→ Phase 5: 落地 + cleanup pass（删被吸收的观察）
→ Phase 6: 更新所有校准样本 prediction 文件底部 Re-scored 行
```

## Phase 0：前置门槛检查

检查项（可软判断）：
- 校准池 ≥ 5 样本 + 至少 1 个跨样本观察有 ≥3 样本支持
- 但如果观察信号特别强（1 篇复合 8.5 实绩 5w 的 ≥3x 偏差）→ 可以提早提议
- Claude 也可以拒绝 bump 即使样本足——如果证据弱
- in_progress_session == null（走完当前流程再 bump）

## Phase 2：全量重打（强制 sub-agent）

**这是 bump 最严格的约束**：

不接受 self-scored fallback——cheat-predict 有 --skip-blind flag，但 cheat-bump 没有。如果 Task tool 不可用 → abort bump。

每条 prediction 的所有 dimension 都由 sub-agent 重新审 script——即使只是调权重不改维度。理由：旧 dim 分本身可能是污染的；权重变了不能保证旧 dim 还成立。

## Phase 3：排序一致性

每个样本算新公式 rank vs 实际播放 rank，delta = |new_rank - actual_rank|

| 样本 | composite(v2) | composite(v2.1) | rank(new) | actual | rank(actual) | delta |
|---|---|---|---|---|---|---|
| 仓鼠 | 9.41 | 9.55 | 1 | 124.8w | 1 | 0 |
| 停止期待 | 8.24 | 9.11 | 2 | 71.1w | 2 | 0 |
| 老板废话 | 7.65 | 8.11 | 4 | 39.6w | 3 | 1 |

**THRESHOLD = 0.8**——新排序与实际排序在 ≥80% 样本上一致 → 通过。
**不允许临时调低**——那本身是一个需要 bump 的元决策。

## Phase 4：跨模型独立审核

调 mcp__llm-chat__chat（qwen-max）审核：

- 排序一致性：新公式 vs 实际表现是否 ≥80% 一致？
- 解释力：新公式比旧公式更好地解释了实绩分布吗？

判定逻辑：本地 PASS + 外部 PASS → 通过。本地 PASS + 外部 REJECT → 视为 REJECT（冲突意味着至少一方解读不稳定）。

## 已知污染诚实标注

两项残余 contamination 在 bump report 里诚实标注：
1. **model_prior_warning**：sub-agent 仍是 Claude，RLHF 共享（true，不可关）
2. **rubric_self_designed**：rubric 是用户自己写的，自然 fit 自己内容（true，不可关）

**意义**：不假装隔离完美，接受"有残余"再去改进。

## 落地后清理

- rubric_notes.md 顶部更新版本号
- 版本速查表加一行（只含版本号+公式签名，不含证据样本）
- rubric-memo.md 追加升级 memo（含实绩、被删除的观察清单、新维度派生逻辑）
- 删被吸收的观察（rubric 是工作台不是博物馆）

## 关键设计洞见

1. **升级有刹车**——全量回测 + 跨模型审核防止 overfit
2. **强制 sub-agent**——不接受自审，防止后视镜打分
3. **诚实污染标注**——不假装隔离完美
4. **一步只做一种事**——不混调公式和 bucket 边界

## 对我们体系的价值：P1

评分标准升级 + 全量回测验证的逻辑可以吃进 fupan-workflow 的"观察积累→阈值触发→升级提议"流程。
