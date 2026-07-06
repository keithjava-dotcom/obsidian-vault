---
author: 小龙
created: 2026-06-23
tags: [cheat-on-content, cheat-shoot, 拍摄, buffer]
---

# cheat-shoot — 拍摄登记

## 定位

把视频从"已写预测、未拍摄"状态推进到"已拍摄、未发布"状态。**Barely 一个独立 skill，但设计精妙**。

## 工作流

```
[用户：拍了 scripts/<id>.md]
→ Phase 0: 解析路径 + 验证 prediction 已存在
→ Phase 1: 检查是否已登记（防重复）
→ Phase 2: 建 videos/<id>/ + 询问"实际拍摄稿一致吗？"
→ Phase 3: 写 videos/<id>/script.md
→ Phase 4: append state.shoots（buffer +1）
→ Phase 5: 输出 buffer 状态
```

## 核心设计：拍 vs 发分两个动作

buffer 警戒系统需要明确区分"拍了"vs"发了"。视频可以批量拍（一天拍 5 条），分散发（每天发 1 条）。

**shoot**: buffer +1 → **publish**: buffer -1

## 核心设计：改稿检测 → v2 重判

询问实际拍摄稿是否与 scripts/<id>.md 一致：
- **一致** → 按草稿拍的，不重判
- **改了一些** → diff 检测 vs V2_TRIGGER_THRESHOLD（30%）
  - 超阈值 → 默认建议 delegate 到 cheat-predict 写 v2 预测
  - 低于阈值 → 询问是否仍要 v2
- **大改了** → 走 _redo 流程

**diff 检测技术**：char-level Levenshtein（normalize 后）。修复了 line-level diff 在口语化转录场景的误报——draft 是 markdown 长句、拍摄稿是 whisper 转录短断句，line-level 算出 ~200% diff 但内容几乎不变。

## 核心设计：v1 vs v2 差异是 rubric 升级信号

v1 给 ER=4，v2 给 ER=5（用户改稿改高了 hook 强度）
→ 告诉 rubric "这个用户的 ER 阈值跟我当前公式不一致"

## 和我们体系关联

b-roll-plan-workflow 可以吸收"拍前预测 vs 拍后改稿"的 diff 检测来收集数据。
