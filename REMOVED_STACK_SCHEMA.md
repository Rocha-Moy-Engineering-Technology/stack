# Removed Stack CSV Schema

Archive of stack entries removed from `stacks.csv` due to maintenance mode, deprecation, or other disqualifying reasons.

## File Location

`bundles/stack/removed_stack.csv`

## Schema Structure

### name

- Type: string
- Constraints: MUST match the `name` value from the original stacks.csv entry at the time of removal
- Example: `TGI`

### reason

- Type: string
- Constraints: brief explanation of why the entry was removed; include replacement recommendations when applicable
- Example: `Maintenance mode; HuggingFace recommends vLLM/SGLang as replacements`

### docs_url

- Type: URL string
- Constraints: required; MUST be the `docs_url` value from the original stacks.csv entry at the time of removal
- Example: `https://huggingface.co/docs/text-generation-inference/`

## Rules

- An entry MUST be added to removed_stack.csv whenever a stack is removed from stacks.csv
- All fields are required; no field may be left empty
- The `name` field preserves the exact name from the original stacks.csv entry
