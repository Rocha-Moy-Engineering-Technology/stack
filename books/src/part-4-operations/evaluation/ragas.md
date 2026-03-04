# Ragas

> Open-source framework for evaluating RAG and LLM application quality

| Field | Value |
|-------|-------|
| Group | Evaluation & Testing |
| Type | SDK |
| Open Source | Yes |
| GitHub | [explodinggradients/ragas](https://github.com/explodinggradients/ragas) |
| Stars | 12679 |
| Documentation | [Official Docs](https://docs.ragas.io/en/stable/) |

## Overview

Ragas is an open-source Python framework for evaluating and optimizing Large Language Model (LLM) applications. The project's central premise is moving teams from informal "vibe checks" to systematic evaluation loops -- replacing subjective assessments with data-driven, repeatable workflows. Ragas provides both LLM-based and traditional metrics for scoring outputs, synthetic test data generation for building evaluation datasets, and an experiments-first approach where developers make changes, run evaluations, observe outcomes, and iterate. [1][2]

The framework targets multiple application types: Retrieval-Augmented Generation (RAG) pipelines, autonomous agents with tool use, prompt engineering workflows, multi-turn conversations, and SQL generation. Ragas integrates with major LLM frameworks (LangChain, LlamaIndex, Haystack, LangGraph) and observability platforms (LangSmith, Arize Phoenix), and supports LLM providers including OpenAI, Anthropic, Google Gemini, Amazon Bedrock, and Vertex AI through factory functions backed by LiteLLM and the Instructor library. [1][3]

The project is licensed under Apache 2.0, has over 12,000 GitHub stars, 273 contributors, and more than 3,400 dependent projects. [2]

## Core Concepts

**Metrics** are the fundamental evaluation primitives. Each metric quantifies a single aspect of application performance and returns a score. Ragas classifies metrics by mechanism (LLM-based versus traditional) and by evaluation type (single-turn versus multi-turn). LLM-based metrics use a language model to judge quality and correlate closely with human judgment but are non-deterministic. Traditional metrics (BLEU, ROUGE, string similarity) are deterministic but have lower correlation with human preferences. [4]

**Metric Output Types** define what a metric returns. Discrete metrics produce categorical values from a predefined set (pass/fail, excellent/good/poor). Numeric metrics return float values within a specified range and support aggregation. Ranking metrics compare multiple outputs simultaneously and return ordered lists. [4]

**SingleTurnSample** represents an individual evaluation case for single-turn interactions. Fields include `user_input` (the query), `response` (the application output), `reference` (the expected ground truth), and `retrieved_contexts` (supporting documents for RAG systems). [5]

**MultiTurnSample** represents evaluation cases for conversational interactions spanning multiple exchanges, used with multi-turn metrics such as Agent Goal Accuracy and Topic Adherence. [4]

**EvaluationDataset** aggregates multiple sample instances into a cohesive test suite for collective evaluation rather than one-at-a-time scoring. [5]

**Experiments** are the core workflow pattern. An experiment consists of loading a dataset, querying your application, scoring responses with metrics, and saving results. The `@experiment()` decorator orchestrates this loop, and results are persisted as CSV files in an `evals/experiments/` directory for comparison across runs. [5][6]

**Knowledge Graph** is the backbone of Ragas's synthetic test data generation. Documents are chunked, entities and relationships are extracted, and a graph structure is built. Queries of varying complexity (single-hop, multi-hop, abstract, specific) are then synthesized from this graph. [7]

**llm_factory** is the factory function for configuring which LLM powers the evaluation. It supports direct providers (OpenAI, Anthropic, Google) and 100+ additional providers via LiteLLM, including Azure OpenAI, AWS Bedrock, and Google Vertex AI. [8]

## Installation and Setup

Install the core package:

```bash
pip install ragas
```

For the latest development version:

```bash
pip install git+https://github.com/explodinggradients/ragas.git
```

For editable development:

```bash
git clone https://github.com/explodinggradients/ragas.git
pip install -e .
```

When using LangChain OpenAI integrations, install compatible versions explicitly to prevent dependency conflicts:

```bash
pip install -U "langchain-core>=0.2,<0.3" "langchain-openai>=0.1,<0.2" openai
```

Configure the LLM provider API key. OpenAI is the default:

```bash
export OPENAI_API_KEY="your-openai-key"
```

For Anthropic:

```bash
export ANTHROPIC_API_KEY="your-anthropic-key"
```

For Google Gemini:

```bash
export GOOGLE_API_KEY="your-google-api-key"
```

Scaffold a new evaluation project using the CLI:

```bash
uvx ragas quickstart rag_eval
cd rag_eval
uv sync
```

This creates a project structure with evaluation scripts, dataset directories, and experiment output folders:

```
rag_eval/
├── README.md
├── pyproject.toml
├── rag.py              # Your RAG application
├── evals.py            # Evaluation workflow
├── __init__.py
└── evals/
    ├── datasets/       # Test data
    ├── experiments/    # Results
    └── logs/           # Logs
```

Run the evaluation:

```bash
uv run python evals.py
```

Disable anonymous analytics if desired:

```bash
export RAGAS_DO_NOT_TRACK=true
```

[1][2][6]

## Architecture

Ragas is organized into four architectural layers:

```
Evaluation Framework      Schemas, metrics, evaluate/aevaluate execution
    |
Test Data Generation      Knowledge graph, scenario generators, synthesizers
    |
Core Infrastructure       Prompt management, LLM/embedding config, tokenizers, caching
    |
Customization Layer       Model adaptation, language localization, metric training
```

**Evaluation Framework** provides the `evaluate()` and `aevaluate()` functions that orchestrate metric scoring across datasets. It defines the `SingleTurnSample` and `MultiTurnSample` schemas, the `EvaluationDataset` container, and the `@experiment()` decorator for structured evaluation loops. Results are output to the console and persisted as CSV. [5]

**Test Data Generation** builds synthetic evaluation datasets from source documents. The pipeline processes documents through a Document Splitter (chunking), Extractors (entity and relationship extraction via LLM or rules), Relationship Builders (connecting nodes by similarity), and a QuerySynthesizer (generating queries of varying complexity and style). The `Parallel` wrapper enables concurrent execution of multiple pipeline components. [7]

**Core Infrastructure** handles LLM and embedding model configuration through `llm_factory` and `embedding_factory`, which abstract over providers using LiteLLM and the Instructor library. Prompt objects manage evaluation prompt templates. RunConfig controls execution parameters (timeouts, retries). A caching layer (DiskCacheBackend) avoids redundant LLM calls. [8]

**Customization Layer** enables prompt modification within metrics, language adaptation for non-English evaluations, persona-based test generation, and metric training via the `align()` method with few-shot examples. [3][8]

### Metric Class Hierarchy

```
Metric (abstract base)
├── MetricWithLLM          Uses LLM for evaluation, has .llm attribute
├── SingleTurnMetric       Scores via single_turn_ascore()
├── MultiTurnMetric        Scores via multi_turn_ascore()
└── SimpleBaseMetric
    ├── SimpleLLMMetric    LLM-based with save/load/align capabilities
    ├── DiscreteMetric     Returns categorical values
    ├── NumericMetric      Returns float values in a range
    └── RankingMetric      Returns ordered lists
```

[4][9]

## Key Features and Functionality

### RAG-Specific Metrics

Metrics designed for evaluating retrieval-augmented generation pipelines:

- **Context Precision**: Measures whether the retrieved context contains only relevant information, penalizing irrelevant chunks that dilute signal
- **Context Recall**: Measures whether all relevant information needed to answer the query was retrieved
- **Context Entities Recall**: Evaluates entity-level coverage in retrieved contexts compared to the reference
- **Response Relevancy**: Scores how relevant the generated response is to the original query
- **Faithfulness**: Determines whether claims in the generated response are supported by the retrieved context, detecting hallucinations
- **Noise Sensitivity**: Measures how much irrelevant context degrades the quality of the generated response
- **Multimodal Faithfulness**: Extends faithfulness evaluation to multimodal (text and image) contexts
- **Multimodal Relevance**: Extends relevancy evaluation to multimodal outputs

### Agent and Tool Use Metrics

Metrics for evaluating autonomous agent applications:

- **Tool Call Accuracy**: Evaluates whether the agent called the correct tools with appropriate parameters
- **Tool Call F1**: Measures precision and recall of tool invocations against expected tool calls
- **Agent Goal Accuracy**: Assesses whether the agent achieved the intended goal of the interaction
- **Topic Adherence**: Evaluates whether the agent stayed on-topic throughout the conversation

### Natural Language Comparison Metrics

Metrics for comparing generated text against references:

- **Factual Correctness**: Checks factual accuracy of generated content against a reference
- **Semantic Similarity**: Measures meaning-level similarity between generated and reference text using embeddings

### Traditional NLP Metrics

Deterministic metrics from established NLP research:

- **BLEU Score**: N-gram precision between generated and reference text
- **ROUGE Score**: Recall-oriented overlap measurement
- **CHRF Score**: Character-level F-score
- **String Presence**: Checks for presence of specific strings
- **Exact Match**: Binary match comparison

### General Purpose Metrics

Flexible metrics for custom evaluation criteria:

- **Aspect Critic**: Evaluates a specific aspect of the response using LLM judgment with a custom rubric
- **Simple Criteria Scoring**: Scores responses against user-defined criteria
- **Rubrics-based Scoring**: Multi-level rubric evaluation with detailed scoring guidelines
- **Instance-specific Rubrics Scoring**: Rubric evaluation with per-instance scoring criteria

### SQL Evaluation Metrics

- **Execution-based Datacompy Score**: Compares SQL query outputs by executing both generated and reference queries
- **SQL Query Equivalence**: Determines whether two SQL queries are semantically equivalent

### Synthetic Test Data Generation

Ragas generates evaluation datasets from source documents through a knowledge graph pipeline:

1. **Document Splitting**: Hierarchical chunking with support for domain-specific splitters (e.g., financial documents split by Income Statement, Balance Sheet, Cash Flow Statement sections)
2. **Entity Extraction**: LLM-based (`LLMBasedExtractor`) and rule-based (`Extractor`, `NERExtractor`) extraction of entities and properties
3. **Relationship Building**: Connecting document nodes by entity overlap using `JaccardSimilarityBuilder`
4. **Query Synthesis**: `QuerySynthesizer` generates queries combining different node pairs, query lengths (short, medium, long), and query styles (web search, chat)

### Custom Metrics

Define custom evaluation logic using the `DiscreteMetric` class:

```python
from ragas.metrics import DiscreteMetric
from ragas.llms import llm_factory

evaluator_llm = llm_factory("gpt-4o")

accuracy_metric = DiscreteMetric(
    name="summary_accuracy",
    prompt="Evaluate if the response accurately summarizes the content. "
           "Return 'accurate' or 'inaccurate'.",
    allowed_values=["accurate", "inaccurate"],
    llm=evaluator_llm,
)
```

[1][3][4][7]

## Use Cases

**RAG Pipeline Evaluation**: Score retrieval quality (Context Precision, Context Recall) and generation quality (Faithfulness, Response Relevancy) across a test dataset to identify whether poor performance stems from retrieval failures or generation hallucinations. [1]

**Agent Testing**: Verify that autonomous agents call the correct tools with appropriate parameters (Tool Call Accuracy), achieve their intended goals (Agent Goal Accuracy), and stay within scope (Topic Adherence). [1]

**Prompt Engineering**: Compare prompt variants by running the same test dataset through different prompts and comparing metric scores across experiments. The experiments-first approach enables systematic A/B testing of prompt changes. [6]

**Regression Testing**: Establish baseline metric scores and detect quality degradation when models, prompts, or retrieval strategies change. CSV-based experiment tracking enables comparison across runs. [5]

**Test Dataset Creation**: Generate synthetic evaluation datasets from domain documents using the knowledge graph pipeline, avoiding the cost and time of manual curation while achieving diverse coverage of single-hop, multi-hop, abstract, and specific query types. [7]

**Multi-Language Evaluation**: Adapt metrics and test generation for non-English applications using language localization features and persona-based generation for diverse user profiles. [3]

**SQL Generation Evaluation**: Validate LLM-generated SQL queries against reference queries using execution-based comparison (Datacompy Score) and semantic equivalence checking. [1]

## API Reference Summary

### Evaluation Functions

- `evaluate(dataset, metrics, llm, embeddings)` -- Synchronous evaluation across a dataset with specified metrics
- `aevaluate(dataset, metrics, llm, embeddings)` -- Asynchronous evaluation with the same interface

### Sample Classes

- `SingleTurnSample(user_input, response, reference, retrieved_contexts)` -- Single interaction evaluation case
- `MultiTurnSample(...)` -- Multi-turn conversation evaluation case

### Dataset

- `EvaluationDataset(samples)` -- Container for evaluation samples; supports iteration and batch operations
- `Dataset(name, backend, root_dir)` -- Named dataset with storage backend for experiment workflows

### Metric Base Classes

- `Metric` -- Abstract base; requires `init(run_config)` implementation
- `MetricWithLLM` -- Adds `.llm` attribute; validates LLM presence on init; supports `train()` for few-shot optimization
- `SingleTurnMetric` -- Implements `single_turn_score()` and `single_turn_ascore()` with timeout
- `MultiTurnMetric` -- Implements `multi_turn_score()` and `multi_turn_ascore()` with timeout
- `SimpleBaseMetric` -- Returns `MetricResult` objects; supports `batch_score()` and `abatch_score()`
- `SimpleLLMMetric` -- Adds `save()`, `load()`, `align()`, `validate_alignment()` for persistence and optimization

### Metric Types

- `DiscreteMetric(name, prompt, allowed_values, llm)` -- Categorical output metric
- `NumericMetric(name, prompt, range, llm)` -- Float/integer output metric within a range
- `RankingMetric(name, prompt, llm)` -- Ordered comparison output metric

### Factory Functions

- `llm_factory(model, provider, client)` -- Create LLM instance for evaluation; supports OpenAI, Anthropic, Google, and 100+ providers via LiteLLM
- `embedding_factory(...)` -- Create embedding model instance

### Test Data Generation

- `TestsetGenerator(llm, knowledge_graph)` -- Generates synthetic test datasets from source documents
- `QuerySynthesizer(...)` -- Generates queries from knowledge graph nodes with configurable length and style
- `apply_transforms(transforms)` -- Executes a sequence of extraction and relationship-building transforms
- `Parallel(transforms)` -- Wrapper for concurrent execution of multiple transforms

### Utilities

- `Ensembler.from_discrete(inputs, attribute)` -- Majority voting across multiple LLM outputs for ensemble scoring
- `create_auto_response_model(name, **fields)` -- Generates Pydantic models for structured metric responses
- `RunConfig(timeout, max_retries)` -- Execution parameter configuration

[4][5][9]

## Configuration and Customization

### LLM Provider Configuration

**OpenAI (default)**:

```python
from ragas.llms import llm_factory

evaluator_llm = llm_factory("gpt-4o")
```

**Anthropic Claude**:

```python
from anthropic import Anthropic
from ragas.llms import llm_factory

client = Anthropic(api_key=os.environ.get("ANTHROPIC_API_KEY"))
evaluator_llm = llm_factory(
    "claude-3-5-sonnet-20241022",
    provider="anthropic",
    client=client,
)
```

**Google Gemini**:

```python
from ragas.llms import llm_factory

evaluator_llm = llm_factory("gemini-pro", provider="google")
```

**Local Ollama**:

```python
from openai import OpenAI
from ragas.llms import llm_factory

client = OpenAI(api_key="ollama", base_url="http://localhost:11434/v1")
evaluator_llm = llm_factory("mistral", provider="openai", client=client)
```

**Azure OpenAI / AWS Bedrock / Vertex AI**: Supported through LiteLLM with provider-specific configuration (deployment names, API base URLs, region settings, credentials). [6][8]

### System Prompts

Customize LLM behavior during evaluation by providing system prompts, useful for fine-tuned models or guiding evaluation consistency:

```python
evaluator_llm = llm_factory(
    "gpt-4o",
    system_prompt="You are a strict evaluator. Score conservatively.",
)
```

### Prompt Modification

Modify the internal prompts used by built-in metrics to tailor evaluation criteria for specific domains or quality standards. [3]

### Language Adaptation

Adapt metrics and test data generation to non-English languages. Configure language localization for both evaluation prompts and synthetic dataset generation. [3]

### Metric Training and Alignment

Optimize LLM-based metrics using few-shot examples with the `align()` method:

```python
metric.align(train_dataset, embedding_model)
metric.validate_alignment(llm, test_dataset)
```

The `validate_alignment()` method computes correlation and agreement metrics between the trained metric and human judgments. Trained metrics can be persisted with `save()` and restored with `load()`. [9]

### Persona-Based Test Generation

Configure or automatically generate personas representing diverse user types for test data generation, creating more realistic and varied evaluation scenarios. [3]

### Caching

DiskCacheBackend avoids redundant LLM calls during evaluation, reducing cost and latency for repeated metric computations. [2]

## Integration Patterns

### With LangChain

Evaluate LangChain RAG pipelines by extracting retrieved contexts and generated responses into Ragas `SingleTurnSample` objects. Ragas metrics score both the retrieval quality and generation fidelity of the chain. [3]

### With LlamaIndex

Supports both RAG and agent-based LlamaIndex applications. The RAG integration evaluates retrieval and generation quality, while the agent integration evaluates tool use and goal completion. [3]

### With Haystack

Evaluate Haystack pipeline outputs using Ragas metrics. The integration supports Haystack's document retrieval and generation components. [3]

### With LangGraph

Evaluate LangGraph agent workflows using agent-specific metrics (Tool Call Accuracy, Agent Goal Accuracy, Topic Adherence) that assess multi-step agentic behavior. [3]

### With LangSmith

Trace evaluator LLM calls through LangSmith for observability into the evaluation process itself. Captures prompt inputs, model outputs, and latency for each metric computation. [3]

### With Arize Phoenix

Monitor evaluation processes through Arize Phoenix's observability platform, providing traces of the evaluator LLMs during Ragas scoring. [3]

### Additional Framework Integrations

Ragas also provides integration guides for Griptape, Amazon Bedrock, Oracle Cloud Infrastructure (OCI) Generative AI, R2R, LlamaStack, Swarm, and the AG-UI Protocol. [3]

## Examples

**Discrete Metric Evaluation**:

```python
from ragas.metrics import DiscreteMetric
from ragas.llms import llm_factory

evaluator_llm = llm_factory("gpt-4o")

correctness = DiscreteMetric(
    name="correctness",
    prompt="Check if the response contains points mentioned in the "
           "grading notes. Return 'pass' or 'fail'.",
    allowed_values=["pass", "fail"],
    llm=evaluator_llm,
)
```

**RAG Evaluation with Experiment Decorator**:

```python
import os
import pandas as pd
from ragas import Dataset
from ragas.metrics import DiscreteMetric
from ragas.llms import llm_factory

evaluator_llm = llm_factory("gpt-4o")

# Define metric
accuracy = DiscreteMetric(
    name="accuracy",
    prompt="Evaluate if the response accurately addresses the query "
           "based on the grading notes. Return 'pass' or 'fail'.",
    allowed_values=["pass", "fail"],
    llm=evaluator_llm,
)

# Load dataset
samples = [
    {
        "question": "What is Ragas?",
        "grading_notes": "Ragas is a framework for evaluating LLM applications",
    },
    {
        "question": "How do metrics work in Ragas?",
        "grading_notes": "Metrics quantify AI application performance using "
                         "LLM-based and traditional scoring methods",
    },
]
df = pd.DataFrame(samples)
df.to_csv("evals/datasets/test_dataset.csv", index=False)
```

**Test Data Generation from Documents**:

```python
from ragas.testset import TestsetGenerator
from ragas.testset.transforms import apply_transforms, Parallel
from ragas.testset.synthesizers import QuerySynthesizer

# Build knowledge graph from documents
transforms = [
    # Document splitter, extractors, relationship builders
]
apply_transforms(transforms)

# Generate synthetic test queries
synthesizer = QuerySynthesizer()
scenarios = synthesizer.generate_scenarios(
    # Configure query length, style, and node pairs
)
```

**Custom LLM Provider with Ollama**:

```python
from openai import OpenAI
from ragas.llms import llm_factory
from ragas.metrics import DiscreteMetric

# Configure local Ollama
client = OpenAI(api_key="ollama", base_url="http://localhost:11434/v1")
local_llm = llm_factory("mistral", provider="openai", client=client)

# Use local model for evaluation
tone_metric = DiscreteMetric(
    name="tone",
    prompt="Evaluate if the response tone is professional. "
           "Return 'professional' or 'casual'.",
    allowed_values=["professional", "casual"],
    llm=local_llm,
)
```

[5][6]

## Limitations and Considerations

**LLM Cost**: LLM-based metrics make one or more LLM calls per sample per metric. Evaluating large datasets with multiple metrics can accumulate significant API costs. Caching mitigates this for repeated runs but not for first-pass evaluations. [1]

**Non-Determinism**: LLM-based metrics are inherently non-deterministic. The same sample evaluated twice may produce different scores. Ragas provides an `Ensembler` for majority voting across multiple runs, but this multiplies cost. [4]

**Evaluator Model Quality**: Metric quality depends on the evaluator LLM. Weaker models may produce less reliable scores. The framework defaults to OpenAI models but supports swapping to any provider. [8]

**Metric Correlation with Humans**: While LLM-based metrics correlate more closely with human judgment than traditional NLP metrics, they are not a substitute for human evaluation on critical applications. The `validate_alignment()` method quantifies this correlation but requires labeled data. [4][9]

**Test Data Representativeness**: Synthetic test data generated from knowledge graphs may not fully represent real user queries in distribution, phrasing, or intent. Production data seeding is available but requires an existing production system. [7]

**API Dependency**: Most evaluation workflows require an LLM API. Local model support via Ollama is available but may produce lower-quality metric scores compared to frontier models. [6][8]

**Rapid Iteration**: The framework is under active development with significant API changes across versions (v0.3 to v0.4). Code written against older versions may require refactoring. [10]

**Single Language Bias**: Default prompts and metrics are English-centric. Non-English evaluation requires explicit language adaptation configuration. [3]

## Changelog Highlights

- **v0.4.3** (January 2026): DSPyOptimizer with MIPROv2 for prompt optimization, system prompt support for InstructorLLM and LiteLLMStructuredLLM, DiskCacheBackend fixes
- **v0.4.2** (December 2024): Migrated multiple metrics (SQL, CHRF, Multimodal) to collections API, AG-UI Protocol integration, new google-genai SDK support, caching for metrics collections and embeddings
- **v0.4.1** (December 2024): Save/load for BasePrompt, Tool Call and Topic Adherence metrics migrated to collections API, improved async embedding support
- **v0.4.0** (December 2024): Major refactor to modular BasePrompt architecture, Instructor provider migration for universal provider support, dual adapter support (Instructor + LiteLLM), GPT-5 and o-series model support
- **Quickstart CLI**: `ragas quickstart` command with project templates for RAG evaluation (agent, benchmark, prompt, and workflow templates coming soon)
- **Knowledge Graph Test Generation**: Document-to-knowledge-graph pipeline for synthetic test data with single-hop and multi-hop query synthesis

[2][10]

## Citations

- [1] Ragas Documentation - <https://docs.ragas.io/en/stable/>
- [2] Ragas GitHub Repository - <https://github.com/explodinggradients/ragas>
- [3] Integrations and Customizations - <https://docs.ragas.io/en/stable/howtos/integrations/>
- [4] Metrics Overview - <https://docs.ragas.io/en/stable/concepts/metrics/overview/>
- [5] Evaluation Getting Started - <https://docs.ragas.io/en/stable/getstarted/evals/>
- [6] Quickstart Guide - <https://docs.ragas.io/en/stable/getstarted/quickstart/>
- [7] Test Data Generation - <https://docs.ragas.io/en/stable/concepts/test_data_generation/>
- [8] Customize Models - <https://docs.ragas.io/en/stable/howtos/customizations/customize_models>
- [9] Metrics API Reference - <https://docs.ragas.io/en/stable/references/metrics/>
- [10] Release Notes - <https://github.com/explodinggradients/ragas/releases>
