# Outlines

> Python library for structured Large Language Model (LLM) generation via JSON Schema, regex, and context-free grammars. Guarantees structured outputs during the generation process itself rather than through post-hoc parsing.

| Field | Value |
|-------|-------|
| Name | Outlines |
| Group | Structured Generation |
| Type | SDK |
| Open Source | Yes |
| GitHub | [dottxt-ai/outlines](https://github.com/dottxt-ai/outlines) |
| Stars | 13838 |
| Docs | [dottxt-ai.github.io/outlines](https://dottxt-ai.github.io/outlines/) |
| License | Apache 2.0 |
| Language | Python |

## Overview

Outlines is a Python library developed by dottxt-ai that enables structured text generation from LLMs. Unlike approaches that generate free-form text and then attempt to parse it into a desired format, Outlines constrains the generation process at the token level using Finite-State Machines (FSMs) and specialized backends. This means every token produced by the model is guaranteed to conform to the specified structure, eliminating parsing failures, broken JSON, and malformed outputs entirely.

The library supports a range of structured output formats including JSON Schema, regular expressions, Context-Free Grammars (CFGs), native Python types, Pydantic models, and multiple-choice selection. It integrates with major LLM providers and inference engines, making it a versatile tool for any workflow that requires reliable, machine-readable output from language models.

Outlines is used in production by organizations including Amazon, Apple, Databricks, and Meta.

## Core Concepts

- **Generation-Time Constraints**: Outlines applies structural constraints during the token generation process rather than after it. Each token is validated against the target schema before being emitted, ensuring 100% conformance without retry loops or post-processing.
- **Finite-State Machine (FSM) Guided Decoding**: The library compiles output schemas (JSON Schema, regex patterns, grammars) into FSMs that mask invalid tokens at each generation step. Only tokens that maintain a valid path through the FSM are considered by the model's sampling procedure.
- **Schema Compilation**: Schemas are compiled into their FSM representations once per session. Subsequent generation calls reuse the compiled representation, amortizing the compilation cost across multiple invocations.
- **Backend Agnosticism**: Outlines decouples the structured generation logic from the model backend. The same schema definition works across OpenAI, Anthropic, vLLM, Hugging Face Transformers, Ollama, and Gemini without modification.
- **Type-Safe Output**: When using Pydantic models or Python type annotations, the generated output is automatically deserialized into the corresponding typed object, providing immediate programmatic access without manual parsing.

## Architecture

Outlines is organized around three primary layers:

- **Schema Layer**: Accepts user-defined output specifications in the form of JSON Schema objects, regex patterns, CFGs, Pydantic models, Python type annotations, or enumerated choices. This layer validates and normalizes the schema definition.
- **Compilation Layer**: Transforms the normalized schema into an FSM or equivalent constraint representation. Compilation happens once per unique schema within a session. The compiled artifact encodes all valid token sequences that satisfy the schema.
- **Generation Layer**: Interfaces with the LLM backend to perform constrained decoding. At each generation step, the FSM state determines which tokens are valid continuations. Invalid tokens are masked (assigned zero probability) before sampling, ensuring the output always conforms to the schema.

The separation of these layers allows Outlines to support multiple backends through a common interface while keeping the constraint logic centralized and reusable.

## Key Features and Functionality

- **JSON Schema Generation**: Define output structure using JSON Schema and receive guaranteed-valid JSON from any supported model. Supports nested objects, arrays, enums, optional fields, and all standard JSON Schema constructs.
- **Regex-Constrained Generation**: Specify output format using regular expressions. Useful for dates, phone numbers, identifiers, and other pattern-based formats.
- **Context-Free Grammar (CFG) Support**: Define output structure using formal grammars for complex, recursive structures that go beyond what regex can express.
- **Pydantic Model Integration**: Pass a Pydantic model class directly and receive a fully instantiated, validated model object as output.
- **Python Type Support**: Use native Python types (str, int, float, bool, lists, dicts) as output specifications for simple structured outputs.
- **Multiple-Choice Selection**: Constrain the model to select from a predefined set of options, useful for classification and decision-making tasks.
- **One-Time Compilation**: Schemas are compiled into FSMs once per session, making repeated generation calls with the same schema efficient.
- **Multi-Provider Support**: Works with OpenAI, Anthropic, vLLM, Hugging Face Transformers, Ollama, and Gemini through a unified interface.

## Use Cases

- **Classification**: Constrain model output to a fixed set of labels for text classification, sentiment analysis, or intent detection tasks.
- **Named Entity Recognition (NER)**: Extract structured entity data from unstructured text with guaranteed output format conformance.
- **Knowledge Graph Construction**: Generate structured triples (subject, predicate, object) from text for knowledge graph population.
- **Question Answering with Citations**: Produce answers that include structured citation references pointing back to source material.
- **PDF and Document Processing**: Extract structured data from unstructured document content with reliable output formatting.
- **ReAct Agents**: Generate structured action-observation-thought sequences for agent-based reasoning frameworks.
- **Data Extraction Pipelines**: Convert unstructured text into structured records for database ingestion or downstream processing.
- **Form Generation**: Produce structured form data from natural language descriptions.

## API Reference Summary

### Model Initialization

```python
import outlines

# OpenAI backend
model = outlines.models.openai("gpt-4o")

# Transformers backend
model = outlines.models.transformers("mistralai/Mistral-7B-v0.1")

# vLLM backend
model = outlines.models.vllm("mistralai/Mistral-7B-v0.1")

# Ollama backend
model = outlines.models.ollama("llama3")
```

### JSON Schema Generation

```python
from pydantic import BaseModel
import outlines

class Customer(BaseModel):
    name: str
    age: int
    email: str

model = outlines.models.openai("gpt-4o")
generator = outlines.generate.json(model, Customer)

result = generator("Alice needs help with her account.")
# result is a Customer instance with guaranteed valid fields
```

### Regex-Constrained Generation

```python
import outlines

model = outlines.models.openai("gpt-4o")
date_pattern = r"\d{4}-\d{2}-\d{2}"
generator = outlines.generate.regex(model, date_pattern)

result = generator("What is today's date?")
# result matches the YYYY-MM-DD pattern
```

### Multiple-Choice Selection

```python
import outlines

model = outlines.models.openai("gpt-4o")
generator = outlines.generate.choice(model, ["positive", "negative", "neutral"])

result = generator("Classify the sentiment: 'I love this product!'")
# result is one of "positive", "negative", or "neutral"
```

### Grammar-Based Generation

```python
import outlines

model = outlines.models.transformers("mistralai/Mistral-7B-v0.1")
grammar = r"""
    start: expression
    expression: term (("+"|"-") term)*
    term: NUMBER
    NUMBER: /[0-9]+/
"""
generator = outlines.generate.cfg(model, grammar)

result = generator("Generate a simple arithmetic expression.")
```

### Text Generation (Unconstrained)

```python
import outlines

model = outlines.models.openai("gpt-4o")
generator = outlines.generate.text(model)

result = generator("Tell me a story.")
```

## Configuration and Customization

- **Schema Compilation Caching**: Compiled FSMs are cached for the duration of the session. No explicit configuration is required; reusing the same generator object across calls leverages the cached compilation.
- **Sampling Parameters**: Generation calls accept standard sampling parameters (temperature, top_p, max_tokens) through the underlying model backend configuration.
- **Backend Selection**: The backend is determined by the model initialization call. Each backend may support additional configuration options specific to the provider (API keys, base URLs, device placement).

## Integration Patterns

### With Pydantic for Validated Outputs

```python
from pydantic import BaseModel, Field
import outlines

class Invoice(BaseModel):
    vendor: str
    amount: float = Field(ge=0)
    currency: str
    date: str

model = outlines.models.openai("gpt-4o")
generator = outlines.generate.json(model, Invoice)

invoice = generator("Extract invoice data: Acme Corp charged $150.00 USD on 2025-03-15")
# invoice.vendor == "Acme Corp", invoice.amount == 150.0, etc.
```

### With vLLM for High-Throughput Inference

```python
import outlines
from pydantic import BaseModel

class Entity(BaseModel):
    name: str
    entity_type: str
    confidence: float

model = outlines.models.vllm("mistralai/Mistral-7B-v0.1")
generator = outlines.generate.json(model, Entity)

results = [generator(text) for text in batch_of_texts]
```

### With ReAct Agent Patterns

```python
from pydantic import BaseModel
from typing import Literal
import outlines

class AgentStep(BaseModel):
    thought: str
    action: Literal["search", "calculate", "respond"]
    action_input: str

model = outlines.models.openai("gpt-4o")
generator = outlines.generate.json(model, AgentStep)

step = generator("The user asked about the weather in Paris. Think step by step.")
# step.action is guaranteed to be one of the valid actions
```

## Examples

### Named Entity Recognition

```python
from pydantic import BaseModel
import outlines

class ExtractedEntities(BaseModel):
    persons: list[str]
    organizations: list[str]
    locations: list[str]

model = outlines.models.openai("gpt-4o")
generator = outlines.generate.json(model, ExtractedEntities)

text = "Tim Cook announced that Apple will open a new office in Austin, Texas."
entities = generator(f"Extract named entities from: {text}")
# entities.persons == ["Tim Cook"]
# entities.organizations == ["Apple"]
# entities.locations == ["Austin", "Texas"]
```

### Text Classification with Confidence

```python
from pydantic import BaseModel
from typing import Literal
import outlines

class Classification(BaseModel):
    label: Literal["spam", "not_spam"]
    confidence: float
    reasoning: str

model = outlines.models.openai("gpt-4o")
generator = outlines.generate.json(model, Classification)

result = generator("Classify this email: 'Congratulations! You won a free iPhone!'")
# result.label is guaranteed to be "spam" or "not_spam"
```

### Structured Q&A with Citations

```python
from pydantic import BaseModel
import outlines

class Citation(BaseModel):
    text: str
    source: str
    page: int

class Answer(BaseModel):
    answer: str
    citations: list[Citation]

model = outlines.models.openai("gpt-4o")
generator = outlines.generate.json(model, Answer)

context = "According to Smith (2024, p.12), transformers revolutionized NLP..."
result = generator(f"Answer with citations based on: {context}\nQuestion: What revolutionized NLP?")
# result.citations contains structured Citation objects
```

## Limitations and Considerations

- **Compilation Overhead**: The initial compilation of a schema into an FSM adds latency to the first generation call. Complex schemas with deeply nested structures or large enumerations increase this overhead.
- **Grammar Support Variability**: CFG support may vary across backends. Not all providers support grammar-based constrained generation natively.
- **Structural vs. Semantic Guarantees**: The constrained decoding operates at the token level, which means the model may produce semantically incorrect but structurally valid output. The structure is guaranteed; the semantic quality depends on the underlying model.
- **Backend Feature Parity**: Not all backends support every generation mode. Some constrained generation features may be available only with local model backends (Transformers, vLLM) and not with API-based providers.
- **Local Model Requirements**: When using Transformers or vLLM backends, adequate GPU memory and compute resources are required to run the models locally.

## Changelog Highlights

Outlines is under active development. The project maintains releases on PyPI and GitHub. Refer to the [GitHub releases page](https://github.com/dottxt-ai/outlines/releases) for version history and detailed changelogs.

## Citations

- [1] [Outlines Documentation](https://dottxt-ai.github.io/outlines/latest/)

## Discovery Signals

> Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search.

### Keywords

- Outlines
- constrained decoding
- token-level constraints
- finite-state machine
- FSM-guided decoding
- JSON Schema generation
- regex-constrained generation
- context-free grammar
- CFG support
- multiple-choice selection
- guaranteed valid output
- schema compilation
- Pydantic generator
- structured generation
- backend-agnostic
- vLLM constrained
- Transformers backend
- Ollama constrained
- one-time compilation
- dottxt-ai

### Verb-Noun Tasks

- Constrain LLM output to a JSON Schema at the token level
- Generate text matching a regex pattern (dates, phone numbers, IDs)
- Restrict output to a fixed set of choices for classification
- Generate output conforming to a context-free grammar
- Compile a Pydantic model into an FSM once and reuse across calls
- Generate typed Python objects directly from a Pydantic class
- Run constrained generation locally via Transformers or vLLM
- Build a ReAct agent with guaranteed-valid action enums
- Extract named entities into a typed list of strings
- Produce structured citations alongside generated answers

### User Intent Phrases

- How do I guarantee an LLM produces valid JSON without retries?
- How can I constrain output to a regex pattern?
- I want generation that's always schema-conformant by construction
- How do I make the model only choose from a fixed list of labels?
- How do I generate output following a formal grammar?
- How can I run constrained decoding on a local Mistral model?
- How do I avoid parse failures on LLM-generated JSON entirely?
- How do I get typed Pydantic instances back without parsing strings?
- How does FSM-guided decoding work?
- Can I use the same schema across OpenAI, vLLM, and Ollama?

### Problem Statements

- Post-hoc parsing of LLM strings fails on malformed JSON
- Retry-based approaches add latency and cost per validation failure
- Free-form generation cannot guarantee schema conformance
- Schema compilation overhead adds latency on the first call
- Grammar support varies across backends (some API providers don't expose it)
- Constrained generation guarantees structure, not semantic correctness
- Local backends (Transformers, vLLM) require GPU resources

### When to Pick This

- Pick Outlines when retries are unacceptable and you need guaranteed valid output at the token level
- Pick Outlines when you have local model access (Transformers, vLLM, Ollama) and want maximum throughput with constrained decoding
- Pick Outlines for latency-critical systems where validation-and-retry is too expensive
- Pick Outlines when output must match a regex or context-free grammar, not just a JSON schema
- Pick Instructor instead when you need broad hosted-provider support and don't have token-level model access
- Pick BAML instead when you want a typed DSL plus code generation across multiple languages
- Pick DSPy instead when you want to optimize the prompt itself against a metric

### Related Terms and Aliases

- guided generation
- structured generation library
- FSM-based decoding
- token-masking generation
- grammar-constrained LLM
- regex-constrained LLM
- JSON-Schema-constrained LLM
- dottxt outlines
- constrained sampling
