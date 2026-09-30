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
- Constraints: MUST be one of the 23 defined group labels (see Group Labels section)
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
- Constraints: `N/A` if entry is open source AND no open-source competitor is meaningful; otherwise set to the most popular/influential open-source alternative. Closed-source entries SHOULD populate this field with the closest open-source counterpart when one exists. If the alternative exists in stacks.csv, use the exact `name` value from stacks.csv; otherwise it may be an external product name. External alternatives SHOULD be recorded in `alternative_stack.csv`.
- Example: `LiteLLM`, `N/A`

### commercial_alternative

- Type: string or `N/A`
- Constraints: set to the most popular/influential commercial competitor when one exists. Open-source entries populate this field with their commercial competitor. Closed-source entries MAY populate this field with another closed-source competitor when no peer is open-source (e.g., LLM provider rows reference each other). Use `N/A` only when no meaningful commercial competitor exists. If the alternative exists in stacks.csv, use the exact `name` value from stacks.csv; otherwise it may be an external product name. External alternatives SHOULD be recorded in `alternative_stack.csv`.
- Example: `Vertex AI`, `Groq`, `Claude`, `N/A`

## Group Labels

Groups are organized into four parts. The part assignment determines the chapter's directory under `bundles/stack/books/src/`.

### Part I — Foundation & Infrastructure

- **LLM Providers** - Hosted large language model APIs (OpenAI, Gemini, Claude)
- **Hosted Inference APIs** - Vendor-hosted inference services on custom silicon or shared accelerators (Groq, Cerebras)
- **Inference Engines** - Self-hosted serving frameworks and model libraries for production inference (vLLM, Triton, BentoML, Hugging Face Transformers)
- **Local Model Runtimes** - Developer-focused desktop and CLI tools for running models locally (Ollama, LM Studio)
- **Model Gateways** - Unified API proxies and routing layers across multiple LLM providers
- **Compute & GPU Infrastructure** - Serverless GPU platforms and distributed compute frameworks (Ray, Modal, RunPod)
- **Managed AI Platforms** - Cloud vendor end-to-end ML/GenAI platforms covering training, serving, and orchestration (Vertex AI, AWS Bedrock)

### Part II — Application Development

- **Agent Frameworks** - Libraries for building autonomous AI agents and multi-agent systems
- **Agent Protocols** - Open standards for connecting agents to tools, data, and other agents (MCP, A2A)
- **Agent Runtimes & Sandboxes** - Isolated execution environments for agent code, tools, and data manipulation
- **Browser Automation** - Headless browser platforms and SDKs for web-acting agents
- **Structured Generation** - Tools for constraining LLM outputs and structured data extraction
- **Memory Systems** - Persistent memory and context management for AI agents and assistants

### Part III — Data & Models

- **RAG Frameworks** - Retrieval-Augmented Generation (RAG) frameworks and knowledge graph tools
- **Embeddings & Reranking** - Hosted and open-source embedding and reranking model providers
- **Vector Databases** - Vector similarity search engines and embedding storage systems
- **Data Pipelines** - ETL/ingestion tools for document parsing, chunking, embedding, and data integration
- **Fine-tuning** - Libraries for parameter-efficient fine-tuning and model training
- **Labeling** - Tools for annotating and labeling training data

### Part IV — Operations & Quality

- **Guardrails** - Input/output validation, content moderation, and LLM security
- **Evaluation** - Frameworks for evaluating and benchmarking LLM application quality
- **Observability** - Tracing, monitoring, experiment tracking, and prompt management platforms
- **Workflow Orchestration** - Pipeline scheduling, workflow engines, and no-code automation

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
