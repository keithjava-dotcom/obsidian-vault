---
author: 小龙
created: 2026-06-23
tags: [cheat-on-content, state, schema, 状态管理]
---

# State Management — 状态管理 Schema

## 核心原则

- .cheat-state.json 是各子 skill 共享上下文的**单一来源**
- 每个内容项目一个 state 文件（不放到全局 ~/.claude/）
- 所有 skill 用 state.get(field, default) 模式读取

## 完整 Schema

```json
{
  "schema_version": "1.4",
  "skill_version": "1.0.0",

  "rubric_version": "v0",
  "content_form": "opinion-video",
  "typical_duration_seconds": 240,
  "target_publish_cadence_days": 2,
  "rubric_form_mismatch": false,

  "benchmark_status": "none",
  "benchmark_name": null,
  "benchmark_sample_count": 0,
  "baseline_plays": null,

  "calibration_samples": 0,
  "calibration_samples_at_last_bump": 0,

  "data_collection": "manual",
  "pool_status": "none",
  "data_layer": "markdown",

  "hooks_installed": false,
  "enabled_trend_sources": ["manual-paste"],
  "enabled_perf_adapters": [],

  "last_bump_at": null,
  "last_bump_self_audited": false,
  "last_published_at": null,
  "last_published_file": null,
  "last_retro_at": null,
  "last_trends_run_at": null,
  "last_trends_added_count": 0,
  "last_prediction_self_scored": false,
  "last_self_scored_at": null,

  "consecutive_directional_errors": [],
  "pending_retros": [],
  "shoots": [],

  "in_progress_session": null,

  "initialized_at": "2026-05-04T15:00:00+08:00"
}
```

## 字段分类

### 配置类（cheat-init 写入）
- schema_version, skill_version, rubric_version, content_form
- typical_duration_seconds, target_publish_cadence_days
- rubric_form_mismatch, benchmark_status/name/count
- baseline_plays, data_collection, pool_status, data_layer
- hooks_installed
- enabled_trend_sources, enabled_perf_adapters
- initialized_at

### 校准统计类（复盘/bump 写入）
- calibration_samples（每次复盘+1）
- calibration_samples_at_last_bump（bump 时写）

### 时间戳类（各 skill 写入）
- last_bump_at, last_published_at, last_retro_at
- last_trends_run_at, last_trends_added_count
- last_prediction_self_scored, last_self_scored_at
- last_bump_self_audited

### 队列类（拍/发/复盘维护）
- consecutive_directional_errors: ["high","low"...]
- pending_retros: ["predictions/file1.md", ...]
- shoots: [{video_folder, prediction_file, shot_at, ...}]

### 会话类
- in_progress_session: {type, file, started_at} / null

## Schema 升级历史

### v1.3 → v1.4（MINOR but BREAKING）
拆 rubric 文件：rubric_notes.md（blind 白名单）→ rubric-memo.md（升级 Memo 含实绩）
state 字段不变，仅 schema_version bump 标识迁移需求

### v1.2 → v1.3（MINOR，兼容）
新增 last_prediction_self_scored, last_self_scored_at
shoots[] 扩展（scripts_path, script_consistency, v2_prediction_written 等）

### v1.0 → v1.1（MINOR，兼容）
基于实战删除冗余字段：mode, prediction_complexity, bucket_scheme

## 读 state 的铁律

```python
# ✅ 好
benchmark = state.get("benchmark_status", "none")
shoots = state.get("shoots", [])

# ❌ 坏（老 state 缺字段会 KeyError）
benchmark = state["benchmark_status"]
```

例外：schema_version 和 rubric_version 允许直接索引——缺失意味着 state 不合法。
