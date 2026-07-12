---
author: Claude
created: 2026-07-09
updated: 2026-07-09
type: 施工单
tags: [帧言, 第3周, 功能完善, Codex派单]
---

# 第 3 周施工单 · Codex 派单(v2 · 功能完善周)

> **v1(支付周)已作废**。柯哥 2026-07-09 中午定调"支付放最后,先把功能完善好"+补出两个新方向:
> 1. **创作者社群圈子**(严格质量管控+让更多人参与)
> 2. **学生分销佣金**(给学生卖,卖了拿佣金——睡后收入第二条腿)
>
> 支付整体推到**第 5 周**。第 3 周(现在)+第 4 周(上云)不动周次,只改主题。
>
> 我是架构师(Claude),Codex 是工程小弟。这份单每单 < 300 行代码 + 有真跑验收,直接派给 Codex 执行。
>
> 回总纲: [[00-产品总纲]]

---

## 一、施工总览(7 单 + 2 待柯哥拍板)

| 编号 | 单名 | 工作量 | 依赖柯哥 | 优先 |
|---|---|---|---|---|
| **F2** | 五棒 fallback 前端徽标(可观测收尾) | 半天 | 无 | 🔴 立刻派 |
| **F6a** | 找回密码接真 SMTP | 半天 | SMTP 账号 | 🟡 |
| **F6b** | 操盘手真搜索(替换 mock) | 半天 | 无 | 🟡 |
| **F1** | 数字柯建军超记忆·基础增强 | 2-3 天 | 我先探路再出细单 | 🔴 主线 |
| **F4** | 学生分销地基(referrer 追踪) | 1-2 天 | 分销规则拍板 | 🔴 主线 |
| **F3** | 创作者社群 MVP | 3-4 天 | 社群形态拍板 | 🔴 主线 |
| **F7** | 后台管理系统 MVP | 2 天 | 无 | 🟡 兜底 |

**总工期估**: 10-13 工时天,配 3-4 天并行(Codex 单线跑,Claude 拆单+验收),够第 3 周吃下核心 4 单(F2/F1/F4/F3),F6/F7 落在第 4 周头。

---

## 二、Codex 通用红线(所有单子)

跟 v1 相同,不重复。摘要:数据隔离 / 不做假活 / 可观测 / 测试自证 / 敏感值只经环境变量 / 只动指定文件 / 不谎报。详见 v1 备份(git 里)。

---

## 三、单子详情

### 🎯 F2 · 五棒 fallback 前端徽标 · **立刻派**

**为什么先做**: 今晌后端已经在 payload 加了 `generatedBy: 'deepseek-v4-flash|pro|fallback'` 字段,前端半天收尾就能让柯哥一眼看出"这条是不是真 AI"。

**动的文件**(前端 only):
- `src/components/skill/FySkillPanelBase.tsx`(五棒共用面板底座): 读 `session.currentSkillPayload?.generatedBy` 或 `session.artifacts?.<stage>?.generatedBy`,在 Panel 顶栏渲染徽标:
  - `deepseek-v4-flash` → 🟢 "AI · flash" 灰底
  - `deepseek-v4-pro` → 🟢 "AI · pro" 金底
  - `fallback` → 🔴 "⚠️ 模板兜底,非 AI 生成" 红底
- `src/components/skill/FyXuantiPanel.tsx`(独立实现的选题棒): 同上,读 `stepOutputs.diverge_topics.generatedBy`
- 类型: `src/state/runnerStore.ts` 或 `src/services/skillService.ts` 里加 `LLMSourceTag = 'deepseek-v4-flash' | 'deepseek-v4-pro' | 'fallback'`

**验收**:
- 真跑一遍五棒(用 admin 账号),截图五棒 Panel 顶栏都有徽标
- 手工把 `bff/.env` 里 `DEEPSEEK_API_KEY` 改成假值,重跑一遍,截图五棒全打红角标
- vitest: 加 `src/components/skill/__tests__/FyGeneratedByBadge.test.tsx`,测三种 tag 渲染正确

**柯哥不用管**: 全自动。

---

### 📧 F6a · 找回密码接真 SMTP

内容跟 v1 的 S3-4 相同。已就绪,等柯哥给 SMTP 账号就派。

---

### 🔍 F6b · 操盘手真搜索

**现状**: `src/components/SettingsModal.tsx:8-49` 里 `MOCK_OPERATORS` 硬编码 3 个假操盘手,`operatorId` 绑定假 ID。真操盘手已经能注册(现代 auth `role='operator'`),存在 `bff/data/users/`,只是前端没接。

**动的文件**:
- 新增 `bff/src/routes/operators.ts`: `GET /api/operators/search?q=<name|email>` 挂 `requireAuth` → 扫 `bff/data/users/` 找 role='operator' 且 nickname/email match 的用户 → 返 `[{userId, nickname, email, avatar}]`
- `src/services/frameyanApi.ts` 加 `searchOperators(query)` 方法
- `src/components/SettingsModal.tsx` 删除 `MOCK_OPERATORS`,改成从 API 拉+搜索框
- `bff/src/routes/auth.ts:165 /api/auth/search-operator` 已有雏形,可以复用或替换

**验收**:
- 用真操盘手 signup 一个测试账号,前端能在下拉搜到
- vitest: `bff/tests/operators-search.test.ts`

**柯哥不用管**。

---

### 🧠 F1 · 数字柯建军超记忆·基础增强 · **主线**

**为什么核心**: 三必杀技第一条,柯哥最在意,别人抄不走的护城河。

**探路结论(2026-07-09 Explore agent)**: 地基完备(存储+5棒4棒写入+prompt注入+用户隔离),但缺 3 块承重墙:

| 缺口 | 后果 | 严重度 |
|---|---|---|
| ① memory 全按 userId 分区,**没有 accountId 维度** | 柯哥自己肥柯+芝加哥顾问两账号 memory 混一个池子,选肥柯选题时 AI 可能带上芝加哥的定位——**立刻串味** | 🔴🔴🔴 |
| ② 每棒重复写 + 无去重 + 无淘汰 | 同 kernel 反复追加,几周后 jsonl 膨胀成检索垃圾场,top-8 名额被重复项挤占 | 🔴🔴 |
| ③ 前端零 UI | 用户看不到 AI 记住了什么、无法纠错、无法遗忘;卖点讲不出,合规也堪忧 | 🔴🔴 |

**拆成 3 单派 Codex(独立可并行)**:

---

#### F1-A · memory 加 accountId 分区(防串味)· 半天

**动的文件**:
- `bff/src/repo/userMemory.ts`:
  - `UserMemoryEntry` 加 `accountId?: string` 字段
  - 存储路径改 `memories/<userId>/<accountId>/memory.jsonl`(物理隔离,防误查)
  - `retrieveMemories` options 加 `accountId?: string`,作为**硬过滤**
  - `appendUserMemory` 强制要求 accountId(缺省 `"__legacy__"`)
- `bff/src/skill-executor/engine.ts`:
  - `rememberCreativeAction` 从 `session.artifacts.kernelLock?.targetAccount || session.inputs.targetAccount` 取 accountId 写入(6 处调用点)
  - `buildUserMemoryPromptSection` 检索时传当前 session 的 accountId
- `bff/scripts/migrate-memory-account.mjs`: 老数据迁移脚本(把 `bff/data/memories/<uid>/memory.jsonl` 移到 `<uid>/__legacy__/memory.jsonl`)

**验收**:
- 加 vitest case: userA 有 accountId=fitness 和 accountId=consulting,写 fitness 的 kernel,检索 consulting 返回空
- 手工:柯哥账号从肥柯切到芝加哥顾问,五棒 prompt 里的记忆段确认不带跨账号内容
- `test:bff-unit` 全绿

---

#### F1-B · 写入去重+滚动上限+token 预算(防膨胀)· 1 天

**动的文件**:
- `bff/src/repo/userMemory.ts`:
  - `appendUserMemory` 前置去重:同 `userId+accountId+type` 下若最近 24h 或同 sourceSessionId 已有相似 summary(`summary.slice(0,80)` 命中),改为**替换**(rewrite jsonl)
  - 新增 `pruneUserMemories(userId, accountId, { maxEntries: 500, pinnedTypes: ['kernel'] })`: kernel 永久保留,其他类型只留最近 500 条;`appendUserMemory` 内每写 50 条低频触发一次
  - `formatMemoriesForPrompt` 加 token 预算参数(默认 1500 token),超出时对每条 summary 从尾部裁剪到 200 字,仍超就丢弃低分条目

**验收**:
- vitest: 写 100 条同 kernel → 最终文件里只有 1 条;写 600 条 topic → prune 后剩 500 条 + kernel 全留
- 手工:清一次柯哥的 memory,跑 30 轮五棒,`memory.jsonl` 行数 < 200

---

#### F1-C · 路由+前端「我的军师档案」页(把护城河变成卖点)· 1-1.5 天

**动的文件**:
- **BFF**:
  - `bff/src/repo/userMemory.ts` 扩展 `pinned?: boolean` 字段 + `deleteMemory(userId, accountId, id)` + `updateMemory(userId, accountId, id, patch)`
  - 新增 `bff/src/routes/memory.ts`,挂 `requireAuth`:
    - `GET /api/memories?type=&accountId=&page=`: 分页列表
    - `DELETE /api/memories/:id`: 单条删除(rewrite jsonl 或 tombstone)
    - `POST /api/memories`: 用户手动补一条(type/summary/tags 白名单)
    - `PATCH /api/memories/:id`: `{pinned: true}` 配合 F1-B 的 pin 保留
- **前端**:
  - 新增 `src/pages/MyMemoryProfile.tsx`,路由 `#/my-memory`
  - 按 type 分栏(灵感/内核/选题/范文/复盘),支持删除 / 置顶 / 手动补内核
  - 每条展示 `createdAt` 和 `sourceSessionId`,让用户能溯源
  - 合规文案:"这些是数字柯建军记住的关于你的东西,你随时可以遗忘"
  - Sidebar 加入口:「我的军师档案」

**验收**:
- vitest: 路由 auth 隔离(userA 不能删 userB 的记忆);pin 保留在 prune 后仍存在
- 手工:柯哥登录 → 点「我的军师档案」→ 看到自己历史 → 删除一条 → 置顶一条 kernel → 后续五棒真读到 pinned 内容
- iOS 真机跑一遍(响应式)

**柯哥不用管细节**,派单顺序建议:**F1-A → F1-B → F1-C**(A 是防串味最紧急,B 挡膨胀基础,C 上体验)。

---

### 💰 F4 · 学生分销地基 · **主线** · 拍板已到位(2026-07-09)

**5 条规则(柯哥 2026-07-09 拍板,不改)**:
- Q4 **1 级分销**——学生→用户,不允许多级;referrer_id 不再向上追溯,循环 referrer 拒绝 signup
- Q5 **一次性 20%**——主套餐 ¥4980 → 学生拿 ¥996/单;Pro/企业订单**不给学生分销佣金**(等柯哥单独拍板)
- Q6 **T+7 观察期**——`mature_at = paid_at + 7d`,提现 API 只允 `now > mature_at`;退款窗口内取消 → 佣金作废(commission_events 状态回 CANCELED)
- Q7 **只柯哥课程学员**——admin 手工标记 `is_distributor: true`,普通 PRO 用户无邀请链接生成入口
- Q8 **签电子协议**——首次生成邀请链接前弹协议签署对话框;协议存 IP+时间戳+版本;协议改版要重新签

**动的文件**:
- **BFF**:
  - `bff/src/routes/auth.ts` StoredUser 加 `referrer_id?: string` + `is_distributor?: boolean` + `distributor_marked_by?: string`;signup 接受 `body.referrer_id`,检查循环拒绝
  - 新增 `bff/src/repo/distribution.ts`: `appendCommissionEvent` + `listMyCommissions` + `markAsMature`(cron 触发 mature_at 到期)
  - 新增 `bff/src/routes/distribution.ts`,挂 `requireAuth`:
    - `GET /api/distribution/me`: 我拉了多少人、多少已 mature、多少 pending
    - `POST /api/distribution/link`: 生成邀请链接(前置检查 is_distributor + 协议已签,否则 403)
    - `POST /api/distribution/agreement/sign`: 签协议(写 `bff/data/distributor-agreements/<userId>.json`,含 IP/时间戳/version)
    - `POST /api/distribution/withdraw`: 提现申请(只允已 mature 的佣金)
  - 新增 `bff/src/routes/admin/distributors.ts`,挂 `requireAdminUser`:
    - `POST /api/admin/distributors/:userId/mark`: 标记 is_distributor + audit log
    - `GET /api/admin/distributors`: 分销员列表
- **前端**:
  - `src/pages/AuthPage.tsx` signup 读 query `?ref=<referrer_id>` 带上
  - 新增 `src/pages/DistributorDashboard.tsx`(路由 `#/distributor`): 邀请数/成交/待 mature/已可提现;侧栏入口仅 is_distributor 用户可见
  - 首次点"生成邀请链接"弹协议对话框(带滚动到底才能签)

**验收**:
- vitest `bff/tests/distribution.test.ts`:
  - 未标记为 distributor 的用户调 `POST /distribution/link` → 403
  - 已标记 + 未签协议 → 403 提示签协议
  - 已签协议 → 返回邀请链接
  - 循环 referrer (A→B, B→A) signup 拒绝
  - 二级 referrer (A→B, B 想标 C 为 B 的下线) 拒绝
  - mature_at 未到不能提现
  - 退款钩子把 CommissionEvent 状态改成 CANCELED
- 手工:柯哥标记一个测试账号为 distributor → 签协议 → 生成链接 → 用另一账号通过链接注册 → 后端记录成功

**Codex 派单模板**(F4):

```
你是帧言项目的 Codex 工程师。项目在 /Users/kejianjun/Documents/柯哥Openclaw项目/staging/frameyan-ui。

单号: F4 (学生分销地基)

规则(柯哥 2026-07-09 拍板,不许改也不许猜):
- 1 级分销(不允许多级链)
- 一次性 20% 佣金,仅主套餐 ¥4980 生效,Pro/企业不给
- T+7 观察期后可提现;退款窗口内取消 → 佣金作废
- 只柯哥课程学员(admin 手工 mark is_distributor)
- 首次生成邀请链接前签电子协议

必须遵守的红线:
1. userId 只信服务端 token 读的
2. 循环 referrer signup 拒绝
3. 非 distributor 或未签协议 → 403,不允许出邀请链接
4. 敏感 audit 事件(admin mark / commission events)全部落 append-only jsonl,不删
5. 完成时贴出 test:bff-unit 输出 + 至少 3 段手工 curl 实证

具体施工内容(读作战室原文): 打开 Obsidian /12帧言产品手册/11-第3周施工单-Codex派单.md 找 "F4" 段。

不许:
- 引入多级链
- 未签协议直接出链接
- Pro/企业订单也给佣金(等柯哥单独拍板)

开工,完成贴 test:bff-unit + 3 段手工 curl。
```

---

### 👥 F3 · 创作者社群 MVP · **主线** · 拍板已到位(2026-07-09)

**3 条规则(柯哥 2026-07-09 拍板,不改)**:
- Q1 **贴吧式**——一个大广场,帖子按 时间/热度 排,评论树形;不做关注/子群
- Q2 **AI 预审**——DeepSeek 打分 >阈值即过,不足才转 admin;新用户前 3 条必审
- Q3 **自动隐藏**——举报累计 >3 次自动 `HIDDEN`,等 admin 复核可恢复/删除

**动的文件**:
- **BFF**:
  - 新增 `bff/src/routes/community.ts`,挂 `requireAuth`:
    - `POST /api/community/posts` 发帖(new_user 前 3 条 → `PENDING_AI_REVIEW`,老用户 → `AI_REVIEWING`)
    - `GET /api/community/posts?sort=time|hot&cursor=<>` 列表(只返 `APPROVED`,自己看得到自己的 pending)
    - `POST /api/community/posts/:id/comments` 评论(树形 parent_id)
    - `POST /api/community/posts/:id/report` 举报(累积 report_count,>3 触发 auto-hide)
    - `POST /api/community/posts/:id/moderate` (admin only) 通过/删除/置顶
  - 新增 `bff/src/services/community-moderator.ts`: 调 DeepSeek 打分 → `{score, verdict, reason}`,分数 <70 → `PENDING_ADMIN`,分数 <30 → `REJECTED`
  - 数据存储 `bff/data/community/posts/<postId>.json` + `bff/data/community/reports.jsonl` + `bff/data/community/moderations.jsonl`
- **前端**:
  - 新增 `src/pages/Community.tsx`(路由 `#/community`): 时间线+热度切换 tab、发帖框、举报按钮、我的帖子(含 pending)
  - 新用户提示"你的前 3 条帖子需要审核后才对其他人可见"

**验收**:
- vitest `bff/tests/community.test.ts` + `community-moderator.test.ts`:
  - 新用户第 1-3 条 → PENDING_AI_REVIEW,通过 AI 后才 APPROVED
  - 老用户帖子 AI 打 <70 分 → PENDING_ADMIN
  - AI 打 <30 直接 REJECTED
  - 举报累积 >3 → auto HIDDEN
  - admin 可 override 恢复
- 手工:测试账号发一条低质帖子(乱字)→ REJECTED;发一条正常帖 → APPROVED

**Codex 派单模板**(F3):

```
你是帧言项目的 Codex 工程师。项目在 /Users/kejianjun/Documents/柯哥Openclaw项目/staging/frameyan-ui。

单号: F3 (创作者社群 MVP)

规则(柯哥 2026-07-09 拍板,不许改也不许猜):
- 贴吧式(时间/热度 排,评论树形,不做关注/子群)
- 新用户前 3 条必 AI 预审
- 举报累积 >3 auto HIDDEN

必须遵守的红线:
1. userId 只信服务端 token 读的
2. AI 预审调 DeepSeek,失败降级为 PENDING_ADMIN(**不允许**静默通过)
3. HIDDEN 状态用户能看到自己的但看不到别人的
4. 完成时贴 test:bff-unit + 3 段手工 curl 实证

具体施工内容(读作战室原文): 打开 Obsidian /12帧言产品手册/11-第3周施工单-Codex派单.md 找 "F3" 段。

不许:
- 加"暂时不审核"字样绕过审核
- AI 失败直接放行
- 举报计数用 localStorage(必须服务端)

开工,完成贴 test:bff-unit + 3 段手工 curl。
```

---

### 📊 F7 · 后台管理系统 MVP · 兜底

**内容**: 按 [[08-后台管理系统蓝图]]。柯哥(admin)登录看板:
- 谁用了多少 token(从 `bff/data/llm-audit.jsonl` 聚合)
- 谁在做什么(session 状态)
- 技能配额调控(编辑 `bff/data/skill-quota.json`)
- 用户/邀请码/反馈管理(已有基础)
- 社群管理(F3 上完接进来)
- 分销结算查看(F4 上完接进来)

**放兜底**是因为它是运营内部工具,晚一周不影响柯哥自己用产品。

**柯哥不用管**。

---

## 四、~~待柯哥拍板(F3+F4)~~ · **2026-07-09 已全部拍板**

柯哥 2026-07-09 中午一句"我觉得你的决策也是我想要的"接受了 8 条推荐答案。规则同步到:
- [[frameyan-community-and-distribution-rules]](Claude memory,永久)
- 施工单 F3/F4 段(已就地更新,含 Codex 派单模板)

**原 8 条决策存档**(备查):

### F3 社群 · 3 条决策

**Q1: 社群形态是什么?**
- (A) **贴吧式**:一个大广场,所有帖子按时间/热度排,评论树形
- (B) **朋友圈式**:关注 + 时间线,只看你关注的人
- (C) **微信群式**:柯哥拉群+按主题分子群
- 我倾向 (A)——最容易起量+管控,B 需要关注关系冷启动难,C 太重

**Q2: 新用户发言门槛?**
- (A) 前 3 条 admin 复核过关才能自由发言
- (B) 前 3 条 AI 预审(打分>阈值即过),不足才转 admin
- (C) 无门槛,只靠举报兜底
- 我倾向 (B)——柯哥不会天天审,AI 兜底让"更多人参与"落地

**Q3: 举报后果?**
- (A) 举报>3 次自动隐藏,等 admin 复核
- (B) 举报>3 次直接删,发帖人扣分,累计到阈值封禁
- (C) 只上报 admin,不自动动
- 我倾向 (A)——温和,不至于误伤

---

### F4 分销 · 5 条决策

**Q4: 分销层级?**
- (A) 只允许 1 级(学生→用户)
- (B) 允许 2 级(学生→用户 A→用户 B,A 也能拿 B 的佣金)
- 我倾向 (A)——2 级容易滑坡到传销,合规风险大

**Q5: 佣金比例?**(等支付上线后配置,现在先定百分比)
- (A) 一次性 10% 主套餐金额(¥498/单)
- (B) 一次性 20%(¥996/单)
- (C) 长期分成:用户续费 5 年内每年都给学生 10%
- 我倾向 (B)——一次性 20% 有吸引力,不用长期结算

**Q6: 结算周期?**
- (A) 用户支付即时到账(高吸引力但退款风险)
- (B) 用户支付满 7 天(退款观察期)后可提现
- (C) 月结
- 我倾向 (B)——平衡吸引力与退款风险

**Q7: 学生分销员资格?**
- (A) 所有 PRO 用户
- (B) 只柯哥课程学员(需 admin 手工标记)
- (C) 需要 admin 审核申请
- 我倾向 (B)——现在起量还没到,先控制在自己人手里

**Q8: 学生分销员是否要签协议?**
- (A) 签(合规,免税务麻烦)
- (B) 不签(轻)
- 我倾向 (A)——涉及钱一定要签,即使是电子版

---

## 五、给柯哥的"外部资质清单"(可以并行推进)

| 事 | 用来 | 周期 | 优先级 |
|---|---|---|---|
| SMTP 账号(腾讯企业邮/阿里云邮件推送) | F6a | 1 天 | 🟡 F2 通过后 |
| 微信支付商户号 | 第 5 周 | 5-10 工作日 | 🔴 今天启动(周期长) |
| 苹果开发者账号($99)+App Store Connect | 第 5 周+iOS 上架 | 1-3 天 | 🟡 F1/F3 中期启动 |
| 备案(国内域名) | 第 4 周上云 | 15-20 工作日 | 🔴 今天启动 |

**支付相关 2 项虽然放到第 5 周开工,但资质申请周期长,今天不启动第 5 周会挡。**

---

## 六、我(Claude)接下来做什么

1. **立刻派 F2 给 Codex**(前端半天,收尾今晌工作,验 Codex 交付质量)
2. **等 Explore agent 摸底 userMemory 完毕**,出 F1 施工细单
3. **柯哥回答 § 四 的 8 条决策**,F3+F4 才能出细单派 Codex
4. **F6a/F6b/F7 排在 F1/F3/F4 之后**,顺手派

---

## 七、Codex 派单模板(F2 版,复制即用)

```
你是帧言项目的 Codex 工程师。项目在 /Users/kejianjun/Documents/柯哥Openclaw项目/staging/frameyan-ui。

单号: F2 (五棒 fallback 前端徽标)

必须遵守的红线:
1. 只动本单指定文件
2. 不谎报,完成时贴出 vitest 输出 + 截图
3. TypeScript 严格,tsc --noEmit 全绿
4. 不改后端(后端 payload.generatedBy 字段已经就绪,你直接用)

具体施工内容(读作战室原文): 打开 Obsidian /12帧言产品手册/11-第3周施工单-Codex派单.md 找 "F2" 段。

验收(必须全部真跑):
- npm run lint 全绿
- npm run test:skill-ui 全绿(含你新加的 FyGeneratedByBadge.test.tsx)
- 真跑一遍五棒,截图五棒 Panel 顶栏都有徽标(绿:AI/红:fallback)
- 手工把 bff/.env 里 DEEPSEEK_API_KEY 改成 sk-invalid 重跑一遍五棒,截图打红

不许:
- 改后端代码
- 用 window.confirm/alert 假活
- 忽略掉某一棒 Panel

开工吧,完成后把 npm run lint 输出 + npm run test:skill-ui 输出 + 4 张截图(真 AI 一张、fallback 一张、五棒各一张 Panel 顶栏)贴给我。
```

---

*—— Claude, 2026-07-09 中午 · v2*
