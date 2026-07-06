---
author: 小龙
created: 2026-06-23
tags: [cheat-on-content, bump, validation, protocol]
---

# Bump Validation Protocol — 升级验证协议

这是项目原则 #2（升级 = 全量重打）的完整规范。

## 触发 bump 的变更

**必须走完整 bump**：
- 公式系数变化（ER×1.5 → ×2.0）
- 维度增减（删 NA，加 MS）
- 维度定义颠覆性改写
- 归一化常数变化

**不走 bump**（但要标注）：
- 维度定义边际细化
- 单维度门槛变严的备注
- 锚点样本更新

## 何时可以提议

**软判断（Claude 判断，可软违反）**：
- 跨样本观察 ≥3 样本支持（可提前到 1-2 强信号）
- 当前 rubric 系统性同向偏差 ≥3 次（可提前到 1-2 极端偏离）
- 候选维度展示独立预测力
- 校准池跨分水岭（5/10/20/50）

**硬约束（不能突破）**：
- in_progress_session 不能为 null
- 上次 bump 后必须有新校准样本（至少 1）

## 完整 5 步流程

### Step 0：写成完整方程
不能只说"ER 权重提高"——必须写完整：
```
v2.1 composite = (ER×2.0 + HP×1.5 + MS×1.5 + QL + SR + TS + SAT) / 9.0 × 2.0
```

### Step 1：校准池全量重打分
- 所有 predictions/*.md 有完整复盘段的文件
- 用新公式重算 composite（维度分数不变，新增维度回追打分）
- **强制 cheat-score-blind sub-agent**（不接受 self-scored fallback）

### Step 2：排序一致性验证

| 样本 | composite(v2) | composite(v2.1) | rank(new) | actual | rank(actual) | delta |
|---|---|---|---|---|---|---|
| 仓鼠 | 9.41 | 9.55 | 1 | 124.8w | 1 | 0 |

**THRESHOLD = 0.8**（4/5 样本一致）
- 加 pairwise no-regression 检查——旧公式做对的 pair 不允许新公式颠倒

**不允许调低 THRESHOLD 绕过**——那是诚实的 self-deception。

### Step 3：跨模型独立审核（Channel C）
调 mcp__llm-chat__chat（qwen-max）审核：
1. 排序一致性真的 ≥80%？
2. 新公式解释力更强？

**判定**：本地 PASS + 外部 PASS → 通过。冲突 → 视为 REJECT。

### Step 4：落地
- 更新 rubric_notes.md 顶部版本号 + 版本速查表
- **写 rubric-memo.md**（含样本名+实绩），不写 rubric_notes.md
- 删除被吸收/被推翻的观察记录
- 更新所有校准样本 prediction 文件：Re-scored under vN 行

## 拒绝机制

| 失败位置 | 处理 |
|---------|------|
| Step 2 排序不一致 | 候选公式回到待验证区 |
| Step 3 外部 REJECT | 完整记录到 rubric-memo.md 的"被拒升级 log" |
| Step 3 内外冲突 | 视为 REJECT |
| Step 4 清算失败 | 回滚到 step 0 |

## 升级成本设计

校准池每多一个样本，bump 成本线性上升（追打新维度、全部重排、跨模型审核）。这是故意的：

**频繁 bump = rubric 在追噪声**。稳定 rubric 的特征是：bump 越来越罕见，bump 越来越大。

参考节奏：v1→v2 约 4 周，v2→v2.1 候选已 4 周仍在等待联合验证。
