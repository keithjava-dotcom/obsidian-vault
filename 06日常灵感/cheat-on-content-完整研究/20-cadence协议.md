---
author: 小龙
created: 2026-06-23
tags: [cheat-on-content, cadence, buffer, 节奏, 协议]
---

# Cadence Protocol — 节奏协议

被 cheat-status、cheat-recommend、cheat-shoot、cheat-publish、SessionStart hook 引用。

## 目标

让用户在每次会话开场就能回答"我现在该拍/该发/该复盘"。让节奏管理自动化，不依赖用户主动感知。

## Buffer 体系

Buffer = state.shoots 数组长度 = 已拍未发布的视频数。

cheat-shoot（+1）vs cheat-publish（-1）两个事件分开，使 buffer 跟踪准确。

### Buffer 颜色（派生自 buffer_days）

```
buffer_days = buffer_count × target_publish_cadence_days
```

| buffer_days | 颜色 | 含义 | 行动 |
|---|---|---|---|
| < 1 | 🔴 红 | 警戒——下个发布日可能断更 | 今天必须拍，只拍稳分 top 1 |
| 1-2 | 🟠 橙 | 偏低 | 应该拍 1-2 条 |
| 3-5 | 🟢 绿 | 正常 | 节奏稳定 |
| > 5 | 🔵 蓝 | 积压 | 暂停拍摄，全力发布存货+复盘 |

示例：
- 日更 + buffer=0 → buffer_days=0 → 🔴
- 周更 + buffer=1 → buffer_days=7 → 🔵（一篇够发七天）
- 灵活模式 → buffer 监控关闭，只显示"已拍未发：N 条"

## 推荐策略

每次推荐 2 条时遵循 **1 稳分 + 1 实验性**：

**稳分**：排序 top 1-3，composite 高 + 议题安全 + 类目不重复
**实验性**：能验证某个待验证假设的样本，composite 不一定高但有信息价值

Buffer 颜色影响推荐策略：
- 🔴 → 只推稳分 top 1
- 🟠 → 1 稳 + 1 实验，优先稳分
- 🟢 → 标准 1+1
- 🔵 → 暂停推荐

## 节奏优先级

1. **Buffer 优先于评分**：🔴 时不要因为"等更好的选题"而断更
2. **复盘优先于新拍**：T+RETRO_WINDOW_DAYS 到期当天先复盘
3. **同步优先于积压**：🔵 时先发存货
4. **实验性最多 1/天**：每天拍 2 条时至少 1 条是稳分

## SessionStart 自动报告

每次会话开场渲染 4-6 行报告：
- 📦 Buffer 状态（颜色+数量）
- ⏰ 待复盘到期项
- 🎯 候选池 top 3（粗排）
- 📅 上次抓热点时间
- ⚠️ 关键 to-do

不主动开始任何动作——等用户决定。

## 关键纪律

- 已发过的 candidate（done）不推
- 用户跳过的 candidate（skip）6 个月内不推
- 同一 category 连发 ≤ 2 条
- 不主动违背节奏——检测到异常（buffer=0 但不拍 / 积压 ≥10 但继续拍）→ 显式报告但不停用
