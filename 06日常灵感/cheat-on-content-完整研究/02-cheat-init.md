---
author: 小龙
created: 2026-06-23
tags: [cheat-on-content, cheat-init, 初始化, onboarding]
---

# cheat-init — 初始化 + 脚手架

## 定位

用户入口，首次使用必跑。6 个问题 + 对标账号建议 + 项目骨架创建。

## 工作流

```
[用户说"初始化"]
→ Phase 0: 检测当前状态（已有 state file？半初始化？）
→ Phase 1: 首屏文案（适用性 + 期望管理）
→ Phase 2: 6 个问题（Q1-Q5 都问；Q2 决定是否 import 历史）
→ Phase 2.5: 对标账号（cold-start 必须问，已发用户可选）
→ Phase 3: 创建骨架（scripts/videos/samples/+模板）
→ Phase 3.5: 历史导入（仅 Q2=有发过+用户同意时）
→ Phase 4: 测试 hook 是否生效
→ Phase 5: 给"下一步该说什么"清单
```

## 6 个问题

### Q1: 内容形态
a) 观点视频 → 匹配内置 rubric
b) 长文 essay
c) 短文/thread
d) 播客/长视频
e) 教程/工具教学
f) 其他（游戏/美食/妆教）
g) 混合

→ 记录 content_form + rubric_form_mismatch（非观点视频标 true，cheat-status 持续提示需要 bump）

### Q1.5: 典型时长（仅视频形态问）
30s / 1-3min / 3-5min（推荐起步）/ 5-10min / 10min+

### Q1.6: 发布频率
日更 / 隔日 / 每周 / 灵活（关闭 buffer 监控）

### Q2: 发过视频吗？
a) 没发过 → calibration_samples=0，纯 brainstorm
b) 发过 → 问平台 + 抓取方式 → 导入历史作为 baseline

### Q3: 数据回收方式
a) 手动粘（必须粘 top 20+ 评论）
b) adapter 自动抓（推荐默认）

### Q4: 候选选题
a) 没有 → 后续/cheat-seed 或/cheat-trends
b) 有 markdown 列表
c) Notion 等其他

### Q5: 装 hook
预测锁 + SessionStart 自动报告 + 静默日志

## 对标账号（Phase 2.5）

cold-start 强烈建议——不找对标，前 5 篇精度 ±50%

## 创建的文件

- .gitignore（护凭证）
- .cheat-state.json（共享状态）
- rubric_notes.md（评分规则）
- scripts/ / videos/ / samples/ 空目录
- benchmark.md / audience.md 骨架

## 关键设计洞见

1. **所有用户都走统一 5 阶段闭环**，唯一区别是有无历史数据
2. **期望管理前置**："早期预测会不准——前 5 篇精度大概 ±50%，这是数学事实。工具用 🔴🟠🟡🟢🔵 标 confidence 等级"
3. **不再因为内容形态拒绝执行**——rubric_form_mismatch 标真，让用户自己在使用中调
4. **gitignore 第一个创建**——防凭证泄漏

## 和我们的定位报告流程

我们的 dingwei-report-workflow（15 模块）比 cheat-init 深得多。但 cheat-init 的"内容形态检测""对标账号导入""期望管理前置"三个设计值得吃进来。
