# BAML

> Domain-specific language for generating structured outputs from Large Language Models (LLMs), providing type-safe definitions, generated client libraries, and production-ready extraction pipelines across multiple programming languages.

| Field        | Value                                                  |
|--------------|--------------------------------------------------------|
| Name         | BAML (Basically a Made-up Language)                    |
| Group        | Structured Generation                                  |
| Type         | SDK                                                    |
| Open Source  | Yes (Apache 2.0)                                       |
| GitHub       | [BoundaryML/baml](https://github.com/BoundaryML/baml) |
| Stars        | 7,642                                                  |
| Docs         | [docs.boundaryml.com](https://docs.boundaryml.com/)   |

## Overview

BAML is a domain-specific language (DSL) created by BoundaryML for defining, generating, and validating structured outputs from LLMs. The framework transforms prompt engineering into schema engineering by providing a declarative approach to specifying data types, LLM functions, prompt templates, and client configurations in dedicated source files (`baml_src/`), then generating fully typed client libraries (`baml_client/`) for use in application code. This architecture separates LLM interaction concerns from business logic, enabling type-safe extraction, classification, and generation workflows that move cleanly from prototyping to production deployment. BAML supports Python, TypeScript, Go, Ruby, Rust, Java, C#, Elixir, and a REST API interface. The framework is built entirely in Rust and operates fully offline with no telemetry [1][2].

BAML's design philosophy positions it as an evolution beyond string-based prompt management, analogous to how JSX modernized web development beyond HTML-in-strings. The framework emphasizes the expressiveness of English combined with the structure of code, allowing developers to view and run prompts directly within their editor without requiring a runtime environment or language-specific setup [1].

A key technical differentiator is BAML's Schema-Aligned Parsing (SAP) algorithm, which applies Postel's Law to LLM outputs: accepting imperfect responses and transforming them to match declared schemas using custom edit distance algorithms. Benchmarks show SAP achieving 92-93% accuracy across models, outperforming provider-native function calling approaches at 87.5%. Additionally, BAML's schema format uses approximately 80% fewer tokens than JSON Schema, reducing cost without sacrificing clarity [3].

## Core Concepts

BAML introduces several foundational abstractions that distinguish it from general-purpose LLM client libraries.

**Type Definitions.** The DSL provides `class` and `enum` keywords for defining the shape of structured data that LLMs should produce. These type definitions serve as the contract between the LLM prompt and the application code, enabling compile-time and runtime validation of outputs. Classes support field-level annotations such as `@description` for guiding LLM output, `@alias` for renaming fields in prompts, `@skip` for excluding fields, and `@@dynamic` for enabling runtime schema modification [4][5].

**Functions.** BAML functions declare the input and output types for an LLM call, binding a prompt template to a specific extraction or generation task. Functions are the primary unit of LLM interaction and are compiled into typed methods in the generated client. Each function specifies a client (LLM provider) and a prompt template [1].

**Template Strings.** Prompt engineering is handled through Jinja-based template strings that compose prompt fragments. Template strings support variable interpolation (`{{ variable }}`), conditional logic, loops, filters, and reuse across multiple functions. The special `{{ ctx.output_format }}` macro injects the output schema instructions into prompts, guiding the LLM to produce correctly structured responses [6].

**LLM Clients.** Provider-specific configuration (model name, API keys, parameters) is declared as client definitions within the DSL. BAML supports over 25 LLM providers including OpenAI, Anthropic, Google AI, Vertex AI, AWS Bedrock, Azure OpenAI, OpenRouter, Groq, Cerebras, HuggingFace, LiteLLM, Ollama, vLLM, and all OpenAI API-compatible endpoints. Shorthand syntax (`client "openai/gpt-4o"`) enables quick provider selection [7].

**Code Generation.** The BAML compiler reads `baml_src/` definitions and generates a `baml_client/` directory containing fully typed client code in the target language. Python types map to Pydantic models, TypeScript generates native TypeScript types, and other languages receive idiomatic equivalents. The generated code handles serialization, deserialization, prompt rendering, robust JSON parsing (including repair of broken JSON), and provider communication [8].

**Testing.** The DSL includes native test declarations that define input/output expectations for functions, with `@@assert` for strict validation and `@@check` for non-exception-raising validation. Tests can be run from the editor playground or via the CLI with parallel execution support [9].

**Checks and Asserts.** BAML provides two validation mechanisms for LLM output quality. `@assert` enforces mandatory rules that halt execution on failure, raising `BamlValidationError` when validation fails. `@check` validates data without interrupting execution, returning results regardless of pass/fail status. Both use Jinja expressions with `this` referencing the current field value [10].

## Installation

BAML provides language-specific installation paths alongside a CLI and editor extensions.

**CLI Installation.** The BAML CLI handles project initialization, code generation, testing, and development server operations:

```bash
# Install via npm (also available through other package managers)
npm install -g @boundaryml/baml

# Initialize a new BAML project
baml init

# Generate client code from baml_src definitions
baml generate

# Run BAML tests
baml-cli test

# Start development server with file watching
baml dev

# Start a REST API server exposing BAML functions
baml serve

# Format BAML source files
baml fmt
```

**Python.**

```bash
pip install baml-py
```

**TypeScript/JavaScript.**

```bash
npm install @boundaryml/baml
```

**Go.**

```bash
go get github.com/boundaryml/baml-go
```

**Ruby.**

```bash
gem install baml
```

**Rust.** Native Rust SDK available since version 0.217.0 via Cargo.

**Java and C#** packages are available through their respective package managers.

**REST API.** For languages without a native SDK, `baml serve` exposes all declared functions as HTTP endpoints with OpenAPI documentation [1][11].

**Editor Extensions.** BAML provides extensions for VSCode, Cursor, JetBrains IDEs, Zed, and Claude Code, offering syntax highlighting, autocompletion, inline diagnostics, live preview of generated prompts, and raw cURL request inspection [1].

## Architecture

BAML follows a two-directory architecture that cleanly separates definitions from generated code.

**`baml_src/` (Source Definitions).** This directory contains all BAML DSL files (`.baml` extension) where types, functions, clients, template strings, and tests are defined. All declarations within the directory are accessible across all files, enabling flexible organization with subdirectories. These files are the single source of truth for LLM interaction contracts. The `baml_src` directory is not required at deployment time [12].

**`baml_client/` (Generated Client Library).** Running `baml generate` compiles the source definitions into a fully typed client library in the target programming language. The generated code includes typed function signatures matching the BAML function definitions, serialization and deserialization logic for all declared types, prompt rendering from template strings with variable binding, provider-specific API communication through declared LLM clients, robust JSON parsing with automatic repair of malformed outputs, and streaming support with partial type generation. The generated client is not intended for manual editing; it is regenerated whenever source definitions change [8].

**Generator Configuration.** A generator block in BAML files configures code generation, specifying the target language via `output_type`, the output directory, client mode (sync or async), and the runtime version matching the installed BAML package [8].

**Compilation Pipeline.** The BAML compiler, built in Rust, parses `.baml` files, validates type consistency and function signatures, resolves template string references, and emits language-specific client code. This compile step catches type mismatches, missing fields, and invalid references before runtime [1].

**High-Level and Modular APIs.** The generated client provides both a high-level API where everything from prompt rendering to response parsing is handled automatically, and a low-level modular API exposing `b.request` (HTTP request generation), `b.parse` (response parsing), and streaming equivalents for custom integration patterns such as OpenAI Batch API workflows [13].

## Key Features

**Streaming with Partial Types.** BAML supports streaming responses from LLMs with partial structured output parsing. As tokens arrive, the generated client provides incrementally populated typed objects where class fields become nullable by default. Semantic streaming attributes provide fine-grained control: `@stream.done` ensures fields stream only when complete, `@stream.not_null` ensures containing objects stream only when the annotated field has a value, and `@stream.with_state` wraps fields in `StreamState` metadata tracking completion status (`incomplete` or `complete`). Number fields are only streamed when the LLM completes them, never as intermediate values. All languages support `get_final_response()` to retrieve the fully validated type after streaming [14].

**Multi-Modal Input.** Functions can accept images, audio files, PDFs, and video as inputs alongside text. Each media type supports creation from URLs or base64-encoded data. The `media_url_handler` configuration controls URL resolution with options including `send_url`, `send_base64`, and `send_base64_unless_google_url` for provider-optimized handling. PDF inputs currently require base64 encoding and are supported by providers including Gemini and Vertex AI [15].

**Concurrent Execution.** Multiple LLM calls can be executed concurrently through language-native patterns: `asyncio.gather()` in Python, `Promise.all()` in TypeScript, goroutines with `sync.WaitGroup` in Go, and thread spawning in Rust. BAML supports advanced parallel patterns including fastest-wins racing (launching multiple provider requests and cancelling slower operations), timeout management, and batch processing with cancellation across remaining batches [16].

**Error Handling.** BAML provides a structured exception hierarchy rooted in `BamlError`. `BamlInvalidArgumentError` covers argument validation failures. `BamlClientError` and `BamlClientHttpError` handle provider communication failures with status code tracking. `BamlClientFinishReasonError` captures LLM finish reason violations. `BamlValidationError` fires when responses cannot be parsed into declared schemas, providing `raw_output`, `prompt`, and `detailed_message` for debugging. `BamlAbortError` signals cancelled operations. For persistent parsing issues, an LLM Fixup pattern is recommended where a dedicated function asks the LLM to repair malformed data [17].

**Dynamic Types (TypeBuilder).** The `TypeBuilder` runtime API enables programmatic type construction for scenarios where output schemas change at runtime. Types marked with `@@dynamic` can have enum values or class properties added dynamically. TypeBuilder supports primitives, literals, collections, unions, and entirely new types not defined in BAML source. The `add_baml()` method allows writing native BAML code for type modifications, and JSON Schema conversion is supported [18].

**Collector (Token Tracking).** The Collector feature enables inspection of BAML function call internals including raw HTTP requests, responses, usage metrics (input tokens, output tokens, cached input tokens), and timing information (`start_time_utc_ms`, `duration_ms`, `time_to_first_token_ms` for streaming). Multiple collectors can be attached to single calls, and reusable collectors accumulate logs across multiple function calls. Custom metadata tagging is supported via the `tags` property [19].

**LLM Client Registry.** The `ClientRegistry` enables runtime modification of LLM client configurations without redeploying code. Developers can register new providers via `add_llm_client`, set primary clients via `set_primary()`, and implement fallback chains and round-robin load balancing through composition providers [20].

**Prompt Caching.** BAML supports provider prompt caching strategies through message role metadata. The `allowed_role_metadata` client configuration safeguards against forwarding incompatible metadata when switching providers. Cache control is applied per-message using role annotations like `{{ _.role("user", cache_control={"type": "ephemeral"}) }}` [21].

**Prompt Optimization.** BAML integrates the GEPA (Genetic Pareto) algorithm from DSPy for automatic prompt refinement. The system optimizes across multiple objectives including accuracy, token usage, and latency. Developers can control scope with flags for trials, evaluations, target functions, and test cases. The optimization pipeline includes customizable BAML functions for improvement proposals, variant merging, and failure analysis [22].

**Prompt Transparency.** BAML's design philosophy ensures prompts are never hidden from the developer. The VSCode playground provides full prompt preview with test cases and raw cURL display of actual API requests made to LLM providers [6].

## Use Cases

**Classification.** Defining enum types for categories and functions that map input text to those categories, producing type-safe classification results with structured confidence or reasoning fields [23].

**PII Extraction.** Declaring class types for personally identifiable information fields such as names, emails, addresses, and phone numbers, then defining functions that extract all PII instances from unstructured text into typed objects [23].

**Action Item Extraction.** Parsing meeting transcripts, emails, or documents into structured action item objects with assignees, deadlines, priorities, and descriptions [23].

**Retrieval-Augmented Generation (RAG).** Structuring RAG pipeline outputs so that retrieved context and generated answers are returned as typed objects with source attribution fields [23].

**Chain-of-Thought Reasoning.** Defining output types that include both a reasoning trace and a final answer, enforcing that the LLM provides its reasoning process in a structured format alongside the conclusion [23].

**AI Agents.** Building agent workflows as while loops that call Chat BAML Functions with state. BAML enables describing tool calls and engineering context within the DSL, with the generated client handling structured tool call parsing and execution [2][24].

**Tools and Function Calling.** Defining tool schemas as BAML types and using SAP to enable function calling on any model, not just those with native function calling support [23].

**Symbol Tuning.** Optimizing enum and class field names in prompts by substituting shorter symbol representations, reducing token usage while maintaining semantic clarity [23].

## API Reference

The BAML DSL provides the following primary constructs.

**Type Declarations.**

```
class Resume {
  name string
  email string
  skills string[]
  experience Experience[]
}

enum Sentiment {
  POSITIVE
  NEGATIVE
  NEUTRAL
}
```

**Function Declarations.**

```
function ExtractResume(resume_text: string) -> Resume {
  client "openai/gpt-4o"
  prompt #"
    Extract resume information from the following text:
    {{ resume_text }}

    {{ ctx.output_format }}
  "#
}
```

**Named Client Declarations.**

```
client<llm> GPT4 {
  provider openai
  options {
    model "gpt-4o"
    temperature 0.0
    base_url "https://custom-endpoint.com/v1"
    headers {
      "custom-header" "value"
    }
  }
}
```

**Template Strings.**

```
template_string ExtractionPreamble() #"
  You are an expert data extraction assistant.
  Always return valid structured data matching the requested format.
"#
```

**Test Declarations.**

```
test ExtractBasicResume {
  functions [ExtractResume]
  args {
    resume_text "John Doe, john@example.com, Python, 5 years at Acme Corp"
  }
  @@assert( {{ this.name == "John Doe" }} )
}
```

**Field Annotations.**

```
class User {
  name string @description("The user's full name")
  age int @alias("user_age")
  internal_id string @skip
  email string @check(valid_email, {{ "@ " in this }})
  @@dynamic
}
```

**Retry and Fallback Strategies.**

```
retry_policy MyRetry {
  max_retries 3
  strategy {
    type exponential_backoff
  }
}

client<llm> MyFallback {
  provider fallback
  options {
    strategy [GPT4, Claude, Gemini]
  }
}

client<llm> MyRoundRobin {
  provider round-robin
  options {
    strategy [GPT4, Claude]
  }
}
```

**CLI Commands.**

- `baml init` -- Initialize a new BAML project with starter files
- `baml generate` -- Compile `baml_src/` and produce `baml_client/`
- `baml-cli test` -- Run declared test cases against LLM providers
- `baml-cli test --parallel 5` -- Run tests concurrently
- `baml-cli test -i "FunctionName::"` -- Filter tests by function
- `baml-cli generate --no-tests` -- Exclude test blocks from production builds
- `baml serve` -- Start a REST API server exposing BAML functions as HTTP endpoints
- `baml dev` -- Start development mode with file watching and auto-regeneration
- `baml fmt` -- Format BAML source files

## Configuration

**LLM Provider Configuration.** Each provider is configured through a client declaration specifying the provider name, model, and provider-specific options (temperature, max tokens, API base URL, timeout, headers). API keys are sourced from environment variables. Shorthand syntax allows inline provider/model selection without a separate client block [7].

**Multi-Provider Setup.** Multiple clients can be declared for different providers or model variants. Functions can reference specific clients, use fallback chains for automatic failover, or round-robin strategies for load balancing. The Client Registry enables runtime provider switching without code changes [20].

**Project Configuration.** A generator block in BAML source identifies the target language for code generation, output directory paths, client mode (sync/async), and runtime version. The `baml_src` directory must be named exactly `baml_src` for tooling compatibility but can be positioned anywhere in the project [8][12].

**Environment Variables.** Provider API keys and environment-specific settings are configured through environment variables (e.g., `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`), keeping secrets out of version-controlled BAML source files [1].

**Media URL Handling.** The `media_url_handler` configuration controls how media URLs are resolved for different providers, with options for sending URLs directly, converting to base64, or conditional handling for Google-compatible URLs [15].

## Integration Patterns

**Direct Client Usage.** The generated `baml_client` is imported directly into application code and called as typed functions. Input parameters and return values are fully typed according to the BAML definitions [1].

```python
from baml_client import b
from baml_client.types import Resume

result = b.ExtractResume("John Doe, john@example.com, Python developer")
# result is a fully typed Resume object
print(result.name)   # "John Doe"
print(result.skills) # ["Python"]
```

**REST API Deployment.** Running `baml serve` exposes all declared functions as HTTP endpoints with OpenAPI documentation, enabling language-agnostic integration and microservice deployment patterns [11].

**React/Next.js Integration.** BAML provides auto-generated React hooks for streaming structured data into frontend applications, with typed input/output/data types for type-safe UI components [5].

**Modular API Integration.** The low-level API exposes `b.request` for HTTP request generation and `b.parse` for response parsing, enabling custom HTTP client usage, request interception, custom authentication flows, and batch API patterns [13].

**Observability with Boundary Cloud.** BoundaryML offers Boundary Cloud as an observability platform for monitoring BAML function calls, tracking latency, inspecting prompts and responses, and analyzing extraction accuracy across production traffic [1].

## Examples

**Basic Classification (Python).**

Given a BAML function `ClassifySentiment(text: string) -> Sentiment`, the generated Python client is used as follows:

```python
from baml_client import b

result = b.ClassifySentiment("This product is excellent and exceeded my expectations")
# result is a typed Sentiment enum value: Sentiment.POSITIVE
```

**Structured Extraction with Streaming (TypeScript).**

```typescript
import { b } from './baml_client';

const stream = b.stream.ExtractResume({ raw_text: documentText });

for await (const partial of stream) {
  // partial is an incrementally populated Resume object
  console.log(partial.name, partial.skills);
}

const final = await stream.getFinalResponse();
// final is a fully validated Resume object
```

**Multi-Modal Input.**

BAML functions can accept image inputs for tasks like document extraction:

```
function ExtractInvoice(invoice_image: image) -> Invoice {
  client "anthropic/claude-sonnet-4-20250514"
  prompt #"
    Extract all invoice fields from this image:
    {{ invoice_image }}

    {{ ctx.output_format }}
  "#
}
```

**Dynamic Types at Runtime (Python).**

```python
from baml_client import b
from baml_client.type_builder import TypeBuilder

tb = TypeBuilder()
tb.Category.add_value("SPORTS")
tb.Category.add_value("TECHNOLOGY")
tb.Category.add_value("POLITICS")

result = b.ClassifyArticle("SpaceX launches new rocket",
    baml_options={"tb": tb})
```

**Collector for Token Tracking (Python).**

```python
from baml_client import b
from baml_client.collector import Collector

collector = Collector(name="my-tracker")
result = b.ExtractResume("...", baml_options={"collector": collector})
print(collector.last.usage)  # input_tokens, output_tokens, cached_input_tokens
print(collector.last.timing) # start_time_utc_ms, duration_ms
```

**Concurrent Calls with Cancellation (TypeScript).**

```typescript
import { b } from './baml_client';

const controller = new AbortController();

const results = await Promise.all([
  b.ClassifyMessage("message1", { signal: controller.signal }),
  b.ClassifyMessage("message2", { signal: controller.signal }),
  b.ClassifyMessage("message3", { signal: controller.signal }),
]);
```

## Limitations

- Generated client code must be regenerated whenever BAML source definitions change, adding a build step to the development workflow
- The DSL introduces a learning curve separate from general-purpose programming languages
- Provider-specific features (native function calling, tool use, structured output modes) are abstracted, which may limit access to provider-specific optimizations in edge cases
- Runtime type validation depends on LLM output quality; malformed responses that do not match the declared schema produce parse errors that must be handled by the application
- Boundary Cloud observability is a separate hosted service, not included in the open-source distribution
- PDF inputs must be provided as base64 data; URL-based PDF inputs are not currently supported
- Ruby does not currently support async/concurrent calls
- Prompt optimization is limited to descriptions and aliases; template string and compound workflow optimization are not yet supported
- OpenAPI does not currently support dynamic types (TypeBuilder)

## Changelog

BAML is under active development with frequent releases. Notable recent versions include:

- **0.219.0** (2026-02-12): PDF handling fixes in `baml-cli serve`, cancel/notify support, enhanced `build_request` across CFFI, Go, and Rust
- **0.218.0** (2026-01-22): `BamlError` base class with improved error hierarchy, Go serde decoding fixes for dynamic types, NDJSON streaming format handling in React
- **0.217.0** (2026-01-10): Native Rust SDK, `@description` wiring to Pydantic models, media type matching via file extension heuristics
- **0.216.0** (2025-12-31): `client` option in `BamlCallOptions`, AWS Bedrock IRSA region handling fixes
- **0.215.0** (2025-12-18): Prompt optimization visualizer, prompt search, TypeScript x86 Alpine and ARM64 Linux support, parser performance improvements
- **0.214.0** (2025-11-24): Static control flow visualizer, `toon` Jinja filter for token-efficient serialization
- **0.212.0** (2025-10-27): Configurable timeouts, `media_url_resolver`, block-level `@@description`, type narrowing for instanceof checks

The full project changelog and release notes are maintained on the GitHub repository at [github.com/BoundaryML/baml/releases](https://github.com/BoundaryML/baml/releases) [25].

## Citations

- [1] BAML Documentation - Welcome. BoundaryML. Available at: [https://docs.boundaryml.com/home](https://docs.boundaryml.com/home)
- [2] GitHub - BoundaryML/baml. Available at: [https://github.com/BoundaryML/baml](https://github.com/BoundaryML/baml)
- [3] Why BAML? BoundaryML. Available at: [https://docs.boundaryml.com/guide/introduction/why-baml](https://docs.boundaryml.com/guide/introduction/why-baml)
- [4] BAML Reference. BoundaryML. Available at: [https://docs.boundaryml.com/ref](https://docs.boundaryml.com/ref)
- [5] BAML Reference - React/Next.js Integration. BoundaryML. Available at: [https://docs.boundaryml.com/ref](https://docs.boundaryml.com/ref)
- [6] Prompting with BAML. BoundaryML. Available at: [https://docs.boundaryml.com/guide/baml-basics/prompting-with-baml](https://docs.boundaryml.com/guide/baml-basics/prompting-with-baml)
- [7] Switching LLMs. BoundaryML. Available at: [https://docs.boundaryml.com/guide/baml-basics/switching-llms](https://docs.boundaryml.com/guide/baml-basics/switching-llms)
- [8] What's baml_client. BoundaryML. Available at: [https://docs.boundaryml.com/guide/introduction/baml_client](https://docs.boundaryml.com/guide/introduction/baml_client)
- [9] Testing Functions. BoundaryML. Available at: [https://docs.boundaryml.com/guide/baml-basics/testing-functions](https://docs.boundaryml.com/guide/baml-basics/testing-functions)
- [10] Checks and Asserts. BoundaryML. Available at: [https://docs.boundaryml.com/guide/baml-advanced/checks-and-asserts](https://docs.boundaryml.com/guide/baml-advanced/checks-and-asserts)
- [11] BAML CLI Reference. BoundaryML. Available at: [https://docs.boundaryml.com/ref](https://docs.boundaryml.com/ref)
- [12] What's the baml_src folder. BoundaryML. Available at: [https://docs.boundaryml.com/guide/introduction/baml_src](https://docs.boundaryml.com/guide/introduction/baml_src)
- [13] Modular API. BoundaryML. Available at: [https://docs.boundaryml.com/guide/baml-advanced/modular-api](https://docs.boundaryml.com/guide/baml-advanced/modular-api)
- [14] Streaming. BoundaryML. Available at: [https://docs.boundaryml.com/guide/baml-basics/streaming](https://docs.boundaryml.com/guide/baml-basics/streaming)
- [15] Multi-Modal (Images / Audio). BoundaryML. Available at: [https://docs.boundaryml.com/guide/baml-basics/multi-modal](https://docs.boundaryml.com/guide/baml-basics/multi-modal)
- [16] Concurrent Calls. BoundaryML. Available at: [https://docs.boundaryml.com/guide/baml-basics/concurrent-calls](https://docs.boundaryml.com/guide/baml-basics/concurrent-calls)
- [17] Error Handling. BoundaryML. Available at: [https://docs.boundaryml.com/guide/baml-basics/error-handling](https://docs.boundaryml.com/guide/baml-basics/error-handling)
- [18] Dynamic Types. BoundaryML. Available at: [https://docs.boundaryml.com/guide/baml-advanced/dynamic-types](https://docs.boundaryml.com/guide/baml-advanced/dynamic-types)
- [19] Collector (Track Tokens). BoundaryML. Available at: [https://docs.boundaryml.com/guide/baml-advanced/collector-track-tokens](https://docs.boundaryml.com/guide/baml-advanced/collector-track-tokens)
- [20] LLM Client Registry. BoundaryML. Available at: [https://docs.boundaryml.com/guide/baml-advanced/llm-client-registry](https://docs.boundaryml.com/guide/baml-advanced/llm-client-registry)
- [21] Prompt Caching / Message Role Metadata. BoundaryML. Available at: [https://docs.boundaryml.com/guide/baml-advanced/prompt-caching-message-role-metadata](https://docs.boundaryml.com/guide/baml-advanced/prompt-caching-message-role-metadata)
- [22] Prompt Optimization. BoundaryML. Available at: [https://docs.boundaryml.com/guide/baml-advanced/prompt-optimization](https://docs.boundaryml.com/guide/baml-advanced/prompt-optimization)
- [23] BAML Examples. BoundaryML. Available at: [https://docs.boundaryml.com/examples](https://docs.boundaryml.com/examples)
- [24] AI Agents Need a New Syntax. BoundaryML Blog. Available at: [https://boundaryml.com/blog/ai-agents-need-new-syntax](https://boundaryml.com/blog/ai-agents-need-new-syntax)
- [25] BAML Changelog. BoundaryML. Available at: [https://github.com/BoundaryML/baml/releases](https://github.com/BoundaryML/baml/releases)
