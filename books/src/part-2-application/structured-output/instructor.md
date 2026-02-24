# Instructor

> Multi-language library for extracting structured, type-safe data from Large Language Models (LLMs) using Pydantic validation and automatic retries.

| Field        | Value                                                        |
|--------------|--------------------------------------------------------------|
| Name         | Instructor                                                   |
| Group        | Structured Output & Prompt Engineering                       |
| Type         | SDK                                                          |
| Open Source  | Yes                                                          |
| License      | MIT                                                          |
| GitHub       | [instructor-ai/instructor](https://github.com/instructor-ai/instructor) |
| Stars        | ~12.4k                                                       |
| Docs         | [python.useinstructor.com](https://python.useinstructor.com/) |
| Downloads    | 3M+ monthly                                                  |
| Contributors | 100+                                                         |

## Overview

Instructor is a library that patches LLM API clients to return structured, validated data instead of raw text. Built on top of Pydantic, it lets developers define response schemas as Python models and have the LLM fill them in directly. When the LLM output fails validation, Instructor automatically retries the request with the validation error context, enabling self-correcting extraction pipelines. The library supports 15+ LLM providers through a unified interface, offers streaming for partial results, and provides full type inference with IDE autocompletion. With over 3 million monthly downloads and 100+ contributors, Instructor has become one of the most widely adopted tools for structured output extraction.

## Core Concepts

### Structured Outputs via Pydantic Models

The fundamental idea behind Instructor is that a Pydantic model defines the expected shape of the LLM response. The library injects the model schema into the LLM request (via function calling, tool use, or JSON mode depending on the provider), parses the raw output, and validates it against the model. The developer receives a fully typed Python object rather than a string.

### Automatic Retries (Reasks)

When the LLM returns output that fails Pydantic validation, Instructor does not simply raise an error. Instead, it feeds the validation error message back to the LLM as context and retries the request. This "reask" loop continues up to a configurable maximum number of retries, giving the model the opportunity to self-correct. This is especially useful for enforcing constraints that are difficult to express purely in a prompt (for example, value ranges, string formats, or cross-field dependencies).

### Client Patching

Instructor works by wrapping (patching) existing LLM client libraries. Rather than replacing the client, it augments it with structured output capabilities. This means developers keep their existing authentication, configuration, and error handling while gaining schema-driven extraction on top.

## Installation

```bash
pip install instructor
```

Provider-specific extras may be required depending on the target LLM backend (for example, `pip install openai` for OpenAI, `pip install anthropic` for Anthropic).

## Architecture

Instructor sits as a thin middleware layer between the application and the LLM provider client:

1. **Application Layer** -- Defines Pydantic response models and sends messages through the Instructor-patched client.
2. **Instructor Layer** -- Injects the Pydantic model schema into the LLM request, parses the response, runs Pydantic validation, and handles retries on failure.
3. **Provider Client Layer** -- The underlying LLM SDK (OpenAI, Anthropic, Google, and others) handles authentication, transport, and raw API communication.

The patching mechanism wraps the provider client's completion method so that all existing client configuration (API keys, base URLs, timeouts) is preserved. Instructor intercepts only the response parsing step.

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

## Key Features

- **Type Safety and IDE Autocompletion** -- Response models are standard Pydantic classes, giving full type inference, autocomplete, and static analysis support in editors and type checkers.
- **Automatic Retries with Validation Context** -- Failed validations trigger retries where the error message is included in the next prompt, allowing the LLM to self-correct.
- **Multi-Provider Support** -- A single `from_provider()` interface supports OpenAI, Anthropic, Google, Ollama, DeepSeek, and 15+ other providers without changing application code.
- **Streaming** -- `create_partial()` streams partial results as the LLM generates tokens, enabling progressive UI updates. `create_iterable()` streams a sequence of complete objects.
- **Custom Pydantic Validators** -- Standard Pydantic field validators and model validators work seamlessly, enabling arbitrarily complex validation logic (regex patterns, cross-field checks, business rules).
- **Async/Await** -- Full async support for non-blocking LLM calls in asynchronous applications.
- **Jinja Templating** -- Prompt templates can use Jinja syntax for dynamic prompt construction with variables and control flow.
- **Hooks** -- Lifecycle hooks allow injecting custom logic at various stages of the request/response cycle (for example, logging, metrics, or transformation).

## Use Cases

- **Data Extraction** -- Pulling structured records (names, dates, amounts, entities) from unstructured text such as emails, documents, or web pages.
- **Classification** -- Categorizing text into predefined enums or labels with guaranteed valid output values.
- **Content Generation with Constraints** -- Generating content that must conform to a specific schema (for example, product descriptions with required fields, quiz questions with exactly four options).
- **Multi-Step Pipelines** -- Chaining structured outputs where the validated result of one step feeds into the next, with type safety preserved throughout.
- **Search and Retrieval Augmented Generation (RAG)** -- Extracting structured queries or filters from natural language to drive database lookups or search APIs.
- **Streaming User Interfaces** -- Progressively rendering structured data in a UI as the LLM generates it, using partial streaming.

## API Reference

### Client Creation

```python
import instructor

# Universal provider interface
client = instructor.from_provider("openai/gpt-5-nano")

# Provider-specific patching
import openai
client = instructor.from_openai(openai.OpenAI())

import anthropic
client = instructor.from_anthropic(anthropic.Anthropic())
```

### Core Methods

- **`client.create(response_model, messages, max_retries=...)`** -- Sends a completion request and returns a validated instance of `response_model`. Retries automatically on validation failure up to `max_retries`.
- **`client.create_with_completion(response_model, messages)`** -- Returns a tuple of `(response_model_instance, raw_completion)`, giving access to both the validated object and the raw provider response (token usage, finish reason, and similar metadata).
- **`client.create_partial(response_model, messages)`** -- Returns an iterator that yields progressively more complete instances of `response_model` as tokens stream in. Fields populate incrementally.
- **`client.create_iterable(response_model, messages)`** -- Returns an iterator of complete `response_model` instances, useful when the LLM is expected to produce a list of structured objects.

### Response Model Definition

```python
from pydantic import BaseModel, Field, field_validator

class Person(BaseModel):
    name: str = Field(description="Full legal name")
    age: int = Field(ge=0, le=150, description="Age in years")

    @field_validator("name")
    @classmethod
    def name_must_not_be_empty(cls, v: str) -> str:
        if not v.strip():
            raise ValueError("Name must not be empty")
        return v.strip()
```

## Configuration

### Retry Configuration

```python
person = client.create(
    response_model=Person,
    messages=[{"role": "user", "content": "Extract: John is 28 years old."}],
    max_retries=3,  # Maximum number of validation retry attempts
)
```

### Provider Selection

The `from_provider()` method accepts a string in the format `"provider/model"`:

```python
client = instructor.from_provider("openai/gpt-5-nano")
client = instructor.from_provider("anthropic/claude-sonnet-4-20250514")
client = instructor.from_provider("google/gemini-2.0-flash")
```

### Mode Configuration

Instructor supports multiple extraction modes depending on provider capabilities:

- **Function Calling / Tool Use** -- The default for providers that support it. The schema is passed as a function or tool definition.
- **JSON Mode** -- Forces the LLM to output valid JSON, which is then parsed against the Pydantic model.
- **Markdown JSON** -- Extracts JSON from markdown code blocks in the response.

## Integration Patterns

### Basic Extraction

```python
import instructor
from pydantic import BaseModel

client = instructor.from_provider("openai/gpt-5-nano")

class Person(BaseModel):
    name: str
    age: int

person = client.create(
    response_model=Person,
    messages=[{"role": "user", "content": "Extract: Jason is 25 years old."}],
)
# person.name == "Jason"
# person.age == 25
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

async_client = instructor.from_provider("openai/gpt-5-nano", async_=True)

async def extract():
    person = await async_client.create(
        response_model=Person,
        messages=[{"role": "user", "content": "Extract: Jason is 25 years old."}],
    )
    return person

result = asyncio.run(extract())
```

### Custom Validators for Business Logic

```python
from pydantic import BaseModel, field_validator

class UserProfile(BaseModel):
    username: str
    email: str
    age: int

    @field_validator("email")
    @classmethod
    def validate_email(cls, v: str) -> str:
        if "@" not in v:
            raise ValueError("Invalid email format")
        return v

    @field_validator("age")
    @classmethod
    def validate_age(cls, v: int) -> int:
        if v < 18:
            raise ValueError("User must be at least 18 years old")
        return v
```

When the LLM produces an email without `@` or an age below 18, Instructor feeds the validation error back to the model and retries, guiding it toward a valid response.

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

class Address(BaseModel):
    street: str
    city: str
    country: str

class Company(BaseModel):
    name: str
    address: Address
    employee_count: int

company = client.create(
    response_model=Company,
    messages=[
        {
            "role": "user",
            "content": "Extract company info: Acme Corp is based at 123 Main St, Springfield, USA with 500 employees.",
        }
    ],
)
```

### With Completion Metadata

```python
person, completion = client.create_with_completion(
    response_model=Person,
    messages=[{"role": "user", "content": "Extract: Jason is 25 years old."}],
)
print(person.name)  # "Jason"
print(completion.usage.total_tokens)  # Access token usage from raw completion
```

## Limitations

- **Provider-Dependent Behavior** -- Extraction quality and reliability vary across LLM providers and models. Smaller models may require more retries or produce lower-quality structured output.
- **Retry Cost** -- Each validation-triggered retry is a full LLM API call, adding latency and token cost. Complex validators on weaker models can lead to retry loops that exhaust the maximum retry count.
- **Schema Complexity Ceiling** -- Deeply nested or very large Pydantic models may exceed the context window or confuse the LLM, leading to incomplete or incorrect extraction.
- **No Guaranteed Correctness** -- Validation ensures structural correctness (types, formats, constraints) but cannot verify factual accuracy of the extracted content. The LLM may hallucinate values that pass validation.
- **Streaming Limitations** -- Partial streaming yields incomplete objects during generation. Application code must handle `None` fields and incomplete state gracefully.

## Changelog

Instructor follows semantic versioning. The library has evolved from OpenAI-only function calling support to a multi-provider, multi-language platform. Key milestones include the introduction of `from_provider()` for unified provider access, streaming support via `create_partial()` and `create_iterable()`, and expansion to 15+ LLM providers. The project maintains an active release cadence with frequent updates.

## Citations

- [1] [Instructor Documentation](https://python.useinstructor.com/)
