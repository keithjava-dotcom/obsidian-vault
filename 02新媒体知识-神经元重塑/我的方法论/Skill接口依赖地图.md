---
author: 小龙
date: 2026-06-08
status: v1.0
---

# Skill 接口依赖地图

> **用途**：看清楚 Skill 之间的「谁产出什么 / 谁消费什么」，新增 Skill 时检查接口对齐。
> **更新规则**：每次新增/修改 Skill 后同步更新此地图。

---

## 一、主生产线：定位 → 选题 → 大纲 → 文案

```
[dingweiwenjuan-workflow]
    产出：定位问卷（0-问卷.md）
         │
         ▼
[dingwei-report-workflow]
    产出：02-客户分析.md / 03-定位方案.md / 04-语言风格指南.md / 05-内容规划.md
         │
         ├──────────────────────────────────────────┐
         ▼                                          ▼
[xuanti-workflow]                          [sucai-mining-workflow]
    产出：选题列表（按15角度发散）               产出：素材缺口清单 / 客户问卷
         │                                          │
         └──────────┬───────────────────────────────┘
                    ▼
            [dagang-workflow]
                产出：大纲粗坯（含金量体检+结构）
                    │  ← yuyin-zhengli-workflow 也产素材进这里
                    ▼
            [fanwen-workflow]
                产出：逐字稿（封面文字+发布文案+IP版+后期版）
                    │
                    ▼
              [export-to-client]
                产出：客户可读版文件
```

---

## 二、侧面输入线：语音 / 沟通 / 碰撞

```
[yuyin-zhengli-workflow] ──→ 07-沟通记录-{主题}.md + 素材 → dagang

[goutongjilu-update-workflow] ──→ 07-沟通记录-{主题}.md（供所有Skill读上下文）

[pengzhuang-xie-gao] ──→ 碰撞后的终稿（柯哥自己写稿时用，不经过dagang→fanwen链）
```

---

## 三、知识沉淀线

```
[learning-processor] ──→ 知识子弹 → 02新媒体知识-神经元重塑/

[chen-dian-gai-wen-an] ──→ 客户文案点评 → 原子笔记沉淀 → 待登记熟料清单 → [dengji] → 调用索引
```

---

## 四、管理 / 复盘线

```
[goal-manager] ──→ 目标追踪 / 进度检查 / 生产优先级建议 → routing-queue → xuanti/dagang/fanwen/fupan

[weekly-skill-review] ──→ 周复盘 → 画像 / 优化建议 / 配置更新建议
```

---

## 五、各 Skill 输入 / 产出速查表

| Skill | 必读（输入） | 产出 |
|-------|------------|------|
| dingweiwenjuan | 无 | 0-问卷.md |
| dingwei-report | 0-问卷.md + 创作DNA | 02/03/04/05 |
| xuanti | 03-定位方案 + 01-原始素材 | 选题列表 |
| **dagang** | 03-定位方案 + 素材（原始/语音整理/选题） | **大纲粗坯** |
| **fanwen** | **dagang输出文件** + 03/04/05 + 创作DNA | **逐字稿** |
| yuyin-zhengli | 语音转文字文件 + 客户档案 | 整理版 + 素材 + 沟通记录 → dagang |
| sucai-mining | 03 + 01 | 素材缺口清单 |
| goutongjilu | 客户档案 | 07-沟通记录-{主题}.md |
| pengzhuang-xie-gao | 柯哥草稿 + 创作DNA | 碰撞后的终稿 |
| chen-dian-gai-wen-an | 客户文案 + 原子笔记库 | 原子笔记更新 + 待登记熟料清单 |
| learning-processor | 输入资料 | 知识子弹 |
| goal-manager | 目标数据 + 内容任务链状态 | 进度追踪 + 生产优先级建议 |
| export-to-client | 成品文件 | 客户可读版 |

---

## 六、关键接口约束

### fanwen ← dagang（硬依赖，机械校验）
- fanwen 必须拿到 dagang 的输出文件路径才能启动
- 校验：文件存在 + 文件内标注「方向已确认」
- 未通过 → 拦截，不继续

### dagang ← yuyin（软依赖）
- yuyin 的 Step 3 产出的内容判断结果，交给柯哥确认后进 dagang
- dagang 对 yuyin 产出的素材无特殊要求，按常规流程处理

### goutongjilu / yuyin / dingwei ← 07-沟通记录（写操作隔离）
- 三个 Skill 都会写入沟通记录文件
- **隔离方案**：每次沟通独立文件 `07-沟通记录-{主题}.md`
- 不存在抢写冲突

---

## 相关链接

- [[Skill接口依赖地图]]
