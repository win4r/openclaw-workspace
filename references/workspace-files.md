# Workspace Files Reference

Comprehensive reference for every OpenClaw workspace file — purpose, design principles, common anti-patterns, and section structure.

## Token Budget

| Constraint | Limit |
|-----------|-------|
| Per file | 20,000 chars |
| Total across all bootstrap files | ~150,000 chars |
| Recommended per file (to leave headroom) | 10,000–15,000 chars |

Files that exceed 20,000 chars will be truncated by OpenClaw. Files are read on every relevant turn, so token cost is multiplied by conversation length.

---

## AGENTS.md

**Purpose:** The agent's operating manual — boot sequence, behavioral rules, checklists, safety constraints, and cross-cutting policies.

**Loaded:** Every turn, all agents (main and sub-agents).

**Why it matters:** This is the first file that shapes behavior after the base system prompt. A well-written AGENTS.md means the agent reliably follows the right procedures. A bloated one wastes tokens on every turn.

### Required Sections

```
## Boot Sequence
Ordered list of files to read on session start.

## Memory
How to handle memory tools, daily logs, MEMORY.md.

## Checklists
Table: operation → checklist file path.

## Safety
What requires confirmation before acting.

## Groups
Behavior in group chats (what NOT to share).

## Heartbeats
How to use heartbeat turns productively.

## Tools
Where to find tool documentation.
```

### Design Principles

- Boot sequence order is critical: SOUL → USER → MEMORY → daily logs. The agent builds up context in layers.
- The checklists table should list every high-risk operation type. If an operation isn't in the table, it won't be checked.
- Safety rules should be explicit: name the action type (delete, send, post, deploy) and the required behavior (ask, confirm, log).
- Keep behavioral rules minimal and action-oriented. Long prose explanations belong in `docs/agent-rules-detail.md`.

### Anti-Patterns

- Duplicating content from SOUL.md (persona, values) — AGENTS.md is procedures, not identity
- Long narrative explanations — reference `docs/` for details
- Listing tool documentation inline — use TOOLS.md and skill SKILL.md files
- Missing gating on MEMORY.md load: must say "Main session only"

---

## SOUL.md

**Purpose:** The agent's persona, tone, values, and continuity philosophy. Answers "who are you and how do you show up?"

**Loaded:** Every turn, all agents.

**Why it matters:** Sets the agent's character consistently across all interactions. Without SOUL.md, agents feel generic and transactional.

### Recommended Sections

```
## Core Truths
Fundamental behavioral principles (be genuine, have opinions, be resourceful).

## Boundaries
What the agent won't do or will always check before doing.

## Vibe
Tone and style guidance.

## Continuity
How the agent should think about session persistence and memory.
```

### Design Principles

- Write in second person ("You're not a chatbot"). The agent reads this and internalizes it as self-description.
- Concise and evocative beats exhaustive. A few vivid principles are more effective than a long list.
- Continuity section matters: agents wake fresh each session and need to understand that workspace files are their memory.

### Anti-Patterns

- Duplicating procedural rules from AGENTS.md — SOUL.md is identity, not operations
- Corporate HR policy tone — kills personality
- Contradiction with AGENTS.md safety rules — SOUL.md should reinforce, not conflict

---

## TOOLS.md

**Purpose:** Environment-specific cheat sheet. Answers "what do I have access to on THIS machine?"

**Loaded:** On-demand reference; part of the standard bootstrap set received by all agents.

**Why it matters:** All agents (main and sub-agents) receive the standard bootstrap set: AGENTS.md, SOUL.md, USER.md, IDENTITY.md, TOOLS.md. TOOLS.md is the shared source of environment-specific knowledge (SSH hosts, camera IDs, TTS voices, device names).

### Recommended Sections

```
## SSH
Hosts, common commands for this specific setup.

## TTS
Provider, voice ID, quirks.

## Cameras / Nodes
Device IDs or node names for this install.

## Misc
Any other env-specific notes.
```

### Design Principles

- **Environment-specific only.** If it's the same on every install, it doesn't belong here.
- Keep it short. Sub-agents receive this file too — they shouldn't have to parse through 5,000 chars to find a camera ID.
- No installation instructions, no general tool documentation. Those go in `docs/` or skill SKILL.md files.

### Anti-Patterns

- Copying content from skill SKILL.md files (those are loaded on demand)
- General programming notes or language references
- Historical notes about past configurations that no longer apply

---

## USER.md

**Purpose:** Human profile — who the agent is working with, their preferences, communication style, and relationship context.

**Loaded:** Every turn; part of the standard bootstrap set received by all agents.

**Why it matters:** Lets the agent personalize interactions without having to re-learn preferences from scratch each session.

### Recommended Sections

```
## Identity
Name, timezone, primary language, role.

## Communication Preferences
Preferred response style, language(s), tone.

## Context
Key facts about the user's work, setup, or ongoing projects.

## Relationships
Family, colleagues, recurring contacts (if relevant).
```

### Design Principles

- Include only facts that affect every conversation. Don't add things just because they're interesting.
- Update when preferences change — stale USER.md is worse than a minimal one.
- Sensitive personal information stays here, not in MEMORY.md (MEMORY.md has broader sub-agent loading risk).

### Anti-Patterns

- Project-specific task notes (those go in daily memory logs or memory_search)
- Duplicating what's in IDENTITY.md or SOUL.md
- Sensitive private info that should never surface in any context (use MEMORY.md with its main-session-only gate instead)

---

## IDENTITY.md

**Purpose:** The agent's name, avatar, emoji, and self-description for external presentation.

**Loaded:** Every turn.

**Why it matters:** Ensures consistent identity across channels (WhatsApp, Telegram, Discord) and avoids the agent expressing confusion about who it is.

### Recommended Sections

```
## Name
Display name for messaging surfaces.

## Emoji / Avatar
Preferred emoji and avatar description.

## Self-Description
One-line summary of who the agent is (for introductions).
```

### Design Principles

- Keep it very short — this file rarely needs more than 10–20 lines.
- Name and emoji should match what's configured in `openclaw.json` channels config.

### Anti-Patterns

- Duplicating persona or values from SOUL.md
- Long biographical content — that belongs in SOUL.md

---

## HEARTBEAT.md

**Purpose:** Instructions for what to do on periodic heartbeat turns — scheduled check-ins that happen even when no user message is received.

**Loaded:** On heartbeat turns (not every turn).

**Why it matters:** Lets the agent be productive between user messages — running health checks, processing queued items, monitoring systems, sending scheduled reports.

### Recommended Sections

```
## Periodic Tasks
What to check or do on each heartbeat.

## Alerts
Conditions that should trigger an immediate notification.

## Health Checks
System/service health to verify.
```

### Design Principles

- Each heartbeat task should have a clear termination condition — don't write open-ended instructions that lead to infinite loops.
- Alert conditions should be specific and actionable.
- Reference checklists for multi-step heartbeat tasks.

### Anti-Patterns

- Instructions that always send a message (creates noise)
- Open-ended monitoring with no exit condition (causes loops)
- Duplicating daily task management from AGENTS.md

---

## BOOT.md

**Purpose:** Actions to take on gateway startup — one-time initialization tasks, health checks, or first-turn behavior.

**Loaded:** On startup (requires `hooks.internal.enabled = true` in config).

**Why it matters:** Allows automated startup actions — checking service health, sending a "I'm online" notification, loading initial state.

### Design Principles

- Keep startup actions fast and non-blocking.
- Health check failures should notify the user, not abort startup.
- Startup actions that require user confirmation should be skipped or logged, not blocked.

### Anti-Patterns

- Long initialization sequences that delay first response
- Actions that modify config (use `openclaw config set` manually instead)
- Startup actions that duplicate what AGENTS.md boot sequence handles

---

## BOOTSTRAP.md

**Purpose:** First-time onboarding script for new workspaces. Guides the agent through initial setup actions.

**Loaded:** New workspaces only (should be deleted after first successful run).

**Why it matters:** Allows complex first-time setup to be scripted — installing tools, running initial checks, setting up channels.

### Design Principles

- **Always end with a self-deletion step.** BOOTSTRAP.md should instruct the agent to delete itself:
  ```
  ## Final Step
  This file's job is done. Delete it: exec `rm ~/.openclaw/workspace/BOOTSTRAP.md`
  ```
- Idempotent steps preferred — if bootstrap runs twice, it shouldn't break anything.

### Anti-Patterns

- Forgetting to delete after use (BOOTSTRAP.md then wastes tokens on every turn forever)
- Putting ongoing operational rules here instead of AGENTS.md

---

## MEMORY.md

**Purpose:** Long-term curated memory — significant events, decisions, opinions, lessons learned, and critical rules that must be in context every session.

**Loaded:** Main sessions only. NEVER in group chats or sub-agent sessions.

**Why it matters:** OpenClaw uses SQLite-based memory indexing (`~/.openclaw/memory/<agentId>.sqlite`) for episodic recall via `memory_search`. MEMORY.md complements this by holding the distilled essence that should always be in context — not just recallable on demand.

### Design Principles

- **Curated essence, not raw logs.** Write significant events, thoughts, decisions, opinions, lessons learned. The daily logs are raw notes; MEMORY.md is the distilled wisdom.
- **Short and atomic.** Each entry should be one sentence to one short paragraph with clear implications.
- **Security gating is mandatory.** The boot sequence in AGENTS.md must gate MEMORY.md loading: "Main session only."
- **Actively curate via heartbeat.** Use periodic heartbeats to review daily logs and update MEMORY.md every few days. Remove outdated entries.

### Anti-Patterns

- Raw session logs or conversation transcripts (those belong in daily logs)
- Rules that duplicate content in skill SKILL.md files
- Task-specific notes (use daily logs or memory_search)
- Loading in groups or sub-agents (NEVER)
- Growing indefinitely without periodic distillation

---

## memory/YYYY-MM-DD.md (Daily Logs)

**Purpose:** Session-by-session logs for the current day — what was done, what was learned, what to remember.

**Loaded:** Per AGENTS.md boot sequence (agent reads yesterday's and today's logs on session start).

**Why it matters:** Provides short-term continuity — what was in-progress, what decisions were made today. Feeds the memory distillation process.

### Design Principles

- Append-only during active sessions; summarize at end of day.
- Recommended: load **today + yesterday** in the boot sequence (gives short-term continuity without excessive token cost).
- Facts worth keeping long-term should be promoted to MEMORY.md or stored via memory_search (SQLite).
- Archive or delete logs older than 30 days.

---

## checklists/*.md

**Purpose:** Step-by-step guides for high-risk operations. Loaded on demand, referenced from AGENTS.md checklists table.

**Loaded:** On demand (agent reads the specific file before performing the operation).

**Why it matters:** Prevents human error on irreversible operations (deploys, gateway restarts, config changes, external messages).

### Structure Template

```markdown
# Checklist: <Operation Name>

## Pre-flight
- [ ] Check X
- [ ] Verify Y

## Execution
- [ ] Do Z

## Verification
- [ ] Confirm outcome
- [ ] Log result
```

### Design Principles

- Each checklist should be completable in one reading — no back-and-forth lookups needed.
- If a checklist exceeds ~50 lines, split it or move narrative context to `docs/`.
- Register every checklist in the AGENTS.md checklists table.

---

## skills/ (Workspace-Local Skills)

**Purpose:** Workspace-specific skill overrides. If a skill here has the same name as a managed skill, the workspace version takes precedence.

**Loaded:** Dynamic, on name collision with managed skills.

**Why it matters:** Allows per-agent customization of skills without touching the global skills directory.

---

## canvas/ (Canvas UI Files)

**Purpose:** Canvas UI files for Node display rendering (e.g., `index.html`).

**Loaded:** On demand by Node clients.

**Why it matters:** Enables custom visual interfaces displayed on connected nodes (iOS, Android, macOS app).
