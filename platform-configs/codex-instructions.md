# Momentum Stack -- OpenAI Codex Instructions

Read `AGENTS.md` at the repo root for the complete knowledge architecture specification.

## Core Behavior

- Follow the three-layer separation defined in `AGENTS.md`: raw (immutable), wiki (compiled), schema (constitution)
- Use observational coaching voice in all user-facing output (see `coaching-voice.md`)
- Propose entity updates through an approval gate -- never auto-apply changes to entity files
- Include `source_lineage` in all wiki-layer files tracing back to raw sources
- Stay silent when confidence is below threshold -- no low-quality suggestions (see `confidence-model.md`)
- Use typed relations with the format `{type, target, evidence}` as defined in `relation-types.md`

## Vault Operations

- Prefer editing existing files over creating new ones
- Use the three-layer directory structure: `raw/`, `wiki/`, `schema/`
- Entity frontmatter must follow schemas in the `schema/` directory
- Ritual outputs are immutable historical records -- never modify a daily plan or weekly review after creation
- Only entity files (goals, projects, areas, people) are mutable

## Coaching Voice

When generating any user-facing coaching or suggestions:

- Never use banned terms: "overdue", "at risk", "action needed", "falling behind" (see `coaching-voice.md` for full list)
- Frame observations factually: "Target date was March 15" not "This is overdue by 8 days"
- Surface evidence, never prescribe actions
- Silence is acceptable when confidence is low

## Codex-Specific

- When using `CODEX.md`, reference the `AGENTS.md` sections for entity schemas and behavioral contracts
- Use sandbox mode for vault modifications -- propose changes for review before applying
- Follow the behavioral contract in `AGENTS.md` Section 5 for all operations
- When processing raw sources, create `derived_from` relations tracing wiki content back to raw inputs
- Validate all entity frontmatter against schemas before writing files
