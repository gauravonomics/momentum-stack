# Momentum Stack

A ritual-driven personal knowledge management architecture for AI agents. The Momentum stack defines how an AI agent should build, maintain, and reason about a personal knowledge vault — including what to store, how to connect it, when to surface it, and when to stay silent.

This repository contains the **system design** — the architecture, schemas, behavioral contracts, and coaching voice that govern a Momentum-compatible vault. It does not contain implementation code, because the implementation varies by platform. The architecture is the hard part; the code is straightforward once you understand the design.

## What This Is (and Isn't)

**What's here:** The knowledge architecture that makes a vault compound over time. Three-layer separation (raw/wiki/schema), 26 entity types with typed frontmatter schemas, 8 semantic relation types with bilateral consistency checking, a coaching voice specification that governs how the AI speaks to the operator, a ritual rhythm (daily/weekly/monthly) that keeps the vault alive, a multi-signal confidence model that decides when to surface suggestions and when to stay silent, and lint categories that detect knowledge integrity defects.

**What's NOT here:** Implementation code. Skills (prompt templates that drive ingestion and rituals) are personal — they encode your workflow, not universal logic. Hook scripts (enforcement automation) are platform-specific — Claude Code uses bash hooks, Cursor uses rules, others use different mechanisms. The semantic search layer is an infrastructure choice, not an architectural one — any vector database works.

**Why release the design without the code?** Because the design is what's hard to figure out. Anyone can write a bash script or a prompt template. What takes months to develop is: which entity types matter, how they relate, what the agent should say (and not say), when to surface information, and how rituals compound knowledge over time. That's what this repo gives you.

## How It Differs

The LLM wiki space has several implementations, each solving part of the problem:

| System | Primary Focus | What It Does Well |
|--------|--------------|-------------------|
| **sage-wiki** | Contradiction detection | Claims database with source tracking, automated consistency checking |
| **SwarmVault** | Multi-agent orchestration | Agent coordination patterns, parallel processing, task decomposition |
| **qmd** | Markdown as database | Structured queries over markdown frontmatter, schema enforcement |
| **nvk/llm-wiki** | Vector-augmented notes | Semantic search integration, embedding-based retrieval |

**Momentum Stack** integrates what these separate:

- **Ritual rhythm** that sage-wiki and qmd lack — systematic daily/weekly/monthly processes that keep the vault alive, not just queryable
- **Coaching voice** that none of them define — a behavioral contract for how the AI communicates (observational, never prescriptive)
- **Typed semantic graph** across all entity types (not just notes) — goals, projects, people, meetings, to-dos all participate in the relation graph
- **Confidence-gated surfacing** — multi-signal scoring that produces silence below threshold, preventing the "helpful but noisy" failure mode
- **Three-layer separation** with schema-driven enforcement — raw sources are immutable, wiki content is compiled and regeneratable, schemas govern everything

These aren't separate features. They're an integrated system: rituals use the confidence model to decide what to surface, the coaching voice governs how to say it, the typed graph enables traversal across entity types, and the three-layer separation ensures everything stays consistent.

## Core Concepts

### Three-Layer Separation

Every file in the vault belongs to exactly one layer:

- **Raw (immutable):** Source material — meeting transcripts, emails, screenshots, web clippings, article highlights. Never modified after creation. This is your evidence base.
- **Wiki (compiled):** Content produced by AI agents from raw sources — entity files (people, projects, goals), zettelkasten notes, ritual outputs. All wiki content must be regeneratable from raw sources + current schema.
- **Schema (constitution):** The specifications that govern wiki operations — entity type definitions, relation type schemas, behavioral rules. Version-controlled and machine-checkable.

### Typed Semantic Graph

Every entity in the vault can declare typed relations to any other entity. Eight relation types (`contradicts`, `supersedes`, `supports`, `references`, `causes`, `trades_off`, `implements`, `derived_from`) enable vault-lint to detect inconsistencies, agents to traverse the knowledge graph, and rituals to discover structural patterns.

Bilateral relations (`contradicts`, `trades_off`) require both entities to declare the relationship. Vault-lint flags unilateral declarations.

### Coaching Voice

The coaching voice specification defines how an AI agent communicates with the vault operator. The core principle: **evidence surfacing is observational and factual — it never scores, judges, or prescribes.** Banned terms include "overdue" (use "past target date"), "action needed" (use "noting for awareness"), and "you should" (surface evidence, let the operator decide). See `coaching-voice.md` for the complete specification.

### Ritual Rhythm

Four rituals keep the vault alive:

1. **Daily Plan** — Three-part morning orientation: what happened since last plan, what's on today's plate, what deserves attention based on vault evidence
2. **Weekly Review** — Structural discovery: goal trajectories, entity file refresh, essay cluster identification, system health
3. **Session Closure** — Lightweight capture of what a work session accomplished (10-15 lines)
4. **Monthly Review** — Goal trajectory assessment, system health, attention allocation patterns

Rituals feed each other: daily plans accumulate evidence, weekly reviews synthesize patterns, monthly reviews assess trajectories. See `ritual-design.md` for the architecture.

### Confidence Gating

The multi-signal confidence model prevents noise. Five signals (recency, entity authority, source count, goal alignment, access recency) combine with per-caller weight profiles. Both relevance and momentum must exceed the caller's threshold for a result to surface. Below threshold = silence, not noise. See `confidence-model.md` for the full model.

## Getting Started

### 1. Copy the starter vault

```bash
git clone https://github.com/gauravonomics/momentum-stack.git
cp -r momentum-stack/starter-vault/ ~/my-vault/
```

The starter vault has the three-layer directory structure with example entities demonstrating all 26 types and all 8 relation types.

### 2. Point your agent at AGENTS.md

Copy `AGENTS.md` to your vault root:

```bash
cp momentum-stack/AGENTS.md ~/my-vault/AGENTS.md
```

This file is the portable schema. Any agent platform that reads `AGENTS.md` can understand and operate on your vault.

### 3. Configure your agent platform

Copy the relevant config stub from `platform-configs/`:

| Platform | Config File | Destination |
|----------|------------|-------------|
| Claude Code | `CLAUDE.md` (in this repo) | `~/my-vault/CLAUDE.md` |
| Cursor | `.cursorrules` | `~/my-vault/.cursor/rules/momentum.md` |
| Windsurf | `.windsurfrules` | `~/my-vault/.windsurfrules` |
| Codex | `codex-instructions.md` | `~/my-vault/CODEX.md` |
| Gemini | `gemini-instructions.md` | `~/my-vault/GEMINI.md` |

### 4. Run your first daily plan

Ask your agent: *"Run a daily plan. Check my calendar, review any pending inbox items, and surface what deserves attention based on vault evidence."*

The agent should:
- Read today's calendar events
- Check raw inbox directories for unprocessed items
- Review recent entity file activity
- Produce an immutable daily plan file following the three-part structure
- Use the coaching voice (observational, not prescriptive)

### 5. Build from here

The starter vault is minimal by design. As you add real content — meeting transcripts, emails, journal entries, notes — the system compounds. Rituals synthesize raw sources into entity files. Typed relations connect entities across types. The confidence model learns which entities are most relevant to your current work.

## Multi-Platform Support

The Momentum architecture is agent-platform-agnostic. `AGENTS.md` is the portable specification. Platform-specific config files extend it with tooling guidance:

- **Claude Code** — Full implementation with skills (prompt templates), hooks (enforcement automation), and MCP integration for semantic search
- **Cursor** — Uses `.cursor/rules/` for project rules, `@file` for AGENTS.md reference, `@folder` for layer-scoped searches
- **Windsurf** — Uses Cascade for multi-file operations and write flows for approval-gated updates
- **Codex** — Uses sandbox mode for vault modifications with review gates
- **Gemini** — Uses `GEMINI.md` for tool mappings and skill activation

All platforms follow the same architectural principles. The behavioral contract, coaching voice, and relation schema apply regardless of which AI agent operates the vault.

## Repository Structure

```
momentum-stack/
  AGENTS.md                  -- Portable schema (the core artifact)
  CLAUDE.md                  -- Claude-specific extensions
  README.md                  -- This file
  LICENSE                    -- MIT License
  coaching-voice.md          -- Coaching voice specification
  ritual-design.md           -- Ritual architecture documentation
  relation-types.md          -- Typed relation specification
  confidence-model.md        -- Multi-signal confidence scoring
  starter-vault/             -- Minimal working vault
    raw/                     -- Immutable source layer
    wiki/                    -- Compiled content layer
    schema/                  -- Schema definitions
  platform-configs/          -- Agent platform config stubs
    .cursorrules
    .windsurfrules
    codex-instructions.md
    gemini-instructions.md
```

## What's NOT Included (and Why)

| Not Included | Why |
|-------------|-----|
| **Skills** (prompt templates) | Skills encode personal workflows — your ingestion process, your coaching preferences, your ritual customizations. They're the most personal part of the system. |
| **Hook scripts** (enforcement automation) | Hooks are platform-specific bash/shell scripts for Claude Code. Other platforms use different enforcement mechanisms. |
| **Semantic search implementation** | Our implementation uses Supabase with pgvector, but any vector database works. The architecture doesn't prescribe infrastructure. |
| **Ritual skill implementations** | The ritual *design* is here (what each ritual does, reads, and writes). The *implementation* (specific prompts, agent coordination) is personal and platform-specific. |
| **Vault content** | No real notes, meetings, people files, or project data. The starter vault has synthetic examples only. |

The boundary is intentional: **architecture is shareable, implementation is personal.** You should build your own skills, hooks, and search layer on top of this architecture. The schema tells you what to build; the code is up to you.

## License

MIT License. See `LICENSE` for details.
