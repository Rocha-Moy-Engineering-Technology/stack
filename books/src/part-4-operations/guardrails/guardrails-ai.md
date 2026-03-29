# Guardrails AI

> Python framework for input/output validation guards on LLM applications

| Field | Value |
|-------|-------|
| Group | Guardrails & Safety |
| Type | SDK |
| Open Source | Yes |
| GitHub | [https://github.com/guardrails-ai/guardrails](https://github.com/guardrails-ai/guardrails) |
| Stars | 6435 |
| Documentation | [Official Docs](https://guardrailsai.com/docs) |

## Overview

Guardrails AI is a Python framework for building reliable AI applications through input/output validation. It provides two primary capabilities: deploying Input/Output Guards that detect, quantify, and mitigate the presence of specific types of risks in Large Language Model (LLM) interactions, and generating structured data from LLM outputs that conform to predefined schemas.

The framework takes a modular approach to LLM safety. Rather than providing a monolithic filtering system, Guardrails decomposes risk management into composable units called Validators. Each validator targets a specific risk category such as toxic language, personally identifiable information (PII) exposure, hallucinated content, or competitor mentions. Multiple validators combine into Guards, which wrap LLM calls and intercept both inputs and outputs to apply validation rules before data reaches the end user.

Guardrails Hub serves as a centralized marketplace of pre-built validators contributed by the community and the Guardrails team. As of early 2025, the Guardrails Index benchmark compares the performance and latency of 24 guardrails across six common risk categories, providing an empirical basis for validator selection.

The project is licensed under Apache 2.0 and supports Python 3.10 through 3.14. It is available as both an in-application library and a standalone server for centralized validation across multiple services.

## Core Concepts

**Guards** are the primary interface for Guardrails. A Guard wraps an LLM call and applies one or more validators to the inputs, outputs, or both. Guards can operate in two modes: as a pass-through validator that checks LLM output after generation, or as an active participant that re-asks the LLM when validation fails. Each Guard maintains a call history for debugging and observability.

**Validators** are the atomic units of risk measurement. Each validator encodes a specific quality criterion and produces a binary outcome: a PassResult when the content meets the criterion (returning the value unchanged) or a FailResult when the content violates the criterion (triggering a configured on-fail action). Validators can be stateless pattern matchers, machine learning classifiers, or LLM-based evaluators depending on the risk category they target.

**On-Fail Actions** determine what happens when a validator produces a FailResult. Eight actions are available: REASK instructs the LLM to regenerate output with feedback about the failure; FIX programmatically corrects the output when a deterministic fix exists; FILTER removes the failing field from structured output while preserving valid fields; REFRAIN returns None when the output is unsafe for end users; NOOP logs the failure without corrective action; EXCEPTION raises an error immediately; FIX_REASK attempts a deterministic fix first and falls back to re-asking if validation still fails; and CUSTOM executes a user-defined handler function.

**Guardrails Hub** is the centralized repository of pre-built validators. Hub validators are installed via the CLI (`guardrails hub install`) or the in-code `install()` function. Validators in the Hub cover six primary risk categories: content safety, PII detection, toxic language, jailbreak detection, topic restriction, and output format validation.

**Structured Data Generation** uses Pydantic models to define the expected schema of LLM output. Guards enforce that LLM responses conform to the schema through either function calling (when the model supports it) or prompt optimization. Field-level validators can be attached to individual fields in the Pydantic model for granular control.

**Validation Metadata** provides runtime context to validators that need information unavailable at initialization time. This metadata is passed to `guard.validate()` or `guard()` calls, allowing validators like `ExtractedSummarySentencesMatch` to access dynamic information such as file paths or reference documents for comparison operations.

## Architecture

Guardrails operates as a middleware layer between application code and LLM providers. The architecture has three primary components:

**Guard Layer.** The Guard object orchestrates the validation pipeline. When invoked, it sends the request to the LLM, receives the response, and passes it through the configured validators. If any validator fails and the on-fail action requires re-asking, the Guard constructs a corrective prompt and sends it back to the LLM. This loop continues up to a configurable `num_reasks` limit.

```text
Application Code
       |
       v
   Guard Object
       |
       +---> LLM Provider (OpenAI, Anthropic, Cohere, HuggingFace)
       |          |
       |          v
       +<--- Raw LLM Response
       |
       v
  Validator Pipeline
       |
       +---> PassResult --> Return validated output
       |
       +---> FailResult --> Apply on-fail action
                |
                +---> REASK: Re-prompt LLM with error feedback
                +---> FIX: Apply deterministic correction
                +---> FILTER: Remove failing field
                +---> REFRAIN: Return None
                +---> EXCEPTION: Raise error
```

**Validator Pipeline.** Validators execute sequentially on the LLM output. Each validator receives the current value and produces either a PassResult or FailResult. Multiple validators compose into a pipeline where the output of one validator feeds into the next. Validators can operate on the full response text or on individual fields within structured output.

**Guardrails Server.** For production deployments, Guardrails can run as a standalone Flask-based REST API server. The server exposes OpenAI-compatible endpoints, allowing any OpenAI SDK client to route through the Guardrails server for transparent validation. Guards are defined in a Python configuration file and loaded at server startup. The server architecture separates validation from application logic, enabling independent scaling of the validation layer.

```text
Client Application
       |
       v
Guardrails Server (Flask/Gunicorn/Uvicorn)
       |
       +---> /guards/{guardName}/openai/v1/chat/completions
       |
       v
  Guard Pipeline --> LLM Provider --> Validators --> Response
```

## Key Features and Functionality

**Input and Output Validation.** Guards can validate both the input sent to an LLM and the output received. Input guards catch prompt injection attempts, off-topic queries, and policy violations before they reach the model. Output guards catch toxic content, PII leaks, hallucinations, and format violations before the response reaches the user.

**Structured Output Generation.** Guardrails converts free-form LLM text into structured data conforming to Pydantic models. Field-level validators provide granular control over individual attributes. When a field fails validation, the on-fail action applies to that specific field rather than the entire response.

**Re-Ask Loop.** When the REASK on-fail action is configured, Guardrails automatically constructs a corrective prompt that includes the original output and a description of the validation failure. The LLM is asked to regenerate its response to meet the specified criteria. The number of re-ask attempts is configurable via `num_reasks`.

**Streaming Support.** Guards can validate streaming LLM responses by processing chunks as they arrive. This allows real-time validation without waiting for the complete response, enabling responsive user experiences while still enforcing safety constraints.

**Async Support.** The `AsyncGuard` class provides async/await support for concurrent validation workflows. Async guards allow making concurrent calls to multiple LLMs and processing response chunks as they arrive, providing better performance in I/O-bound applications.

**Call History.** Guards maintain a history of all LLM calls and validation results. Starting with version 0.8.0, history is capped at 10 entries by default, configurable via the `history_max_length` parameter. This history is available for debugging, observability, and audit purposes.

**OpenAI-Compatible Server.** The Guardrails Server exposes endpoints compatible with the OpenAI Chat Completions API. Applications using the OpenAI SDK can route through the Guardrails Server by changing their `base_url`, gaining validation without code changes to the application layer.

**Hub Ecosystem.** The Guardrails Hub provides a growing collection of pre-built validators that can be installed and composed without writing custom validation logic. Hub validators cover categories including content safety, PII detection, toxic language, jailbreak prevention, format validation (JSON, SQL, regex), and business rule enforcement.

## Use Cases

**Content Safety Enforcement.** Filter toxic language, hate speech, and inappropriate content from LLM outputs using validators like `ToxicLanguage` (powered by the Detoxify multi-label classifier) and custom content policies. On-fail actions control whether unsafe content is removed, replaced, or causes an exception.

**PII Protection.** Detect and redact personally identifiable information such as names, email addresses, phone numbers, and social security numbers from LLM outputs using the `DetectPII` validator (powered by Microsoft Presidio). This is critical for applications handling user data subject to privacy regulations.

**Competitor Mention Filtering.** Prevent LLM responses from mentioning competitors using the `CompetitorCheck` validator. Configure a list of competitor names and the guard either removes mentions (FIX) or rejects the response entirely (EXCEPTION/REFRAIN).

**Structured Data Extraction.** Extract structured information from unstructured LLM text. Define a Pydantic model with the desired schema, attach field-level validators, and the guard ensures the LLM output conforms to the schema with all validation rules satisfied.

**Prompt Injection Defense.** Input guards detect and block prompt injection attempts before they reach the LLM. Validators identify common injection patterns, jailbreak attempts, and off-topic inputs that could cause the model to deviate from its intended behavior.

**API Response Validation.** Validate that LLM-generated API responses, SQL queries, or code snippets meet format and safety requirements before execution. Format validators ensure syntactic correctness while content validators prevent injection attacks in generated code.

**Bias and Fairness Monitoring.** Detect bias in LLM outputs across demographic categories using specialized validators. NOOP on-fail actions allow logging bias occurrences without blocking responses, enabling monitoring and analysis of bias patterns over time.

## API Reference Summary

### Guard Class

The primary interface for validation. Core methods:

**`Guard()`** -- Create a new Guard instance. Accepts optional `name` parameter for identification when using Guardrails Server.

**`guard.use(validator, **kwargs)`** -- Add a single validator to the guard with configuration parameters and an `on_fail` action.

**`guard.use_many(*validators)`** -- Add multiple validators to the guard simultaneously.

**`guard(model, messages, **kwargs)`** -- Invoke the guard with an LLM call. Mirrors standard LLM SDK call signatures. Returns a `GuardResponse` containing the raw output, validated output, and validation status.

**`guard.parse(llm_output, num_reasks=0)`** -- Validate pre-generated LLM output. With `num_reasks=0`, operates as a pure post-processor. With higher values, enables re-asking the LLM on failure.

**`guard.validate(value, metadata=None)`** -- Validate a value against the guard's validators without making an LLM call. Useful for validating cached or pre-fetched responses.

**`Guard.for_pydantic(output_class, prompt=None)`** -- Create a guard configured for structured data generation using a Pydantic model.

### GuardResponse

Returned by guard invocations:

- `raw_llm_output` -- The unmodified LLM response text
- `validated_output` -- The output after validation and any corrections
- `validation_passed` -- Boolean indicating if all validators passed
- `reask` -- Information about any re-ask attempts

### Validation Results

**`PassResult`** -- Returned by validators when content meets criteria. Contains the validated value, which in most cases is the original value unchanged.

**`FailResult`** -- Returned by validators when content violates criteria. Contains an `error_message` describing the failure and an optional `fix_value` for deterministic corrections.

### OnFailAction Enum

```python
from guardrails import OnFailAction

OnFailAction.REASK       # Re-ask LLM with failure feedback
OnFailAction.FIX         # Apply deterministic fix_value
OnFailAction.FILTER      # Remove failing field from structured output
OnFailAction.REFRAIN     # Return None for unsafe content
OnFailAction.NOOP        # Log failure, return original value
OnFailAction.EXCEPTION   # Raise ValidationError
OnFailAction.FIX_REASK   # Try FIX first, then REASK if still failing
OnFailAction.CUSTOM      # Execute custom handler function
```

## Configuration and Customization

### Guard Configuration

Guards accept configuration at multiple levels:

```python
from guardrails import Guard, OnFailAction
from guardrails.hub import ToxicLanguage, DetectPII, CompetitorCheck

# Basic guard with a single validator
guard = Guard(name="content-safety").use(
    ToxicLanguage,
    threshold=0.5,
    validation_method="sentence",
    on_fail=OnFailAction.REFRAIN,
)

# Guard with multiple validators
guard = Guard(name="production-guard").use_many(
    ToxicLanguage(threshold=0.5, on_fail=OnFailAction.REFRAIN),
    DetectPII(on_fail=OnFailAction.FIX),
    CompetitorCheck(
        competitors=["Apple", "Microsoft", "Google"],
        on_fail=OnFailAction.FIX,
    ),
)
```

### Structured Output with Pydantic

```python
from pydantic import BaseModel, Field
from guardrails import Guard

class UserProfile(BaseModel):
    name: str = Field(description="The user's full name")
    email: str = Field(description="The user's email address")
    age: int = Field(description="The user's age in years", ge=0, le=150)

guard = Guard.for_pydantic(output_class=UserProfile)
```

### Custom Validators

Create validators using the class-based approach:

```python
from guardrails.validators import Validator, register_validator, PassResult, FailResult

@register_validator(name="custom/word_count", data_type="string")
class WordCount(Validator):
    def __init__(self, min_words: int, max_words: int, on_fail=None, **kwargs):
        super().__init__(on_fail=on_fail, min_words=min_words, max_words=max_words, **kwargs)
        self.min_words = min_words
        self.max_words = max_words

    def _validate(self, value, metadata=None) -> PassResult | FailResult:
        word_count = len(value.split())
        if self.min_words <= word_count <= self.max_words:
            return PassResult()
        return FailResult(
            error_message=f"Expected {self.min_words}-{self.max_words} words, got {word_count}.",
            fix_value=" ".join(value.split()[:self.max_words]),
        )
```

### Server Configuration

Define guards in a Python config file for the Guardrails Server:

```python
# config.py
from guardrails import Guard, OnFailAction
from guardrails.hub import ToxicLanguage, DetectPII

content_guard = Guard(name="content-guard").use_many(
    ToxicLanguage(threshold=0.5, on_fail=OnFailAction.REFRAIN),
    DetectPII(on_fail=OnFailAction.FIX),
)
```

Start the server:

```bash
guardrails start --config=./config.py
```

### Environment Variables

- `GUARDRAILS_BASE_URL` -- Base URL for the Guardrails Server (default: `http://localhost:8000`)
- `GUARDRAILS_API_KEY` -- API key for authenticating with the Guardrails Server
- `OPENAI_API_KEY` -- API key for the OpenAI provider when using OpenAI models
- `ANTHROPIC_API_KEY` -- API key for the Anthropic provider when using Claude models

## Integration Patterns

### Direct In-Application Usage

The most common pattern wraps LLM calls directly in application code:

```python
from guardrails import Guard, OnFailAction
from guardrails.hub import ToxicLanguage

guard = Guard().use(
    ToxicLanguage,
    threshold=0.5,
    on_fail=OnFailAction.EXCEPTION,
)

result = guard(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Summarize this article."}],
)

print(result.validated_output)
```

### OpenAI SDK Proxy

Route existing OpenAI SDK calls through the Guardrails Server without changing application code:

```python
from openai import OpenAI

# Point the OpenAI client at the Guardrails Server
client = OpenAI(
    base_url="http://localhost:8000/guards/content-guard/openai/v1/",
    api_key="your-api-key",
)

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Hello, world!"}],
)
```

### LiteLLM Integration

Guardrails integrates with LiteLLM for multi-provider routing with validation:

```python
import litellm
from guardrails import Guard, OnFailAction
from guardrails.hub import ToxicLanguage

guard = Guard().use(
    ToxicLanguage,
    threshold=0.5,
    on_fail=OnFailAction.REFRAIN,
)

# Use LiteLLM model strings with Guardrails
result = guard(
    model="anthropic/claude-3-5-sonnet-latest",
    messages=[{"role": "user", "content": "Explain quantum computing."}],
)
```

### Production Deployment with Gunicorn

Deploy the Guardrails Server behind a production Web Server Gateway Interface (WSGI) server:

```bash
gunicorn \
    --bind 0.0.0.0:8000 \
    --timeout=90 \
    --workers=4 \
    'guardrails_api.app:create_app(None, "config.py")'
```

Worker count recommendation: `(2 x num_cores) + 1` as a baseline. Adjust based on whether validators are CPU-bound (static validators) or I/O-bound (LLM-based validators).

### Docker Deployment

```dockerfile
FROM python:3.12-slim

RUN pip install guardrails-ai
RUN guardrails configure

COPY config.py /app/config.py
WORKDIR /app

CMD ["guardrails", "start", "--config=./config.py"]
```

## Examples

### Basic Output Validation

```python
from guardrails import Guard, OnFailAction
from guardrails.hub import RegexMatch

# Validate phone number format
guard = Guard().use(
    RegexMatch,
    regex=r"\(?\d{3}\)?-? *\d{3}-? *-?\d{4}",
    on_fail=OnFailAction.EXCEPTION,
)

# This passes validation
guard.validate("123-456-7890")

# This raises an exception
try:
    guard.validate("not a phone number")
except Exception as e:
    print(f"Validation failed: {e}")
```

### Multi-Validator Guard

```python
from guardrails import Guard, OnFailAction
from guardrails.hub import CompetitorCheck, ToxicLanguage

guard = Guard().use_many(
    CompetitorCheck(
        competitors=["Apple", "Microsoft", "Google"],
        on_fail=OnFailAction.FIX,
    ),
    ToxicLanguage(
        threshold=0.5,
        validation_method="sentence",
        on_fail=OnFailAction.REFRAIN,
    ),
)

result = guard(
    model="gpt-4o",
    messages=[{
        "role": "user",
        "content": "Compare our product to competitors in the market.",
    }],
)

if result.validation_passed:
    print(result.validated_output)
else:
    print("Response was filtered for safety.")
```

### Structured Data Extraction with Pydantic

```python
from pydantic import BaseModel, Field
from guardrails import Guard

class Pet(BaseModel):
    name: str = Field(description="The pet's name")
    species: str = Field(description="The pet's species")
    age: int = Field(description="The pet's age in years")
    favorite_toy: str = Field(description="The pet's favorite toy")

guard = Guard.for_pydantic(
    output_class=Pet,
    prompt="Tell me about a golden retriever named Buddy.",
)

result = guard(
    model="gpt-4o",
    messages=[{
        "role": "user",
        "content": "Tell me about a golden retriever named Buddy.",
    }],
)

pet = result.validated_output
print(f"{pet['name']} is a {pet['species']}, age {pet['age']}")
```

### On-Fail Action Comparison

The following demonstrates how different on-fail actions handle the same toxic input:

```python
from guardrails import Guard, OnFailAction
from guardrails.hub import ToxicLanguage

toxic_text = "damn you!"

# FIX: Removes toxic portion, returns "you!"
guard_fix = Guard().use(ToxicLanguage, threshold=0.5, on_fail=OnFailAction.FIX)
result = guard_fix.validate(toxic_text)
print(f"FIX: {result.validated_output}")

# REFRAIN: Returns None
guard_refrain = Guard().use(ToxicLanguage, threshold=0.5, on_fail=OnFailAction.REFRAIN)
result = guard_refrain.validate(toxic_text)
print(f"REFRAIN: {result.validated_output}")

# NOOP: Returns original text unchanged, logs failure
guard_noop = Guard().use(ToxicLanguage, threshold=0.5, on_fail=OnFailAction.NOOP)
result = guard_noop.validate(toxic_text)
print(f"NOOP: {result.validated_output}")

# EXCEPTION: Raises an error
guard_exc = Guard().use(ToxicLanguage, threshold=0.5, on_fail=OnFailAction.EXCEPTION)
try:
    guard_exc.validate(toxic_text)
except Exception as e:
    print(f"EXCEPTION: {e}")
```

### Custom On-Fail Handler

```python
from guardrails import Guard, OnFailAction
from guardrails.hub import ToxicLanguage

def custom_handler(value, fail_result):
    """Log the failure and return a sanitized placeholder."""
    print(f"Validation failed: {fail_result.error_message}")
    return "[Content moderated]"

guard = Guard().use(
    ToxicLanguage,
    threshold=0.5,
    on_fail=OnFailAction.CUSTOM,
    on_fail_handler=custom_handler,
)
```

## Limitations and Considerations

**Validator Latency.** Machine learning-based validators (toxic language, PII detection) add measurable latency to each LLM call. Static validators (regex, format checks) are fast, but ML-based validators may take tens to hundreds of milliseconds per invocation. The Guardrails Index benchmark provides empirical latency data for planning purposes.

**Re-Ask Cost.** The REASK on-fail action triggers additional LLM calls, which increases both latency and token cost. Each re-ask is a full LLM invocation with the original prompt plus corrective feedback. Setting `num_reasks` too high can lead to significant cost amplification.

**ML Validator Memory Footprint.** Validators powered by machine learning models (such as ToxicLanguage using Detoxify) require loading model weights into memory. In server deployments, this memory cost multiplies with the number of worker processes. Multithreading via `--threads` reduces memory overhead compared to multiprocessing but may introduce race conditions in history manipulation.

**Python Only for Core Framework.** While a JavaScript client exists, the core validation framework and custom validator development require Python. Applications in other languages must interact through the Guardrails Server REST API.

**Hub Dependency.** Some Hub validators require network access to download model artifacts or contact external APIs. Deployments in air-gapped environments may need to pre-download all required validator models and dependencies.

**False Positives and Negatives.** ML-based validators are probabilistic. Threshold tuning is required per use case to balance false positive rates (blocking legitimate content) against false negative rates (allowing violating content through). The `threshold` parameter on validators like `ToxicLanguage` controls this trade-off.

**Structured Output Reliability.** Structured data generation depends on the LLM's ability to produce output conforming to the Pydantic schema. Smaller or less capable models may require more re-ask attempts. Function calling support in the underlying model significantly improves structured output reliability.

**Server Concurrency.** The Flask-based Guardrails Server does not inherently support high-concurrency workloads. Production deployments should use Gunicorn or Uvicorn with appropriate worker configuration. Guard history manipulation is not thread-safe, requiring careful tuning of worker threads versus processes.

## Changelog Highlights

- **v0.9.0** (February 2026) -- Major release requiring migration from v0.8.x. Migration guide available at guardrailsai.com.
- **v0.8.0** (February 2026) -- Introduced `history_max_length` parameter capping Guard history to 10 entries by default. Limits memory consumption in long-running applications.
- **v0.8.1** (February 2026) -- Fixed custom validators failing without Hub API key. Resolved temperature handling issues. Removed previously deprecated methods (breaking change).
- **v0.7.0** (November 2025) -- Added LangChain 1.x support for framework interoperability.
- **v0.7.3** (February 2026) -- Added OpenAI 2.x SDK support.
- **v0.6.8** (November 2025) -- Added Python 3.13 support. Upgraded Click and Typer dependency versions.
- **v0.6.7** (September 2025) -- Replaced deprecated `pkg_resources` with modern packaging alternatives.
- **v0.5.0** (July 2024) -- Major milestone release establishing the Guard/Validator/Hub architecture.
- **v0.1.0** (March 2023) -- Initial public release.

The project has been under active development since January 2023, with approximately 72 contributors and consistent release cadence averaging multiple minor releases per month.

## Citations

- [Guardrails AI GitHub Repository](https://github.com/guardrails-ai/guardrails) -- Source code, issues, and releases under Apache 2.0 license.
- [Guardrails AI Official Documentation](https://guardrailsai.com/docs) -- Concepts, API reference, how-to guides, and deployment documentation.
- [Guardrails Hub](https://guardrailsai.com/hub) -- Centralized repository of pre-built validators with documentation and installation instructions.
- [Guardrails Index](https://guardrailsai.com/) -- Benchmark comparing performance and latency of 24 guardrails across six common risk categories (launched February 2025).
- [guardrails-ai on PyPI](https://pypi.org/project/guardrails-ai/) -- Package distribution with version history and dependency information.
- [Guardrails AI Validator Template](https://github.com/guardrails-ai/validator-template) -- Template repository for creating and submitting custom validators to the Hub.
