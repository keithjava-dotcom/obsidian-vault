---
author: 小龙
created: 2026-06-23
tags: [cheat-on-content, templates, 模板]
---

# Templates — 模板文件完整清单

原项目共 13 个模板，由 cheat-init 在创建脚手架时复制到用户项目。

## 模板清单

| 模板 | 用途 | 我是否写了独立文件 |
|------|------|-------------------|
| **prediction.template.md** | 预测日志完整 7 组件模板 | ✅ 29-预测文件模板 |
| retro.template.md | 复盘段格式模板 | ⬜ 内置在 09-cheat-retro.md |
| rubric_notes.template.md | rubric_notes.md 骨架 | ⬜ 内置在 19+28+33 |
| rubric-memo.template.md | 升级 Memo 积累档案 | ⬜ bump 验证协议里提到 |
| **benchmark.template.md** | 对标账号信息 | ⬜ 未单独写 |
| audience.template.md | 受众画像骨架 | ⬜ 内置在 10-cheat-persona.md |
| script_patterns.template.md | 写作模式沉淀 | ⬜ 内容在 03-cheat-learn-from.md |
| **candidates.template.md** | 候选池条目 | ✅ 24-候选池schema |
| candidates.template.json | JSON 版候选池 | ⬜ 与 24 同内容不同格式 |
| **status.template.md** | 看板模板 | ⬜ 内置在 14-cheat-status.md |
| **workflow.template.md** | 5 阶段流程文档 | ⬜ 00-总览覆盖 |
| **gitignore.template** | 护凭证 | ⬜ 02-cheat-init.md 提到 |
| content.db.schema.sql | SQLite 升级（batch 3） | ⬜ SQL 文件未做 |

## 核心模板结构

### benchmark.template.md 结构
```markdown
# benchmark: [对标账号名]

## 基础信息
- URL: [链接]
- 平台: [douyin/bilibili/...]
- 粉丝量级: [Nw/Nk]
- 调性: [一句话]

## 导入样本
| # | 标题 | 形态 | 时长 | 用户印象 | 播放 | 点赞 | 评论 | 转发 |
|---|---|---|---|---|---|---|---|---|

## 脚本模式
所有来自对标的 pattern 标 **Imported, untested**。

## 选题方向感
[主题分布 + 调性分析]

## Rubric 初始信号（仅定性）
[基于样本的维度信号]
```

### script_patterns.template.md 结构
```markdown
# Script Patterns

> 这是你的写作 pattern 沉淀库。每次复盘发现新 pattern，追加到这里。
> 对标账号的 pattern 标 "Imported, untested"；自己的 pattern 有 ≥2 次复盘验证后升正。

## 🧭 Cheat Sheet（写稿前扫一眼）
[骨架分段：开头 / 主体 / 收尾 / 风格]

## 核心 Pattern
### Pattern 1: [名称]
- **适用**: [哪种脚本]
- **钩子类型**: [场景代入/IS/数据反转]
- **结构**: [几段 + 怎么切]
- **证据**: [样本 X / Y，N 样本]

## 用户改稿历史观察
| 日期 | 视频 | 用户改了什么 | 流量影响 | 是否升pattern |
|---|---|---|---|---|

## 新发现的 Pattern
标 "≥N 样本待验证"
```

## 模板设计原则

1. **每个模板都带写前扫一眼指南**——降低"用户打开空白文件不知写什么"的启动成本
2. **骨架强制包含关键字段**——但留出弹性空间（用户可加自己的）
3. **cold-start 期望管理**——每份模板的首屏说明段解释"你现在在哪个阶段"

## 关于 examples/

原项目含 `examples/script_patterns.example.md`（参考实现例子）。
未公开的参考实现（reference-implementation 项目）是 v1→v2 升级的完整实录，约 30 篇已发作品的校准数据。这个不属于公开代码，没有写独立的文件。
