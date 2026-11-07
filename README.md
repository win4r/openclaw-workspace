[Bilibili profile](https://space.bilibili.com/3493277319825652)

# openclaw-workspace

A [Claude Code](https://claude.ai/claude-code) skill for maintaining and optimizing [OpenClaw](https://openclaw.ai) workspace files — the markdown files that form an AI agent's soul, memory, and operating procedures.

[中文文档 →](README_CN.md)

---

## What is this?

OpenClaw is a self-hosted, multi-channel AI agent gateway (WhatsApp, Telegram, Discord, Slack, and more). Every OpenClaw agent has a **workspace** — a directory of markdown files that are injected into the system prompt on each turn, giving the agent its identity, behavioral rules, environmental knowledge, and long-term memory.

These workspace files are powerful but require careful management:
- **Too much content** wastes tokens on every single turn
- **Redundancy** between files creates confusion and contradictions
- **Stale content** (old SSH hosts, deprecated rules, finished tasks) silently degrades agent quality
- **Missing security gates** can leak private memory into group chats

This skill gives Claude Code the knowledge to audit, optimize, and build OpenClaw workspace files correctly.

---

## Workspace Files Overview

| File | Purpose | Loaded When | Visible to Sub-agents? |
|------|---------|-------------|----------------------|
| `AGENTS.md` | Boot sequence, checklists, behavioral rules | Every turn (all agents) | Yes |
| `SOUL.md` | Persona, tone, values, continuity philosophy | Every turn (all agents) | Yes |
| `TOOLS.md` | Env-specific notes (SSH, TTS, cameras, devices) | Every turn (main + sub-agents) | Yes |
| `USER.md` | Human profile, preferences, relationship context | Every turn (main sessions only) | No |
| `IDENTITY.md` | Name, emoji, avatar, self-description | Every turn | Yes |
| `HEARTBEAT.md` | Periodic check tasks and health routines | Every heartbeat turn | Depends |
| `BOOT.md` | Startup actions (`hooks.internal.enabled` required) | On gateway startup | No |
| `BOOTSTRAP.md` | First-time onboarding script — delete after use | New workspaces only | No |
| `MEMORY.md` | Long-term curated facts and iron-law rules | Main sessions only | **Never** |
| `memory/YYYY-MM-DD.md` | Daily session logs | Per AGENTS.md boot sequence | No |
| `checklists/*.md` | Step-by-step ops guides | On demand (referenced from AGENTS.md) | No |

### Token Budget

| Constraint | Limit |
|-----------|-------|
| Per file hard cap | 20,000 chars (truncated if exceeded) |
| Total across all bootstrap files | ~150,000 chars |
| Recommended target per file | 10,000–15,000 chars |

### Security Rule

> **`MEMORY.md` must NEVER be loaded in group chats or sub-agent sessions.** It contains private user context that must not leak. The boot sequence in `AGENTS.md` must gate its loading explicitly: `"Main session only: Read MEMORY.md"`.

---

## Workspace Directory Structure

```
~/.openclaw/workspace/
├── AGENTS.md          # Operating manual — boot sequence, rules, checklists table
├── SOUL.md            # Persona, tone, values
├── TOOLS.md           # Env-specific: SSH hosts, TTS voices, camera IDs
├── USER.md            # Human profile (main sessions only)
├── IDENTITY.md        # Name, emoji, avatar
├── HEARTBEAT.md       # Periodic task instructions
├── BOOT.md            # Startup hook actions
├── BOOTSTRAP.md       # First-run onboarding (delete after use)
├── MEMORY.md          # Iron-law rules (main sessions only)
├── memory/
│   ├── 2026-03-10.md  # Daily session logs
│   └── archive/       # Old logs (> 30 days)
├── checklists/
│   ├── deploy-agent.md
│   ├── gateway-restart.md
│   └── config-patch.md
└── docs/              # On-demand docs (NOT auto-loaded every turn)
    ├── agent-rules-detail.md
    └── ssh-reference.md
```

---

## What This Skill Does

When invoked, this skill guides Claude Code through five main workflows:

### 1. Audit Existing Workspace

Reads all workspace files, checks character counts, identifies bloat, spots redundancy between files, finds stale content, and proposes targeted edits.

```bash
# Quick size audit
wc -c ~/.openclaw/workspace/*.md
```

Files over 10,000 chars are prime candidates for trimming or moving content to `docs/` (loaded on demand).

### 2. Set Up a New Workspace from Scratch

Creates workspace files in the correct order (SOUL → AGENTS → IDENTITY → USER → TOOLS → MEMORY → optional files). Ensures the boot sequence, security gates, and checklist table are all properly wired up.

**Minimal viable workspace:** `AGENTS.md` + `SOUL.md` + `TOOLS.md`. Everything else is optional.

### 3. Memory Distillation

Processes `memory/YYYY-MM-DD.md` daily logs into `MEMORY.md`. Promotes recurring mistakes and hard-won rules to iron-law format, archives old logs, and checks for rules that have matured enough to move to a skill's `SKILL.md` instead.

### 4. Add or Update Checklists

Creates `checklists/<operation>.md` files for high-risk operations (deploys, config changes, gateway restarts) and registers them in the `AGENTS.md` checklists routing table.

### 5. Update TOOLS.md

Adds new environment-specific entries (SSH hosts, TTS voices, camera/device IDs) and cleans out stale entries.

---

## How to Install

This is a [Claude Code skill](https://docs.openclaw.ai/skills). Place it in your skills directory:

```bash
# Clone into your Claude Code skills directory
git clone https://github.com/win4r/openclaw-workspace ~/.claude/skills/openclaw-workspace
```

Claude Code will automatically detect and register the skill. It will appear in the skill list as:

> **openclaw-workspace** — Use when maintaining or optimizing OpenClaw workspace files...

### Manual Installation

Copy the skill directory anywhere Claude Code looks for skills:
- `~/.claude/skills/openclaw-workspace/` (recommended)
- Per-project: `.claude/skills/openclaw-workspace/`

---

## How to Use

Once installed, Claude Code will automatically invoke this skill when you:

- "Review my AGENTS.md for token efficiency"
- "Help me set up a new OpenClaw workspace"
- "Distill my memory logs into MEMORY.md"
- "Add a checklist for gateway restarts"
- "Audit my workspace files for redundancy"
- "My MEMORY.md is getting too large, help me clean it up"

You can also explicitly ask: *"Use the openclaw-workspace skill to..."*

---

## Common Issues

### File exceeds token limit (> 20,000 chars)
Move content to `docs/` (loaded on demand, not every turn). Keep only what needs to be in context on every single turn.

### MEMORY.md leaking to group chats
Add explicit gating in AGENTS.md boot sequence: `"2. Main session only: Read MEMORY.md"`. Without this gate, the agent may load it in any context.

### Boot sequence not loading files
The agent follows AGENTS.md boot sequence instructions — it won't auto-discover files. Make sure each file is explicitly named in the boot sequence.

### Workspace changes not taking effect
Workspace files are read at session start. Start a new session or restart the gateway for changes to apply.

### MEMORY.md growing unbounded
Run memory distillation monthly. Promote mature rules to skill `SKILL.md` files (more appropriate home for stable, tool-specific rules). Delete rules about completed tasks.

---

## File Reference

| File | Description |
|------|-------------|
| [`SKILL.md`](SKILL.md) | Main skill — all workflows, common issues, workspace paths |
| [`references/workspace-files.md`](references/workspace-files.md) | Deep-dive on every workspace file: purpose, design principles, anti-patterns, section structure |
| [`references/optimization-guide.md`](references/optimization-guide.md) | Token efficiency strategies, audit commands, memory distillation process, redundancy audit table |

---

## Related

- [OpenClaw](https://openclaw.ai) — the gateway this skill supports
- [openclaw skill](https://github.com/win4r/openclaw) — gateway operations, channel setup, multi-agent routing (separate skill)

---

## License

MIT

## Buy Me a Coffee
[!["Buy Me A Coffee"](https://storage.ko-fi.com/cdn/kofi2.png?v=3)](https://ko-fi.com/aila)

## My WeChat Group and My WeChat QR Code

<img src="https://github.com/win4r/AISuperDomain/assets/42172631/d6dcfd1a-60fa-4b6f-9d5e-1482150a7d95" width="186" height="300">
<img src="https://github.com/win4r/AISuperDomain/assets/42172631/7568cf78-c8ba-4182-aa96-d524d903f2bc" width="214.8" height="291">
<img src="https://github.com/win4r/AISuperDomain/assets/42172631/fefe535c-8153-4046-bfb4-e65eacbf7a33" width="207" height="281">



