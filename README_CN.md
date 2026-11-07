# [🚀哔哩哔哩主页](https://space.bilibili.com/3493277319825652)

# openclaw-workspace

一个用于维护和优化 [OpenClaw](https://openclaw.ai) 工作区文件的 [Claude Code](https://claude.ai/claude-code) 技能包。这些 Markdown 文件构成了 AI Agent 的"灵魂"、记忆系统与操作规程。

[English →](README.md)

---

## 这是什么？

OpenClaw 是一个自托管的多渠道 AI Agent 网关，支持 WhatsApp、Telegram、Discord、Slack 等平台。每个 OpenClaw Agent 都有一个**工作区**（workspace）——一个包含多个 Markdown 文件的目录，这些文件在每次对话轮次时被注入到系统提示词中，赋予 Agent 身份认同、行为规则、环境知识和长期记忆。

工作区文件功能强大，但需要精心管理：
- **内容过多** 会在每次对话轮次中浪费大量 Token
- **文件间的重复内容** 会导致混乱和自相矛盾
- **过时内容**（旧的 SSH 主机、废弃的规则、已完成的任务）会悄无声息地降低 Agent 质量
- **缺少安全隔离** 可能导致私密记忆泄露到群聊中

这个技能包让 Claude Code 具备正确审计、优化和构建 OpenClaw 工作区文件的能力。

---

## 工作区文件一览

| 文件 | 用途 | 加载时机 | 子 Agent 可见？ |
|------|------|---------|----------------|
| `AGENTS.md` | 启动序列、操作清单、行为规则 | 每次轮次（所有 Agent） | 是 |
| `SOUL.md` | 人格、语气、价值观、连续性哲学 | 每次轮次（所有 Agent） | 是 |
| `TOOLS.md` | 环境特定信息（SSH、TTS、摄像头、设备） | 每次轮次（主 Agent + 子 Agent） | 是 |
| `USER.md` | 用户画像、偏好、关系背景 | 每次轮次（仅主会话） | 否 |
| `IDENTITY.md` | 名称、Emoji、头像、自我描述 | 每次轮次 | 是 |
| `HEARTBEAT.md` | 周期性检查任务和健康例程 | 每次心跳轮次 | 视情况而定 |
| `BOOT.md` | 启动时执行的操作（需要 `hooks.internal.enabled`） | 网关启动时 | 否 |
| `BOOTSTRAP.md` | 首次初始化脚本——用完即删 | 仅新工作区 | 否 |
| `MEMORY.md` | 长期精华事实与铁律规则 | 仅主会话 | **永不** |
| `memory/YYYY-MM-DD.md` | 每日会话日志 | 按 AGENTS.md 启动序列加载 | 否 |
| `checklists/*.md` | 高风险操作的逐步指南 | 按需加载（从 AGENTS.md 引用） | 否 |

### Token 预算

| 约束 | 限制 |
|------|------|
| 单文件硬上限 | 20,000 字符（超出将被截断） |
| 所有启动文件总计 | 约 150,000 字符 |
| 建议单文件目标 | 10,000–15,000 字符 |

### 安全规则

> **`MEMORY.md` 绝对不能在群聊或子 Agent 会话中加载。** 它包含不应泄露的私密用户上下文。`AGENTS.md` 中的启动序列必须明确设置加载条件：`"仅主会话：读取 MEMORY.md"`。

---

## 工作区目录结构

```
~/.openclaw/workspace/
├── AGENTS.md          # 操作手册——启动序列、规则、清单路由表
├── SOUL.md            # 人格、语气、价值观
├── TOOLS.md           # 环境专属信息：SSH 主机、TTS 声音、摄像头 ID
├── USER.md            # 用户画像（仅主会话）
├── IDENTITY.md        # 名称、Emoji、头像
├── HEARTBEAT.md       # 周期性任务指令
├── BOOT.md            # 启动钩子操作
├── BOOTSTRAP.md       # 首次初始化脚本（用完即删）
├── MEMORY.md          # 铁律规则（仅主会话）
├── memory/
│   ├── 2026-03-10.md  # 每日会话日志
│   └── archive/       # 归档的旧日志（超过 30 天）
├── checklists/
│   ├── deploy-agent.md
│   ├── gateway-restart.md
│   └── config-patch.md
└── docs/              # 按需文档（不会每次自动加载）
    ├── agent-rules-detail.md
    └── ssh-reference.md
```

---

## 各文件详解

### AGENTS.md — 操作手册

这是每次会话中最先塑造 Agent 行为的文件（基础系统提示词之后）。它包含：

- **启动序列**：有序列出需要在会话开始时读取的文件（SOUL → USER → MEMORY → 日志）
- **清单路由表**：高风险操作 → 对应清单文件路径的映射
- **安全规则**：哪些操作需要确认、哪些可以自主执行
- **群聊规则**：在群聊中不应分享什么

**注意**：AGENTS.md 是规程文件，不是身份文件。人格和价值观属于 SOUL.md。

### SOUL.md — 灵魂

用第二人称书写（"你不是一个聊天机器人，你正在成为某个人"），Agent 读取后将其内化为自我描述。

包含：核心价值观、边界与底线、语气风格、关于连续性的哲学（每次会话都是全新开始，工作区文件就是记忆）。

### TOOLS.md — 本地环境备忘录

这是工作区中最容易被误用的文件。它应该是**当前机器的环境专属速查表**：SSH 主机、TTS 声音 ID、摄像头设备名称等。

子 Agent 也会收到此文件——这是它们唯一的环境知识来源。请保持简洁，50 行以内为佳。

### USER.md — 用户画像

包含影响每次对话的人物相关事实：姓名、时区、语言偏好、沟通风格。

仅在主会话中加载——绝不在群聊或子 Agent 会话中加载。

### MEMORY.md — 铁律规则

只保存"遗忘了就会出严重问题"的规则。每条规则要短且原子化，具有明确的行动指导意义。

定期精炼（每月一次）。已经几个月没有出过问题的规则，可以考虑迁移到技能的 `SKILL.md` 文件中（更合适的归宿）。

### checklists/*.md — 操作清单

高风险操作（部署、网关重启、配置变更）的逐步指南。Agent 执行操作前主动读取对应清单。

**正确模式**：AGENTS.md 只保留一行路由表条目，完整清单放在 `checklists/` 目录中（按需加载，不占用每轮 Token 预算）。

---

## 这个技能能做什么

安装后，Claude Code 将在以下五种主要工作流中调用此技能：

### 1. 审计现有工作区

读取所有工作区文件，检查字符数，识别冗余内容，发现过时条目，提出针对性的修改建议。

```bash
# 快速大小审计
wc -c ~/.openclaw/workspace/*.md
```

超过 10,000 字符的文件是修剪或将内容迁移到 `docs/` 的首要候选。

### 2. 从零创建新工作区

按正确顺序创建工作区文件（SOUL → AGENTS → IDENTITY → USER → TOOLS → MEMORY → 可选文件），确保启动序列、安全隔离和清单路由表都正确配置。

**最小可用工作区**：`AGENTS.md` + `SOUL.md` + `TOOLS.md`。其余均为可选。

### 3. 记忆精炼

处理 `memory/YYYY-MM-DD.md` 每日日志，将其提炼进 `MEMORY.md`。将反复出现的错误和来之不易的规则提升为铁律格式，归档旧日志，并检查是否有已成熟的规则可以迁移到技能的 `SKILL.md` 中。

**铁律格式：**
```
N. **规则名称（分类）**：一句话规则。必要时在同一句中补充背景。
```

### 4. 添加或更新操作清单

为高风险操作创建 `checklists/<操作>.md` 文件，并在 `AGENTS.md` 清单路由表中注册。

**清单结构：**
```markdown
# Checklist: <操作名称>

## 准备阶段
- [ ] 检查 X
- [ ] 验证 Y

## 执行阶段
- [ ] 执行 Z

## 验证阶段
- [ ] 确认结果
- [ ] 记录到记忆系统
```

### 5. 更新 TOOLS.md

添加新的环境专属条目（SSH 主机、TTS 声音、摄像头/设备 ID），清理过时条目。

---

## 如何安装

这是一个 [Claude Code 技能](https://docs.openclaw.ai/skills)。将其放入技能目录：

```bash
# 克隆到 Claude Code 技能目录
git clone https://github.com/win4r/openclaw-workspace ~/.claude/skills/openclaw-workspace
```

Claude Code 会自动检测并注册该技能，在技能列表中显示为：

> **openclaw-workspace** — 维护或优化 OpenClaw 工作区文件时使用...

### 手动安装

将技能目录复制到 Claude Code 能识别的路径：
- `~/.claude/skills/openclaw-workspace/`（推荐）
- 项目级：`.claude/skills/openclaw-workspace/`

---

## 如何使用

安装后，当你说以下内容时，Claude Code 会自动调用此技能：

- "检查我的 AGENTS.md 的 Token 效率"
- "帮我从零创建一个新的 OpenClaw 工作区"
- "把我的记忆日志精炼到 MEMORY.md 中"
- "为网关重启添加一个操作清单"
- "审计我的工作区文件是否有冗余"
- "我的 MEMORY.md 太大了，帮我清理一下"

也可以明确指定：*"使用 openclaw-workspace 技能来……"*

---

## Token 优化策略

### 什么时候把内容移到 docs/

`docs/` 目录存放按需加载的内容，不会在每次轮次自动加载。这是降低每轮 Token 成本的主要手段。

**应该移到 `docs/` 的内容：**
- 只在特定操作类型时需要的内容（例如详细的渠道配置步骤）
- 叙述性说明而非规则或事实
- 特定过去任务的历史背景

**应该保留在工作区文件中的内容：**
- 影响每次轮次的内容（人格、安全规则、清单路由表）
- 在 Agent 收到任何消息之前就必须在上下文中的内容（启动序列文件）

### 冗余审计速查

| 来源 A | 来源 B | 检查内容 |
|--------|--------|---------|
| SOUL.md（价值观） | AGENTS.md（规则） | 安全规则两处都有？保留在 AGENTS.md，删除 SOUL.md 中的 |
| TOOLS.md | MEMORY.md | 相同的工具备注或 SSH 主机？保留在 TOOLS.md |
| MEMORY.md | 技能 SKILL.md | 相同规则两处都有？将稳定规则迁移到 SKILL.md |
| USER.md | SOUL.md | 用户偏好重复？只保留在 USER.md |

### 文件大小阈值

| 大小 | 状态 |
|------|------|
| 5,000 字符以下 | 健康 |
| 5,000–10,000 字符 | 可接受，注意增长趋势 |
| 10,000–15,000 字符 | 审查修剪机会 |
| 超过 15,000 字符 | 需要优化 |
| 超过 20,000 字符 | 将被 OpenClaw 截断——立即处理 |

---

## 常见问题

### 文件超出 Token 限制（> 20,000 字符）
将内容移到 `docs/`（按需加载，不是每次都加载）。只保留每次轮次都必须在上下文中的内容。

### MEMORY.md 泄露到群聊
在 AGENTS.md 启动序列中添加明确的加载条件：`"2. 仅主会话：读取 MEMORY.md"`。没有这个条件，Agent 可能会在任何上下文中加载它。

### 启动序列未加载文件
Agent 遵循 AGENTS.md 启动序列指令——它不会自动发现文件。确保每个文件都在启动序列中被明确命名。

### 工作区变更未生效
工作区文件在会话开始时读取。变更生效需要开始新会话或重启网关。

### MEMORY.md 无限增长
每月执行一次记忆精炼。将成熟的规则迁移到技能 `SKILL.md` 文件中（针对工具特定规则的更合适归宿）。删除关于已完成任务的规则。

---

## 参考文件

| 文件 | 说明 |
|------|------|
| [`SKILL.md`](SKILL.md) | 主技能文件——所有工作流、常见问题、工作区路径 |
| [`references/workspace-files.md`](references/workspace-files.md) | 每个工作区文件的深度解析：用途、设计原则、反模式、章节结构 |
| [`references/optimization-guide.md`](references/optimization-guide.md) | Token 效率策略、审计命令、记忆精炼流程、冗余审计表格 |

---

## 相关资源

- [OpenClaw](https://openclaw.ai) — 本技能支持的网关
- [openclaw 技能](https://github.com/win4r/openclaw) — 网关操作、渠道配置、多 Agent 路由（独立技能包）

---

## 许可证

MIT

## Buy Me a Coffee
[!["Buy Me A Coffee"](https://storage.ko-fi.com/cdn/kofi2.png?v=3)](https://ko-fi.com/aila)

## My WeChat Group and My WeChat QR Code

<img src="https://github.com/win4r/AISuperDomain/assets/42172631/d6dcfd1a-60fa-4b6f-9d5e-1482150a7d95" width="186" height="300">
<img src="https://github.com/win4r/AISuperDomain/assets/42172631/7568cf78-c8ba-4182-aa96-d524d903f2bc" width="214.8" height="291">
<img src="https://github.com/win4r/AISuperDomain/assets/42172631/fefe535c-8153-4046-bfb4-e65eacbf7a33" width="207" height="281">



