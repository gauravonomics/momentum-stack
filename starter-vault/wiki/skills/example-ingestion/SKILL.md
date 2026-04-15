---
name: example-ingestion
description: "Example ingestion skill that reads raw transcript files and produces entity-linked meeting files. Triggers: 'process transcripts', 'ingest meetings'. Produces: inputs/meetings/YYYY/MM/ files with frontmatter."
version: "1.0.0"
created: 2026-01-10
updated: 2026-01-15
category: processing
status: active
---

# Example Ingestion Skill

## Purpose

Demonstrates the structure of an ingestion skill that reads from the raw layer and writes compiled content to the wiki layer.

## Triggers

- "process transcripts"
- "ingest meetings"
- "convert raw transcripts"

## Process

1. Scan `raw/transcripts/` for unprocessed files
2. Parse transcript content and extract metadata (date, participants, topics)
3. Link entities: match participants to `people/`, topics to `projects/` and `goals/`
4. Write entity-linked meeting file to `inputs/meetings/YYYY/MM/`
5. Never modify raw source files

## Output Format

Each output file includes frontmatter with `type: meeting-transcript`, `source: transcript-tool`, entity wikilinks, and the full transcript content in the body.

## Constraints

- Raw layer files are immutable — read only, never modify
- All output files must have `created` date matching the meeting date
- Entity links must point to existing vault files
