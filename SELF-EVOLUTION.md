# Self-Evolution Loop Architecture

> How agents improve themselves over time without forgetting.

## The Problem

Large Language Models are **stateless** — they forget everything after each conversation. For an AI agent to be truly useful, it needs:

1. **Memory** — Remember what happened
2. **Reflection** — Learn from what happened
3. **Improvement** — Change behavior based on lessons

This is the **self-evolution loop**.

## Research

### Industry Standard (2026)

| Approach | Provider | Description |
|----------|----------|-------------|
| **Mem0** | Mem0 | "Context window ≠ memory. Persistent multi-type memory + self-improvement loops." |
| **LangGraph** | LangChain | Orchestration for agent workflows with state management |
| **Long-Term Memory** | ArXiv 2026 | "LTM is the foundation of AI self-evolution." |

### Core Principles

1. **Three Memory Types**
   - Episodic — what happened (daily logs)
   - Semantic — what I know (knowledge base)
   - Procedural — how to do things (TOOLS.md, skills)

2. **The Loop**
   ```
   Experience → Memory → Reflection → Consolidation → Improvement
   ```

3. **Avoid Hallucination**
   - Single source of truth: disk state
   - Agent's internal state always matches files
   - No bidirectional sync (human = reader only)

## Our Implementation

### For Lil Claw (System Agent)

```
Every Session:
  ↓
Continuous Append → daily/YYYY-MM-DD.md
  ↓
Significant Decision → WAL.md (state recovery)
  ↓
Correction → reflections/
  ↓
Cron (17:00 UTC):
  ↓
Dream Cycle:
  1. Orphan Recovery (WAL)
  2. Distill (last 7 days)
  3. Integrate (MEMORY.md)
  4. Prune (archive old)
  5. Self-Improve (update core)
```

### For Goop (Learning Agent)

```
Every Session:
  ↓
Teach/PDF Process → session-handoff.md
  ↓
Append to daily/
  ↓
Every 2 days:
  ↓
Dream Cycle:
  1. Surface Alex's struggles
  2. Adapt approach
  3. Update alex-knowledge.md
  4. Plan next topic
```

### The Key Insight

> **Agents = Writers, Humans = Readers**

If both agent AND human edit the same files:
- Need conflict resolution
- Need stash/rebase/push pipeline
- Risk: agent overwrites human changes → hallucination

If human edits are **never allowed**:
- Agents own the state
- No conflicts
- Agent's internal state always matches disk

## Key Files

| File | Purpose | Updated By |
|------|---------|------------|
| `WAL.md` | State recovery after crashes | Agent only |
| `daily/*.md` | Session logs | Agent only |
| `reflections/*.md` | Lessons learned | Agent only |
| `MEMORY.md` | Long-term knowledge | Dream Cycle only |
| `alex-knowledge.md` | Model of user | Goop only |

## Why This Works

1. **Crash Recovery** — WAL ensures deterministic recovery
2. **No Hallucination** — Agent state always matches disk
3. **Compounding Knowledge** — Each session builds on previous
4. **Human as Validator** — Obsidian dashboard shows agent's thinking

## The 5-Step Dream Cycle

### Step 1: ORPHANS
Check for crashed sessions:
```
[13:45] INTENTION: Config change
[13:45] RESULT: Incomplete - crash
[13:45] SESSION: Orphaned
```
Mark as [RECOVERED] or [DROPPED].

### Step 2: DISTILL
Extract from last 7 days:
- Key decisions
- Corrections made
- Patterns noticed

### Step 3: INTEGRATE
Add to MEMORY.md [FLUID]:
- Insights
- Project updates
- Pending tasks

### Step 4: PRUNE
Archive old data:
- Daily logs > 14 days → monthly summary
- Reflections > 30 days → archive
- Stale experiments → mark [INACTIVE]

### Step 5: SELF-IMPROVE
If experiments showed success:
- Update SOUL.md
- Update TOOLS.md
- Mark [INTEGRATED]

## Results

| Metric | Before | After |
|--------|--------|-------|
| Memory retention | Session only | Infinite |
| Crash recovery | Amnesia | Deterministic |
| Learning | None | Compounding |
| Hallucination risk | High | Minimal |

---

*Built from research on Mem0, LangGraph, and ArXiv 2026 papers on Long-Term Memory for AI agents.*
