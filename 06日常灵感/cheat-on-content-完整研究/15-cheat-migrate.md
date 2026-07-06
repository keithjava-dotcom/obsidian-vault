---
author: 小龙
created: 2026-06-23
tags: [cheat-on-content, cheat-migrate, schema, 迁移]
---

# cheat-migrate — Schema 迁移

## 定位

把老用户的 .cheat-state.json 升级到当前 schema 版本。**幂等、不跳版、失败停在原地、备份硬约束**。

## 工作流

```
[用户：迁移 / SessionStart 提示后跑]
→ Phase 0: 读 state + migrations/registry.md → 确定迁移链
→ Phase 1: dry-run（默认展示迁移计划，等确认）
→ Phase 2: 备份 .cheat-state.json → .backup-<timestamp>
→ Phase 3: 按顺序对每个 step 应用对应迁移文件
→ Phase 4: 验证升级后 state 能被解析 + schema_version 已更新
→ Phase 5: 报告 + 提示备份清理
```

## 核心设计

### 幂等
在已升过的 state 上重跑 → 立刻退出"无需迁移"。不重复应用步骤。靠 current_version == target 实现。

### 不跳版
1.0 → 1.3 必须按 1.0→1.1→1.2→1.3 顺序。不允许"直接升 1.0→1.3 的合并 migration"。

### 失败停在原地
第 N 步失败时 schema_version 停在 N-1 已成功的版本，不回滚到迁移前。重跑能从断点继续。

### 备份是硬约束
写前必有备份。即使 --dry-run: false，备份仍执行。

### 不降级
schema 演进单向。降级请手动 cp 历史 git 快照。

## 迁移文件格式

每个 migrations/<from>-to-<to>.md：

```
## WHAT
...（版本间变化）

## WHY
...（为什么这么改）

## HOW (Claude steps for /cheat-migrate)
1. ...
2. ...
3. ...

## Manual fallback
...（如果自动化失败怎么办）
```

迁移是 Claude 读 markdown 跑的——不是 python 脚本。这是关键设计：迁移是自然语言步骤。

## 和我们的体系

我们目前的 state 管理没有这么复杂的 schema 版本机制。但经验值得借鉴——所有 schema 变更都要有迁移文件 + 回滚能力。
