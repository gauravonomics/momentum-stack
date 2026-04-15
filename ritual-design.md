# Ritual Design -- The Rhythm of a Knowledge System

## Overview

A vault without rituals is a graveyard of unconnected notes. Knowledge accumulates, but nothing compiles it into insight. Rituals are the systematic recurring processes that keep a knowledge system alive -- they read what the vault knows, synthesize it into actionable intelligence, and feed updated understanding back into the system.

Ritual design follows a simple principle: each ritual reads evidence from the vault, produces an immutable output file, and optionally updates mutable entity files. The immutable outputs form a historical record. The mutable updates keep entity files fresh. Together, they create a compounding cycle where each ritual run builds on every previous one.

## The Four Rituals

### Daily Plan

**Purpose:** Orient the day using vault evidence, calendar, and pending items.

**Three-part structure:**

1. **Backward look:** What happened since the last plan? Sessions completed, meetings processed, to-dos finished.
2. **Today's plate:** What is on the calendar? What inbox items need processing? What to-dos are pending?
3. **What deserves attention:** Evidence-surfaced coaching -- goals with silence, patterns across days, items past their target date.

**Data flow:**
- **Reads:** Calendar events, vault entity files (goals, projects, areas, people), inbox items (email, messages, journal entries), recent session closures, semantic search queries
- **Writes:** Immutable daily plan file in `rituals/daily-plans/`
- **Coaching voice:** All coaching sections must follow the coaching voice specification (see `coaching-voice.md`)

### Weekly Review

**Purpose:** Structural discovery -- step back from daily execution to see patterns.

**Core activities:**

1. **Goal trajectory assessment:** Green/yellow/red for each goal based on the week's evidence.
2. **Entity file refresh:** Update synthesis fields (pulse, signal, coaching) on areas, goals, projects, people, habits.
3. **Essay cluster identification:** Find atomic notes that cluster around a theme, surfacing potential essay ideas.
4. **System health check:** Orphan detection, stale claims, broken links.
5. **Pattern recognition:** What is the story of this week across all work?

**Data flow:**
- **Reads:** All daily plans for the week, session closures, entity files, zettelkasten notes
- **Writes (immutable):** Weekly review file in `rituals/weekly-reviews/`
- **Writes (mutable):** Updates to entity synthesis fields

**Note:** Entity file updates are the ONLY mutable output. The weekly review file itself is immutable once written.

### Session Closure

**Purpose:** Lightweight capture of what a work session accomplished.

**Structure:** Three sections -- What Was Completed, Key Decisions, Follow-Up Items.

**Length:** 10-15 lines maximum.

**Pattern:** Created as a placeholder early in the session, updated as work progresses, finalized at session end.

**Data flow:**
- **Reads:** Current session context
- **Writes:** Immutable session closure file

### Monthly Review

**Purpose:** Goal trajectory assessment across the full month.

**Core activities:**

1. Goal progress against objectives.
2. System health (accumulated lint defects, orphan count, stale claims).
3. Ritual cadence assessment (were daily plans and weekly reviews run consistently?).
4. Attention allocation (which goals got the most sessions? which got silence?).

**Data flow:**
- **Reads:** All weekly reviews for the month, goal files, system health metrics
- **Writes:** Immutable monthly review file

## The Ritual Chain

```
Daily Plan --> accumulates evidence --> Weekly Review --> synthesizes patterns --> Monthly Review
     ^                                       |                                        |
     |                                       v                                        v
     +--- Entity file updates <--- synthesis field refresh <--- goal trajectory assessment
```

Daily plans generate the raw evidence. Weekly reviews synthesize evidence into patterns and update entity files. Monthly reviews assess goal trajectories and system health. The cycle repeats.

## Data Contracts

| Ritual | Reads | Writes (Immutable) | Writes (Mutable) |
|--------|-------|-------------------|-------------------|
| Daily Plan | Calendar, inboxes, entity files, recent closures | Daily plan file | -- |
| Weekly Review | Daily plans, closures, entity files, ZK notes | Weekly review file | Entity synthesis fields |
| Session Closure | Current session context | Session closure file | -- |
| Monthly Review | Weekly reviews, goal files, metrics | Monthly review file | -- |

## Design Principles

1. **Ritual outputs are immutable.** Once written, a daily plan or weekly review is a historical record. Only entity files get updated.
2. **Evidence over assertion.** Every coaching observation must trace to vault evidence. No inference from absence.
3. **The vault is the memory.** Rituals compile what the vault knows into actionable insight. They do not introduce new information -- they synthesize existing evidence.
4. **Rituals feed each other.** Each level of ritual builds on the one below. Breaking the chain (skipping weekly reviews) degrades monthly review quality.
5. **Silence is a signal.** If a ritual has nothing confident to say about a goal, it says nothing. This is by design.

## Implementing Rituals

Each ritual needs three things to operate:

1. **A data reader** that can traverse vault files, parse frontmatter, and extract structured information from entity files.
2. **A coaching engine** that follows the coaching voice specification (`coaching-voice.md`) and the confidence model (`confidence-model.md`) to decide what to surface.
3. **A file writer** that produces the immutable output file with correct frontmatter and places it in the correct ritual directory.

The ritual chain works regardless of what LLM or automation layer drives it. The contracts above define what each ritual reads and writes -- the implementation can be a Claude skill, a script, or a manual process.
