# Memory Architecture Design

> How we designed a three-tier memory system for AI agents.

## The Problem

Most AI agents have one memory problem:

1. **Context window** is limited (200k tokens)
2. **Files are static** — agent doesn't update them
3. **No retrieval** — agent doesn't know what it knows
4. **No hierarchy** — everything at same importance

## Our Solution

### Three-Tier Architecture

```
┌─────────────────────────────────────────┐
│         TIER 3: LONG-TERM               │
│   MEMORY.md + wiki (persistent)          │
│   - Facts, knowledge, rules             │
│   - Updated by Dream Cycle              │
│   - Searched when needed                │
├─────────────────────────────────────────┤
│         TIER 2: WORKING                 │
│   Daily logs + reflections              │
│   - Session context                     │
│   - Appended continuously              │
│   - Searched for recent context         │
├─────────────────────────────────────────┤
│         TIER 1: SHORT-TERM              │
│   Current session context               │
│   - What I'm doing right now            │
│   - Very recent context                 │
└─────────────────────────────────────────┘
```

### Tier 1: Short-Term (Session)

**What:** Current conversation + immediate context

**How:** Native context window

**When:** Now — what I'm thinking about this turn

---

### Tier 2: Working (Session Logs)

**What:** Daily logs + reflections

**Where:** `agents/*/memory/daily/*.md`

**Format:**
```
[14:32 UTC] Turn #1: User asked about X
[14:33 UTC] Tool: exec → checked Y
[14:35 UTC] Decision: Changed Z
```

**When:** Appended after every 3rd turn or tool execution

**Why:** Crash recovery + recent context without full MEMORY.md

---

### Tier 3: Long-Term (Persistent)

**What:** MEMORY.md + wiki

**Format:** Structured knowledge

```
# MEMORY.md

## [IMMUTABLE] — DO NOT EDIT
Core rules, identity, critical system facts

## [FLUID] — Dream Cycle can update
Insights, projects, pending tasks
```

**When:** Dream Cycle updates (nightly)

**Why:** Strategic knowledge that persists across sessions

---

## Retrieval Strategy

### Don't Load Everything

```
At session start:
  1. Read MEMORY.md (index only) — 500 tokens
  2. Read recent daily logs — 200 tokens
  3. That's it — specific context loaded on demand

When context needed:
  "Tell me about X"
  → grep/semantic search → pull relevant files
  → load only what's needed
```

### Token Budget

| Source | Tokens | When |
|--------|--------|------|
| SOUL.md | ~1k | Always |
| MEMORY.md | ~500 | Session start |
| Recent daily | ~200 | Session start |
| Relevant files | ~1-5k | When needed |
| **Total** | **~3-7k** | Per session |

**Savings:** vs loading all history (~50k+ tokens) every session

---

## File Structure

```
workspace/
├── agents/
│   ├── lil-claw/
│   │   └── memory/
│   │       ├── daily/        ← Tier 2: Session logs
│   │       ├── reflections/   ← Tier 2: Lessons
│   │       ├── experiments/  ← Tier 3: Active tests
│   │       └── WAL.md        ← Crash recovery
│   │
│   └── goop/
│       └── memory/
│           ├── daily/        ← Tier 2: Session logs
│           ├── alex-profile/ ← Tier 3: User model
│           └── reflections/   ← Tier 2: Lessons
│
├── market-intel/
│   └── wiki/               ← Tier 3: Knowledge base
│
└── MEMORY.md               ← Tier 3: Long-term index
```

---

## Partitioned Memory

### Why Partition?

1. **Lil Claw** manages system — needs system knowledge
2. **Goop** manages learning — needs user model
3. **Shared** is read-only — prevents cross-contamination

### Rules

| File | Lil Claw | Goop | Shared |
|------|----------|------|--------|
| `agents/lil-claw/memory/*` | Write | Read | — |
| `agents/goop/memory/*` | Read | Write | — |
| `market-intel/wiki/` | Own | Own | Read |
| `MEMORY.md` | Write | Read | — |

---

## The Write-Ahead Log (WAL)

### Purpose

Crash recovery — not a diary.

### Format

```
[TIMESTAMP] INTENTION: [what I was about to do]
[TIMESTAMP] RESULT: [success/failure/incomplete]
[TIMESTAMP] SESSION: [completed normally / orphaned]
```

### Example

```
[13:45 UTC] INTENTION: Set allowInsecureAuth = false
[13:45 UTC] RESULT: Success - gateway restarted
[13:45 UTC] SESSION: Completed normally
```

### Why Not A Diary?

LLMs don't pause mid-thought cleanly. Writing a 5-part essay after every decision = token waste + cognitive load.

WAL = **state recovery only**.

---

## Continuous Append

### Why Not Session End?

Sessions don't end cleanly:
- Terminals crash
- Context windows max out
- Users close laptops
- Gateway throws 1006 errors

### Solution

Append after every 3rd turn OR after every tool execution.

```
User message
    ↓
Count turns
    ↓ 3rd turn?
    ↓ YES
Append to daily/YYYY-MM-DD.md
    ↓
Continue
```

---

## Pruning Strategy

### Daily Logs

- **< 14 days:** Keep raw
- **> 14 days:** Compress to monthly summary
- **> 90 days:** Archive

### Reflections

- **< 30 days:** Keep
- **> 30 days:** Archive

### MEMORY.md

- **Facts:** Keep forever (once established)
- **Project notes:** Keep if active, archive if stale
- **Lessons:** Keep, compound over time

---

## Git-as-Dashboard

### The Pattern

1. Agent writes to files
2. Agent commits to GitHub
3. User's Obsidian syncs
4. User reads (never writes)

### Why This Works

- **No merge conflicts** — single writer
- **Real-time updates** — push after every significant change
- **Human validates** — Obsidian shows agent's thinking
- **Simple** — no conflict resolution needed

---

## Results

| Metric | Before | After |
|--------|--------|-------|
| Token per session | ~50k | ~5k |
| Crash recovery | Amnesia | Deterministic |
| Memory growth | Unlimited | Managed |
| Retrieval | Everything | On-demand |

---

*Design principles inspired by Mem0's three-tier memory architecture.*
