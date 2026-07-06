---
author: 小龙
created: 2026-06-23
tags: [cheat-on-content, cheat-publish, 发布, buffer]
---

# cheat-publish — 发布登记

## 定位

把作品的发布元数据写入预测文件 header + 更新 state。**只动 metadata，不动预测段**。

## 工作流

```
[用户：已发布 https://...]
→ Phase 0: 找到对应的预测文件（通过 in_progress_session 或匹配）
→ Phase 1: 解析 URL → 平台/发布时间
→ Phase 2: 更新 prediction 文件 header（仅 metadata 段）
→ Phase 3: 更新 .cheat-state.json，清除 in_progress_session
```

## 核心设计

### 自动平台识别

从 URL 模式自动匹配：douyin/bilibili/youtube/xhs/wechat/substack/twitter 等。未知 → 询问用户。

### Buffer 出队

从 state.shoots[] 移除发布项（buffer -1）。如没找到 → 警告"跳过 cheat-shoot？"——不阻塞但提示。

### Pending retros 自动入队

发布后自动把这条加进 pending_retros 列表。cheat-status 基于此 + RETRO_WINDOW_DAYS 显示"今天该复盘哪些"。

### 诚信声明

```
⚠️ 从此刻起，你看到任何关于这条作品的播放/点赞/评论数据
   都会破坏盲度声明的诚信。如果不小心看到，告诉我——
   我会在文件里追加一个 integrity warning。
```

## 写入的 metadata

```
Published at: 2026-05-04T14:32:00+08:00
Platform: douyin
URL: https://v.douyin.com/abc123
Video Folder: videos/2026-05-04_a3f2c1d4_停止期待/
Aweme ID: 7234567890123456789（平台特定 ID）
```

## 约束

- ❌ 不动预测段（即使是修复笔误）
- ❌ 不抓数据（发布是登记，不是数据回收）
- ✅ 发布 URL 可后续补
- ✅ 重复登记需明示

## 对我们体系

轻量功能，但 pending_retros 自动排队值得吃进 fupan-workflow。
