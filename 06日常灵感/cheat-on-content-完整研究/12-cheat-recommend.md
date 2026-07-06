---
author: 小龙
created: 2026-06-23
tags: [cheat-on-content, cheat-recommend, 候选池, 推荐]
---

# cheat-recommend — 候选选题推荐

## 定位

从 candidates.md 按当前 rubric 排序推荐 top N 选题。**候选池不存在时给引导而不是报错**。

## 工作流

```
[用户：推荐选题]
→ Phase 0: 检查 candidates.md 存在性（不存在→引导）
→ Phase 1: 解析候选列表
→ Phase 2: 过滤（tier/安全性/已发过）
→ Phase 3: 按 composite 排序 + 找锚点
→ Phase 4: 输出 top N + 每条 rationale + 锚点对比
```

## 核心设计：Buffer 颜色驱动推荐策略

| Buffer 颜色 | 推荐策略 |
|---|---|
| 🔴 红（断更风险） | 只推 top 1 稳分，不推实验性 |
| 🟠 橙 | 标准 1 稳 + 1 实验，但提示"优先拍稳分" |
| 🟢 绿（默认） | 1 稳分 + 1 实验性 |
| 🔵 蓝（积压） | **拒绝推荐**，提示先发存货+复盘 |

## 推荐格式

```
🎯 候选池推荐（rubric: v2 / buffer: 🟢 绿 / cadence: 隔日更）

📌 第 1 条 — 稳分（推荐立即拍）：
  [tier1] [composite 9.18] "为你好"高密体系
  - 维度：ER=5 HP=5 QL=4 NA=4 AB=5 SR=5 SAT=4
  - 粗预测桶：30-100w（中枢 ~60w）
  - rationale：ER+SR 双 5 顶配，"高密度家庭议题"普适且分享安全
  - 锚点：仓鼠 (composite 9.41, 实绩 124w)

🧪 第 2 条 — 实验性（验证特定假设）：
  [tier1] [composite 8.71] 哈哈长度
  - **测试目标**：v2.1 候选维度 MS+TS 双 5
  - 信息价值：拍这条能强证据/弱推翻 v2.1 升正
  - 锚点：谁问你了 (composite 8.24, 实绩 11.7w)
```

## 无候选池引导

candidates.md 不存在时不报错，输出 4 个建立方式：
1. 跑/cheat-seed（一次性的种子动作）
2. 跑/cheat-trends（日常补充）
3. 手动建 candidates.md
4. 从 Notion/RSS 导入

## 关键设计洞见

1. **Buffer 颜色驱动推荐策略**——避免断更风险还推试验性内容
2. **每条附锚点 + rationale**——不让用户只看 composite 做决策
3. **候选池不存在不报错**——给引导，不是拒绝服务
4. **重复类别过滤**——避免连续推同类内容审美疲劳
