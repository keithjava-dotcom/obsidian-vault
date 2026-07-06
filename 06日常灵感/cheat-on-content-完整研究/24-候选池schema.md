---
author: 小龙
created: 2026-06-23
tags: [cheat-on-content, candidate, schema, 候选池]
---

# Candidate Schema — 候选池 Schema

被 cheat-trends、cheat-recommend、cheat-init 和所有 adapter 引用。

## 核心字段

| 字段 | 类型 | 说明 |
|------|------|------|
| id | string(12) | 稳定 hash: sha256(source+title+url)[:12] |
| title | string | 人类可读标题 |
| source | string | 标识：`<adapter-type>:<source-name>` |
| snapshot_text | string | 打分输入（adapter 必须把 URL 展开为可读文本） |
| snapshot_at | ISO 8601 | 抓取时间 |
| url | string | 原始链接（可选） |
| tier | enum | tier1/2/3/skip/risky/done |
| composite_score | float | rubric 综合分（null 表示未打分） |
| dimension_scores | object | 各维度整数分 |
| predicted_bucket | string | 粗预测桶 |
| predicted_reason | string | 一句话理由 |
| category | string | 分类标签 |
| note | string | 自由文本 |

## 去重算法

```python
def candidate_id(source, title, url=None):
    normalized_title = title.strip().lower().replace(" ", "")
    url_path = url.split("?")[0].rstrip("/") if url else ""
    raw = f"{source.split(':')[0]}|{normalized_title}|{url_path}"
    return hashlib.sha256(raw.encode()).hexdigest()[:12]
```

**特点**：
- source 取冒号前的 adapter type（不取具体 source name）——同一标题被 HN 和 Reddit 都抓到，视为同一候选
- title lowercase + 去空格
- URL 砍 query string

## 去重三重保护

1. candidates.md 已有此 id → 跳过
2. predictions/*.md 已发布 → 跳过
3. trends-history.jsonl 有 rejected_at → 6 个月内不推

## Markdown 表示

```markdown
### [tier1] 为什么我们都讨厌主动联系朋友
- **id**: a3f2c1d4e5b6
- **source**: pool:markdown-list
- **snapshot_at**: 2026-05-04
- **category**: 社交
- **composite (v0)**: 7.4 — ER=4 HP=4 QL=5 NA=3 AB=5 SR=3 SAT=3
- **predicted bucket**: 5-30w
```

## Tier 含义

| Tier | 含义 | 行动 |
|------|------|------|
| tier1 | 强候选 | 进推荐排序池 |
| tier2 | 中等备选 | 进排序池但权重低 |
| tier3 | 弱候选 | 不进推荐池 |
| skip | 用户跳过 | 6 个月不出现 |
| risky | 议题敏感 | 推荐时额外标注 |
| done | 已发布 | 移出候选池 |
