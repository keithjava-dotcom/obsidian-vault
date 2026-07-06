---
author: 小龙
created: 2026-06-23
tags: [cheat-on-content, cheat-predict, 预测, immutable, 核心]
---

# cheat-predict — ⭐盲预测日志

## 定位

整个校准循环的核心动作——给最终稿写一份 immutable 的预测日志。预测段一旦落盘不可改，由 hook 强制。

## 工作流

```
[用户：启动预测 scripts/<id>.md]
→ Phase 0: blind check 自检（见过任何数据？→拒绝）
→ Phase 0.7: 模式判定 — v1（新建）还是 v2（append）
→ Phase 1: 读 script + rubric + state + 派生 confidence
→ Phase 2: 委派 cheat-score-blind sub-agent 拿 9 维盲打
→ Phase 2.5: 主 Claude review——|delta|≥2 弹用户裁定（a/b/c）
→ Phase 3: 找锚点对比（2-4 个 composite 邻近的旧样本）
→ Phase 4: 给 bucket + 概率分布 + 中枢（confidence 低时分布更平）
→ Phase 5: 写反事实场景 + 关键校准假设
→ Phase 5.5: 用户 review → "ok"落盘 / 挑刺→改→循环
→ Phase 6: 落盘——v1 写新文件 / v2 append
→ Phase 7: 更新 state
```

## Blind Check（Phase 0）

严格检查是否见过实绩数据：

| 信息 | 破坏 blind？ | 例外 |
|------|-------------|------|
| 播放数/阅读数 | ✗ 破坏 | 无 |
| 点赞/评论/转发数 | ✗ 破坏 | 无 |
| 具体评论内容 | ✗ 破坏 | 无 |
| 算法推荐位 | ✗ 破坏 | 无 |
| 发布后截图/后台数据 | ✗ 破坏 | 无 |
| 同期其他人作品表现 | ○ 不破坏 | — |
| 历史上类似主题表现 | ○ 不破坏 | 正是锚点对比要做的 |
| 发布前稿子内容 | ○ 不破坏 | 这是预测的输入 |
| 用户口述"我感觉还行" | △ 谨慎 | 标用户偏见 |

**判断捷径**：只要这条信息只能在作品发布后才能获得，就算"数据"。

## 7 组件预测文件

```
# 标题 — 预测日志

## Header（组件1）
- Article ID, Rubric Version, 预测时间, Script Path, Script Hash
- Calibration Samples, Confidence
- Prediction Basis（pre_shoot / post_shoot_pre_publish）
- BlindScored By, BlindScore Disagreement（全维度delta=0也记）
- 预测时数据状态: **blind**（关键声明）

## 输入快照（组件2）
- 7 维分数 + 用户改写要点 vs Claude 草稿
- 如用户原创稿 → 标"用户原创稿，无 Claude 草稿对照"

## 预测 v1（组件3）⭐IMMUTABLE 起点
- Bucket: `30-100w`
- 概率分布: <5w→3%, 5-30w→22%, 30-100w→55%, >100w→17%, >150w→3%
- 中枢: ~50w
- 一句话 reason（浓缩到 DB 字段可检索）

## 推理因素（组件4）
| 因素 | 方向 | 置信度 | 说明 |
|---|---|---|---|
| ER=5 | 强+ | 高 | "半夜三点翻聊天记录"极端具象 |
| SR=2 | 强- | 高 | 纯个人情感，天花板有限 |

## 锚点对比（组件5）
校准池不够时写"N/A 段"——不是删除段落

## 反事实场景（组件6）
每个可能 bucket 写一段"如果落在这里，意味着什么rubric假设被验证/推翻"

## 关键校准假设（组件7）
这次预测作为实验的明确赌注——"我押本篇 > 谁问你了（比率 1.5-2x）"

---

## 复盘（仅追加，IMMUTABLE 边界）
实绩数据 + top 评论聚类 + 验证/推翻 + 新观察
```

## 关键设计洞见

1. **概率分布必须 100%**——这是逼你诚实的工具
2. **cold-start 不简化**——统一 7 组件格式，用 Confidence 等级标可信度（不改结构只改标注）
3. **v2 追加**——拍后改稿 ≥30% 走 v2 重判，append 不覆盖，v1 留作历史档案
4. **BlindScore Disagreement 全记**——即使 delta=0 也记，用于分析"哪类维度 sub-agent 与主 Claude 系统性分歧"
5. **Prediction Basis 字段**——区分 pre_shoot（标准盲）vs post_shoot_pre_publish（软盲）

## 对我们体系的价值：P0

这是最该移植的设计。fupan-workflow 目前是事后分析；加了盲预测后变成"事前×事后双环校准"。
