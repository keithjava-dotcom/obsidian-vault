---
author: 小龙
created: 2026-06-23
tags: [cheat-on-content, starter-rubrics, 冷启动]
---

# Starter Rubrics — 各内容形态冷启动占位

原项目共 7 个 starter-rubrics，覆盖主流内容形态。所有 zero 版都是等权 + 比率桶的冷启动模板，核心结构和 opinion-video-zero.md（文件 28）一致。

## 完整清单

| 文件 | 形态 | 说明 |
|------|------|------|
| **opinion-video.md** | 观点视频（2-5min） | v2 已校准版（25+ 样本） |
| **opinion-video-zero.md** | 观点视频 | v0 等权占位 |
| **long-essay-zero.md** | 长文（公众号/Substack/Medium） | 等权 vs cold+rate |
| **short-text-zero.md** | 短文（X thread/微博/即刻） | 等权+短文节奏 |
| **podcast-zero.md** | 播客/长视频 | 等权+时长衍生指标 |
| **tutorial-builder-zero.md** | 教程/工具教学 | 等权+实用度维度 |
| **other-zero.md** | 其他（游戏/美食/妆教/剧情） | 等权+通用，需自行拆维度 |

## 各形态的差异化

相同点：7 维等权 + 比率桶 + confidence 派生 + cold-start 行为指南

不同点：

### long-essay-zero
- ER/SR/HP/QL/NA/AB/SAT 7 维等权
- HP（钩子力）的重要性在文字场景低于视频：前 3 秒可回看/跳过率不同
- QL（金句密度）在文字场景更重要：段落标题/引用是传播主力
- 比率桶用"阅读数"代替"播放数"
- 「基础盘」范围：50-500 阅读（0粉新号）

### short-text-zero
- 7 维等权，NA（叙事弧线）几乎不适用——短文天然没有三幕结构
- HP 退化到"前 3 个词能不能让人读完这一条"——更短更快
- SAT（讽刺深度）在短文场景（X/微博）权重更高——戏仿格式天然适配
- 比率桶用"互动数"（likes+reposts）作为主指标

### podcast-zero
- 引入非线性指标——播客的"完播段"比"完播率"更有意义（大多数用户跳跃收听）
- 建议降低 HP 权重——用户在播客场景主动选择了内容，钩子的"撕开注意力"价值低于视频
- WA（完播段留存）作为新增候选维度

### tutorial-builder-zero
- 引入 UT（实用度/Usefulness Threshold）维度
- 删 SR（社会议题共振）——教程场景几乎不适用
- SAT 权重降至最低——教程场景讽刺深度接近零
- 比率桶用"保存/收藏"作为主指标——比"播放"更能反映教程价值

### other-zero
- 通用等权占位
- 建议用户自己跑 5-10 篇后拆合适维度
- 最大风险：通用 rubric 在垂直形态上完全不预测——rubric_form_mismatch=true

## 通用冷启动原则

所有 zero 版共享 opinion-video-zero.md 的核心行为指南（文件 28）：
- 前 5 篇精度 ±50%（数学事实）
- 概率分布要平（30/30/20/15/5 而非 5/40/45/8/2）
- Cold-start 期主动选维度组合差异最大的样本
- 第 5 篇后必须跑 /cheat-bump --bucket-only
- 第 5 篇前不要基于 composite 决定发不发

不同形态有各自的首次 bump 建议时间（内容形态越垂直，越早 bump）：
- opinion-video: 5 篇
- tutorial-builder: 3 篇（UT 维度急需校准）
- long-essay: 5 篇
- short-text: 10 篇（短文样本密集，更多数据再 bump 更稳）
- podcast: 5-8 篇
- other: 8-10 篇
