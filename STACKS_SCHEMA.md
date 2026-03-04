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
- Constraints: MUST be one of the 16 defined group labels (see Group Labels section)
- Example: `Agent Frameworks`, `Inference Engines`

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
- Constraints: `N/A` if entry is open source; otherwise set to the most popular/influential open-source alternative. If the alternative exists in stacks.csv, use the exact `name` value from stacks.csv; otherwise it may be an external product name. External alternatives SHOULD be recorded in `alternative_stack.csv`.
- Example: `LiteLLM`, `N/A`

### commercial_alternative

- Type: string or `N/A`
- Constraints: `N/A` if entry is not open source; if entry is open source, set to the most popular/influential commercial competitor. If the alternative exists in stacks.csv, use the exact `name` value from stacks.csv; otherwise it may be an external product name. External alternatives SHOULD be recorded in `alternative_stack.csv`.
- Example: `Vertex AI`, `Groq`, `N/A`

## Group Labels

- **LLM Providers** - Hosted large language model APIs (OpenAI, Gemini, Claude)
- **Agent Frameworks** - Libraries for building autonomous AI agents and multi-agent systems
- **RAG Frameworks** - Retrieval-Augmented Generation (RAG) frameworks and knowledge graph tools
- **Structured Generation** - Tools for constraining LLM outputs and structured data extraction
- **Model Gateways** - Unified API proxies and routing layers across multiple LLM providers
- **Vector Databases** - Vector similarity search engines and embedding storage systems
- **Inference Engines** - Engines and platforms for hosting and serving model inference
- **GPU Infrastructure** - Cloud GPU providers, managed AI platforms, and distributed compute
- **Workflow Orchestration** - Pipeline scheduling, workflow engines, and no-code automation
- **Observability** - Tracing, monitoring, experiment tracking, and prompt management platforms
- **Evaluation** - Frameworks for evaluating and benchmarking LLM application quality
- **Guardrails** - Input/output validation, content moderation, and LLM security
- **Data Pipelines** - ETL/ingestion tools for document parsing, chunking, embedding, and data integration
- **Fine-tuning** - Libraries for parameter-efficient fine-tuning and model training
- **Memory Systems** - Persistent memory and context management for AI agents and assistants
- **Labeling** - Tools for annotating and labeling training data

## Type Classifications

- **SDK** - Frameworks and libraries
- **API** - Hosted inference and gateway services
- **UI** - Visual tools and dashboards
- **Infra** - Compute and serving platforms
- Stacks may span multiple type classifications joined by `/` (e.g., `API/SDK`, `SDK/Infra`, `API/SDK/UI`)

## Alternative Name Matching

- If an alternative value matches an existing stack `name` (case-insensitive), it SHOULD use the exact `name` spelling from stacks.csv
- If an alternative is not present in stacks.csv, the value may be an external product name (and should be recorded in `alternative_stack.csv`)

## Maintenance Rules

- All stack entries in maintenance mode MUST be flagged and removed from stacks.csv; the catalog shall only contain actively maintained tools
- When any stack entry is removed from stacks.csv, it MUST be added to `removed_stack.csv` with all fields filled out (see REMOVED_STACK_SCHEMA.md)
- Documentation URLs point to official docs and should be verified periodically
