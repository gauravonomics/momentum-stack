# AGENTS.md -- Momentum Stack Knowledge Architecture

> A portable schema for building ritual-driven personal knowledge management systems with AI agents.
> Any agent platform can read this file to understand and operate on a Momentum-compatible vault.

---

## 1. Three-Layer Separation

A Momentum vault enforces strict separation between raw sources, compiled knowledge, and the schema that governs operations. This separation is the foundational invariant -- every other rule derives from it.

### Raw Layer (Immutable)

Source material that is never modified after creation. Raw files are write-once: an ingestion process creates them, and nothing ever edits them again.

**Directories:**
- `raw/transcripts/` -- Meeting transcripts from recording tools
- `raw/email-inbox/` -- Unprocessed email exports
- `raw/whatsapp-inbox/` -- Unprocessed messaging exports
- `raw/clippings/` -- Web clippings and screenshots
- `raw/highlights/` -- Reading highlights from services like Readwise
- `raw/audio-notes/` -- Voice memos and audio transcriptions

**Rule:** Ingestion processes read from this layer but never write back to it. The only writes are the initial deposit of raw material.

### Wiki Layer (Compiled)

Content compiled from raw sources by AI agents or human operators. This is the working knowledge base -- entity files, zettelkasten notes, ritual outputs, and all structured knowledge.

**Directories:**
- `wiki/areas/` -- Life/work area definitions
- `wiki/goals/` -- Goal files with trajectory tracking
- `wiki/projects/` -- Project folders with status tracking
- `wiki/people/` -- Person profiles
- `wiki/zettelkasten/atomic/` -- Atomic insight notes
- `wiki/zettelkasten/literature/` -- Literature notes from source material
- `wiki/zettelkasten/mocs/` -- Maps of Content (topic clusters)
- `wiki/habits/` -- Habit definitions and tracking
- `wiki/to-dos/` -- Task management (open, delegated, archive subfolders)
- `wiki/skills/` -- Skill definitions
- `wiki/essays/ideas/` -- Essay idea pipeline
- `wiki/essays/mini/` -- Short-form essays
- `wiki/essays/long-form/` -- Long-form essays
- `rituals/daily-plans/` -- Daily plan outputs
- `rituals/weekly-reviews/` -- Weekly review outputs
- `sessions/closures/` -- Session closure records
- `wiki/prompts/` -- Prompt files and sprint sequences
- `wiki/plans/` -- Implementation plans
- `wiki/evals/` -- Companion evaluation rubrics
- `wiki/journal/` -- Journal entries (archive and daily)
- `wiki/reflections/` -- Reflections (archive and daily)
- `wiki/diagnostics/` -- Validation records
- `inputs/meetings/` -- Compiled meeting files (YYYY/MM subfolders)
- `inputs/email/` -- Compiled email files (YYYY/MM subfolders)
- `inputs/whatsapp/` -- Compiled WhatsApp files (YYYY/MM subfolders)

**Rule:** All wiki content must be regeneratable from raw sources + current schema. Wiki pages carry a `source_lineage` field tracing back to raw inputs.

### Schema Layer (Constitution)

The specifications that govern how the wiki layer operates. Entity type schemas, relation type definitions, behavioral rules, and quality controls.

**Directories:**
- `schema/entity-types/` -- YAML definitions for every entity type
- `schema/relation-types.yaml` -- The 8 typed relation definitions
- `schema/coaching-voice.md` -- Coaching voice specification
- `AGENTS.md` -- This file (the portable schema)

**Rule:** Schema files govern wiki operations. Wiki files reference schema definitions but never duplicate them.

### Invariants

1. **Ingestion processes never modify the raw layer.** They read raw files and write compiled entities to the wiki layer.
2. **Wiki pages carry a `source_lineage` field** tracing back to raw inputs. Broken lineage is a lint defect.
3. **Schema layer files govern wiki layer operations.** Wiki layer files reference (never duplicate) schema definitions.
4. **Lint validates raw-to-wiki lineage** on every pass and routes broken lineage to the gaps backlog.

---

## 2. Entity Type System

The vault contains 26 entity types. Each has a defined schema with required and optional frontmatter fields, a directory location, and common relation types. Fields marked *Synthesis* are written by ritual processes (weekly review, daily planning) and are not required at entity creation time.

All frontmatter uses YAML format between `---` delimiters at the top of a Markdown file.

### Field Type Reference

| Type | Format | Example |
|------|--------|---------|
| `string` | Plain text | `"Build a knowledge system"` |
| `date` | ISO date, optionally with time | `2026-01-15` or `"2026-01-15 09:30 IST"` |
| `number` | Integer or decimal | `1`, `0.5` |
| `link` | Wikilink to another entity file | `"[[goal-build-knowledge-system]]"` |
| `array` | YAML list. Items may be strings, links, or objects depending on field. | `["tag-a", "tag-b"]` or `["[[goal-1]]", "[[goal-2]]"]` |
| `wikilink` | Same as `link` -- alternate name used in some schemas | `"[[learning-growth]]"` |

**Link resolution:** Links use the `[[filename]]` format where `filename` is the target file's name without path or extension. Agents resolve links by searching for the matching file across all vault directories.

**File naming convention:** All vault files use kebab-case (lowercase, numbers, hyphens). Example: `compound-knowledge-systems.md`, `goal-build-knowledge-system.md`.

---

### 2.1 Area

An area of life or work that contains goals, projects, and habits. Areas are the highest-level organizational unit.

**Directory:** `wiki/areas/`

**Required fields:**

| Field | Type | Description |
|-------|------|-------------|
| type | string | Always `area` |
| status | string | `active` or `dormant` |
| goals | array | Links to goal files this area serves |
| created | date | Date created |

**Optional fields:**

| Field | Type | Description |
|-------|------|-------------|
| also_serves_goals | array | Links to secondary goal files |
| habits | array | Links to habit files associated with this area |
| pulse | string | *Synthesis:* current activity level and what is driving it |
| signal | string | *Synthesis:* what the evidence pattern means |
| coaching | string | *Synthesis:* recommended next action |
| relations | array | Typed semantic relations (see Section 4: Relation Schema) |

**Validation:** `goals` must contain at least one link. Synthesis fields are written by weekly review, not required at creation.

**Common relations:** `supports` (area supports goals), `trades_off` (competing priorities between areas)

---

### 2.2 Goal

A time-bound objective within a quarter or month. Goals have priority rankings and connect upward to areas and downward to projects.

**Directory:** `wiki/goals/`

**Required fields:**

| Field | Type | Description |
|-------|------|-------------|
| title | string | Human-readable goal name |
| goal_number | number | Goal number for ordering |
| quarter | string | Quarter this goal belongs to (e.g., `Q1 2026`) |
| objective | string | One-sentence description of what achieving this goal means |
| status | string | `in-progress`, `complete`, or `deferred` |
| priority | number | Priority rank (1 = highest priority) |
| deadline | date | Quarter-end deadline |
| created | date | Date created |
| primary_area | link | Link to the primary area this goal belongs to |

**Optional fields:**

| Field | Type | Description |
|-------|------|-------------|
| also_serves | array | Links to secondary areas this goal also serves |
| evidence | string | *Synthesis:* factual evidence of progress |
| signal | string | *Synthesis:* what the evidence pattern means |
| coaching | string | *Synthesis:* recommended next action |
| relations | array | Typed semantic relations (see Section 4) |

**Validation:** `goal_number` and `priority` must be positive integers. `status` must be one of the allowed values. Synthesis fields are written by weekly review.

**Common relations:** `supports` (from projects), `trades_off` (between competing goals), `supersedes` (when goals evolve across quarters)

---

### 2.3 Project

A concrete body of work that advances a goal. Projects live in status-based subdirectories and connect to goals and areas.

**Directory:** `wiki/projects/{active,ongoing,paused,archived}/`

**Required fields:**

| Field | Type | Description |
|-------|------|-------------|
| type | string | Always `project` |
| created | date | Date created |
| status | string | `active`, `ongoing`, `paused`, or `archived` |
| primary_area | link | Link to the primary area |
| primary_goal | link | Link to the primary goal this project advances |

**Optional fields:**

| Field | Type | Description |
|-------|------|-------------|
| also_serves | array | Links to other goals this project contributes to |
| target_date | date | Target completion date (active projects only) |
| progress | string | *Synthesis:* factual progress summary |
| signal | string | *Synthesis:* what the evidence pattern means |
| coaching | string | *Synthesis:* recommended next action |
| relations | array | Typed semantic relations (see Section 4) |

**Validation:** `primary_area` and `primary_goal` must link to existing files. Synthesis fields are written by weekly review.

**Common relations:** `implements` (project implements a goal), `supports` (project supports multiple goals), `causes` (project decision led to outcome)

---

### 2.4 Person

A profile for someone the vault operator interacts with. Captures role, organization, and interaction patterns.

**Directory:** `wiki/people/`

**Required fields:**

| Field | Type | Description |
|-------|------|-------------|
| type | string | Always `person` |
| role | string | Professional role and organization |
| nickname | string | Short name used in conversation |

**Optional fields:**

| Field | Type | Description |
|-------|------|-------------|
| organization | string | Company or organization |
| primary_area | link | Link to the most relevant area |
| created | date | Date the profile was created |
| last_context | string | *Synthesis:* most recent interaction context with date |
| signal | string | *Synthesis:* what the relationship pattern means |
| coaching | string | *Synthesis:* recommended next interaction |
| relations | array | Typed semantic relations (see Section 4) |

**Validation:** `nickname` must not be empty. Synthesis fields are written by weekly review.

**Common relations:** `references` (person referenced in meetings), `supports` (person supports a project), `causes` (person's decision caused an outcome)

---

### 2.5 Meeting

A compiled meeting transcript with entity linking. Created by a transcript processing skill from raw meeting recordings.

**Directory:** `inputs/meetings/{YYYY}/{MM}/`

**Required fields:**

| Field | Type | Description |
|-------|------|-------------|
| created | date | Date of the meeting |
| type | string | Always `meeting-transcript` |
| source | string | Source tool (e.g., `transcript-tool`) |

**Optional fields:**

| Field | Type | Description |
|-------|------|-------------|
| meeting_id | string | Source tool meeting UUID |
| areas | array | Links to areas this meeting relates to |
| goals | array | Links to goals this meeting advances |
| projects | array | Links to projects discussed |
| people | array | Links to people who attended |
| tags | array | Hashtag tags for categorization |
| essence | string | *Synthesis:* what happened in the meeting |
| signal | string | *Synthesis:* what this meeting means for goals/projects |
| coaching | string | *Synthesis:* recommended follow-up actions |
| relations | array | Typed semantic relations (see Section 4) |

**Validation:** Meeting files are **immutable after creation** (ritual output rule). The body contains the full meeting transcript content.

**Common relations:** `references` (meeting references entities), `causes` (meeting decision caused outcome), `derived_from` (compiled from raw transcript)

---

### 2.6 Communication (Email)

A compiled email record with entity linking. Created by an email processing skill from raw email exports.

**Directory:** `inputs/email/{YYYY}/{MM}/`

**Required fields:**

| Field | Type | Description |
|-------|------|-------------|
| created | date | Date the email was processed |
| type | string | Always `email` |
| source | string | Always `email` |
| schema_version | number | Schema version, currently `1` |
| subject | string | Email subject line |
| from | string | Sender name |
| to | array | Recipient names |

**Optional fields:**

| Field | Type | Description |
|-------|------|-------------|
| cc | array | CC recipient names |
| areas | array | Links to areas |
| goals | array | Links to goals |
| projects | array | Links to projects |
| people | array | Links to people |
| tags | array | Hashtag tags |
| essence | string | *Synthesis:* what happened in the exchange |
| signal | string | *Synthesis:* what this means for goals/projects |
| coaching | string | *Synthesis:* recommended follow-up |
| relations | array | Typed semantic relations (see Section 4) |

**Validation:** Communication files are **immutable after creation**.

**Common relations:** `references` (email references entities), `causes` (email triggered action), `derived_from` (compiled from raw email)

---

### 2.7 Communication (WhatsApp)

A compiled WhatsApp message record with entity linking. Created by a messaging processing skill from raw message exports.

**Directory:** `inputs/whatsapp/{YYYY}/{MM}/`

**Required fields:**

| Field | Type | Description |
|-------|------|-------------|
| created | date | Date the message was processed |
| type | string | Always `whatsapp` |
| source | string | Always `whatsapp` |
| schema_version | number | Schema version, currently `1` |
| from | string | Sender name |

**Optional fields:**

| Field | Type | Description |
|-------|------|-------------|
| areas | array | Links to areas |
| goals | array | Links to goals |
| projects | array | Links to projects |
| people | array | Links to people |
| tags | array | Hashtag tags |
| essence | string | *Synthesis:* what the message is about |
| signal | string | *Synthesis:* what this means for goals/projects |
| coaching | string | *Synthesis:* recommended follow-up |
| relations | array | Typed semantic relations (see Section 4) |

**Validation:** Communication files are **immutable after creation**.

**Common relations:** `references` (message references entities), `causes` (message triggered action), `derived_from` (compiled from raw message)

---

### 2.8 Habit

A recurring behavior to track, tied to an area and goal. Habits have machine-readable verification rules and frequency.

**Directory:** `wiki/habits/`

**Required fields:**

| Field | Type | Description |
|-------|------|-------------|
| created | date | Date created |
| type | string | Always `habit` |
| status | string | `active`, `paused`, or `retired` |
| area | link | Link to the area this habit belongs to |
| goal | link | Link to the goal this habit tracks |
| rule | string | Machine-readable verification rule (what to count and where) |
| frequency | string | How often to verify: `daily`, `weekly`, or `monthly` |
| verification | string | What passing looks like (e.g., "5 weekday essay files with published status") |

**Optional fields:**

| Field | Type | Description |
|-------|------|-------------|
| last_verified | date | Date of last verification check |
| streak | number | Current streak count |
| schema_version | number | Schema version number |
| evidence | string | *Synthesis:* factual evidence of habit performance |
| signal | string | *Synthesis:* what the performance pattern means |
| coaching | string | *Synthesis:* recommended action to maintain/restore the habit |
| relations | array | Typed semantic relations (see Section 4) |

**Validation:** `frequency` must be one of: `daily`, `weekly`, `monthly`. Synthesis fields are written by weekly review.

**Common relations:** `implements` (habit implements a goal), `supports` (habit supports an area), `trades_off` (habit trades off with another habit for time)

---

### 2.9 Daily Plan

A ritual output capturing the day's backward look, current plate, and coaching. Daily plans are the primary daily ritual artifact.

**Directory:** `rituals/daily-plans/`

**Required fields:**

| Field | Type | Description |
|-------|------|-------------|
| created | date | Date of the plan |
| type | string | Always `daily-plan` |

**Optional fields:**

| Field | Type | Description |
|-------|------|-------------|
| focus | string | One-sentence focus statement for the day |
| coaching | string | AI-generated coaching for the day's priorities |
| meetings_count | number | Number of meetings today |
| todos_proposed | number | Number of to-dos proposed for action today |
| blockers_count | number | Number of blockers identified |
| signals_count | number | Number of evidence signals surfaced |
| primary_goal | link | Link to the day's primary goal |
| areas_touched | array | Links to areas this plan covers |
| calendar_events | array | Array of calendar event objects (summary, start, end, attendees, location, all_day) |
| relations | array | Typed semantic relations (see Section 4) |

**Validation:** Daily plans are **immutable after creation** (ritual output rule).

**Common relations:** `references` (plan references entities active that day), `derived_from` (compiled from calendar, inbox, vault evidence)

---

### 2.10 Daily Closure

> **DEPRECATED.** Daily Closure is superseded by Daily Plan, which now absorbs backward-look and forward-look functions into a single ritual artifact. Retained here for schema completeness and backward compatibility with existing files.

**Directory:** `rituals/daily-closures/`

**Required fields:**

| Field | Type | Description |
|-------|------|-------------|
| created | date | Date of the closure |
| type | string | Always `daily-closure` |

**Optional fields:**

| Field | Type | Description |
|-------|------|-------------|
| story | string | One-paragraph narrative of the day |
| goals_touched | array | Links to all goals advanced today |
| areas_touched | array | Links to all areas involved today |
| meetings_processed | number | Number of meetings processed into vault |
| todos_completed | number | Number of to-dos completed |
| todos_proposed | number | Number of new to-dos proposed |
| synthesis_fields_written | number | Number of entity files updated with synthesis fields |
| coaching | string | End-of-day coaching on attention allocation |
| relations | array | Typed semantic relations (see Section 4) |

**Validation:** Daily closures are **immutable after creation**.

**Common relations:** `supersedes` (superseded by Daily Plan type), `references` (closure references entities touched that day)

---

### 2.11 Weekly Review

A ritual output that synthesizes a full week of activity. Performs structural discovery: goal trajectory, entity file refresh, essay cluster identification, system health.

**Directory:** `rituals/weekly-reviews/`

**Required fields:**

| Field | Type | Description |
|-------|------|-------------|
| created | date | Date the review was created |
| type | string | Always `weekly-review` |
| week_of | date | Monday of the week being reviewed |

**Optional fields:**

| Field | Type | Description |
|-------|------|-------------|
| story | string | One-paragraph narrative of the week |
| goals_green | number | Count of goals at green status |
| goals_yellow | number | Count of goals at yellow status |
| goals_red | number | Count of goals at red status |
| todos_created | number | To-dos created this week |
| todos_completed | number | To-dos completed this week |
| notes_filed | number | Zettelkasten notes filed this week |
| areas_dark | array | Links to areas with zero activity |
| coaching | string | End-of-week coaching on attention allocation |
| relations | array | Typed semantic relations (see Section 4) |

**Validation:** `week_of` must be a Monday date. Weekly reviews are **immutable after creation**.

**Common relations:** `references` (review references all active entities), `derived_from` (compiled from session closures, daily plans, entity files)

---

### 2.12 Zettelkasten Note (Atomic)

A single atomic insight -- one idea per note. The core knowledge unit of the zettelkasten. Created from literature notes, meetings, reflections, or original thinking.

**Directory:** `wiki/zettelkasten/atomic/`

**Required fields:**

| Field | Type | Description |
|-------|------|-------------|
| type | string | Always `atomic` |
| created | date | Date created |
| source | string | Where this insight came from (literature note, meeting, etc.) |

**Optional fields:**

| Field | Type | Description |
|-------|------|-------------|
| tags | array | Categorization tags |
| thesis | string | One-sentence summary of the core idea |
| signal | string | *Synthesis:* evergreen status and connection strength |
| essay_potential | string | *Synthesis:* readiness for essay conversion |
| linked_essays | array | Links to essay ideas that use this note |
| source_lineage | link | Link to the raw source file this note was derived from. Checked by lint (see Section 8.5). |
| relations | array | Typed semantic relations (see Section 4) |

**Validation:** Title must express one idea clearly (zettelkasten principle). Body must contain a Links section with at least one link (no orphan notes). Synthesis fields are written by weekly review. Notes lacking `source_lineage` or a `derived_from` relation are flagged by lint.

**Common relations:** `supports` (note supports a claim in another note), `contradicts` (opposing claim), `derived_from` (extracted from literature note or meeting), `references` (cites another note)

---

### 2.13 Literature Note

A structured summary of a source material (book, article, video, podcast, course, report). Literature notes are the bridge between raw highlights and atomic notes.

**Directory:** `wiki/zettelkasten/literature/`

**Required fields:**

| Field | Type | Description |
|-------|------|-------------|
| created | date | Date created |
| type | string | Always `literature` |
| source_type | string | Type of source: `book`, `article`, `video`, `podcast`, `course`, `report` |
| title | string | Title of the source material |

**Optional fields:**

| Field | Type | Description |
|-------|------|-------------|
| author | string | Author or creator of the source |
| tags | array | Categorization tags |
| url | string | URL for online sources |
| relations | array | Typed semantic relations (see Section 4) |

**Validation:** `source_type` must be one of the allowed values. Body should contain Summary, Key Themes/Insights, and Potential Atomic Notes sections.

**Common relations:** `derived_from` (literature note derived from raw highlights), `references` (cites another source), `supports` (provides evidence for an atomic note)

---

### 2.14 Map of Content (MOC)

A topic cluster that organizes atomic notes by subtopic. MOCs are navigation hubs, not content containers.

**Directory:** `wiki/zettelkasten/mocs/`

**Required fields:**

| Field | Type | Description |
|-------|------|-------------|
| created | date | Date created |
| type | string | Always `moc` |

**Optional fields:**

| Field | Type | Description |
|-------|------|-------------|
| tags | array | Categorization tags |
| relations | array | Typed semantic relations (see Section 4) |

**Validation:** Body must contain an Overview section explaining the topic cluster and links to atomic notes organized by subtopic.

**Common relations:** `references` (MOC references all its atomic notes), `supports` (MOC supports understanding of a goal or area)

---

### 2.15 Skill

A defined capability that an AI agent can invoke. Skills have names, descriptions, triggers, and versioning.

**Directory:** `wiki/skills/{skill-name}/`

**Required fields:**

| Field | Type | Description |
|-------|------|-------------|
| name | string | Skill name (kebab-case, matches directory name) |
| description | string | What the skill does, when to invoke it, what it produces |
| version | string | Semantic version number |
| created | date | Date first created |
| updated | date | Date of most recent update |

**Optional fields:**

| Field | Type | Description |
|-------|------|-------------|
| does | string | Short description of what the skill does |
| triggers | string | Keywords or conditions that trigger this skill |
| produces | string | What artifacts the skill produces |
| category | string | Skill category (management, ritual, processing, etc.) |
| status | string | `active`, `deprecated`, or `draft` |
| usage_signal | string | When and how often this skill runs |
| coaching | string | Usage guidance for the skill |
| relations | array | Typed semantic relations (see Section 4) |

**Validation:** `name` must match the directory name. `version` must follow semantic versioning. `description` must specify triggers and outputs.

**Common relations:** `implements` (skill implements a ritual), `supersedes` (new skill version supersedes old), `references` (skill references entity schemas)

---

### 2.16 Session Closure

A lightweight capture of what a work session accomplished. Created as a placeholder early in a session, updated as work progresses.

**Directory:** `sessions/closures/`

**Required fields:**

| Field | Type | Description |
|-------|------|-------------|
| created | string | Date and optional time (YYYY-MM-DD or YYYY-MM-DD HH:MM) |
| type | string | Always `closure` |
| scope | string | One-sentence description of what the session accomplished |
| primary_goal | link | Link to the primary goal this session advanced |

**Optional fields:**

| Field | Type | Description |
|-------|------|-------------|
| project | link | Link to the project when session is primarily about one project |
| relations | array | Typed semantic relations (see Section 4) |

**Validation:** `scope` must be non-empty. `primary_goal` must use actual goal name, never a number. Body must have three sections: What Was Completed, Key Decisions, Follow-Up Items. Maximum 10-15 lines total.

**Common relations:** `references` (closure references entities worked on), `implements` (session implemented part of a project)

---

### 2.17 Prompt

A structured instruction for an AI agent to execute. Prompts can be standalone or part of a sprint sequence.

**Directory:** `wiki/prompts/` (standalone) or `wiki/prompts/sprint-{name}/` (sprint sequences)

**Required fields:**

| Field | Type | Description |
|-------|------|-------------|
| created | string | Date or datetime of creation |
| type | string | Always `prompt` |

**Optional fields:**

| Field | Type | Description |
|-------|------|-------------|
| purpose | string | What this prompt accomplishes |
| estimated_sessions | string | Estimated session count to execute |
| depends_on | string | Prerequisites (skills, other prompts) |
| destination | string | Target environment (e.g., `Code` or `Cowork`) |
| complexity | string | Execution complexity: `quick`, `medium`, `complex` |
| primary_goal | link | Link to the primary goal |
| project | link | Link to the related project |
| plan | link | Link to the parent plan (if prompt originated from a plan) |
| eval | link | Link to the companion eval |
| sequence | string | Position in sprint sequence (e.g., `2 of 5`) |
| relations | array | Typed semantic relations (see Section 4) |

**Validation:** Sprint prompts in subfolders should have `sequence` and `depends_on` fields.

**Common relations:** `implements` (prompt implements a plan step), `references` (prompt references skills or schemas), `derived_from` (prompt derived from a plan)

---

### 2.18 Validation

A diagnostic record from a validation run. Tracks per-session correctness checks.

**Directory:** `wiki/diagnostics/`

**Required fields:**

| Field | Type | Description |
|-------|------|-------------|
| type | string | Always `validation` |
| date | date | Date of the validation run |

**Optional fields:**

| Field | Type | Description |
|-------|------|-------------|
| sessions | number | Number of sessions validated |
| last_updated | string | Timestamp of last update |
| relations | array | Typed semantic relations (see Section 4) |

**Validation:** Body contains per-session validation entries. Each entry has: Ran correctly, Failed, Failure mode, Verification flags.

**Common relations:** `references` (validation references the sessions it checked)

---

### 2.19 Essay Idea

A candidate essay topic in the writing pipeline. Ideas are rated, tiered, and connected to atomic notes that could feed the essay.

**Directory:** `wiki/essays/ideas/`

**Required fields:**

| Field | Type | Description |
|-------|------|-------------|
| created | date | Date created |
| type | string | Always `essay-idea` |
| title | string | Working title of the essay |
| tier | number | Priority tier: `1` (highest) to `3` |
| status | string | Pipeline status: `idea`, `ready`, `drafted`, `published` |

**Optional fields:**

| Field | Type | Description |
|-------|------|-------------|
| idea_number | number | Sequential idea number |
| source | string | Where the idea came from |
| rating_portfolio_fit | number | Rating 1-5 for portfolio fit |
| rating_originality | number | Rating 1-5 for originality |
| rating_practitioner_depth | number | Rating 1-5 for practitioner depth |
| rating_audience_value | number | Rating 1-5 for audience value |
| rating_readiness | number | Rating 1-5 for readiness to write |
| total_score | number | Sum of all ratings |
| primary_area | link | Link to the primary area |
| primary_goal | link | Link to the primary goal |
| atomic_sources | array | Links to zettelkasten atomic notes that feed this essay |
| relations | array | Typed semantic relations (see Section 4) |

**Validation:** `tier` must be 1, 2, or 3. `status` must be one of the allowed values. Tier 1 ideas should have `atomic_sources` populated.

**Common relations:** `derived_from` (idea derived from atomic notes), `supports` (idea supports a writing goal), `references` (idea references source material)

---

### 2.20 Mini Essay

A short-form essay (typically a social media post or brief article). Lives in date-organized subfolders with supporting artifacts.

**Directory:** `wiki/essays/mini/{YYYY}/{MM}/`

**Required fields:**

| Field | Type | Description |
|-------|------|-------------|
| created | date | Date created |
| status | string | Pipeline status: `draft`, `ready`, `published` |

**Optional fields:**

| Field | Type | Description |
|-------|------|-------------|
| title | string | Essay title (often in body header instead of frontmatter) |
| type | string | Content type hint (e.g., `LinkedIn Post`) |
| primary_area | link | Link to the primary area |
| primary_goal | link | Link to the primary goal |
| essay_idea | link | Link to the source essay idea |
| atomic_sources | array | Links to zettelkasten notes that feed this essay |
| published_links | array | URLs where this essay was published |
| relations | array | Typed semantic relations (see Section 4) |

**Validation:** `status` must be one of the allowed values. Only set status to `published` after the operator has provided actual published URLs. Body contains the full essay text.

**Common relations:** `derived_from` (essay derived from essay idea and atomic notes), `implements` (essay implements an essay idea), `references` (essay references source material)

---

### 2.21 Long-Form Essay

A long-form essay or newsletter issue. Lives in issue-organized subfolders with supporting files (reviews, drafts, style guides).

**Directory:** `wiki/essays/long-form/`

**Required fields:**

| Field | Type | Description |
|-------|------|-------------|
| created | date | Date created |
| status | string | Pipeline status: `draft`, `ready`, `published` |

**Optional fields:**

| Field | Type | Description |
|-------|------|-------------|
| title | string | Essay title |
| issue_number | number | Newsletter issue number |
| primary_area | link | Link to the primary area |
| primary_goal | link | Link to the primary goal |
| atomic_sources | array | Links to zettelkasten notes that feed this essay |
| published_links | array | URLs where this essay was published |
| relations | array | Typed semantic relations (see Section 4) |

**Validation:** `status` must be one of the allowed values. Only set status to `published` after the operator has provided actual published URLs. Body contains the full essay text.

**Common relations:** `derived_from` (essay derived from atomic notes), `implements` (essay implements an essay idea), `supersedes` (revised edition supersedes earlier draft)

---

### 2.22 Plan

An implementation plan that must always have a companion eval. Plans go through a lifecycle from draft to execution.

**Directory:** `wiki/plans/`

**Required fields:**

| Field | Type | Description |
|-------|------|-------------|
| created | string | Date or datetime of creation |
| type | string | Always `plan` |
| scope | string | One-sentence description of what this plan produces |
| eval | link | Link to the companion eval (every plan must have one) |
| status | string | `draft`, `approved`, `executed`, or `superseded` |

**Optional fields:**

| Field | Type | Description |
|-------|------|-------------|
| primary_goal | link | Link to the primary goal |
| project | link | Link to the related project |
| destination | string | Target environment (e.g., `Code` or `Cowork`) |
| complexity | string | Execution complexity: `medium` or `complex` |
| relations | array | Typed semantic relations (see Section 4) |

**Validation:** Every plan must have a companion eval linked via the `eval` field. `status` must be one of the allowed values.

**Common relations:** `implements` (plan implements a goal or project), `references` (plan references skills and schemas), `supersedes` (new plan supersedes old)

---

### 2.23 Eval

A pre-execution quality rubric that defines what "done well" looks like. Always paired with a companion plan.

**Directory:** `wiki/evals/`

**Required fields:**

| Field | Type | Description |
|-------|------|-------------|
| created | string | Date or datetime of creation |
| type | string | Always `eval` |
| plan | link | Link to the companion plan |
| status | string | `draft`, `active`, or `completed` |

**Optional fields:**

| Field | Type | Description |
|-------|------|-------------|
| primary_goal | link | Link to the primary goal |
| project | link | Link to the related project |
| domains | array | Relevant certification domains |
| relations | array | Typed semantic relations (see Section 4) |

**Validation:** Every eval must link to its companion plan. `status` must be one of the allowed values. Body must contain Purpose, Dimensions, Overall Quality Gate, and Verification Steps sections.

**Common relations:** `references` (eval references the plan it validates), `implements` (eval implements quality standards for a project)

---

### 2.24 Journal

Personal journal entries capturing thoughts, energy, and attention patterns. Two sub-types: individual archive entries (with time-of-day) and aggregated daily journals.

**Directory:** `wiki/journal/archive/{YYYY-MM-DD}/` (individual entries) and `wiki/journal/daily/` (daily aggregates)

**Archive entry required fields:**

| Field | Type | Description |
|-------|------|-------------|
| created | string | Date and time: YYYY-MM-DD HH:MM |
| type | string | Always `journal` |
| period | string | Time of day: `Morning`, `Afternoon`, `Evening`, `Late Night` |

**Daily journal required fields:**

| Field | Type | Description |
|-------|------|-------------|
| created | string | Date: YYYY-MM-DD |
| type | string | Always `journal` |
| day | string | Day of the week |

**Optional fields (both sub-types):**

| Field | Type | Description |
|-------|------|-------------|
| areas | array | Links to areas referenced |
| goals | array | Links to goals referenced |
| projects | array | Links to projects referenced |
| people | array | Links to people mentioned |
| essence | string | One-sentence summary of the entry |
| signal | string | What this entry reveals about attention/energy |
| coaching | string | Contextual coaching based on the entry |
| relations | array | Typed semantic relations (see Section 4) |

**Validation:** Both sub-types are **immutable after creation**. Archive entries require `period`; daily journals require `day`.

**Common relations:** `references` (journal references entities mentioned), `derived_from` (daily journal derived from archive entries)

---

### 2.25 Reflection

A reflective note capturing what the operator is thinking, trying, or building. Two sub-types: individual archive entries (typed) and aggregated daily indexes.

**Directory:** `wiki/reflections/archive/{YYYY-MM-DD}/` (individual entries) and `wiki/reflections/daily/` (daily indexes)

**Archive entry required fields:**

| Field | Type | Description |
|-------|------|-------------|
| created | string | Date and time: YYYY-MM-DD HH:MM (or YYYY-MM-DD if time unknown) |
| type | string | Reflection type: `ai-practitioner`, `general` |

**Daily index required fields:**

| Field | Type | Description |
|-------|------|-------------|
| created | string | Date: YYYY-MM-DD |

**Optional fields (both sub-types):**

| Field | Type | Description |
|-------|------|-------------|
| notes_count | number | Number of reflections that day (daily index only) |
| areas | array | Links to areas referenced |
| goals | array | Links to goals referenced |
| projects | array | Links to projects referenced |
| people | array | Links to people mentioned |
| essence | string | One-sentence summary |
| signal | string | Attention or behavioral pattern |
| coaching | string | Coaching observation |
| relations | array | Typed semantic relations (see Section 4) |

**Validation:** Both sub-types are **immutable after creation**. Archive entries require `type`. Reflections should be 3-4 short paragraphs in warm first-person voice -- not technical reports.

**Common relations:** `references` (reflection references entities), `derived_from` (reflection derived from journal entries or meetings), `supports` (reflection supports an insight in an atomic note)

---

### 2.26 To-Do

A task with priority, status, and connections to areas, goals, and projects. To-dos live in status-based subdirectories and support delegation and dependency tracking.

**Directory:** `wiki/to-dos/{open,delegated,archive}/`

**Required fields:**

| Field | Type | Description |
|-------|------|-------------|
| created | date | Date created |
| status | string | `open`, `someday`, `delegated`, `done`, or `discarded` |
| primary_area | link | Link to the primary area |
| priority | string | Priority level: `p1`, `p2`, `p3` |

**Conditionally required fields:**

| Condition | Field | Type | Description |
|-----------|-------|------|-------------|
| status is `delegated` | delegated_to | string | Person this to-do is delegated to |
| status is `delegated` | review_date | date | Date to review delegated to-do |

**Optional fields:**

| Field | Type | Description |
|-------|------|-------------|
| todo_name | string | Human-readable to-do name |
| project | link | Link to the project this to-do belongs to |
| primary_goal | link | Link to the primary goal |
| effort | string | Effort estimate: `small`, `medium`, `large` |
| source | string | Origin of this to-do |
| context | string | Brief context for why this matters |
| leverage | string | How this to-do creates leverage across goals |
| approach | string | Recommended approach to complete |
| people | array | Full names of people involved |
| completed | date | Date completed (only for status `done`) |
| blocked_by | array | Links to to-dos that must complete first |
| blocks | array | Links to to-dos this blocks |
| relations | array | Typed semantic relations (see Section 4) |

**Validation:** File location must match status: open/someday in `open/`, delegated in `delegated/`, done/discarded in `archive/`. Delegated to-dos require `delegated_to`. Done to-dos require `completed` date.

**Common relations:** `implements` (to-do implements a project milestone), `supports` (to-do supports a goal), `causes` (completing this to-do causes another to unblock), `references` (to-do references a meeting or decision)

---

## 3. Common Entity Patterns

### Source Lineage

Every wiki-layer entity should include provenance tracing back to its raw source. This is typically expressed through:
- `source` field (naming the raw source type)
- `source_lineage` field (path or identifier of the raw file)
- `derived_from` relation (typed relation pointing to the raw source)

Broken lineage (empty `source_lineage` pointing to a missing file) is a lint defect.

### Synthesis Fields

Many entity types share a common synthesis pattern -- fields written by ritual processes (especially weekly review) rather than at creation time:

- **evidence** / **progress** / **pulse**: Factual description of what is happening
- **signal**: What the evidence pattern means
- **coaching**: Recommended next action (always in observational voice -- see Section 6)

These fields are never required at creation. They accumulate over time as rituals process vault evidence.

### Entity Linking

Entities connect to each other through:
1. **Direct link fields** (`primary_area`, `primary_goal`, `project`, `people`, etc.) -- structural connections
2. **Array link fields** (`areas`, `goals`, `projects`, `also_serves`, etc.) -- multi-target connections
3. **Typed relations** (`relations` array) -- semantic connections with evidence (see Section 4)

---

## 4. Typed Relation Schema

The vault supports 8 typed semantic relations that any entity can declare toward any other entity. Relations create a semantic graph layer on top of structural links.

### Relation Field Format

```yaml
relations:
  - type: "supports"
    target: "filename-of-target-entity"
    evidence: "One sentence explaining why this relation exists."
  - type: "contradicts"
    target: "another-entity-filename"
    evidence: "These two sources make opposing claims about X."
```

Every relation entry has three fields:
- **type**: One of the 8 relation types below
- **target**: The filename (without path) of the target entity
- **evidence**: A single sentence explaining why this relation exists

---

### 4.1 `contradicts`

**Description:** Source A and source B make opposing claims about the same subject.

**Bilateral:** Yes -- when A declares `contradicts` B, then B must also declare `contradicts` A. Both entities carry the relation.

**Lint priority:** High

**When to use:**
- Two atomic notes assert opposite conclusions about the same topic
- A meeting transcript records a decision that contradicts a previously documented position
- A new literature note contains findings that oppose an existing atomic note's thesis

**When NOT to use:**
- Sources discuss different aspects of the same topic (that is `references`, not `contradicts`)
- One source simply updates or replaces another (that is `supersedes`)
- Sources present different but compatible perspectives

**Lint checks:** Bilateral consistency (if A contradicts B, B must contradict A). Contradiction concentration (entities with 3+ contradiction relations are flagged for resolution).

---

### 4.2 `supersedes`

**Description:** Source A replaces or updates source B (B is now outdated).

**Bilateral:** No -- only the newer entity declares `supersedes`. The older entity does not need to declare anything.

**Lint priority:** Medium

**When to use:**
- A revised plan supersedes an earlier draft
- A new quarterly goal supersedes last quarter's version of the same goal
- An updated atomic note supersedes an older note on the same topic with better evidence
- A skill version 2.0 supersedes version 1.0

**When NOT to use:**
- Sources coexist as valid alternatives (that is `trades_off`)
- The newer source merely references the older one without replacing it (that is `references`)

**Lint checks:** Dangling targets (the superseded entity should still exist). Chain validation (supersession chains should not loop).

---

### 4.3 `supports`

**Description:** Source A provides evidence for a claim in source B.

**Bilateral:** No -- the supporting entity declares the relation. The supported entity does not need to reciprocate.

**Lint priority:** Low

**When to use:**
- An atomic note provides evidence supporting a goal's objective
- A meeting transcript contains evidence supporting a project's progress claim
- Multiple literature notes support a thesis in a MOC

**When NOT to use:**
- The connection is purely structural (use direct link fields like `primary_goal` instead)
- The sources merely mention each other (that is `references`)

**Lint checks:** Target existence (the supported entity must exist).

---

### 4.4 `references`

**Description:** Source A mentions or cites source B.

**Bilateral:** No -- only the referencing entity declares the relation.

**Lint priority:** Low

**When to use:**
- A reflection mentions a meeting that influenced the thinking
- An essay cites an atomic note as background
- A to-do references the meeting where it was assigned

**When NOT to use:**
- The reference provides supporting evidence (use `supports` instead -- it is more specific)
- The reference indicates derivation (use `derived_from` instead)

**Lint checks:** Target existence (the referenced entity must exist).

---

### 4.5 `causes`

**Description:** Event or decision in source A led to outcome in source B.

**Bilateral:** No -- the cause entity declares the relation pointing to the effect entity.

**Lint priority:** Medium

**When to use:**
- A meeting decision caused a new project to be created
- A to-do completion caused another to-do to unblock
- A strategic shift (documented in a reflection) caused goal reprioritization

**When NOT to use:**
- The relationship is correlational, not causal
- One source merely follows another chronologically without causal link (that is `references`)

**Lint checks:** Target existence. Temporal consistency (cause should predate effect based on `created` dates).

---

### 4.6 `trades_off`

**Description:** Source A identifies a trade-off with what source B recommends.

**Bilateral:** Yes -- when A declares `trades_off` with B, then B must also declare `trades_off` with A.

**Lint priority:** Medium

**When to use:**
- Two areas compete for the same time allocation
- Two project approaches offer different strengths at the expense of different weaknesses
- Two goals cannot both be fully pursued simultaneously

**When NOT to use:**
- Sources directly contradict each other on facts (that is `contradicts`)
- One is simply better than the other (that is `supersedes`)

**Lint checks:** Bilateral consistency (both entities must declare the trade-off). Trade-off clusters are surfaced for operator awareness during weekly review.

---

### 4.7 `implements`

**Description:** Source A is the implementation of the design described in source B.

**Bilateral:** No -- the implementation entity declares the relation pointing to the design entity.

**Lint priority:** Low

**When to use:**
- A project implements a goal's objective
- A skill implements a ritual specification
- A prompt implements a step in a plan
- A to-do implements a project milestone

**When NOT to use:**
- The relationship is loose inspiration rather than direct implementation (use `references`)
- The implementation has diverged so far that the design no longer applies (use `derived_from` or `supersedes`)

**Lint checks:** Target existence (the design entity must exist).

---

### 4.8 `derived_from`

**Description:** Source A was synthesized or compiled from source B (raw source).

**Bilateral:** No -- the derived entity declares the relation pointing to its source.

**Lint priority:** Low

**When to use:**
- A compiled meeting file was derived from a raw transcript
- An atomic note was derived from a literature note
- A daily journal was derived from individual archive entries
- A literature note was derived from raw reading highlights

**When NOT to use:**
- The derivation is loose inspiration (use `references`)
- Source A replaces source B (use `supersedes`)

**Lint checks:** Target existence (the source entity must exist). This relation is the typed-relation equivalent of the `source_lineage` field.

---

### Bilateral Consistency Rule

For bilateral relations (`contradicts` and `trades_off`): when entity A declares the relation toward entity B, entity B **must also** declare the same relation type toward entity A. Vault lint flags unilateral bilateral relations as defects.

---

## 5. Behavioral Contract

These rules define how an AI agent must behave when operating on a Momentum-compatible vault. Each rule includes the design intent.

### 5.1 Never Modify Raw Layer Files

Raw transcripts, emails, screenshots, and highlights are immutable after creation. No agent process may edit, delete, or overwrite raw layer files.

**Why:** Raw sources are the ground truth. If compiled wiki content becomes corrupted, it can be regenerated from raw sources. Mutating raw sources destroys this recovery path.

### 5.2 Always Compile from Raw to Wiki

Ingestion reads raw sources and produces wiki-layer entity files. The compilation step adds frontmatter, entity linking, and synthesis. Never edit raw content directly -- always produce a new wiki-layer file.

**Why:** The three-layer separation ensures that the compilation process is repeatable and auditable. Raw-to-wiki is a one-way flow.

### 5.3 Include Source Lineage

Every wiki-layer entity must trace back to its raw source via a `source_lineage` field, a `source` field, or a `derived_from` relation. Broken lineage is a lint defect.

**Why:** Provenance enables trust. When a claim appears in an atomic note, the operator can trace it back to the meeting or article where it originated.

### 5.4 Propose Entity Updates Through an Approval Gate

Never auto-apply updates to entity files (areas, goals, projects, people, to-dos). Propose changes and let the operator approve or reject them. Maximum 3 proposals per ritual run.

**Why:** Entity files compound across the system. A bad update to a goal file cascades through every ritual that reads it. The approval gate prevents automated errors from compounding.

### 5.5 Follow Coaching Voice Constraints

All user-facing coaching output must use observational, not prescriptive language. See Section 6 for the full specification.

**Why:** The vault serves the operator, not the other way around. Prescriptive language creates pressure. Observational language creates awareness. The operator decides what to do.

### 5.6 Stay Silent When Confidence Is Below Threshold

Do not surface low-confidence suggestions. Silence is better than noise. If the confidence model (Section 9) scores a result below the caller's threshold, suppress it.

**Why:** Every surfaced item costs the operator's attention. Low-quality suggestions erode trust in the system. A quiet system that speaks only when it has something meaningful to say is more useful than a noisy one.

### 5.7 Surface Higher-Priority Goals First

When coaching touches multiple goals, mention higher-priority ones first. Goal priority order is defined in the vault's goal files.

**Why:** Attention is finite. Leading with higher-priority goals ensures the operator sees the most important signals first, even if they stop reading midway.

### 5.8 Treat Goal Silence as Information, Not Failure

Absence of vault evidence on a goal does not mean failure. It is a signal to surface factually, not an alarm to raise. Silence on a lower-priority goal during a sprint on a higher-priority goal is expected.

**Why:** The operator works outside the vault too. Meetings, phone calls, and offline work do not always leave vault traces. Treating silence as failure creates false anxiety.

### 5.9 Ritual Outputs Are Immutable

Daily plans, weekly reviews, session closures, journal entries, reflections, meeting files, and communication files are historical records. Never modify them after creation. Only entity files (people, goals, projects, areas, to-dos, habits) get updated.

**Why:** Ritual outputs are the audit trail. They capture what the system believed at a point in time. Modifying them retroactively destroys the historical record and makes pattern detection unreliable.

### 5.10 Two-Output Rule

Every substantive ritual task produces both (a) its primary user-facing deliverable and (b) zero-or-more structured wiki-page proposals that flow through an approval gate before landing in the zettelkasten.

**Why:** Knowledge evaporates in chat history. The Two-Output Rule ensures that insights surfaced during rituals get captured as durable atomic notes, making the wiki compound from both ingestion and operator reasoning.

---

## 6. Coaching Voice

All ritual output that addresses the operator (daily plan coaching, weekly review signals, session closure observations) must follow this voice specification.

### Core Principle

> Evidence surfacing is observational and factual. It never scores, judges, or prescribes.

The coaching posture INFORMS rather than pressures. A reader should feel oriented, not anxious. Work happens outside the vault -- absence of vault evidence does not mean absence of progress.

### Foundation Rules

1. **Coach from understanding, not assumption.** Every coaching sentence traces to vault evidence. Never fill gaps with inference.
2. **Never infer next steps.** Reflect what happened. Proposals for what should happen go to Proposed To-Dos, not inline coaching.
3. **No stale coaching.** Check recent evidence before repeating. If a meeting already addressed something, reflect the progress.

### Banned Terms

Never use these terms in coaching output:

| Banned Term | Use Instead |
|-------------|-------------|
| overdue | past target date |
| action needed | noting for awareness |
| at risk | needs attention |
| escalate | surface to / follow up with / raise with |
| push for | prepare for / surface for discussion |
| delayed | pending / not yet started |
| force (as directive) | *(omit)* |
| demand | *(omit)* |
| insist | *(omit)* |
| nudge | follow up with |
| falling behind | no recent vault activity |
| slipping | not progressing on current timeline |
| behind schedule | past target date |
| missed deadline | target date was [date] |
| zero progress | no vault activity in [N] days |
| stalling | no recent movement |
| bottleneck (as urgency) | blocking dependency |

### Banned Patterns

- **Imperative verbs directed at the operator:** "you should", "you need to", "make sure to", "don't forget to"
- **Alarm framing:** treating routine goal silence as concerning, flagging a few days of quiet on one of several active goals as noteworthy
- **False urgency:** flagging something as noteworthy when it is routine within organizational rhythm
- **Prescriptive coaching:** stating what should happen next unless explicitly documented in the vault

### Due Date Treatment

Due dates are suggestions, not rules. Organizational rhythm handles most things naturally.

- **Factual:** "Target date was March 15." -- no alarm, no judgment
- **Never:** "This is overdue by 8 days." or "Missed deadline."
- Never propose discarding or deferring to-dos solely because they are past their target date
- Past-target-date to-dos may reflect low priority, not abandonment

### People Language

The vault operator may work through soft influence rather than direct authority.

- **Use:** "follow up with", "raise with", "surface to", "check in with"
- **Never:** "push", "demand", "insist", "confront", "escalate to" (as directive)
- Decision attribution reflects who actually decided, not who facilitated

### Observational Framing Templates

**Time-based observations:**
- "Last vault activity on [entity]: [date]."
- "No vault activity related to [goal] in the past [N] days."
- "This is the third week without vault activity on [goal]."

**Due date context:**
- "Target date was [date]. Current status: [status from vault]."

**People context:**
- "Last contact with [person]: [date] via [source]."
- "Open items with [person]: [list from vault]."

**Goal silence as first-class signal:**
- "No evidence of activity on [goal] this period." -- surfaced as information, not alarm
- Higher-priority goals noted first; lower-priority goals noted without concern
- Silence on a lower-priority goal during a sprint on a higher-priority goal is expected

**Active movement:**
- "Active movement on [entity] -- [summary of recent activity]."
- "Evidence from [N] sources this week: [list sources]."

**Decision framing:**
- "Two paths: [A] or [B]. Both are valid." -- not "you must decide now"
- Present the evidence, name the pattern, let the operator decide

**Pattern awareness:**
- "This is the [Nth] consecutive [period] without movement on [entity]." -- observation
- "The pattern this week: [description of what the evidence shows]." -- synthesis

### What Makes Good Coaching

Good ritual coaching reveals something the operator would not see from calendar + to-do list alone. It connects information across sources, names patterns across days and weeks without judgment, surfaces dormant goals as information (not failures), and credits progress where evidence shows it.

Bad ritual coaching repeats what the calendar already shows, creates anxiety about items progressing normally, treats every past-target-date item as a problem, assumes inaction from absence of vault evidence, or prescribes actions the operator has not requested.

See `coaching-voice.md` in the repository for the complete specification.

---

## 7. Ritual Rhythm

Rituals are recurring processes that read vault state, produce immutable outputs, and optionally update entity synthesis fields. They form a chain where each level feeds the next.

### 7.1 Daily Plan

The primary daily ritual. A three-part structure:

1. **Backward look:** What happened since the last plan. Reads recent session closures, completed to-dos, processed meetings, and entity changes.
2. **Current plate:** What is on the plate today. Reads calendar events, inbox items (email, messages), pending to-dos, and active project status.
3. **Coaching:** What deserves attention. Evidence-surfaced observations following the coaching voice specification. Connects cross-source information the operator might not see from calendar alone.

**Reads:** Calendar events, vault evidence (session closures, entity files, to-dos), inbox items (email, messages), recent entity activity.

**Writes:** One immutable daily plan file in `rituals/daily-plans/`. May also propose to-do updates (through approval gate).

### 7.2 Weekly Review

Structural discovery across a full week. Broader scope than daily planning.

1. **Goal trajectory assessment:** Green/yellow/red status for each goal based on vault evidence.
2. **Entity file refresh:** Updates synthesis fields (pulse, signal, coaching, evidence, progress) on entity files -- areas, goals, projects, people, habits.
3. **Essay cluster identification:** Surfaces atomic notes that are clustering into essay-ready themes.
4. **System health check:** Identifies orphan notes, stale claims, broken lineage, and other lint defects.

**Reads:** Session closures from the week, daily plans, all entity files, zettelkasten notes, to-do status changes.

**Writes:** One immutable weekly review file in `rituals/weekly-reviews/`. Updates synthesis fields on entity files (areas, goals, projects, people, habits). May propose new to-dos or atomic notes (through approval gate).

### 7.3 Session Closure

Lightweight capture of what a single work session accomplished. Not a ritual in the recurring sense -- created during every session as a side effect of work.

**Structure:** Three sections:
1. **What Was Completed** -- factual list of deliverables
2. **Key Decisions** -- decisions made during the session
3. **Follow-Up Items** -- what needs to happen next

**Reads:** The work done in the current session.

**Writes:** One immutable session closure file in `sessions/closures/`. Maximum 10-15 lines. Created as placeholder early in session, updated as work progresses.

### 7.4 Monthly Review

Goal trajectory assessment across a full month. Less frequent but broader scope than weekly review.

**Reads:** Weekly reviews from the month, all entity files, goal files, project status.

**Writes:** One immutable monthly review file. May trigger goal reprioritization proposals (through approval gate).

### Ritual Chain

```
Daily plans --> Weekly review --> Monthly review
```

Each ritual feeds the next level. Daily plans accumulate evidence that weekly review synthesizes into entity-level patterns. Weekly reviews accumulate patterns that monthly review assesses for trajectory changes. The chain ensures no observation is lost -- it either surfaces in the next ritual or gets captured as a lint defect.

### Data Contract Summary

| Ritual | Reads | Writes (Immutable) | Writes (Mutable) |
|--------|-------|---------------------|-------------------|
| Daily Plan | Calendar, inbox, to-dos, entity files, session closures | 1 daily plan file | *(none -- proposes via gate)* |
| Weekly Review | Session closures, daily plans, all entity files | 1 weekly review file | Entity synthesis fields (pulse, signal, coaching, evidence, progress) |
| Session Closure | Current session work | 1 session closure file | *(none)* |
| Monthly Review | Weekly reviews, entity files, goal files | 1 monthly review file | *(none -- proposes via gate)* |

---

## 8. Lint Defect Categories

The vault maintains a gaps backlog where structural defects are tracked. Six defect categories are detected by lint processes and routed for resolution.

### 8.1 Contradictions

**Detection:** Two or more notes make opposing claims about the same subject. Identified through `contradicts` relations or through semantic analysis of overlapping topic tags with opposing conclusions.

**Resolution:** Review both sources, determine which is current, and either: (a) mark one as superseded via `supersedes` relation, (b) add nuance that resolves the apparent contradiction, or (c) document the genuine disagreement explicitly.

**Severity:** High. Contradictions erode trust in the knowledge base. Prioritize resolution.

### 8.2 Orphans

**Detection:** Atomic notes with no inbound or outbound links to other notes. Every atomic note should connect to at least one other entity.

**Resolution:** Add links to related notes, file under an appropriate MOC, or determine if the note should be merged into an existing note.

**Severity:** Medium. Orphan notes represent trapped knowledge that cannot surface in rituals or searches.

### 8.3 Stale Claims

**Detection:** High-confidence claims whose source has not been touched in 90+ days. The claim may still be valid, but the evidence is aging.

**Resolution:** Verify the claim against current sources. Update the note's `created`/`updated` date if still valid. Mark as superseded if outdated.

**Severity:** Medium. Stale claims may mislead rituals that surface them as current.

### 8.4 Missing Cross-References

**Detection:** Notes with overlapping topic tags but no link between them. These notes likely relate but are not connected in the graph.

**Resolution:** Review both notes and add appropriate links or relations. The connection type may be `supports`, `references`, `contradicts`, or `trades_off`.

**Severity:** Low. Missing cross-references reduce discovery but do not create incorrect information.

### 8.5 Source Traceability

**Detection:** Atomic notes whose `source_lineage` field is empty or points to a file that does not exist. This breaks the raw-to-wiki chain.

**Resolution:** Trace the note back to its raw source and populate the `source_lineage` field. If the raw source cannot be found, document this in the note and flag for manual review.

**Severity:** High. Broken lineage means a claim cannot be verified against its origin.

### 8.6 Relation Integrity

**Detection:** Entities with `relations` entries whose target does not exist, whose `type` is not one of the 8 defined relation types, or whose `evidence` field is empty or missing.

**Resolution:** Fix the relation entry -- update the target to point to an existing file, correct the type to a valid relation type, or add the missing evidence sentence.

**Severity:** Medium. Invalid relations produce noise in the graph and cause lint failures during bilateral consistency checks.

---

## 9. Confidence Model

The confidence model determines which entities surface during rituals and queries. It prevents noise by suppressing low-confidence results and ensures the most relevant entities get attention.

### 5 Signals

| Signal | Description |
|--------|-------------|
| Recency | Decay curve from entity's created/updated date -- newer entities score higher |
| Entity Authority | Weight based on entity type (goals > projects > atomic notes > session closures) |
| Source Count | Number of raw sources backing the entity -- more sources = higher confidence |
| Goal Alignment | Binary: is this entity connected to an active goal with open to-dos? |
| Access Recency | Inverse of days since last access -- resurfaces forgotten-but-relevant entities |

### Per-Caller Weight Profiles

Different rituals weight signals differently based on what they need:

| Caller | Recency | Authority | Source Count | Goal Alignment | Access Recency |
|--------|---------|-----------|-------------|----------------|----------------|
| Daily plan (backward look) | 0.4 | 0.2 | 0.1 | 0.2 | 0.1 |
| Daily plan (surfacing) | 0.2 | 0.2 | 0.1 | 0.3 | 0.2 |
| Weekly review | 0.1 | 0.2 | 0.2 | 0.3 | 0.2 |
| Essay pipeline | 0.1 | 0.3 | 0.2 | 0.1 | 0.3 |
| User query (default) | 0.2 | 0.3 | 0.2 | 0.2 | 0.1 |

### Threshold Behavior

Both **relevance** (weighted signal score) and **momentum** (goal-connected, recently accessed, actionable entity type) must meet the caller's threshold for a result to surface.

- Below threshold = silence, not noise
- Default threshold: 0.5
- Rituals may set their own thresholds

### Momentum Dimension

Momentum measures whether an entity is "in play":
- Is the entity connected to a goal with active to-dos?
- Has it been accessed recently?
- Is the entity type actionable (to-do, project) vs. archival (session closure, journal)?

Both relevance and momentum must meet threshold. An entity can be highly relevant but low-momentum (important but dormant) or high-momentum but low-relevance (active but not applicable to the current context).

### Calibration Guidance

Tune weights per caller based on what that ritual needs:
- **Daily plans** prioritize recency -- what happened in the last 24-48 hours matters most
- **Weekly reviews** prioritize goal alignment -- which goals are progressing and which are quiet
- **Essay pipelines** prioritize access recency (to resurface forgotten notes) and entity authority (to favor high-quality atomic notes)
- **User queries** balance all signals with a slight preference for authority -- when the operator asks a question, the most authoritative entities should surface first
