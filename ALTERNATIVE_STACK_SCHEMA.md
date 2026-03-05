# Alternative Stack CSV Schema

Tracking file for alternative tools referenced in `stacks.csv` that are NOT currently present as catalog entries.

## File Location

`bundles/stack/alternative_stack.csv`

## Schema Structure

### alternative_name

- Type: string
- Constraints: required; should be the external product name as written in `stacks.csv`
- Example: `Zapier`

### alternative_kind

- Type: string (enum)
- Constraints: required; MUST be one of:
  - `open_source` (for `open_source_alternative`)
  - `commercial` (for `commercial_alternative`)
- Example: `commercial`

### referenced_by

- Type: string
- Constraints: required; MUST be the `name` of the stack row in `stacks.csv` that references this alternative
- Example: `n8n`

### referenced_group

- Type: string
- Constraints: required; SHOULD match the `group` of the referencing stack row in `stacks.csv`
- Example: `Workflow Orchestration`

### docs_url

- Type: URL string
- Constraints: required; MUST point to the alternative's official documentation; verified via Playwright browser navigation before insertion
- Example: `https://neo4j.com/docs/aura/`

## Rules

- Add a row whenever `open_source_alternative` or `commercial_alternative` in `stacks.csv` points to an alternative that is not present as a `name` in `stacks.csv`
- An alternative MUST NOT appear in `alternative_stack.csv` if it already exists as a `name` in `stacks.csv` (in that case, use the canonical `name` in the alternative field and omit it from `alternative_stack.csv`)
- Duplicates are allowed if the same alternative is referenced by multiple stacks (this preserves provenance)
- All fields are required; no field may be left empty
- `docs_url` MUST be verified reachable and containing relevant documentation content via Playwright before insertion
- When the same `alternative_name` appears in multiple rows (referenced by different stacks), `docs_url` MUST be identical across all rows
