---
author: 小龙
created: 2026-05-31
source: dream-promotion (memory/2026-05-10.md)
tags: [工具选型, 语音转写, AI工具]
---

# 语音转写工具选型：paraformer-v2

## 核心结论

**视频转文字用 paraformer-v2，不用 qwen-omni-turbo。**

## 成本对比

| 模型 | 每条6分钟视频 | 290分钟(56条) | 10元能转 |
|------|-------------|--------------|---------|
| qwen-omni-turbo | ~¥0.18 | ~¥10 | ~55条 |
| paraformer-v2 | ¥0.029 | ¥1.45 | ~340条 |

**便宜6倍以上。**

## 技术管线

```
本地MP3 → DashScope Files API上传 → 获得file_id
→ 获取OSS下载URL → 提交异步转写任务(paraformer-v2)
→ 轮询任务状态 → 下载转写结果
```

## 适用场景

- 批量处理视频素材转文字（碰撞优质视频）
- 客户/学员视频内容分析
- 柯哥自己视频的文稿存档

## 不适用

- 实时转写（异步，需等待）
- 需要说话人识别（paraformer-v2 不支持diarization，用 funasr 替代方案）

---

> 验证记录：2026-05-10 用 paraformer-v2 转写交叉科学 DeepSeek V4 视频，¥0.03，结果满意。
