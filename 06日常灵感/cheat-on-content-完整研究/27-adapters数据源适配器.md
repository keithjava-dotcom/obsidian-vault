---
author: 小龙
created: 2026-06-23
tags: [cheat-on-content, adapters, 数据源, 平台适配]
---

# Adapters — 数据源适配器

适配器层负责从各平台抓取数据（复盘数据和热点数据），统一 normalize 为内部 schema。

## 复盘数据 Adapters（perf-data/）

用于 cheat-retro 的 T+N 天数据回收。

| Adapter | 平台 | 调用方式 | 需认证？ |
|---------|------|---------|---------|
| douyin-session | 抖音 | bash run.sh <aweme_id> <video_folder> | ✅ Playwright + 扫码登录 |
| xhs-explore | 小红书 | bash run.sh <note_id> <video_folder> | ✅ cookie（.auth-xhs/） |
| linkedin-session | LinkedIn | bash run.sh <activity_id> <video_folder> | ✅ cookie（.auth-linkedin/） |
| bilibili-stat | B 站 | bash run.sh <bvid> <video_folder> | ❌ 公开接口 |
| youtube-data-api | YouTube | planned — batch 3 | ✅ API key |

**douyin-session 特殊处理**：
- 短链 resolve 提取 aweme_id
- cookie 存在 .auth/ 目录
- 输出 renderer.py → report.md

**xhs-explore 特殊处理**：
- URL 含 xsec_token → 提取 note_id
- 字段已校准（view_count 等已写死）
- 评论可能因为 xsec_token 缺失抓不到 → 降级 manual

**linkedin-session 特殊处理**：
- 只能抓本人帖（LinkedIn 单帖分析仅作者可见）
- 评论只给数不给正文（分析页限制）→ 降级要求用户 manual 粘

**bilibili-stat 特殊处理**：
- B 站 video view / reply 都是公开接口，无 wbi 签名
- 纯 httpx，没有 login 步骤

## 热点数据 Adapters（trend-sources/）

用于 cheat-trends 的每日热点抓取。

| Adapter | 源 | 需配置？ |
|---------|----|---------|
| manual-paste | 用户粘贴 | ❌ 永远可用（兜底） |
| aihot | AI 热点聚合 | ❌ |
| weibo-hot | 微博热搜 | ❌ |
| zhihu-hot | 知乎热榜 | ❌ |
| trendradar-mcp | TrendRadar MCP | ✅ 配置 MCP 服务 |

**adapter 的契约**：
1. 只写 .auth/ 的 cookie 到用户项目根（不写到 skill 源码目录——v1.3 修复的漏洞）
2. 输出 normalize 到 candidate-schema
3. 单 adapter 失败 → skip 不抛异常

## 脚本提取 Adapters（script-extraction/）

用于 cheat-learn-from 的对标账号 script 提取（whisper 转录）。

- whisper-cpp（推荐，快）
- openai-whisper（Python，慢）
- 转录准确度不如手动粘文本
