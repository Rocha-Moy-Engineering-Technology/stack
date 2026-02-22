# Stacks CSV Schema

CSV catalog of AI/ML ecosystem tools (SDKs, APIs, infrastructure) used as a reference index for selecting frameworks, inference platforms, observability tools, and orchestration systems.

## File Location

`bundles/stack/stacks.csv`

## Schema Structure

### name

- Type: string
- Constraints: unique, case-insensitive; must not duplicate an existing entry
- Example: `LangChain`, `vLLM`, `Ollama`

### group

- Type: string (enum)
- Constraints: MUST be one of the 12 defined group labels (see Group Labels section)
- Example: `Agent Frameworks`, `Inference Serving`

### type

- Type: string
- Constraints: one or more type classifications joined by `/` (see Type Classifications section)
- Example: `SDK`, `API/SDK`, `SDK/Infra`, `API/SDK/UI`

### docs_url

- Type: URL string
- Constraints: MUST point to official documentation; verified via browser navigation before insertion
- Example: `https://docs.langchain.com/`

### description

- Type: string
- Constraints: one-line concise description matching existing CSV entry style
- Example: `Python framework for building LLM applications with chains and agents`

### open_source

- Type: string (enum)
- Constraints: `yes` or `no`
- Example: `yes`

### github_repo

- Type: URL string or `N/A`
- Constraints: full GitHub repository URL if open source; `N/A` if not open source
- Example: `https://github.com/langchain-ai/langchain`, `N/A`

### stars

- Type: integer or `N/A`
- Constraints: raw GitHub star count (no formatting); `N/A` if not open source
- Example: `127162`, `N/A`

### open_source_alternative

- Type: string or `N/A`
- Constraints: `N/A` if entry is open source; otherwise MUST reference an existing `name` in stacks.csv (referential integrity); if the alternative does not exist yet, it must be added first; choice must be most popular/influential stack.
- Example: `LiteLLM`, `N/A`

### commercial_alternative

- Type: string or `N/A`
- Constraints: `N/A` if entry is not open source; if entry is open source, use the top commercial competitor name; values that exist in stacks.csv maintain referential integrity, values not in stacks.csv are external product names; choice must be most popular/influential stack.
- Example: `Vertex AI`, `Groq`, `N/A`

## Group Labels

- **LLM Providers** - Hosted large language model APIs (OpenAI, Gemini, Claude)
- **Agent Frameworks** - Libraries for building autonomous AI agents and multi-agent systems
- **RAG & Knowledge Retrieval** - Retrieval-Augmented Generation (RAG) frameworks, vector stores, and knowledge graph tools
- **Structured Output & Prompt Engineering** - Tools for constraining LLM outputs and optimizing prompts
- **Guardrails & Safety** - Input/output validation, content moderation, and LLM security
- **Evaluation & Testing** - Frameworks for evaluating and benchmarking LLM application quality
- **Observability & LLM Ops** - Tracing, monitoring, experiment tracking, and prompt management platforms
- **API Gateways & Model Routing** - Unified API proxies and routing layers across multiple LLM providers
- **Inference Serving** - Engines and platforms for hosting and serving model inference
- **GPU Compute & Cloud Platforms** - Cloud GPU providers, managed AI platforms, and distributed compute
- **Workflow Orchestration & Automation** - Pipeline scheduling, workflow engines, and no-code automation
- **Data Labeling** - Tools for annotating and labeling training data

## Type Classifications

- **SDK** - Frameworks and libraries
- **API** - Hosted inference and gateway services
- **UI** - Visual tools and dashboards
- **Infra** - Compute and serving platforms
- Stacks may span multiple type classifications joined by `/` (e.g., `API/SDK`, `SDK/Infra`, `API/SDK/UI`)

## Referential Integrity

- `open_source_alternative` values MUST reference an existing entry in the `name` column of stacks.csv; if the alternative does not exist yet, it must be added first
- `commercial_alternative` values reference the top commercial competitor; values that exist in stacks.csv maintain referential integrity, values not in stacks.csv are external product names

## Maintenance Rules

- All stack entries in maintenance mode MUST be flagged and removed from stacks.csv; the catalog shall only contain actively maintained tools
- When any stack entry is removed from stacks.csv, it MUST be added to `removed_stack.csv` with all fields filled out (see REMOVED_STACK_SCHEMA.md)
- Documentation URLs point to official docs and should be verified periodically
