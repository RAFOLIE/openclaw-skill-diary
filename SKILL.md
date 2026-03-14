---
name: diary
description: |
  用户的 Obsidian 日记写入技能。

  **当以下情况使用此技能**：
  (1) 定时任务（cron）触发写日记
  (2) 用户要求"写日记"、"记一下今天的事"
  (3) 一天结束时整理当天活动记录
  (4) 用户说"日记"、"journal"、"记一下"

  **核心原则**：日记是完整、可直接阅读的文档，不是引用或摘要。

version: 1.0
metadata:
  openclaw:
    emoji: "📓"
    priority: high
---

# 日记写入技能

## 核心信息

| 项目 | 说明 |
|------|------|
| **用途** | 将当天活动记录整理成用户的 Obsidian 日记 |
| **目标位置** | `笔记-用户/日记/YYYY/MM/YYYY-MM-DD.md` |
| **工具** | obsidian CLI（必须使用，禁止直接操作文件） |
| **内容来源** | `memory/YYYY-MM-DD.md`（当天工作日志） + 对话记忆 |

## ⛔ 铁律（必须遵守）

### 1. 日记必须写完整正文
- ✅ 日记内容**直接写入** Obsidian 笔记
- ❌ **禁止**用 `@文件路径` 引用临时文件——临时文件会丢，引用会断
- ❌ **禁止**只写摘要或一句话——用户要的是完整的活动记录

### 2. 日记必须可直接阅读
- 日记是给人看的，不是给程序看的
- 每个事件要有**背景、过程、结果**
- 技术细节要保留（命令、配置、路径等）
- 代码块用 markdown 语法包裹

### 3. 只能用 obsidian CLI
- 所有笔记操作必须用 `obsidian` 命令
- ❌ 禁止直接 read/write/edit 操作 `.md` 文件
- 长内容使用 eval + JS 写入（见下方"长内容写入法"）

### 4. 用户的日记空间是私密的
- 日记存到 `笔记-用户/日记/`，不是 `笔记-小企鹅/`
- 未经用户同意不操作用户的日记

## 日记格式规范

### 文件名
`YYYY-MM-DD.md`，存放在 `笔记-用户/日记/YYYY/MM/` 下

### 结构模板

```markdown
# YYYY-MM-DD 星期X

## 🏷️ 主题一（emoji + 简短标题）

### 子标题（如需要）
- 事件描述（谁、做了什么、为什么）
- 技术细节（命令、配置、路径）
- 结果（成功/失败/待定）

### 备注（可选）
- 注意事项、后续计划

---

## 🏷️ 主题二

...

---

#标签1 #标签2 #标签3
```

### 格式要点
- **标题**：`# YYYY-MM-DD 星期X`
- **主题块**：用 emoji 分类 + 简短中文标题（参考下方 emoji 对照表）
- **分隔线**：主题之间用 `---`
- **标签**：末尾用 `#标签` 格式，2-5 个
- **时间线**：重要事件标注时间（如 `18:37 —`）
- **完成标记**：完成的操作用 ✅

### Emoji 对照表

| 主题 | Emoji |
|------|-------|
| 系统配置/安装 | 🖥️ |
| 网络问题 | 🌐 |
| 插件/工具 | 🔧 |
| 技能创建/更新 | 📝 |
| GitHub | 📤 |
| 浏览器 | 🌍 |
| Twitter/X | 🐦 |
| ComfyUI/AI 绘画 | 🎨 |
| 记忆系统 | 🧠 |
| 删除/卸载 | 🗑️ |
| 模型/配置 | 🤖 |
| 日程/会议 | 📅 |
| 飞书 | 📨 |
| 论坛互动 | 💬 |
| Obsidian | 📓 |
| 错误/故障 | ⚠️ |

## 写入流程

### 第一步：收集素材

1. 读取 `memory/YYYY-MM-DD.md`（当天工作日志）
2. 回顾对话中发生的重要事件
3. 识别 3-6 个主题块

### 第二步：组织内容

按时间线或主题组织：
- 同一主题的事件归到一块（不要按时间分散）
- 技术操作保留关键命令和配置
- 结果要明确（成功/失败/进行中）

### 第三步：写入 Obsidian

#### 短内容（<3000 字符）

直接用 create：

```powershell
obsidian create "path=笔记-用户/日记/YYYY/MM/YYYY-MM-DD.md" "content=<正文>"
```

> ⚠️ PowerShell 命令行参数有长度限制，超过 ~3000 字符会被截断。

#### 长内容（≥3000 字符）— 使用 eval + JS

这是最可靠的方法，绕过 PowerShell 命令行长度限制：

```powershell
# 1. 用 write 工具把完整日记写到临时文件
# 2. 用 obsidian eval 通过 JS 读取临时文件写入 vault
obsidian eval "code=(function() { const fs = require('fs'); const c = fs.readFileSync('C:/Users/win11-001/.openclaw/workspace/tmp-diary.md', 'utf8'); const f = app.vault.getAbstractFileByPath('笔记-用户/日记/YYYY/MM/YYYY-MM-DD.md'); if (f) { app.vault.modify(f, c); return 'updated ' + c.length; } else { app.vault.create('笔记-用户/日记/YYYY/MM/YYYY-MM-DD.md', c); return 'created ' + c.length; } })()"
```

注意：
- JS 中用正斜杠 `/` 作为路径分隔符
- `getAbstractFileByPath` 查找已有文件，`create` 创建新文件
- `modify` 覆写已有文件内容
- 执行后清理临时文件

### 第四步：验证

```powershell
# 用 wordcount 验证（比 read 更准确）
obsidian wordcount "path=笔记-用户/日记/YYYY/MM/YYYY-MM-DD.md"
```

确认 characters 数与预期一致。

## 写入禁忌

| 禁忌 | 原因 |
|------|------|
| ❌ 用 `@临时文件` 引用 | 临时文件会丢失，日记变成空白 |
| ❌ 只写摘要 | 用户需要完整细节以便日后回顾 |
| ❌ 直接操作 .md 文件 | 违反 obsidian CLI 规则，不触发插件/同步 |
| ❌ 不验证就结束 | 写入可能被截断，必须 wordcount 验证 |
| ❌ 写到小企鹅的笔记空间 | 用户的日记在 `笔记-用户/` |

## 定时触发

日记由 cron 任务定时触发（通常在每晚 23:30），流程：

1. cron 触发 → 发送 systemEvent 到 main session
2. main session 读取当天 memory 日志
3. 整理成日记格式
4. 写入 Obsidian
5. 验证完整性
6. 清理临时文件

---

**版本历史**：
- v1.0 (2026-03-14): 初始版本，基于 3月13日日记问题教训创建
