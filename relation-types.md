# Typed Relation Schema

## Overview

Typed relations are structured semantic links between any two vault entities. Unlike plain wikilinks that only say "A mentions B," typed relations carry meaning: A *supports* B, A *contradicts* B, A *implements* B. This semantic layer enables vault-lint to detect inconsistencies, agents to traverse the knowledge graph intelligently, and the operator to understand how their knowledge connects.

Without typed relations, a vault is a flat collection of notes with undifferentiated links. With typed relations, it becomes a semantic graph where contradictions surface automatically, implementation gaps become visible, and evidence chains are traceable from raw source to compiled insight.

## The 8 Relation Types

### Frontmatter Format

Every entity in the vault can declare typed relations in its frontmatter:

```yaml
relations:
  - type: supports
    target: goal-build-knowledge-system
    evidence: "This note provides evidence that compound systems improve decision quality"
  - type: derived_from
    target: raw/transcripts/2026-01-15-meeting
    evidence: "Insight extracted from weekly standup discussion"
```

Each relation has three required fields:
- **type**: One of the 8 defined types below
- **target**: The filename (without extension) of the related entity
- **evidence**: A one-sentence justification for why this relation exists

### contradicts

**Bilateral: Yes** -- Both entities must declare the relation.
**Description:** Source A and source B make opposing claims about the same subject.
**Lint priority:** High -- contradictions are the most important relations to track.

**When to use:**
- Two atomic notes make opposing claims about the same topic
- A literature note presents evidence that challenges an existing insight
- A meeting discussion contradicts a previous decision

**When NOT to use:**
- Entities that discuss different aspects of the same topic (that is not a contradiction)
- Opinions vs. facts -- use only when both are making factual claims
- When one clearly supersedes the other (use `supersedes` instead)

**Bilateral rule:** If note-A declares `contradicts` targeting note-B, then note-B MUST also declare `contradicts` targeting note-A. Vault-lint flags unilateral contradictions.

### supersedes

**Bilateral: No**
**Description:** Source A replaces or updates source B (B is now outdated).
**Lint priority:** Medium

**When to use:**
- A revised decision replaces an earlier one
- Updated research invalidates previous findings
- A new version of a process replaces the old

**When NOT to use:**
- When both sources remain valid (use `supports` or `references` instead)
- When the relationship is disagreement rather than replacement (use `contradicts`)

### supports

**Bilateral: No**
**Description:** Source A provides evidence for a claim in source B.
**Lint priority:** Low

**When to use:**
- An atomic note provides evidence for a goal or project thesis
- A literature note corroborates an existing insight
- Meeting minutes confirm a hypothesis

**When NOT to use:**
- When A merely mentions B without providing evidence (use `references`)
- When A is the raw source that B was compiled from (use `derived_from`)

### references

**Bilateral: No**
**Description:** Source A mentions or cites source B.
**Lint priority:** Low

**When to use:**
- An entity explicitly cites another entity
- A meeting note references a project or person
- A literature note cites its source

**When NOT to use:**
- When A provides substantive evidence for B (use `supports`)
- When A was compiled from B (use `derived_from`)
- Use sparingly -- only for explicit citations, not general topic overlap

### causes

**Bilateral: No**
**Description:** Event or decision in source A led to outcome in source B.
**Lint priority:** Medium

**When to use:**
- A decision in a meeting led to a new project
- An organizational change caused a shift in strategy
- A technical failure led to a process redesign

**When NOT to use:**
- When the causal link is speculative (causes requires documented evidence)
- When A merely preceded B chronologically without causal connection

### trades_off

**Bilateral: Yes** -- Both entities must declare the relation.
**Description:** Source A identifies a trade-off with what source B recommends.
**Lint priority:** Medium

**When to use:**
- Two design approaches that cannot both be pursued
- A decision that improves one metric at the expense of another
- Competing priorities where advancing one necessarily deprioritizes the other

**When NOT to use:**
- When there is no genuine tension between the entities (use `references`)
- When one is simply wrong (use `contradicts`)

**Bilateral rule:** If note-A declares `trades_off` targeting note-B, then note-B MUST also declare `trades_off` targeting note-A. Vault-lint flags unilateral trade-offs.

### implements

**Bilateral: No**
**Description:** Source A is the implementation of the design described in source B.
**Lint priority:** Low

**When to use:**
- A project file implements a goal
- A skill file implements a design spec
- A configuration file implements an architecture decision

**When NOT to use:**
- When A merely references B's design without actually implementing it (use `references`)
- When A extends or builds upon B rather than implementing it (use `supports`)

### derived_from

**Bilateral: No**
**Description:** Source A was synthesized or compiled from source B (raw source).
**Lint priority:** Low

**When to use:**
- A wiki-layer note was compiled from a raw transcript
- An atomic note was extracted from a literature note
- A meeting summary was derived from a raw meeting recording

**When NOT to use:**
- When A merely cites B (use `references`)
- When A provides evidence for B (use `supports` -- the direction matters)

## Vault-Lint Rules

### Bilateral Consistency

For `contradicts` and `trades_off`, vault-lint checks that both participating entities declare the relation. If only one side declares it, lint raises a defect in the `relation-integrity` category.

### Dangling Targets

Vault-lint checks that every `target` in a relation points to an existing file. Missing targets are flagged as defects in the `relation-integrity` category.

### Type Validation

Vault-lint validates that every `type` value is one of the 8 defined types. Unknown types are flagged.

### Evidence Requirement

Every relation must have a non-empty `evidence` field. Relations without evidence are flagged.

### Contradiction Concentration

When multiple entities contradict the same target, vault-lint flags this as a potential knowledge conflict zone requiring resolution. These are high-priority defects.

## Relation Extraction During Ingestion

When processing raw sources (transcripts, emails, articles), agents should:

1. Identify factual claims and check against existing entities.
2. Flag contradictions between new and existing claims.
3. Create `derived_from` relations tracing wiki content to raw sources.
4. Create `supports` relations when new evidence strengthens existing claims.
5. Use `references` sparingly -- only for explicit citations, not general topic overlap.

## YAML Reference

```yaml
relation_types:
  - name: contradicts
    bilateral: true
    lint_priority: high
  - name: supersedes
    bilateral: false
    lint_priority: medium
  - name: supports
    bilateral: false
    lint_priority: low
  - name: references
    bilateral: false
    lint_priority: low
  - name: causes
    bilateral: false
    lint_priority: medium
  - name: trades_off
    bilateral: true
    lint_priority: medium
  - name: implements
    bilateral: false
    lint_priority: low
  - name: derived_from
    bilateral: false
    lint_priority: low
```
