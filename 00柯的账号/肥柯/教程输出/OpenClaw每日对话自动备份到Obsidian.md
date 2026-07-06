# OpenClaw 每日对话自动备份到 Obsidian

## 这是什么

每天凌晨自动把你前一天跟AI的所有聊天记录，备份成一篇整洁的Markdown文件，存到你的Obsidian知识库里。

**效果**：每天醒来，Obsidian里多了一篇《X月X日对话记录》，你和AI聊过的所有东西都在，可搜索、可回顾、可链接。

---

## 原理（两句话）

1. OpenClaw 的所有对话都存在你电脑的 `~/.openclaw/agents/main/sessions/` 目录下（JSONL格式）
2. 一个 Python 脚本 + 一个定时任务，每天凌晨自动把昨天的对话提取出来，转成Markdown，存到 Obsidian

---

## 第一步：创建备份脚本

在你的电脑上找个地方放脚本。建议放在 OpenClaw 的 workspace 下：

```bash
mkdir -p ~/.openclaw/workspace/scripts
```

创建文件 `~/.openclaw/workspace/scripts/daily_chat_backup.py`，复制以下内容：

```python
#!/usr/bin/env python3
"""每日对话备份：提取昨天CST时间范围内的所有对话消息，写入Obsidian"""
import json, os, sys
from datetime import datetime, timezone, timedelta
from pathlib import Path

CST = timezone(timedelta(hours=8))  # 北京时间

# ====== 改这里：你的路径 ======
OBSIDIAN_VAULT = Path.home() / '你的Obsidian仓库路径'  # 例如 Documents/Obsidian/我的知识库
BACKUP_DIR_NAME = '每日对话'  # 存在仓库的哪个子目录下
# ============================

SESSION_DIR = Path.home() / '.openclaw' / 'agents' / 'main' / 'sessions'
OUT_DIR = OBSIDIAN_VAULT / BACKUP_DIR_NAME


def main():
    now = datetime.now(CST)
    yesterday = (now - timedelta(days=1)).date()
    today = yesterday

    start_utc = datetime(today.year, today.month, today.day, 0, 0, 0, tzinfo=CST).astimezone(timezone.utc)
    end_utc = datetime(today.year, today.month, today.day, 23, 59, 59, tzinfo=CST).astimezone(timezone.utc)

    msgs = []

    if not SESSION_DIR.exists():
        print("❌ 没找到 OpenClaw session 目录，确认 OpenClaw 已安装并运行过")
        return 1

    for fpath in sorted(SESSION_DIR.glob('*.jsonl')):
        if 'trajectory' in fpath.name or '.bak' in fpath.name or 'reset' in fpath.name or 'deleted' in fpath.name:
            continue

        try:
            with open(fpath) as f:
                for line in f:
                    try:
                        d = json.loads(line)
                    except:
                        continue
                    ts = d.get('timestamp', '')
                    if not ts:
                        continue
                    try:
                        dt = datetime.fromisoformat(ts.replace('Z', '+00:00'))
                    except:
                        continue
                    if dt < start_utc or dt > end_utc:
                        continue

                    msg = d.get('message', {})
                    role = msg.get('role', '')
                    content = msg.get('content', '')
                    text = ''
                    if isinstance(content, list):
                        for c in content:
                            if isinstance(c, dict) and c.get('type') == 'text':
                                text += c['text']
                    elif isinstance(content, str):
                        text = content
                    text = text.strip()
                    if not text:
                        continue
                    if role not in ('user', 'assistant'):
                        continue
                    if 'heartbeat' in text.lower() or text.startswith('HEARTBEAT'):
                        continue
                    if text == '[assistant turn failed before producing content]':
                        continue
                    if 'Continue the OpenClaw runtime event' in text:
                        continue

                    cst_time = dt.astimezone(CST).strftime('%H:%M')
                    label = '我' if role == 'user' else 'AI'
                    msgs.append((dt, cst_time, label, text))
        except Exception as e:
            print(f"⚠️ 读取出错 {fpath}: {e}")

    if not msgs:
        print(f"{yesterday}: 没有对话消息")
        return 0

    msgs.sort(key=lambda x: x[0])
    date_str = yesterday.strftime('%Y-%m-%d')

    lines = [
        "---",
        "date: " + date_str,
        "---",
        "",
        f"# {date_str} 对话记录",
        "",
        f"共 {len(msgs)} 条消息",
        "",
    ]
    for _, cst_time, label, text in msgs:
        lines.append(f"**{cst_time} {label}**")
        lines.append("")
        lines.append(text)
        lines.append("")
        lines.append("---")
        lines.append("")

    OUT_DIR.mkdir(parents=True, exist_ok=True)
    outpath = OUT_DIR / f'{date_str}.md'
    with open(outpath, 'w') as f:
        f.write('\n'.join(lines))

    print(f"✅ {yesterday}: {len(msgs)} 条消息 → {outpath}")


if __name__ == '__main__':
    sys.exit(main())
```

**改什么**：脚本顶部有两个路径要改成你自己的——

```python
OBSIDIAN_VAULT = Path.home() / '你的Obsidian仓库路径'
BACKUP_DIR_NAME = '每日对话'
```

> 比如你的Obsidian仓库在 `~/Documents/Obsidian/我的知识库/`，就写成：
> `OBSIDIAN_VAULT = Path.home() / 'Documents' / 'Obsidian' / '我的知识库'`

---

## 第二步：测试一下

```bash
python3 ~/.openclaw/workspace/scripts/daily_chat_backup.py
```

跑完去你的Obsidian里看，`每日对话/` 目录下有没有生成昨天的对话文件。有就对了。

---

## 第三步：设置每天自动跑

用 OpenClaw 的 cron 功能（不需要搞系统的 crontab）。

在 OpenClaw 的 WebChat 或微信里，对你的AI助手说：

> 帮我创建一个cron任务，每天早上00:05自动跑 `python3 ~/.openclaw/workspace/scripts/daily_chat_backup.py`，备份昨天的对话记录到Obsidian。静默执行不发通知。

或者你自己用命令行建：

```bash
openclaw cron add '{
  "name": "每日对话备份到Obsidian",
  "schedule": { "kind": "cron", "expr": "5 0 * * *", "tz": "Asia/Shanghai" },
  "sessionTarget": "isolated",
  "payload": {
    "kind": "agentTurn",
    "message": "运行备份脚本：python3 ~/.openclaw/workspace/scripts/daily_chat_backup.py\n脚本会自动备份昨天的对话。备份完静默退出，不发通知。",
    "timeoutSeconds": 300
  },
  "delivery": { "mode": "none" },
  "enabled": true
}'
```

---

## 验证

第二天早上起来，打开Obsidian → `每日对话/` → 看看有没有生成 `YYYY-MM-DD.md`。打开看看内容对不对。

---

## 常见问题

**Q: 跑完没生成文件？**
A: 确认两件事：（1）昨天确实跟AI聊过天；（2）OBSIDIAN_VAULT 路径写对了。

**Q: 时间不对？**
A: 脚本用的是北京时间（CST, UTC+8）。如果你在其他时区，改脚本顶部的 `CST = timezone(timedelta(hours=8))`。

**Q: 不想存Obsidian，想存别的地方？**
A: 改 `OUT_DIR` 路径就行，不需要Obsidian也能用。

---

> 🦞 这套东西，就是柯建军AI工作流基建的一部分。能用起来的，打个卡告诉我。
