---
author: Hermes
created: 2026-08-04
type: 施工单
tags: [帧言, 架构改造, Hermes底座, 施工]
---

# 帧言架构改造施工单 · 接 Hermes 底座（步骤 3）

> 前置：功能测试 ✅（17 号报告）+ 工程验收 ✅（18 号报告）全绿
> 原则：蓝图 v2「不造轮子」——帧言壳保留，砍自研引擎，换 Hermes 大脑
> 状态：**方案待柯哥拍板后动工**（大改核心，不趁人不在偷跑）

---

## 一、改造对象（已实测盘点）

| 自研模块 | 规模 | 功能 | Hermes 替代 |
|---------|------|------|------------|
| `bff/src/skill-executor/` | 17 个 .ts | 技能加载/步骤执行/提示词组装/交接卡/事实闸/循环闸/会话存储 | `hermes chat -s <skill>`（技能预载+工具+记忆） |
| `bff/src/llm/providers/` | 8 个 .ts | anthropic/deepseek/openai 三套接入 | Hermes 多模型通道（Nous 全包） |
| `bff/src/repo/calibration.ts` | 1 文件 | 校准闭环 | Hermes fupan 校准闭环（已融合） |
| 会话/记忆存储（repo/sessions 等） | — | 每用户会话 | Hermes 多会话/多 profile |

**依赖方 6 处**：routes/skills.ts、routes/session.ts、routes/skill-merge.ts、repo/skillVersions.ts、repo/handoffCards.ts、repo/sessions.ts

## 二、对接方案（两层，先易后稳）

### 层 1（快速落地）：BFF 子进程调 `hermes chat CLI`
```
帧言前端 → 帧言 BFF（用户/订阅/运营）→ hermes chat -z "用户消息" -s <技能包> [--continue 会话ID]
                                     ← 回复 + 会话ID
```
- ✅ 零协议开发，`--continue` 天然续会话，`-s` 预载技能，工具/记忆/多模型全白拿
- ✅ 每用户一个 Hermes 会话 ID = 天然多用户隔离
- ⚠️ 进程级开销（每次启动 ~秒级）——用户量大了再升级层 2

### 层 2（长期稳定）：BFF 对接 `hermes serve`（JSON-RPC/WebSocket）
- 桌面 app 同款后端协议，headless 常驻
- 需要实现 JSON-RPC 客户端（中等工作量）——**用户量起来后做**

## 三、改造步骤（建议顺序）

| 步 | 动作 | 验收 |
|----|------|------|
| 1 | 写 Hermes 适配器（`bff/src/hermes-adapter/`）：封装 chat CLI 调用、会话映射、超时/重试 | 适配器单测过 |
| 2 | skills 路由改走适配器（先 xuanti 一个技能灰度） | 创作主链路回归测试过（17 号报告同款流程） |
| 3 | 全部技能切换 + 砍 skill-executor/llm providers | 全套测试过（18 号报告命令） |
| 4 | 校准/记忆切换 Hermes | 校准流程回归 |
| 5 | 清理死代码 + 更新 docs | build 全绿 |

**每步可回滚**（git 分支 + 旧模块保留一个版本周期）。

## 四、风险与对策

| 风险 | 对策 |
|------|------|
| CLI 子进程性能（每消息 ~1-3s 启动） | 层 1 只验证闭环；正式前上层 2；或 hermes chat 常驻会话复用 |
| 技能执行语义差异（帧言自研执行器 vs Hermes 技能运行方式） | 步骤 2 先灰度 xuanti，逐技能对齐；差异记入适配器 |
| 多用户隔离 | 每用户独立 Hermes 会话 ID；profile 隔离（长期） |
| Hermes 更新破坏兼容 | 适配器是唯一对接面，锁 hermes 版本 + 更新测试 |

## 五、待柯哥拍板

1. **动工时间**：方案 OK 就开工？（预计 3-5 个工作日，Codex/Claude 主力 + 我验收）
2. **层 1 vs 直接层 2**：先 CLI 快速闭环（推荐），还是直接上 JSON-RPC？
3. **灰度范围**：先 xuanti 一个技能试水，还是全量切换？
4. **开发主力**：Claude（claude-sonnet-4）接手改造，还是我直接改？
