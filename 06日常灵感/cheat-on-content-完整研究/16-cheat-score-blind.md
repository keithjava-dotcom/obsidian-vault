---
author: 小龙
created: 2026-06-23
tags: [cheat-on-content, cheat-score-blind, sub-agent, 污染隔离]
---

# cheat-score-blind — ⭐隔离打分 Sub-agent

## 定位

内部 sub-agent，只能由 cheat-score / cheat-predict / cheat-bump 通过 Task tool spawn。**Channel B 的核心**——一份未受污染的打分 anchor。

**⚠️ 不是用户可直接调用的 skill**。用户在主对话调它没有意义——主对话已经被污染了。

## 为什么需要隔离

Chanel scoring 的污染路径：

- 用户对话历史（含偶然提到的播放数/评论/情绪）
- 已发布作品的实绩数据
- 历史 predictions/*.md 含复盘段
- 用户的赞美/抱怨/期待

**在 cheat-bump Phase 2 校准池重打分时最严重**——主 Claude 知道每条实绩才回追 TN/CC 分，rank 一致性可能 overfit 不是真信号。

## 白名单（只读这两个文件）

**唯一允许读的两个文件**：
1. `scripts/<id>.md`（pre-shoot 草稿）
2. `rubric_notes.md`（评分公式 + 维度定义）

**硬拒绝读**（即使主 Claude 在 Task prompt 里手滑塞进来）：

| 路径 | 为什么禁 | refusal_code |
|------|---------|-------------|
| .cheat-state.json | 含 calibration_samples/pending_retros 等后视数据 | blocked_contaminated_input |
| predictions/*.md | 含复盘段 = 实绩 | blocked_contaminated_input |
| videos/*/report.md | T+3d 真实数据 | blocked_contaminated_input |
| rubric-memo.md | **最大泄漏入口**（PR#11 实测）——含真实视频名+实绩 | blocked_rubric_memo |
| audience.md | 含评论派生的实绩信号 | blocked_audience |
| 任何含"播放/万/w/k/M"的文件 | 直接污染 | blocked_contaminated_input |

## 输出格式（严格 JSON）

```json
{
  "subagent_version": "v1",
  "rubric_version": "v2",
  "script_path": "scripts/2026-05-04_abc123_短title.md",
  "script_hash": "<sha256:12>",
  "scored_at": "<ISO 8601>",
  "dimensions": {
    "ER": { "score": 4, "confidence": "high", "reason": "PPT加油猫猫开头—具象画面" },
    "SR": { "score": 3, "confidence": "medium", "reason": "AI焦虑是议题但非热点对峙" },
    "HP": { "score": 5, "confidence": "high", "reason": "首句具象反差" },
    "QL": { "score": 5, "confidence": "high", "reason": "双关金句" },
    "NA": { "score": 4, "confidence": "medium", "reason": "单线反思+收束" },
    "AB": { "score": 4, "confidence": "medium", "reason": "一人公司但AI焦虑普适" },
    "SAT": { "score": 2, "confidence": "high", "reason": "共情调，几乎无讽刺" }
  },
  "input_status": { "rubric_notes_read": true, "script_read": true, "any_other_file_read": false },
  "self_check": { "any_contamination_signal": false },
  "refusal": null
}
```

## 自检 contamination

读完 rubric_notes.md 后必跑 grep 排查实绩数字/播放/万等 pattern。命中则：

- `self_check.any_contamination_signal: true`
- `refusal: "non_blind_warning"`
- 所有维度 confidence 降 medium
- 违禁 snippet 摘抄进 contamination_note

仍输出 dimensions——拒绝输出比误判更糟，但要诚实标注。

## 子 agent 的行为约束

- 不要 Read benchmark.md（Channel A context）
- 不要 Glob predictions/（污染源）
- 不要 Read .cheat-state.json（不需要知道主 Claude 跑了多少篇）
- 如果 Task prompt 漏传了某条路径 → 主动问"我只允许读 script + rubric_notes，缺哪个？"

## 主 Claude 调用契约

Task prompt 只含：

```
Spawn cheat-score-blind sub-agent.
Input:
 script_path: scripts/<id>.md
 rubric_notes_path: rubric_notes.md

Task: 按 rubric_notes 当前公式打分。返回严格 JSON。
不要读 state file / predictions/ / videos/ 任何其他文件。
不要询问用户。
```

**禁止塞进 Task prompt**：用户对话引用、前一次预测、实际播放数、任何含"万/w/k/M"字符串。

调用前 grep 自检：echo prompt | grep -Ei '播放|阅读|点赞|评论数|实际|retro|复盘|实绩|w$|万$' 命中 → 改 prompt 重发。

## 关键设计洞见

1. **污染诚实标注**——不假装隔离完美。接受"sub-agent 仍是 Claude，RLHF prior 共享"和"rubric 用户自己写的自然偏自己内容"两个残余
2. **拒绝但要输出**——contamination 时仍输出 dimensions（只是降 confidence）——不让用户陷入"不知道出了什么问题"的状态
3. **rubric_notes.md leak guard**——写完 rubric_notes.md 后 grep 自检，发现实绩 pattern → abort + 回滚

## 对我们体系的价值：P1

OpenClaw 的 sessions_spawn 可以实现类似 Channel B 隔离：
- fanwen 写完 → spawn sub-agent 只收文本+rubric → 纯 JSON 打分回传
