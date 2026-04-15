# CLAUDE.md — Claude-Specific Extensions

> This file extends `AGENTS.md` with Claude Code-specific guidance. Read `AGENTS.md` first for the complete knowledge architecture specification.

## Tool Selection

| Operation | Tool | Why |
|-----------|------|-----|
| Edit file content | Edit tool | Preserves metadata, produces reviewable diffs |
| Move/rename files | `obsidian move` / `obsidian rename` | Updates wikilinks across the vault (if using Obsidian CLI) |
| Set properties | `obsidian property:set` | Through Obsidian's property system (if available) |
| Create new files | Write tool | Safe for new files |
| Search files by name | Glob tool | Fast pattern matching |
| Search file content | Grep tool | Regex-capable content search |
| Multi-file operations | Agent tool (sub-agents) | Parallelize independent work across files |

**Never use `sed -i` or `awk -i` on vault files.** BSD `sed -i` on macOS replaces files (delete + create), destroying creation and modification timestamps.

**Never use `mv` to move vault files.** It breaks wikilinks. Use Obsidian CLI if available, or update all references manually.

## Sub-Agent Strategy

Use the Agent tool for any task involving 3+ files or 2+ vault layers:

- **Ingestion agents:** Process raw sources into wiki entities. Read from `raw/`, write to `wiki/`.
- **Ritual agents:** Run daily plans, weekly reviews. Read across layers, write ritual output files.
- **Lint agents:** Check relation consistency, orphan detection, source traceability.

Sub-agents should return maximum signal, minimum tokens. The 1M context window exists to be used — don't artificially constrain reads.

## Approval Gate Pattern

When proposing entity updates:

1. Read the current state of the target entity file
2. Identify what needs updating (synthesis fields, relations, status)
3. Present the proposed changes to the operator in chat
4. Wait for approval before writing
5. Never auto-apply updates to entity files

This applies to all wiki-layer entity files (people, goals, projects, areas, to-dos, habits). Ritual output files (daily plans, weekly reviews, session closures) are written directly — they're immutable records, not updateable entities.

## Coaching Voice in Claude

Claude has a natural tendency toward helpfulness that can conflict with the coaching voice specification. Specifically:

- Claude defaults to prescriptive suggestions ("You should...") — override with observational framing ("Last vault activity on this goal: 3 days ago")
- Claude defaults to treating silence as concerning — override with the principle that goal silence is information, not failure
- Claude defaults to being comprehensive — override with confidence gating (below threshold = silence)

Read `coaching-voice.md` for the full specification. The banned terms table is non-negotiable.

## Session Protocol

Every work session should:

1. Read `AGENTS.md` for the knowledge architecture
2. Check the ritual calendar (when was the last daily plan? weekly review?)
3. Understand the three-layer separation before modifying any files
4. Create a session closure file when substantive work is done
5. Follow the behavioral contract for all operations

## Relation Extraction

When processing raw sources, Claude should:

1. Identify factual claims and compare against existing wiki entities
2. Flag contradictions (highest priority — bilateral relation required)
3. Create `derived_from` relations from wiki entities to raw sources
4. Propose `supports` relations when evidence strengthens existing claims
5. Use `references` only for explicit citations
6. Run proposed relations through the approval gate before writing
