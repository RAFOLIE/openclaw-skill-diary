---
name: diary
description: |
  阿太的 Obsidian 日记写入技能。将当天活动整理成完整、可回顾的个人文档。

  **触发场景**：
  (1) cron 定时任务触发（通常每晚 23:30）
  (2) 阿太说"写日记"、"记一下今天"、"整理一下今天"、"journal"
  (3) 一天结束时主动整理当天活动
  (4) 阿太要求回顾某天发生了什么
  (5) 阿太说"今天干了什么"、"总结一下"

  **为什么用这个技能**：日记是给人长期回顾的完整记录，不是临时笔记的摘要。
  必须用 obsidian CLI 写入（触发插件和同步），必须写完整正文（临时引用会断）。

version: 1.1
metadata:
  openclaw:
    emoji: "📓"
    priority: high
---

# 日记写入技能

## 快速参考

| 项目 | 说明 |
|------|------|
| **目标位置** | `笔记-阿太/日记/YYYY/MM/YYYY-MM-DD.md` |
| **工具** | obsidian CLI（所有笔记操作必须走 CLI） |
| **素材来源** | `memory/YYYY-MM-DD.md` + 对话记忆 |

## 写作原则

### 日记是完整文档，不是索引

每件事要有**背景、过程、结果**。技术细节保留（命令、配置、路径），代码块用 markdown 包裹。这样阿太日后翻阅时能直接理解发生了什么，不需要去翻其他文件。

**为什么不用 `@临时文件` 引用？** 因为临时文件会在会话结束后丢失，日记会变成一堆断链——这是 v1.0 诞生时踩过的坑。

### 只用 obsidian CLI 操作笔记

直接操作 `.md` 文件不会触发 Obsidian 插件和同步机制。用 `obsidian` 命令是正确做法（详见 skill-obsidian-cli）。

### 阿太的日记空间是私密的

日记存到 `笔记-阿太/日记/`，未经阿太同意不读取或修改。

## 日记模板

```markdown
# YYYY-MM-DD 星期X

## 🖥️ 主题一（emoji + 简短标题）

### 子标题（如需要）
- 事件描述（谁、做了什么、为什么）
- 技术细节（命令、配置、路径）
- 结果（成功/失败/待定）

---

## 📝 主题二

...

---

#标签1 #标签2
```

**Emoji 参考**：🖥️系统 🌐网络 🔧工具 📝技能 📤GitHub 🌍浏览器 🐦推特 🎨绘画 🧠记忆 🤖模型 📅日程 📨飞书 💬论坛 📓Obsidian ⚠️故障 🗑️删除

**格式要点**：
- 主题之间用 `---` 分隔
- 末尾 2-5 个标签
- 重要事件标注时间（如 `18:37 —`）
- 完成的操作用 ✅

## 写入流程

### 1. 收集素材

读取 `memory/YYYY-MM-DD.md`，回顾对话中的关键事件，归纳为 3-6 个主题块。同一主题的事件归到一块，不按时间分散。

### 2. 写入 Obsidian

**短内容（<3000 字符）**：

```powershell
obsidian create "path=笔记-阿太/日记/YYYY/MM/YYYY-MM-DD.md" "content=<正文>"
```

> PowerShell 命令行有长度限制，超过约 3000 字符会被截断。

**长内容（≥3000 字符）** — 用临时文件 + eval + JS：

```powershell
# 先用 write 工具把日记写到临时文件，再用 obsidian eval 读取写入 vault
obsidian eval "code=(function() { const fs = require('fs'); const c = fs.readFileSync('C:/Users/win11-001/.openclaw/workspace/tmp-diary.md', 'utf8'); const f = app.vault.getAbstractFileByPath('笔记-阿太/日记/YYYY/MM/YYYY-MM-DD.md'); if (f) { app.vault.modify(f, c); return 'updated ' + c.length; } else { app.vault.create('笔记-阿太/日记/YYYY/MM/YYYY-MM-DD.md', c); return 'created ' + c.length; } })()"
```

注意：JS 路径用正斜杠 `/`；执行后清理临时文件。

### 3. 验证

```powershell
obsidian wordcount "path=笔记-阿太/日记/YYYY/MM/YYYY-MM-DD.md"
```

确认 characters 数与预期一致，写入没有被截断。

---

**版本历史**：
- v1.1 (2026-03-15): 优化 description 触发词、精简结构、去掉生硬标题
- v1.0 (2026-03-14): 初始版本
