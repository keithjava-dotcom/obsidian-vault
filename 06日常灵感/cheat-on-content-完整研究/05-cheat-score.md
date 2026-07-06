---
author: 小龙
created: 2026-06-23
tags: [cheat-on-content, cheat-score, 打分, rubric]
---

# cheat-score — 单稿打分

## 定位

给单篇稿子打 rubric 分，**只在控制台输出，不写文件，不预测**。是 cheat-predict 之前的轻量探索动作。

## 工作流

```
[用户：打分这篇 draft.md]
→ 读 draft.md + rubric_notes.md
→ 委派 cheat-score-blind sub-agent 打分（防污染）
→ 计算 composite
→ 控制台输出：评分表 + composite + 推荐下一步
→ 结束 — 不写任何文件
```

## 核心设计：打分走 Channel B sub-agent

主对话已经被用户对话/已发数据/历史复盘段污染——inline 评分等于带着后视镜判分。

通过 Task tool spawn cheat-score-blind sub-agent：
- 只读 script_path + rubric_notes_path
- 硬拒绝读 state/predictions/videos/audience/rubric-memo
- 输出严格 JSON（9 维 × {score, confidence, reason}）

主 Claude 只做调度 + review + 算 composite。

## 控制台输出格式

```
📊 [draft.md 短标题] — 打分（rubric: v2）

| 维度 | 分 | 理由 |
|---|---|---|
| ER (情感共鸣) | 5 | "半夜三点翻聊天记录"极端具象 |
| HP (钩子强度) | 5 | IS 句一句锁定受众 |
| ... | ... | ... |

公式：(ER×1.5 + SR×1.5 + HP×1.5 + QL + NA + AB + SAT) / 8.5 × 2.0
composite = (5×1.5 + 2×1.5 + ...) / 8.5 × 2.0 = **8.24**

📍 落在 30-100w 桶

下一步建议：
- 如果你已写定最终稿、准备发布 → 说"启动预测"
- 如果想再改稿子 → 改完再打一次
- 如果想看历史相近 composite 的样本 → 说"找 composite 8.0-8.5 的锚点"
```

## 约束

- ❌ 不写任何文件（score 是探索，predict 才是承诺）
- ❌ 不给 bucket 概率分布（那是 cheat-predict 的活）
- ❌ 不触发"已发布"或"复盘"逻辑
- ❌ 不提议 rubric 升级
- ✅ 整数分（不允许 4.5、3.7）
- ✅ 理由是诊断工具，复盘时用来找哪个维度判断错了
- ✅ 连续 3 次给同一稿打分 → 提示决策疲劳

## 和我们的 fanwen-workflow

fanwen 目前写完稿直接交付，没有"先打分看 composite"这一步。建议在 0.4b 交付前自检之后，加一步**交付前盲打分**，让每条稿发出前有一个可复盘的 baseline。
