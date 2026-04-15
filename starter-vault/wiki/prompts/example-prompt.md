---
created: 2026-01-15
type: prompt
purpose: "Validate starter vault entity frontmatter against schema definitions"
destination: Code
complexity: medium
primary_goal: "[[goal-build-knowledge-system]]"
project: "[[personal-knowledge-system]]"
---

# Validate Starter Vault Schemas

## Task

Read every entity file in the starter vault. For each file, load the corresponding schema YAML from `schema/`. Verify that all required frontmatter fields are present and have valid values. Report any missing fields, type mismatches, or invalid enum values.

## Expected Output

A validation report listing each file with PASS or FAIL status, and for failures, the specific field violations.

## Success Criteria

- All 26 entity type examples pass validation
- All 8 relation types appear at least once
- All bilateral relations (contradicts, trades_off) appear in both participating files
- Zero dangling wikilink targets in relation declarations
