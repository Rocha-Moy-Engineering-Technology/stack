# GraphRAG

> Microsoft Research's structured approach to Retrieval-Augmented Generation (RAG) that creates knowledge graphs from input corpora to enhance Large Language Model (LLM) reasoning over complex, interconnected information.

| Field | Value |
|-------|-------|
| Name | GraphRAG |
| Group | RAG & Knowledge Retrieval |
| Type | SDK |
| Open Source | Yes |
| GitHub | [https://github.com/microsoft/graphrag](https://github.com/microsoft/graphrag) |
| Stars | 31020 |
| Documentation | [Official Docs](https://microsoft.github.io/graphrag/) |

## Overview

GraphRAG is a knowledge-graph-augmented retrieval system developed by Microsoft Research. It addresses fundamental limitations of baseline RAG pipelines, specifically the inability to connect information across disparate sources and the lack of holistic understanding of an entire corpus. Where traditional vector-based RAG retrieves semantically similar chunks in isolation, GraphRAG constructs a structured knowledge graph that captures entities, relationships, and hierarchical community structures, enabling the LLM to reason over the corpus as an interconnected whole rather than as a bag of independent passages.

The system operates in three primary stages: Indexing, Querying, and Prompt Tuning. During indexing, raw text is decomposed into TextUnits, entities and relationships are extracted, hierarchical clustering groups related entities into communities, and summaries are generated at each level of the hierarchy. During querying, multiple search strategies leverage the graph structure to answer different classes of questions. Prompt tuning allows dataset-specific optimization of the extraction and summarization prompts.

## Core Concepts

- **TextUnits**: The fundamental unit of text segmentation. The input corpus is sliced into TextUnits, which serve as the atomic building blocks for entity and relationship extraction. Each TextUnit maintains provenance back to its source document.
- **Entities**: Named concepts, people, organizations, locations, events, or other noun phrases extracted from TextUnits. Entities become nodes in the knowledge graph and carry descriptions synthesized from all mentions across the corpus.
- **Relationships**: Directed or undirected edges between entities, representing how entities interact or relate within the text. Relationships carry descriptions and strength scores derived from co-occurrence and contextual signals.
- **Communities**: Groups of densely connected entities discovered through hierarchical clustering using the Leiden algorithm. Communities form a multi-level hierarchy, from fine-grained clusters to broad thematic groupings, enabling reasoning at different levels of abstraction.
- **Community Summaries**: Bottom-up summaries generated for each community at every level of the hierarchy. These summaries distill the collective knowledge of all entities and relationships within a community into a coherent narrative that an LLM can consume during query time.
- **Knowledge Graph**: The assembled graph structure containing all entities, relationships, and community annotations. This graph is the central artifact that distinguishes GraphRAG from traditional vector-only RAG systems.

## Installation and Setup

Install GraphRAG from PyPI:

```bash
pip install graphrag
```

Initialize a new GraphRAG project:

```bash
graphrag init --root ./my-project
```

This creates the project directory structure with default configuration files and prompt templates. Place input documents in the `input/` directory, then run the indexing pipeline:

```bash
graphrag index --root ./my-project
```

After indexing completes, run queries against the built knowledge graph:

```bash
graphrag query --root ./my-project --method global --query "What are the main themes?"
```

## Architecture

GraphRAG's architecture is organized around three stages that transform raw text into a queryable knowledge graph.

### Stage 1: Indexing Pipeline

The indexing pipeline transforms a raw corpus into a structured knowledge graph through a sequence of steps:

1. **Text Chunking**: The input corpus is segmented into TextUnits of configurable size. Each TextUnit retains metadata linking it to its source document and position.
2. **Entity and Relationship Extraction**: An LLM processes each TextUnit to identify entities (nodes) and relationships (edges). Multiple extraction passes can be configured to improve recall.
3. **Entity Resolution**: Duplicate and near-duplicate entities are merged. Descriptions from all mentions are synthesized into a single canonical description per entity.
4. **Graph Construction**: Extracted entities and relationships are assembled into a knowledge graph data structure.
5. **Hierarchical Community Detection**: The Leiden algorithm is applied to the graph to discover communities of densely connected entities. This produces a hierarchy of communities at multiple granularity levels, from small, tightly-knit clusters to large, thematic groupings.
6. **Community Summarization**: For each community at each level of the hierarchy, an LLM generates a summary that captures the collective knowledge of the community's entities and relationships. Summarization proceeds bottom-up, with higher-level summaries incorporating information from lower levels.

### Stage 2: Query Engine

The query engine provides multiple search strategies, each suited to different question types:

- **Global Search**: Answers broad, corpus-wide questions (e.g., "What are the main themes across all documents?"). Uses community summaries at the appropriate hierarchy level to construct a map-reduce style answer. Community summaries are scored for relevance, and the top-ranked summaries are aggregated into a final response.
- **Local Search**: Answers entity-specific questions (e.g., "What is the relationship between Entity A and Entity B?"). Starts from relevant entities in the graph, traverses their local neighborhoods, and combines entity descriptions, relationship details, community context, and source TextUnits into a context window for the LLM.
- **DRIFT Search**: A hybrid approach combining entity-level detail with community-level context. Starts with entity-focused retrieval, then expands to incorporate community summaries, providing both specificity and broader thematic grounding.
- **Basic Search**: Traditional vector similarity search over TextUnits, provided as a baseline comparison and fallback for questions that do not benefit from graph structure.

### Stage 3: Prompt Tuning

GraphRAG supports dataset-specific prompt optimization. The prompt tuning pipeline analyzes a sample of the input corpus and generates tailored prompts for entity extraction, relationship extraction, and community summarization. This improves extraction quality for domain-specific terminology and relationships that generic prompts may miss.

## Key Features and Functionality

- **Cross-document reasoning**: Connects information across multiple source documents through the shared knowledge graph, enabling answers that synthesize dispersed facts.
- **Hierarchical community structure**: The Leiden-based community detection produces a multi-level hierarchy, allowing queries to be answered at the appropriate level of abstraction.
- **Bottom-up summarization**: Community summaries are built from the ground up, preserving detail at lower levels while providing thematic coherence at higher levels.
- **Multiple search strategies**: Global, Local, DRIFT, and Basic search modes address different question types without requiring the user to restructure their data.
- **Prompt tuning**: Dataset-specific prompt optimization improves extraction accuracy for specialized domains.
- **Provenance tracking**: TextUnits maintain links to source documents, and entities maintain links to the TextUnits from which they were extracted, supporting traceability and citation.
- **Configurable pipeline**: Chunking sizes, extraction parameters, community detection resolution, and LLM model selection are all configurable.

## Use Cases

- **Corporate knowledge bases**: Synthesizing information across thousands of internal documents, reports, and communications where answers require connecting facts from multiple sources.
- **Scientific literature review**: Building knowledge graphs from research papers to identify cross-paper relationships, emerging themes, and gaps in the literature.
- **Legal document analysis**: Extracting entities (parties, clauses, precedents) and relationships across large volumes of contracts, regulations, and case law.
- **Intelligence analysis**: Connecting entities and events across disparate intelligence reports to surface patterns and relationships not visible in any single document.
- **Due diligence and compliance**: Mapping relationships between organizations, individuals, and transactions across financial and regulatory filings.
- **Dataset exploration**: Answering high-level questions about unfamiliar datasets to quickly understand structure, themes, and key entities before deeper analysis.

## API Reference Summary

### Command-Line Interface (CLI)

```bash
# Initialize project structure
graphrag init --root <project-directory>

# Run the indexing pipeline
graphrag index --root <project-directory>

# Run prompt tuning
graphrag prompt-tune --root <project-directory>

# Execute a query
graphrag query --root <project-directory> --method <global|local|drift|basic> --query "<question>"
```

### Python API

```python
import graphrag

# Indexing
from graphrag.index import run_pipeline

# Querying
from graphrag.query import GlobalSearch, LocalSearch, DRIFTSearch, BasicSearch
```

The Python API exposes the same pipeline stages as the CLI, allowing programmatic integration into larger applications. The indexing pipeline can be run as an async workflow, and query engines can be instantiated and invoked directly.

## Configuration and Customization

GraphRAG uses a `settings.yaml` file in the project root for configuration. Key configuration sections include:

```yaml
llm:
  model: gpt-4o
  api_key: ${GRAPHRAG_API_KEY}

chunks:
  size: 1200
  overlap: 100

entity_extraction:
  max_gleanings: 1

community_detection:
  max_cluster_size: 10

storage:
  type: file
  base_dir: output
```

- **llm**: Model selection and API credentials for the LLM used during indexing and querying.
- **chunks**: TextUnit size and overlap parameters controlling how the input corpus is segmented.
- **entity_extraction**: Parameters governing how many extraction passes (gleanings) are performed per TextUnit.
- **community_detection**: Controls for the Leiden algorithm, including maximum cluster size and resolution.
- **storage**: Output storage configuration for the knowledge graph artifacts.

Environment variables can be referenced using `${VARIABLE_NAME}` syntax within the configuration file.

## Integration Patterns

### As a standalone indexing and query system

Run GraphRAG as an independent pipeline that indexes a document corpus and serves queries through its CLI or Python API. This is the simplest integration pattern and requires no external dependencies beyond the LLM provider.

### As a retrieval backend for an LLM application

Use GraphRAG's query engines as a retrieval layer within a larger LLM application. The application submits queries to GraphRAG's search functions and incorporates the retrieved context into its own prompt construction before calling the LLM.

### Combined with traditional vector RAG

Use GraphRAG's Global or Local search alongside a traditional vector store. Route corpus-wide or relationship-heavy questions to GraphRAG's graph-based search, and route straightforward factual lookups to the vector store. The Basic Search mode provides a built-in baseline for comparison.

### Incremental indexing workflows

For evolving corpora, re-run the indexing pipeline periodically as new documents arrive. The pipeline processes the full corpus each time, regenerating the knowledge graph with updated entities, relationships, and community structures.

## Examples

### Global Search: Corpus-wide question

```bash
graphrag query \
  --root ./my-project \
  --method global \
  --query "What are the top 5 themes discussed across all documents?"
```

Global Search leverages community summaries to answer broad thematic questions without requiring the user to specify which documents or entities to examine.

### Local Search: Entity-specific question

```bash
graphrag query \
  --root ./my-project \
  --method local \
  --query "What is the relationship between Microsoft Research and knowledge graphs?"
```

Local Search starts from the entities mentioned in the query, traverses their graph neighborhoods, and assembles a focused context for the LLM.

### Python API: Programmatic querying

```python
from graphrag.query import LocalSearch

search = LocalSearch(root_dir="./my-project")
result = search.search("Describe the key contributions of Entity X.")
print(result.response)
```

## Limitations and Considerations

- **Indexing cost**: The indexing pipeline requires multiple LLM calls per TextUnit for entity extraction, relationship extraction, and community summarization. For large corpora, this translates to significant token consumption and cost.
- **Indexing latency**: Building the knowledge graph is a batch process that can take substantial time for large document collections. It is not designed for real-time or streaming ingestion.
- **LLM dependency for extraction**: The quality of the knowledge graph is directly dependent on the LLM's ability to accurately extract entities and relationships. Domain-specific or highly technical content may require prompt tuning to achieve acceptable extraction quality.
- **Full re-indexing**: Updates to the corpus currently require re-running the full indexing pipeline rather than incrementally updating the existing graph.
- **Community summary staleness**: If the underlying documents change, community summaries must be regenerated to remain accurate.
- **Graph structure assumptions**: The effectiveness of graph-based search depends on the corpus containing meaningful entity relationships. Corpora with few cross-document entity connections may not benefit significantly over traditional vector RAG.

## Changelog Highlights

GraphRAG is under active development by Microsoft Research. The project follows semantic versioning on PyPI. Refer to the GitHub repository's releases page for detailed version history and breaking changes.

## Citations

[1] Microsoft GraphRAG Documentation. https://microsoft.github.io/graphrag/
