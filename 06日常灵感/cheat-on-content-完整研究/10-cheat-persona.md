---
author: 小龙
created: 2026-06-23
tags: [cheat-on-content, cheat-persona, 受众画像, 评论聚类]
---

# cheat-persona — 受众画像

## 定位

从复盘评论数据聚类出账号真实受众画像。和 rubric 平行的第二个派生物——rubric 答"怎么打分"，persona 答"谁在看"。

## 关系

```
复盘数据（评论 + 完播 + 转粉）
 ├──→ rubric 进化（cheat-bump）—— "怎么打分"
 └──→ 受众画像（cheat-persona）—— "谁在看"
```

两者都喂给 cheat-seed，但用途不同——rubric 影响打分，persona 影响选题和写稿角度。

**绝不混**：persona 不进打分公式。audience.md 在 cheat-score-blind 的 hard refusal list 里。

## 工作流

```
[用户：构造受众画像]
→ Phase 0: 扫 predictions/*.md 复盘段评论 + benchmark.md
→ Phase 1: 数据量判定 → 派生 Confidence 等级
→ Phase 2: 评论四维聚类
→ Phase 3: persona × rubric 交叉检验
→ Phase 4: 写 audience.md（覆盖式重建）
→ Phase 5: 控制台报告 + 跟上版画像的 diff
```

## 评论四维聚类

### 维度 1：自我认同
统计哪些身份反复出现——"我也是…"/"这就是我"/"作为一个[大厂打工人/一人公司/考研党…]"

### 维度 2：情绪寄存
观众来评论是为了什么情绪？
- 被验证（"说得太对了"）
- 宣泄（"我也好累"）
- 抬杠（"我不同意"）
- 求助（"那该怎么办"）

### 维度 3：反驳点
哪些观点引来稳定的反对声——这是 persona 边界

### 维度 4：语言
怎么说话——玩梗密度、真诚 vs 戏谑、有没有复制你的金句造句

## Confidence 等级

| 数据基础 | Confidence |
|---------|-----------|
| 0 篇复盘 + 无 benchmark | 🔴 无数据 |
| 0 篇复盘 + 有 benchmark | 🟠 benchmark-seed 未验证 |
| 1-2 篇复盘 | 🟡 早期信号 |
| 3-5 篇复盘 | 🟢 数据扎实 |
| ≥6 篇复盘 | 🔵 稳健 |

## 写入 audience.md 的结构

- **验证特征**（≥3 条评论证据 + 出处）
- **假设特征**（证据不足的候选）
- **反画像**（你以为的受众 vs 实际评论的人不一样）
- **persona × rubric 交叉检验**（persona 说受众爱 X→校准池里 X 真的 over-perform 吗？）
- 版本号 + 数据基础 + 版本历史

## 关键设计洞见

1. **数据驱动不手写**——persona 必须来自评论聚类。用户想手动加 → 标 user-asserted 放"假设"段
2. **证据强制**——验证特征至少 3 条评论证据，否则降为假设
3. **反画像强制**——防止 persona 变成讨好自己的虚构
4. **矛盾不调和**——persona × rubric 冲突时如实 flag，不编故事
5. **覆盖式重建**——每次 rebuild 重写全文，只有版本历史段 append
6. **不进打分**——persona 永远不喂 cheat-predict/cheat-score-blind

## 和我们体系

dingwei-report-workflow 的"用户画像"模块是基于柯哥对客户的理解。这个是**从数据反向聚类**出真实受众。两者可以互为补充。
