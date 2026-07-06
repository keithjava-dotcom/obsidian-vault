---
author: 小龙
created: 2026-06-23
tags: [cheat-on-content, cheat-trends, 热点, 候选池]
---

# cheat-trends — 热点抓取

## 定位

从配置的热点源抓今天的热门话题 → 去重 + 粗打分 + 写入候选池。**让"我没素材"问题在 onboarding 第二步就消失**。

## 工作流

```
[用户：抓热点]
→ Phase 0: 读 state 拿 enabled adapters
→ Phase 1: 对每个 adapter 调 fetch
→ Phase 2: normalize 到 candidate-schema
→ Phase 3: 去重（vs candidates / predictions / trends-history）
→ Phase 4: 对每个新 item 粗打分
→ Phase 5: 排序 + 询问用户哪些加入 candidates.md
→ Phase 6: 写入 + 更新 trends-history.jsonl 缓存
```

## 热点源（Adapter 模式）

| Adapter | 实现 | 需配置 |
|---------|------|--------|
| manual-paste（兜底） | 用户粘贴 URL/标题 | ❌ |
| aihot | AI 热点聚合 | ❌ |
| weibo-hot | 微博热搜 | ❌ |
| zhihu-hot | 知乎热榜 | ❌ |
| trendradar-mcp | TrendRadar MCP 服务 | ✅ 配置 |

## 核心设计：去重三重保护

1. 检查 candidates.md 已有此 id → 跳过
2. 检查 predictions/*.md 已发布 → 跳过
3. 检查 trends-history.jsonl 已拒绝过（用户说"none"）→ 6 个月内不重复推

## 核心设计：粗打分

AUTO_SCORE=true 时，对每条新 item 按当前 rubric 打 7 维分 + 算 composite。`MIN_COMPOSITE_TO_SUGGEST = 6.0`，低于此分不推荐用户加入候选池。

## 核心设计：用户选择权

排序后展示 → 用户说"all"全部加 / "1,3,5"选 / "none"都不要。选中的写进 candidates.md，不要的记录到 trends-history 避免下次重复推。

## 关键设计洞见

1. **manual-paste 永远是兜底**——即使其他所有 adapter 都坏了
2. **用户拒绝过的 6 个月不重复推**——防厌烦
3. **粗打分诚实标注**——composite (rough, snapshot-based) vs prediction 的精打分区分
4. **单 adapter 失败 → skip**，不抛异常
