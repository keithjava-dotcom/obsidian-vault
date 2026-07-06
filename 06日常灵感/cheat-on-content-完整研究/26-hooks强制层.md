---
author: 小龙
created: 2026-06-23
tags: [cheat-on-content, hooks, 强制层, prediction-immutability, meta-logging]
---

# Hooks — 强制层

三个 hook 构成系统的工程保障，防止"自己骗自己"。

## Hook 1 — Prediction Immutability（预测锁）

**文件**：hooks/prediction-immutability.json + hooks/prediction-immutability.sh

**作用**：在 PreToolUse(Edit|Write) 上检查 predictions/ 下文件。命中 `## 预测` 与下一个二级标题之间的任何 diff → exit 1 阻塞。只有 `## 复盘` 段的追加 → 放行。

**影响**：预测段一旦写完不可能被任何人修改——包括用户本人。这是校准循环的信号源保护。

**绕过**：允许用户设置 `CHEAT_BYPASS_IMMUTABILITY=1` 环境变量单次绕过（仅用于 markdown 排版格式修复），绕过在 git history 留痕。

**hook 不拦的场景**：
- 预测文件不小心被人手编辑了 → 不自动回滚（破坏更大）。下次 cheat-retro 检测到不一致 → 在复盘段追加 Integrity warning，不计入 bump 校准池
- 预测文件遗失/被删 → git log 找回。找不到 → rubric_notes.md 记录"预测文件遗失"

## Hook 2 — SessionStart 自动报告

**文件**：hooks/session-start.json + hooks/session-start.sh

**作用**：每次新会话开场自动渲染 4-6 行报告：
- 📦 Buffer 状态（颜色+数量）
- ⏰ 待复盘到期项
- 🎯 候选池 top 3（粗排）
- ⚠️ 关键 to-do

**不主动开始任何动作**——等用户决定。设计上区分"看板"和"执行"。

**状态检测触发**：
- 任何已发未复盘+时间到 → 顶部高亮
- Schema mismatch → 建议跑 /cheat-migrate
- last_prediction_self_scored && days_since >= 7 → 红色警告

## Hook 3 — Meta-logging（静默使用日志）

**文件**：hooks/meta-logging.json + hooks/log-event.sh

**作用**：异步记录使用频率（不记录用户内容）。只存 prompt_present(bool) + prompt_chars(长度)，不存 prompt 内容。

**用途**：将来诊断用——哪些 skill 用得最多、校准循环的完成率等。

**历史修复**：v1.3 曾把每条用户 prompt 前 120 字存进 usage.jsonl（过度采集），v1.4 修复为只存 bool+长度。
