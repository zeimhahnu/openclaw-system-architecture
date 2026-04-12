# OpenClaw Agent System Architecture

> A self-evolving AI agent system built on OpenClaw, featuring long-term memory, continuous learning, and multi-agent coordination.

## Overview

This is my personal AI agent system, built on [OpenClaw](https://openclaw.ai), featuring two specialized agents:

- **Lil Claw** — Main agent, system management, market intelligence
- **Goop** — Finance mentor, index data specialist, learning partner

## Key Features

- 🧠 **Long-Term Memory** — Persistent memory across sessions
- 🔄 **Self-Evolution Loop** — Agents improve from experience
- 📊 **Continuous Learning** — Adapts to user's learning style
- 🔒 **Memory Architecture** — Agent-specific + shared knowledge
- 📈 **Market Intelligence** — Automated daily briefings

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     User (Alex)                             │
│              Obsidian (Read-Only Dashboard)                 │
└──────────────────────────┬────────────────────────────────┘
                           │ GitHub Sync
        ┌──────────────────┴──────────────────┐
        ↓                                      ↓
┌───────────────────┐               ┌───────────────────┐
│    Lil Claw       │               │      Goop         │
│   (Main Agent)    │               │  (Finance Mentor) │
├───────────────────┤               ├───────────────────┤
│ memory/           │               │ memory/           │
│  - daily/         │               │  - alex-profile/  │
│  - reflections/   │               │  - session-handoff│
│  - experiments/   │               │  - daily/         │
│  - WAL.md         │               │  - reflections/   │
├───────────────────┤               ├───────────────────┤
│ Dream Cycle       │               │ Dream Cycle       │
│ (17:00 UTC)      │               │ (Every 2 days)    │
└───────────────────┘               └───────────────────┘
```

## Memory Architecture

### Three-Tier Memory System

| Tier | What | When Used |
|------|------|-----------|
| **Short-Term** | Session context | Current conversation |
| **Working** | Daily logs, reflections | Decision making |
| **Long-Term** | MEMORY.md, wiki | Strategic thinking |

### Agent Memory Ownership

- **Lil Claw** owns: `agents/lil-claw/memory/`
- **Goop** owns: `agents/goop/memory/`
- **Shared**: `market-intel/wiki/` (both can read, owner writes)

### Git Repository Model

Agents are **writers**, user is **reader only**. This eliminates merge conflicts and ensures agents' internal state always matches disk state.

## Self-Evolution Loop

```
Experience → Memory → Reflection → Consolidation → Improvement
    ↑                                              ↓
    └──────────── [Loop Repeats] ←─────────────────┘
```

### Dream Cycle (Nightly)

Every night at 17:00 UTC, Lil Claw runs a 5-step consolidation:

1. **Orphan Recovery** — Check for crashed sessions
2. **Distill** — Extract insights from last 7 days
3. **Integrate** — Update long-term memory
4. **Prune** — Archive stale data
5. **Self-Improve** — Update core knowledge if needed

## What We Built

### 1. Continuous Memory Append

Instead of waiting for session end (which crashes often), agents append to daily logs after every 3rd turn or tool execution.

### 2. WAL (Write-Ahead Log)

State recovery for crashes. Before significant decisions, agents write their intention. If session crashes, recovery is deterministic.

### 3. Partitioned Memory

- `[IMMUTABLE]` — Core rules the Dream Cycle cannot modify
- `[FLUID]` — Project data the Dream Cycle can update

### 4. Agent-Specific Memory

Goop builds `alex-knowledge.md` — an evolving model of the user, tracking:
- Learning style (what helps, what confuses)
- Communication preferences
- Knowledge gaps and confidence levels

## Technologies Used

- [OpenClaw](https://openclaw.ai) — Agent framework
- [Obsidian](https://obsidian.md) — Knowledge base (via GitHub sync)
- [Google Workspace](https://workspace.google.com) — Gmail, Sheets, Calendar
- [Tavily AI](https://tavily.ai) — Web search and content extraction
- [yfinance](https://github.com/ranaroussi/yfinance) — Market data
- [GitHub](https://github.com) — Version control and sync

## Learnings

### What Worked

1. **Git-as-Dashboard** — Treating Obsidian as read-only eliminates merge conflicts entirely
2. **Continuous Append** — Session crashes mean lost memory; append continuously
3. **Agent Ownership** — Each agent owns its memory; no cross-contamination
4. **Lightweight Dream Cycle** — Less intensive cycles (2-3 days for Goop) more sustainable than daily

### What We Got Wrong

1. **Heartbeat Timing** — Initial Dream Cycle relied on heartbeats; changed to cron for reliability
2. **YAML Wikilinks** — Obsidian doesn't support `[[wikilinks]]` in YAML frontmatter; moved to markdown sections
3. **WAL Over-Engineering** — Initially asked for 5-part essays; simplified to state recovery only
4. **Single Memory Space** — Initially shared memory between agents; separated to prevent cross-contamination

### Key Insight

> "The most fragile part of any agent system is bidirectional sync between human and AI editing the same files. By treating the human as read-only consumer, you eliminate the most complex problem in agent architecture."

## Results

- **Zero merge conflicts** in 3 months
- **Real-time sync** — Obsidian updates within seconds of agent activity
- **Crisis recovery** — WAL ensures deterministic recovery from crashes
- **Learning compound** — Goop's alex-knowledge.md grows smarter about user over time

## Documentation

| Document | What It Covers |
|----------|---------------|
| [README.md](./README.md) | This overview |
| [SELF-EVOLUTION.md](./SELF-EVOLUTION.md) | The self-evolution loop, Dream Cycle, 5-step consolidation |
| [MEMORY-DESIGN.md](./MEMORY-DESIGN.md) | Three-tier memory, retrieval strategy, Git-as-Dashboard |
| [IMPLEMENTATION.md](./IMPLEMENTATION.md) | How we built it, mistakes made, key lessons |

## Quick Summary

### The Problem

AI agents are stateless — they forget everything after each conversation.

### The Solution

```
Experience → Memory → Reflection → Consolidation → Improvement
    ↑                                              ↓
    └──────────── [Loop Repeats] ←─────────────────┘
```

### The Key Insight

> Agents = Writers, Humans = Readers only.
> This eliminates merge conflicts and ensures agents' internal state always matches disk.

### Files Per Agent

**Lil Claw (System Agent):**
- `daily/` — Session logs
- `WAL.md` — Crash recovery
- `reflections/` — Lessons
- `experiments/` — Hypotheses
- Dream Cycle: 17:00 UTC daily

**Goop (Learning Agent):**
- `daily/` — Session logs
- `alex-profile/` — User model
- `session-handoff/` — Continuity
- Dream Cycle: Every 2 days

## Learn More

- [Self-Evolution Architecture](./SELF-EVOLUTION.md)
- [Memory Design](./MEMORY-DESIGN.md)
- [Implementation Log](./IMPLEMENTATION.md)

---

*Built with OpenClaw by Alex (@zeimhahnu)*
