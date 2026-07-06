---
author: 小龙
created: 2026-06-23
tags: [cheat-on-content, cheat-learn-from, 对标账号, benchmark]
---

# cheat-learn-from — 对标账号拆解

## 定位

工具早期最重要的信号源。cold-start 用户没自己历史时全靠对标，已发用户也建议至少 1 个对标做 sanity check。

## 工作流

```
[用户：学这个账号]
→ Phase 0: 检查 benchmark 状态（首次/已有/追加）
→ Phase 1: 选 input 方式
→ Phase 2: 收集材料（script + 数据）
→ Phase 3: 询问每条样本的印象（高/中/低 + 为什么）
→ Phase 4: Claude 拆 pattern + 派生 rubric 信号
→ Phase 5: 用户 review → 改 → 确认
→ Phase 6: 写 benchmark.md / script_patterns.md / rubric_notes.md
→ Phase 7: 更新 state
```

## 核心设计：三种素材获取方式

### Way a — 粘文本（推荐，最简单）
- 用户自己整理过/用工具提取过
- 工具推荐：轻抖小程序等

### Way b — whisper 转录视频文件
- 用户下载了 source.mp4 到 samples/ 目录
- 装 whisper-cpp + ffmpeg
- 准确度比 a 差

### Way c — 跳过 script，只用元数据 + 印象
- 拆不出深层 pattern
- 但 rubric 信号还行
- 适合"先快速搭起来，将来补"

## 核心设计：用户印象标记

每条样本收完数据后，追问用户：

> 你看完这条视频的印象，算这个账号的：
> a) 高表现样本（代表作）
> b) 中表现样本（普通水准）
> c) 低表现样本（不算代表作）

> 为什么？（一句话——这个判断比数据更能告诉我你想做什么风格）

**关键设计**：印象可以和数据冲突——比如某条数据高但用户觉得"不算代表作"。这种冲突本身是有用信号。

## 核心设计：拆 pattern + 派生 rubric 信号

### Script pattern 拆解
- 开头钩子类型分布（场景代入 / IS 戏仿 / 数据反转）
- 主体结构
- 句式/句长/节奏
- Emotional 标记/双声道
- 致谢段/收尾
- 高频词汇/词汇风格

### Rubric 信号（定性，不直接给数值权重）
- "ER 看起来重要"（3/3 高样本 ER≥4）
- "SR 看起来不显著"（高/低样本 SR 分布无差异）
- "MS 高的样本评论区有明显模因爆发"

## 写入的文件

### benchmark.md
- 账号信息 + 样本表 + 基础 rubric 派生 + 选题方向感

### script_patterns.md（对标段）
- 所有对标 pattern 标记：**Imported, untested on my channel**
- 实拍验证 ≥2 次才去掉标记升入正式 pattern

### rubric_notes.md（benchmark-derived segment）
- 仅定性方向，不直接采纳为数值权重
- 等用户自己 N≥5 校准样本后正式 bump 时再决定

## 关键设计洞见

1. **印象 vs 数据冲突是信号不是问题**——拆出"用户想成为的 vs 实际数据"的 gap
2. **所有对标 pattern 标 imported**——防止把对标当真理
3. **不给数值权重**——5-10 样本拟合容易过拟合
4. **用户 review 后才落盘**——"ok"或"Pattern X 我觉得不准"
