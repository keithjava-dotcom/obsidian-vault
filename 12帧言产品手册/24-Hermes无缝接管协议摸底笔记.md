---
author: Hermes (claude-sonnet-4 via Nous)
maintainer: Hermes
created: 2026-08-04
type: 技术笔记
tags: [帧言, Hermes底座, 无缝接管, 协议]
---

# Hermes 无缝接管 SkillRunner · 协议摸底笔记（内部工作用）

> 目标：让 Hermes 五棒无缝接管现有 SkillRunner 创作，前端零改动、体验无缝。

## 前端交互协议（Fy*Panel → skillService）

每个 FyPanel（Xuanti/Dagang/Fanwen/Broll/Fupan）通过 `skillService` 三件事操作会话：
1. `startSkill({ skillId, userId, forkId, initialInput, launchContext })` → 建会话，返回 Session
2. `stepSession(sessionId, userInput)` / `continueSkill(session.id, userInput)` → 推进当前棒，返回新 Session
3. `getSession(sessionId)` → 恢复会话

后端对应路由：`POST /api/session/:id/step` → `executeSkillStep` → `runTurn`（自研引擎 LLM）。

## Session 关键字段（前端渲染面）
- `session.id` / `skillId` / `status` / `currentStepId`
- `session.spec.stateMachine.steps` — 步骤定义（前端用 currentStep.kind 渲染不同 UI）
- `session.messages` — SkillMessage[]（对话流：guidance/user/artifact/blocked/handoff）
- `session.artifacts.divergencePoints` — 选题发散点（diverge_topics 步骤渲染）
- `session.inputs.launchContext` — 账号上下文
- `session.blockedPayload.formSchema` — 被拦时表单

## 无缝接管关键结论
- 前端**完全通过 skillService **调后端，不直接碰引擎 → 后端 `executeSkillStep`/`step` 路由是唯一接管点
- Hermes 分支要"无缝"：返回的 Session 必须让 FyPanel 正常渲染
- 最小可行：Hermes 会话把产出作为 `messages` 里的 `guidance`/`artifact` 追加；depth 内容（divergencePoints）后续按需映射
- **不试图复刻整个自研 stateMachine 协议**（那是死胡同），用轻量 Hermes 会话 + 前端已用的消息流

## 已实现（方案甲独立通道，作为中间验证）
- 后端 `/api/hermes/run` + `/api/hermes/skills`（routes/hermes.ts）
- 前端 HermesPipeline 独立页（后作为深入 SkillRunner 的辅助）
- HermesAdapter (hermesChat) + profile 记忆隔离

## 下一步（无缝接管实现）
在后端 `executeSkillStep`（或新增 session/hermes 端点）加 Hermes 分支：
- 当 session.skillId ∈ {xuanti,dagang,fanwen,broll,fupan} → 调 hermesChat(skill)，产出作为消息 append，推进 step，返回兼容 Session
- 前端零改动
