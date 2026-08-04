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

## 二、对接方案（多用户容量导向 · 2026-08-04 柯哥提示"架构层面思考容纳更多用户"）

> 🔴 **架构决策（修正版）**：不做 CLI 子进程模型（每用户消息起一个进程，~50 活跃用户即打爆本机）。
> **直接按层 2（长驻服务）设计**：BFF 通过统一 Adapter 接口调 `hermes serve`（JSON-RPC/WebSocket 常驻），
> 一个常驻进程服务所有用户，天然并发。CLI 只作为适配器开发的调试工具，不进生产路径。

### 目标架构（多用户版）

```
用户（APP / 小程序 / 微信）
    ↓
帧言 BFF（无状态 · 可水平扩展）
    用户/订阅/运营/商城/队列/限流
    ↓ HermesAdapter（唯一对接面，接口抽象）
    ↓
Hermes 节点（有状态）
    hermes serve 常驻（JSON-RPC/WebSocket）
    每用户一个 profile（独立记忆/技能/会话隔离）
    会话映射表在 BFF（用户ID → profile → 会话ID）
```

### 容量阶梯（设计目标）

| 规模 | 部署形态 | 关键动作 |
|------|---------|---------|
| ≤50 活跃 | 本机 M4（现状演进） | hermes serve 本地 + 现有 JSON 存储 |
| ≤500 活跃 | 1 台云服务器 4C8G | serve 常驻 + SQLite + BFF 队列（每用户串行、全局并发上限） |
| ≤5000 活跃 | 2-3 节点 | BFF 无状态化 + 用户按 hash 路由节点 + Postgres |
| 10 万级 | 容器化集群 | Hermes 节点池 + 自动扩缩（**Adapter 接口不变，只换路由**） |

### 现在就要写进代码的 4 个架构件

1. **HermesAdapter 接口抽象**：`chat / continueSession / createSession / getState` 四个方法，
   BFF 业务层只依赖接口——今天接本地 serve，明天换远程节点池，业务零改动
2. **会话映射表**：用户ID → profile → 会话列表，进 DB 结构（现在 JSON 文件即可，字段先定对）
3. **BFF 队列**：每用户消息串行（LLM 生成慢，防自爆），全局并发上限（可配置）
4. **每用户一个 Hermes profile**：独立记忆/技能/会话（用户拍板"每个用户要有自己的记忆"的落地）

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

1. **动工时间**：方案 OK 就开工？（预计 3-5 个工作日，开发主力 + 我验收）
2. ~~层 1 vs 直接层 2~~ → **已定：直接层 2（JSON-RPC 长驻）**，CLI 仅作调试工具（多用户容量考量，2026-08-04 柯哥拍板方向）
3. **灰度范围**：先 xuanti 一个技能试水，还是全量切换？
4. **开发主力**：Claude（claude-sonnet-4）接手改造，还是我直接改？

> 补充确认（多用户架构）：≤50 用户阶段先本机跑通、字段/接口按 5000 用户的标准设计——是否认可这个"架构件先行、容量后置"的节奏？
