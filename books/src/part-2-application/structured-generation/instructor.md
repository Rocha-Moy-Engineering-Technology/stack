# Instructor

> Multi-language library for extracting structured data from LLMs

| Field | Value |
|-------|-------|
| Name | Instructor |
| Group | Structured Generation |
| Type | SDK |
| Open Source | Yes |
| GitHub | [instructor-ai/instructor](https://github.com/instructor-ai/instructor) |
| Stars | 12,413 |
| Docs | [python.useinstructor.com](https://python.useinstructor.com/) |

## Overview

Instructor is a Python library for extracting structured, validated data from Large Language Models (LLMs). With over 3 million monthly downloads, 12,500+ GitHub stars, and 100+ contributors, it is one of the most widely adopted tools for structured output extraction. Built on top of Pydantic, Instructor lets developers define response schemas as Python models and have the LLM fill them in directly. When the LLM output fails validation, Instructor automatically retries the request with the validation error context, enabling self-correcting extraction pipelines. The library supports 23+ LLM providers through a unified interface, offers streaming for partial results, and provides full type inference with IDE autocompletion. Instructor is available in Python, TypeScript, Go, Ruby, Elixir, and Rust, though the Python implementation is the most mature and widely used. The project is licensed under the MIT License and authored by Jason Liu.

Instructor positions itself as a focused tool for structured extraction rather than a full agent framework. As the documentation notes: "Instructor for extraction, PydanticAI for agents." When a project requires quality gates, shareable runs, or built-in observability, the Pydantic team recommends PydanticAI as the complementary agent runtime that works alongside Instructor models.

## Core Concepts

### Structured Outputs via Pydantic Models

The fundamental idea behind Instructor is that a Pydantic model defines the expected shape of the LLM response. The `response_model` parameter serves three purposes: defining the schema and prompts for the language model, validating API responses, and returning Pydantic model instances. Docstrings, types, and field annotations are used to generate the prompt automatically. The library injects the model schema into the LLM request (via function calling, tool use, or JSON mode depending on the provider), parses the raw output, and validates it against the model. The developer receives a fully typed Python object rather than a string.

### Automatic Retries (Reasks)

When the LLM returns output that fails Pydantic validation, Instructor does not simply raise an error. Instead, it feeds the validation error message back to the LLM as context and retries the request. This "reask" loop continues up to a configurable maximum number of retries (`max_retries`), giving the model the opportunity to self-correct. The system defends against two error types: Pydantic validation failures and JSON decoding errors. This is especially useful for enforcing constraints that are difficult to express purely in a prompt (value ranges, string formats, cross-field dependencies). Instructor also integrates with the Tenacity library for more advanced retry strategies including exponential backoff, error-specific retries, and result-based retries.

### Client Patching

Instructor works by wrapping (patching) existing LLM client libraries. Rather than replacing the client, it augments it with structured output capabilities. The patching process intercepts calls to completion methods like `create()`, transforms Pydantic models into provider-specific formats, checks outputs against the defined model, and handles retries when validation fails. Developers keep their existing authentication, configuration, and error handling while gaining schema-driven extraction on top. The patched client gains three new parameters: `response_model` (defines expected output structure), `max_retries` (retry attempts on validation failure), and `context` (additional validation context and Jinja template variables).

### Extraction Modes

Instructor supports multiple extraction modes depending on provider capabilities:

- **TOOLS** -- Uses the provider's function/tool calling API. The default and recommended mode for OpenAI, Anthropic, Google, and Ollama.
- **JSON_SCHEMA** -- Native schema support when providers offer it. Strict schema enforcement.
- **MD_JSON** -- Extracts JSON from markdown code blocks in the response. Useful for providers without tool calling support.
- **PARALLEL_TOOLS** -- Multiple tool calls in a single response for batch extraction.
- **RESPONSES_TOOLS** -- OpenAI Responses API tools integration.

The `from_provider()` function automatically selects the optimal mode for each provider, though modes can be overridden manually.

## Installation

Install the core package:

```bash
pip install instructor
```

Alternative package managers:

```bash
uv add instructor
poetry add instructor
```

Core dependencies installed automatically: `openai`, `pydantic`, `typer`, and `docstring-parser`. Python 3.9 or later is required.

Provider-specific client libraries must be installed separately depending on the target LLM backend:

```bash
pip install openai       # OpenAI
pip install anthropic    # Anthropic
pip install google-genai # Google Gemini
pip install ollama       # Ollama (local models)
pip install cohere       # Cohere
pip install mistralai    # Mistral
pip install litellm      # LiteLLM (multi-provider)
```

## Architecture

Instructor sits as a thin middleware layer between the application and the LLM provider client:

1. **Application Layer** -- Defines Pydantic response models and sends messages through the Instructor-patched client.
2. **Instructor Layer** -- Injects the Pydantic model schema into the LLM request, parses the response, runs Pydantic validation, and handles retries on failure.
3. **Provider Client Layer** -- The underlying LLM SDK (OpenAI, Anthropic, Google, and others) handles authentication, transport, and raw API communication.

The main execution pipeline flows through several stages: caching and templating, retry mechanisms powered by Tenacity, provider communication, and response dispatching. The dispatcher routes responses through mode-specific handlers (streaming, partial, standard) before parsing into Pydantic models. If validation fails, a reask handler prepares error feedback for retry attempts.

### Retry Flow

```
Application -> Instructor -> LLM Provider
                  |
                  v
          Parse response
                  |
           Validate with Pydantic
                  |
         [Pass] -> Return typed object
         [Fail] -> Append validation error to messages -> Retry LLM call
```

When retry attempts are exhausted, Instructor raises `InstructorRetryException` containing the full attempt history, final completion data, and reproduction parameters.

### Instrumentation Points

The framework provides hooks at critical points in the pipeline:

- `completion:kwargs` -- Before provider invocation
- `completion:response` -- After receiving response
- `completion:error` -- Error before retries
- `parse:error` -- On validation failures
- `completion:last_attempt` -- Before retry exhaustion

## Key Features

- **Type Safety and IDE Autocompletion** -- Response models are standard Pydantic classes, giving full type inference, autocomplete, and static analysis support in editors and type checkers.
- **Automatic Retries with Validation Context** -- Failed validations trigger retries where the error message is included in the next prompt, allowing the LLM to self-correct. Integrates with Tenacity for exponential backoff, error-specific retries, and result-based retries.
- **Multi-Provider Support** -- A single `from_provider()` interface supports OpenAI, Anthropic, Google, Ollama, DeepSeek, and 23+ other providers without changing application code.
- **Streaming** -- `create_partial()` streams partial results as the LLM generates tokens, enabling progressive UI updates. `create_iterable()` streams a sequence of complete objects. Both support async iteration.
- **Custom Pydantic Validators** -- Standard Pydantic field validators and model validators work seamlessly, enabling complex validation logic (regex patterns, cross-field checks, business rules).
- **LLM-Based Validation** -- The `llm_validator` function uses the LLM itself to validate outputs against semantic criteria, generating human-readable error messages for reasking.
- **Async/Await** -- Full async support for non-blocking LLM calls in asynchronous applications via `async_client=True`.
- **Jinja Templating** -- Prompt templates can use Jinja syntax for dynamic prompt construction with variables, loops, and conditionals. Templates are rendered in a sandboxed environment for security.
- **Hooks System** -- Lifecycle hooks allow injecting custom logic at various stages of the request/response cycle for logging, metrics, monitoring, or transformation.
- **Multimodal Extraction** -- Unified, provider-agnostic interface for extracting structured data from images, PDFs, and audio files with automatic format handling.
- **CLI Tools** -- Built-in command-line utilities for API usage monitoring (`instructor usage`), fine-tuning management (`instructor finetune`), and documentation access (`instructor docs`).
- **Dynamic Model Creation** -- Pydantic's `create_model()` enables runtime model generation when schemas are determined by database queries, user configurations, or other dynamic sources.

## Use Cases

- **Data Extraction** -- Pulling structured records (names, dates, amounts, entities) from unstructured text such as emails, documents, or web pages.
- **Classification** -- Categorizing text into predefined enums or labels with guaranteed valid output values.
- **Content Generation with Constraints** -- Generating content that must conform to a specific schema (product descriptions with required fields, quiz questions with exactly four options).
- **Multi-Step Pipelines** -- Chaining structured outputs where the validated result of one step feeds into the next, with type safety preserved throughout.
- **Search and Retrieval Augmented Generation (RAG)** -- Extracting structured queries or filters from natural language to drive database lookups or search APIs.
- **Streaming User Interfaces** -- Progressively rendering structured data in a UI as the LLM generates it, using partial streaming.
- **Document Processing** -- Extracting structured data from PDFs, images, and audio files using multimodal capabilities.
- **Content Moderation** -- Using `llm_validator` to check outputs against semantic criteria and reject objectionable content.

## API Reference

### Client Creation

```python
import instructor

# Universal provider interface (recommended)
client = instructor.from_provider("openai/gpt-4o")

# Async client
async_client = instructor.from_provider("openai/gpt-4o", async_client=True)

# Provider-specific patching
import openai
client = instructor.from_openai(openai.OpenAI())

import anthropic
client = instructor.from_anthropic(anthropic.Anthropic())

# With mode override
client = instructor.from_provider("openai/gpt-4o", mode=instructor.Mode.JSON)

# With caching
client = instructor.from_provider("openai/gpt-4o", cache=True)
```

The `from_provider()` function accepts a model string in the format `"provider/model"` and automatically handles provider-specific configurations. Additional provider-specific constructors include `from_openai()`, `from_anthropic()`, `from_google()`, `from_litellm()`, `from_ollama()`, and others.

### Core Methods

- **`client.create(response_model, messages, max_retries=3, validation_context=None, context=None, strict=None, hooks=None)`** -- Sends a completion request and returns a validated instance of `response_model`. Retries automatically on validation failure up to `max_retries`.
- **`client.create_with_completion(response_model, messages)`** -- Returns a tuple of `(response_model_instance, raw_completion)`, giving access to both the validated object and the raw provider response (token usage, finish reason, metadata).
- **`client.create_partial(response_model, messages)`** -- Returns an iterator that yields progressively more complete instances of `response_model` as tokens stream in. All fields become `Optional` during streaming. Validators are not applied until the final iteration.
- **`client.create_iterable(response_model, messages)`** -- Returns an iterator of complete `response_model` instances, useful when the LLM produces a list of structured objects.

### Hooks API

```python
# Register hooks
client.on("completion:kwargs", handler_function)
client.on("completion:response", lambda response: print(response))
client.on("completion:error", lambda error: log_error(error))
client.on("parse:error", lambda error: track_validation_failure(error))
client.on("completion:last_attempt", lambda: alert_exhaustion())

# Remove hooks
client.off("completion:kwargs", handler_function)
client.clear("completion:kwargs")  # Clear all handlers for event
client.clear()                      # Clear all hooks

# Per-call hooks
result = client.create(
    response_model=User,
    messages=[...],
    hooks={"completion:kwargs": lambda **kw: print(kw)},
)
```

### Response Model Definition

```python
from pydantic import BaseModel, Field, field_validator
from typing import Optional
from enum import Enum

class Priority(str, Enum):
    LOW = "low"
    MEDIUM = "medium"
    HIGH = "high"
    CRITICAL = "critical"

class Person(BaseModel):
    """Extract person information from text."""
    name: str = Field(description="Full legal name")
    age: int = Field(ge=0, le=150, description="Age in years")
    occupation: Optional[str] = Field(None, description="Current job title")

    @field_validator("name")
    @classmethod
    def name_must_not_be_empty(cls, v: str) -> str:
        if not v.strip():
            raise ValueError("Name must not be empty")
        return v.strip()
```

### Multimodal Input

```python
from instructor.multimodal import Image, Audio, PDF

# Image extraction
result = client.create(
    response_model=ImageDescription,
    messages=[{
        "role": "user",
        "content": ["Describe this image.", Image.from_url("https://example.com/photo.jpg")],
    }],
)

# PDF extraction
result = client.create(
    response_model=InvoiceData,
    messages=[{
        "role": "user",
        "content": ["Extract invoice data.", PDF.from_path("invoice.pdf")],
    }],
)
```

Image, Audio, and PDF classes support `from_url()`, `from_path()`, `from_base64()`, `from_gs_url()` (Google Cloud Storage), and `autodetect()` methods.

## Configuration

### Retry Configuration

```python
# Built-in retry configuration
person = client.create(
    response_model=Person,
    messages=[{"role": "user", "content": "Extract: John is 28 years old."}],
    max_retries=3,
)

# Tenacity integration for advanced retry strategies
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(stop=stop_after_attempt(3), wait=wait_exponential(multiplier=1, min=4, max=10))
def extract_with_backoff(text: str) -> Person:
    return client.create(
        response_model=Person,
        messages=[{"role": "user", "content": text}],
    )
```

Recommended retry settings by error type: rate limits (5 attempts, 1-120s delay), validation errors (2-3 attempts, 1-10s delay), network errors (4 attempts, 2-30s delay).

### Provider Selection

The `from_provider()` method accepts a string in the format `"provider/model"`:

```python
client = instructor.from_provider("openai/gpt-4o")
client = instructor.from_provider("anthropic/claude-sonnet-4-20250514")
client = instructor.from_provider("google/gemini-2.0-flash")
client = instructor.from_provider("ollama/llama3")
client = instructor.from_provider("deepseek/deepseek-chat")
```

### Mode Configuration

Override the default extraction mode when needed:

```python
import instructor

# Force JSON mode
client = instructor.from_provider("openai/gpt-4o", mode=instructor.Mode.JSON)

# Use strict JSON schema
client = instructor.from_provider("openai/gpt-4o", mode=instructor.Mode.JSON_SCHEMA)

# Markdown JSON for broader compatibility
client = instructor.from_provider("databricks/model", mode=instructor.Mode.MD_JSON)

# Parallel tool calls
client = instructor.from_provider("openai/gpt-4o", mode=instructor.Mode.PARALLEL_TOOLS)
```

### Jinja Templating

```python
response = client.create(
    messages=[{
        "role": "user",
        "content": "Extract the information from the following text: {{ data }}",
    }],
    response_model=User,
    context={"data": "John Doe is thirty years old"},
)
```

Context variables are also accessible within Pydantic field validators through `ValidationInfo`, enabling dynamic validation rules based on input context.

## Integration Patterns

### Basic Extraction

```python
import instructor
from pydantic import BaseModel

client = instructor.from_provider("openai/gpt-4o")

class Person(BaseModel):
    name: str
    age: int

person = client.create(
    response_model=Person,
    messages=[{"role": "user", "content": "Extract: Jason is 25 years old."}],
)
# person.name == "Jason", person.age == 25
```

### Streaming Partial Results

```python
for partial_person in client.create_partial(
    response_model=Person,
    messages=[{"role": "user", "content": "Extract: Jason is 25 years old."}],
):
    print(partial_person)
    # Yields progressively: Person(name=None, age=None) -> Person(name="Ja", age=None) -> ...
```

### Iterable Extraction

```python
users = client.create_iterable(
    response_model=Person,
    messages=[
        {"role": "user", "content": "Extract all people: Jason is 25. Sarah is 30."}
    ],
)
for user in users:
    print(user)
    # Person(name="Jason", age=25)
    # Person(name="Sarah", age=30)
```

### Async Usage

```python
import asyncio
import instructor

async_client = instructor.from_provider("openai/gpt-4o", async_client=True)

async def extract():
    person = await async_client.create(
        response_model=Person,
        messages=[{"role": "user", "content": "Extract: Jason is 25 years old."}],
    )
    return person

result = asyncio.run(extract())
```

### LLM-Based Validation

```python
from pydantic import BaseModel, BeforeValidator
from typing_extensions import Annotated
from instructor import llm_validator

client = instructor.from_provider("openai/gpt-4o-mini")

class QuestionAnswer(BaseModel):
    question: str
    answer: Annotated[
        str,
        BeforeValidator(llm_validator("don't say objectionable things", client=client)),
    ]
```

When the answer contains objectionable content, the LLM-based validator generates a human-readable error message that is fed back for reasking.

### Context-Based Validation

```python
from pydantic import BaseModel, field_validator, ValidationInfo

class CityExtraction(BaseModel):
    city: str

    @field_validator("city")
    @classmethod
    def validate_city(cls, v: str, info: ValidationInfo) -> str:
        allowed = info.context.get("allowed_cities", [])
        if allowed and v not in allowed:
            raise ValueError(f"City must be one of {allowed}")
        return v

result = client.create(
    response_model=CityExtraction,
    messages=[{"role": "user", "content": "Extract the city: I live in Paris."}],
    validation_context={"allowed_cities": ["Paris", "London", "Tokyo"]},
)
```

### With Completion Metadata

```python
person, completion = client.create_with_completion(
    response_model=Person,
    messages=[{"role": "user", "content": "Extract: Jason is 25 years old."}],
)
print(person.name)                    # "Jason"
print(completion.usage.total_tokens)  # Access token usage from raw completion
```

## Examples

### Classification with Enums

```python
from enum import Enum
from pydantic import BaseModel

class Sentiment(str, Enum):
    POSITIVE = "positive"
    NEGATIVE = "negative"
    NEUTRAL = "neutral"

class SentimentResult(BaseModel):
    sentiment: Sentiment
    confidence: float

result = client.create(
    response_model=SentimentResult,
    messages=[{"role": "user", "content": "Classify: I love this product!"}],
)
# result.sentiment == Sentiment.POSITIVE
```

### Nested Models

```python
from pydantic import BaseModel
from typing import List

class Address(BaseModel):
    street: str
    city: str
    country: str

class Company(BaseModel):
    name: str
    address: Address
    employee_count: int
    departments: List[str]

company = client.create(
    response_model=Company,
    messages=[{
        "role": "user",
        "content": "Extract: Acme Corp at 123 Main St, Springfield, USA with 500 employees in Engineering, Sales, and Marketing.",
    }],
)
```

### Hooks for Logging

```python
import instructor
from pydantic import BaseModel

client = instructor.from_provider("openai/gpt-4o-mini")

client.on("completion:kwargs", lambda **kw: print("Called with:", kw))
client.on("completion:error", lambda e: print(f"Error: {e}"))
client.on("completion:response", lambda r: print(f"Tokens: {r.usage.total_tokens}"))

class UserInfo(BaseModel):
    name: str
    age: int

user_info = client.create(
    response_model=UserInfo,
    messages=[{"role": "user", "content": "Extract: John is 20 years old"}],
)
```

### Failed Attempt Tracking

```python
from instructor.exceptions import InstructorRetryException

try:
    result = client.create(
        response_model=StrictModel,
        messages=[{"role": "user", "content": "Extract data..."}],
        max_retries=3,
    )
except InstructorRetryException as e:
    for attempt in e.failed_attempts:
        print(f"Attempt {attempt.attempt_number}: {attempt.exception}")
```

## Limitations

- **Provider-Dependent Behavior** -- Extraction quality and reliability vary across LLM providers and models. Smaller models may require more retries or produce lower-quality structured output.
- **Retry Cost** -- Each validation-triggered retry is a full LLM API call, adding latency and token cost. Complex validators on weaker models can lead to retry loops that exhaust the maximum retry count.
- **Schema Complexity Ceiling** -- Deeply nested or very large Pydantic models may exceed the context window or confuse the LLM, leading to incomplete or incorrect extraction.
- **No Guaranteed Correctness** -- Validation ensures structural correctness (types, formats, constraints) but cannot verify factual accuracy of the extracted content. The LLM may hallucinate values that pass validation.
- **Streaming Validator Limitation** -- Partial streaming (`create_partial`) does not support Pydantic validators during intermediate iterations due to the streaming nature of the response. Validators are only applied on the final complete object.
- **Literal Type Streaming** -- Models using `Literal` values in partial streaming must inherit from `PartialLiteralMixin` to avoid parsing errors with incomplete values.
- **Mode Variability** -- Different extraction modes may perform differently for the same use case. The documentation recommends testing with actual data and models to find the optimal mode.

## Changelog

Instructor follows semantic versioning. The current version is v1.14.5. The library has evolved from OpenAI-only function calling support to a multi-provider, multi-language platform. Key milestones include the introduction of `from_provider()` for unified provider access, streaming support via `create_partial()` and `create_iterable()`, expansion to 23+ LLM providers, multimodal extraction capabilities for images, PDFs, and audio, the hooks system for lifecycle instrumentation, Jinja templating for dynamic prompts, and the addition of TypeScript, Go, Ruby, Elixir, and Rust implementations. The project maintains an active release cadence with frequent updates.

## Citations

- [1] [Instructor Documentation - Home](https://python.useinstructor.com/)
- [2] [Instructor - Patching Concepts](https://python.useinstructor.com/concepts/patching/)
- [3] [Instructor - Retry Mechanism](https://python.useinstructor.com/concepts/retrying/)
- [4] [Instructor - Hooks System](https://python.useinstructor.com/concepts/hooks/)
- [5] [Instructor - Partial Streaming](https://python.useinstructor.com/concepts/partial/)
- [6] [Instructor - Iterable Streaming](https://python.useinstructor.com/concepts/lists/)
- [7] [Instructor - Integrations](https://python.useinstructor.com/integrations/)
- [8] [Instructor - Validation and Reasking](https://python.useinstructor.com/concepts/reask_validation/)
- [9] [Instructor - Jinja Templating](https://python.useinstructor.com/concepts/templating/)
- [10] [Instructor - Mode Comparison](https://python.useinstructor.com/modes-comparison/)
- [11] [Instructor - API Reference](https://python.useinstructor.com/api/)
- [12] [Instructor - Architecture](https://python.useinstructor.com/architecture/)
- [13] [Instructor - CLI Reference](https://python.useinstructor.com/cli/)
- [14] [Instructor - Installation](https://python.useinstructor.com/installation/)
- [15] [Instructor - Pydantic Models](https://python.useinstructor.com/concepts/models/)
- [16] [Instructor - Multimodal Capabilities](https://python.useinstructor.com/concepts/multimodal/)
- [17] [Instructor GitHub Repository](https://github.com/instructor-ai/instructor)
