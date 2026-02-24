# BAML

> Domain-specific language for generating structured outputs from Large Language Models (LLMs), providing type-safe definitions, generated client libraries, and production-ready extraction pipelines across multiple programming languages.

| Field        | Value                                                  |
|--------------|--------------------------------------------------------|
| Name         | BAML                                                   |
| Group        | Structured Output & Prompt Engineering                 |
| Type         | SDK                                                    |
| Open Source  | Yes                                                    |
| GitHub       | [BoundaryML/baml](https://github.com/BoundaryML/baml) |
| Stars        | 7642                                                   |
| Docs         | [docs.boundaryml.com](https://docs.boundaryml.com/)   |

## Overview

BAML is a domain-specific language (DSL) created by BoundaryML for defining, generating, and validating structured outputs from LLMs. Described as the "easiest way to use LLMs," BAML provides a declarative approach to specifying data types, LLM functions, prompt templates, and client configurations in dedicated source files (`baml_src/`), then generates fully typed client libraries (`baml_client/`) for use in application code. This architecture separates LLM interaction concerns from business logic, enabling type-safe extraction, classification, and generation workflows that move cleanly from prototyping to production deployment. BAML supports Python, TypeScript/JavaScript, Go, Ruby, Elixir, and a REST API interface, making it accessible across a broad range of technology stacks [1].

## Core Concepts

BAML introduces several foundational abstractions that distinguish it from general-purpose LLM client libraries.

**Type Definitions.** The DSL provides `class` and `enum` keywords for defining the shape of structured data that LLMs should produce. These type definitions serve as the contract between the LLM prompt and the application code, enabling compile-time and runtime validation of outputs.

**Functions.** BAML functions declare the input and output types for an LLM call, binding a prompt template to a specific extraction or generation task. Functions are the primary unit of LLM interaction and are compiled into typed methods in the generated client.

**Template Strings.** Prompt engineering is handled through Jinja-based template strings that compose prompt fragments. Template strings support variable interpolation, conditional logic, and reuse across multiple functions, enabling modular prompt construction.

**LLM Clients.** Provider-specific configuration (model name, API keys, parameters) is declared as client definitions within the DSL. BAML supports over 20 LLM providers including OpenAI, Anthropic, Google, AWS Bedrock, Azure, Groq, Ollama, and LiteLLM.

**Code Generation.** The BAML compiler reads `baml_src/` definitions and generates a `baml_client/` directory containing fully typed client code in the target language. This generated code handles serialization, deserialization, prompt rendering, and provider communication.

**Testing.** The DSL includes native test declarations that define input/output expectations for functions, enabling structured testing of LLM behavior without leaving the BAML ecosystem.

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
baml test

# Start development server with file watching
baml dev
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

**Ruby and Elixir** packages are available through their respective package managers.

**Editor Extensions.** BAML provides extensions for VSCode, Cursor, JetBrains IDEs, Zed, and Claude Code, offering syntax highlighting, autocompletion, inline diagnostics, and live preview of generated prompts.

## Architecture

BAML follows a two-directory architecture that cleanly separates definitions from generated code.

**`baml_src/` (Source Definitions).** This directory contains all BAML DSL files (`.baml` extension) where types, functions, clients, template strings, and tests are defined. These files are the single source of truth for LLM interaction contracts. Developers author and version-control these files directly.

**`baml_client/` (Generated Client Library).** Running `baml generate` compiles the source definitions into a fully typed client library in the target programming language. The generated code includes typed function signatures matching the BAML function definitions, serialization and deserialization logic for all declared types, prompt rendering from template strings with variable binding, provider-specific API communication through declared LLM clients, and streaming support where applicable. The generated client is not intended for manual editing; it is regenerated whenever source definitions change.

**Compilation Pipeline.** The BAML compiler parses `.baml` files, validates type consistency and function signatures, resolves template string references, and emits language-specific client code. This compile step catches type mismatches, missing fields, and invalid references before runtime.

## Key Features

**Streaming.** BAML supports streaming responses from LLMs with partial structured output parsing. As tokens arrive, the generated client can provide incrementally populated typed objects, enabling real-time UI updates while maintaining type safety.

**Multi-Modal Input.** Functions can accept images, audio files, PDFs, and video as inputs alongside text. The DSL provides type annotations for multi-modal content, and the generated client handles encoding and provider-specific formatting.

**Concurrent Execution.** Multiple LLM calls can be executed concurrently through the generated client, with BAML managing parallel provider requests and aggregating results according to function return types.

**Error Handling and Timeouts.** The generated client provides structured error types for provider failures, parsing errors, and timeout conditions. Timeout configuration is declarable at the client or function level.

**Prompt Caching.** BAML supports prompt caching strategies for providers that offer them, reducing latency and cost for repeated or similar prompts.

**TypeBuilder.** A runtime API for dynamically constructing BAML types programmatically, enabling scenarios where the output schema is not known at compile time.

**Symbol Tuning.** BAML can optimize enum and class field names in prompts by substituting shorter symbol representations, reducing token usage while maintaining semantic clarity for the LLM.

**Jinja Templating.** Template strings use Jinja syntax for prompt composition, supporting conditionals, loops, filters, and macro definitions for reusable prompt fragments.

## Use Cases

**Classification.** Defining enum types for categories and functions that map input text to those categories, producing type-safe classification results with structured confidence or reasoning fields.

**PII Extraction.** Declaring class types for personally identifiable information (PII) fields such as names, emails, addresses, and phone numbers, then defining functions that extract all PII instances from unstructured text into typed objects.

**Action Item Extraction.** Parsing meeting transcripts, emails, or documents into structured action item objects with assignees, deadlines, priorities, and descriptions.

**Retrieval-Augmented Generation (RAG).** Structuring RAG pipeline outputs so that retrieved context and generated answers are returned as typed objects with source attribution fields.

**Chain-of-Thought Reasoning.** Defining output types that include both a reasoning trace and a final answer, enforcing that the LLM provides its reasoning process in a structured format alongside the conclusion.

**Hallucination Reduction.** Using strict type definitions and validation to constrain LLM outputs to declared schemas, catching responses that do not conform to expected structures before they reach application logic.

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
function ExtractResume(raw_text: string) -> Resume {
  client GPT4
  prompt #"
    Extract resume information from the following text:
    {{ raw_text }}

    {{ ctx.output_format }}
  "#
}
```

**Client Declarations.**

```
client<llm> GPT4 {
  provider openai
  options {
    model "gpt-4"
    temperature 0.0
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
    raw_text "John Doe, john@example.com, Python, 5 years at Acme Corp"
  }
}
```

**CLI Commands.**

- `baml init` -- Initialize a new BAML project with starter files
- `baml generate` -- Compile `baml_src/` and produce `baml_client/`
- `baml test` -- Run declared test cases against LLM providers
- `baml serve` -- Start a REST API server exposing BAML functions as HTTP endpoints
- `baml dev` -- Start development mode with file watching and auto-regeneration
- `baml fmt` -- Format BAML source files

## Configuration

**LLM Provider Configuration.** Each provider is configured through a client declaration specifying the provider name, model, and provider-specific options (temperature, max tokens, API base URL, timeout). API keys are typically sourced from environment variables.

**Multi-Provider Setup.** Multiple clients can be declared for different providers or model variants, and functions can reference specific clients or use fallback chains.

**Project Configuration.** The BAML project root is identified by a configuration file that specifies the target language for code generation, output directory paths, and global settings.

**Environment Variables.** Provider API keys and environment-specific settings are configured through environment variables, keeping secrets out of version-controlled BAML source files.

## Integration Patterns

**Direct Client Usage.** The generated `baml_client` is imported directly into application code and called as typed functions. Input parameters and return values are fully typed according to the BAML definitions.

**REST API Deployment.** Running `baml serve` exposes all declared functions as HTTP endpoints with OpenAPI documentation, enabling language-agnostic integration and microservice deployment patterns.

**Docker Deployment.** BAML applications can be containerized with the generated client and served via the built-in REST server or embedded in application containers.

**AWS Deployment.** BAML provides deployment patterns for AWS Lambda and container services, with documentation covering serverless and long-running service configurations.

**Observability with Boundary Studio.** BoundaryML offers Boundary Studio as an observability platform for monitoring BAML function calls, tracking latency, inspecting prompts and responses, and analyzing extraction accuracy across production traffic.

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
  client GPT4Vision
  prompt #"
    Extract all invoice fields from this image:
    {{ invoice_image }}

    {{ ctx.output_format }}
  "#
}
```

## Limitations

- Generated client code must be regenerated whenever BAML source definitions change, adding a build step to the development workflow
- The DSL introduces a learning curve separate from general-purpose programming languages
- Provider-specific features (function calling, tool use, structured output modes) are abstracted, which may limit access to provider-specific optimizations in edge cases
- Runtime type validation depends on LLM output quality; malformed responses that do not match the declared schema produce parse errors that must be handled by the application
- Boundary Studio observability is a separate hosted service, not included in the open-source distribution

## Changelog

BAML is under active development with frequent releases. The project changelog and release notes are maintained on the GitHub repository at [github.com/BoundaryML/baml/releases](https://github.com/BoundaryML/baml/releases). Consult the official documentation for migration guides between major versions [1].

## Citations

- [1] BAML Documentation. BoundaryML. Available at: [https://docs.boundaryml.com/](https://docs.boundaryml.com/)
