---
author: 小龙
created: 2026-06-23
tags: [cheat-on-content, cheat-status, 看板, 状态]
---

# cheat-status — 状态看板

## 定位

任何时候可调、无副作用的状态看板。显示当前模式/rubric 版本/校准进度/待复盘/pool 状态/是否该 bump。

## 工作流

```
[用户：状态]
→ Phase 1: 读 .cheat-state.json + 扫文件系统
→ Phase 2: 计算派生指标
→ Phase 3: 检测建议触发器
→ Phase 4: 输出看板
```

## 派生指标

| 指标 | 算法 |
|------|------|
| Buffer 数 | len(state.shoots) |
| Buffer 颜色 | buffer_days <1 红 / 1-2 橙 / 3-5 绿 / >5 蓝 |
| Confidence 等级 | 从 calibration_samples 派生 |
| 校准样本数 | 中含完整复盘段的文件数 |
| 待复盘 | pending_retros 中已过 RETRO_WINDOW_DAYS 的 |
| 池大小 | candidates.md 中 tier!=skip 的 entry 数 |
| 同向偏差队列 | state.consecutive_directional_errors |

## 优先级触发

按优先级（高→低）逐项检查：

- **🔴 buffer 红** → 第一行高优先级警戒
- **🔵 buffer 蓝** → 高优先级提示暂停拍摄
- **拍摄超 14 天未发** → "议题时效流失风险"
- **待复盘 ≥ 1** → 高优先级
- **pool_status=none + calibration_samples=0** → "5 分钟拿 5 个候选"
- **bump 信号** → 建议

## Confidence 事件

| 阈值 | 提示 |
|------|------|
| 0→1 篇 | 🎉 confidence 升级 |
| 跨过 5 篇 | "可以第一次正式 bump 了" |
| 跨过 10 篇 | "可以跑 percentile bucket 重校" |

## 输出示例

```
🎛️ cheat-on-content 状态

内容形态：opinion-video / 时长 3-5min / cadence: 隔日更
当前 rubric：v2 (上次 bump: 2026-04-22)
校准样本：18 篇
Confidence: 🟢 较高
Baseline: 4.2w 中位数

📦 Buffer：3 篇（🟢 绿色）

🎬 待办（按紧急度）
 🚨 复盘 1 篇（已过 T+3d）
 ⚠️ 同向偏差 3 次（high, high, high）→ 建议 /cheat-bump

🔥 候选池
 - candidates.md: 27 条
 - 距上次抓热点: 4 天

下一步建议（按推荐优先级）：
1. /cheat-retro predictions/... ← 最紧急
2. /cheat-bump ← 同向偏差 3 次
```

## 关键设计洞见

1. **无副作用**——读多写零。任何状态修改是其他 skill 的事
2. **不假装数据可用**——字段缺失显式标"未知"，不猜
3. **建议带优先级 + 确切命令**——用户 copy-paste 执行
4. **自动检测建议触发器**——每个跨阈值事件都有对应提示
