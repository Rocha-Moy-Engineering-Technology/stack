# NeMo Guardrails

> NVIDIA toolkit for programmable safety guardrails in conversational AI

| Field | Value |
|-------|-------|
| Group | Guardrails & Safety |
| Type | SDK |
| Open Source | Yes |
| GitHub | [https://github.com/NVIDIA/NeMo-Guardrails](https://github.com/NVIDIA/NeMo-Guardrails) |
| Stars | 5683 |
| Documentation | [Official Docs](https://docs.nvidia.com/nemo-guardrails/index.html) |

## Overview

NeMo Guardrails is an open-source toolkit from NVIDIA for adding programmable guardrails to LLM-based conversational systems. It provides a framework for controlling model output, enforcing safety policies, and steering dialogue flow through a combination of a domain-specific language called Colang and YAML-based configuration. The toolkit intercepts user input and model output at multiple stages of the conversation pipeline, applying configurable validation, filtering, and transformation rules before responses reach the end user.

NeMo Guardrails operates independently of the underlying LLM provider. It supports OpenAI, NVIDIA-hosted models, LLaMa-2, Falcon, Vicuna, Mosaic, and other models through configurable engine backends. The toolkit integrates with LangChain as a composable Runnable, supports FastAPI-based server deployment, and provides built-in mechanisms for jailbreak detection, sensitive data masking, fact-checking, toxicity filtering, and topic restriction.

The project is licensed under Apache 2.0 and has accumulated over 5,600 GitHub stars. It was introduced in a 2023 EMNLP paper by Rebedea et al. and continues active development, with the latest release at v0.20.0.

## Core Concepts

**Colang** is a Python-like domain-specific language for defining conversational guardrails. It models user intents, bot responses, and dialogue flows in a readable, declarative format. Two versions exist: Colang 1.0 (the default, simpler syntax for standard guardrails) and Colang 2.0 (enhanced capabilities for complex workflows with Jinja2 templating and async operations). Colang files use the `.co` extension.

**Rails** are the guardrail mechanisms that intercept and process data at specific stages of the conversation pipeline. NeMo Guardrails defines five rail types, each operating at a different point in the request-response lifecycle: input rails, dialog rails, retrieval rails, execution rails, and output rails.

**Flows** are sequences of steps defined in Colang that describe conversation patterns and guardrail logic. Flows match user intents, execute actions, evaluate conditions, and produce bot responses. They are the primary building block for both dialogue management and safety enforcement.

**Actions** are Python functions that flows invoke to interact with external systems, perform computations, or apply validation logic. Actions can be asynchronous, access conversation context, and return structured data that flows consume for branching decisions.

**Configuration** is split between YAML files (`config.yml`) for model selection and rail activation, Colang files (`.co`) for flow definitions, and Python files (`config.py`, `actions.py`) for custom initialization and action registration.

**LLMRails** is the central Python class that loads configuration, registers actions, and processes messages through the configured rail pipeline. It exposes both synchronous and asynchronous methods for generating responses.

**RailsConfig** is the configuration container that loads and validates Colang content, YAML settings, and custom code. It can be constructed from a directory path or from inline content strings.

## Installation and Setup

NeMo Guardrails requires Python 3.10, 3.11, 3.12, or 3.13. A C++ compiler is needed for the `annoy` dependency.

Install the core package with NVIDIA model support:

```bash
pip install "nemoguardrails[nvidia]"
```

Available extras for additional functionality:

```bash
pip install "nemoguardrails[nvidia]"       # NVIDIA-hosted model integration
pip install "nemoguardrails[openai]"       # OpenAI model support
pip install "nemoguardrails[sdd]"          # Sensitive data detection via Presidio
pip install "nemoguardrails[eval]"         # Evaluation tools
pip install "nemoguardrails[tracing]"      # OpenTelemetry support
pip install "nemoguardrails[jailbreak]"    # YARA-based jailbreak detection
pip install "nemoguardrails[multilingual]" # Language detection
pip install "nemoguardrails[gcp]"          # Google Cloud Platform services
pip install "nemoguardrails[all]"          # All extras
```

Install from source using Poetry:

```bash
git clone https://github.com/NVIDIA/NeMo-Guardrails.git
cd NeMo-Guardrails
poetry install --extras "nvidia"
```

Set the API key for your chosen provider:

```bash
export NVIDIA_API_KEY="nvapi-..."   # NVIDIA models via build.nvidia.com
# or
export OPENAI_API_KEY="sk-..."      # OpenAI models
```

Verify the installation:

```python
from nemoguardrails import LLMRails, RailsConfig

config = RailsConfig.from_content(
    colang_content="""
define user express greeting
  "hello"
  "hi"

define flow greeting
  user express greeting
  bot express greeting
    "Hello! How can I help you today?"
""",
    yaml_content="""
models:
  - type: main
    engine: openai
    model: gpt-4
"""
)

rails = LLMRails(config)
response = rails.generate(messages=[{"role": "user", "content": "Hello!"}])
print(response["content"])
```

Compiler setup for the `annoy` dependency on Linux/macOS:

```bash
apt-get install gcc g++ python3-dev
```

On Windows, install Microsoft C++ Build Tools version 14.0 or higher.

## Architecture

NeMo Guardrails processes every conversation turn through a pipeline of five rail types, each intercepting data at a specific stage:

```
User Input
    |
    v
[Input Rails]       -- Validate, reject, or alter user input
    |
    v
[Dialog Rails]      -- Influence LLM prompting and conversation flow
    |
    v
[Retrieval Rails]   -- Filter RAG chunks before LLM processing
    |
    v
[LLM Generation]    -- Core model inference
    |
    v
[Execution Rails]   -- Monitor custom action inputs/outputs
    |
    v
[Output Rails]      -- Validate or modify generated responses
    |
    v
Bot Response
```

**Input Rails** trigger when the system receives a new user message. They can reject messages outright (returning a canned refusal), modify the content (masking sensitive data), or pass the message through unchanged. Common input rails include jailbreak detection, toxicity checking, and Personally Identifiable Information (PII) filtering.

**Dialog Rails** control how the LLM is prompted and how conversation flow proceeds. They match user intents against Colang flow definitions, determine which actions to execute, and select bot response templates. Dialog rails implement the conversational logic layer.

**Retrieval Rails** trigger when the system retrieves context chunks from a knowledge base for Retrieval-Augmented Generation (RAG). They filter, mask, or reject retrieved documents before they are injected into the LLM prompt.

**Execution Rails** monitor the inputs and outputs of custom actions invoked during flow execution. They provide a checkpoint for validating that action parameters and return values meet safety requirements.

**Output Rails** trigger after the system generates a bot message. They validate the response against fact-checking rules, hallucination detectors, toxicity filters, topic restrictions, and sensitive data policies before delivering it to the user.

The standard configuration directory structure:

```
config/
  config.yml        -- Model selection, rail activation, parameters
  config.py         -- Custom initialization code (optional)
  actions.py        -- Custom action definitions (optional)
  rails.co          -- Colang flow definitions for guardrails
  prompts.co        -- Custom prompt templates (optional)
```

## Key Features and Functionality

**Jailbreak Detection**: Built-in heuristics and YARA-based pattern matching detect prompt injection and jailbreak attempts. Enable through input rail configuration:

```yaml
rails:
  input:
    flows:
      - jailbreak detection heuristics
```

**Sensitive Data Detection**: Integration with Microsoft Presidio for detecting and masking PII across input, output, and retrieval rails. Supports configurable entity types:

```yaml
rails:
  config:
    sensitive_data_detection:
      input:
        entities:
          - PERSON
          - EMAIL_ADDRESS
          - PHONE_NUMBER
          - CREDIT_CARD
      output:
        entities:
          - PERSON
          - EMAIL_ADDRESS
  input:
    flows:
      - mask sensitive data on input
  output:
    flows:
      - mask sensitive data on output
  retrieval:
    flows:
      - mask sensitive data on retrieval
```

**Fact-Checking and Hallucination Detection**: Output rails that verify generated responses against retrieved context, flagging or blocking responses that contain unsupported claims:

```yaml
rails:
  output:
    flows:
      - self check facts
      - self check hallucination
```

**Toxicity Filtering**: Content moderation for both input and output using configurable thresholds:

```yaml
rails:
  config:
    guardrails_ai:
      validators:
        - name: toxic_language
          parameters:
            threshold: 0.5
            validation_method: "sentence"
  input:
    flows:
      - check toxicity
```

**Topic Restriction**: Constrain conversations to approved topics and block off-topic requests:

```yaml
rails:
  config:
    guardrails_ai:
      validators:
        - name: restricttotopic
          parameters:
            valid_topics: ["technology", "science", "education"]
  output:
    flows:
      - guardrailsai check output $validator="restricttotopic"
```

**Selective Rail Execution**: Programmatically enable or disable specific rails per request using `GenerationOptions`:

```python
from nemoguardrails.rails.llm.options import GenerationOptions

options = GenerationOptions(
    rails={
        "input": ["self check input", "mask sensitive data on input"],
        "output": ["self check output"],
        "dialog": True,
        "retrieval": False
    }
)

response = rails.generate(
    messages=[{"role": "user", "content": "Hello"}],
    options=options
)
```

**Knowledge Base Retrieval**: Built-in support for RAG workflows with retrieval rails that filter chunks before they reach the LLM:

```colang
define extension flow generate bot message
  priority 100
  bot ...
  execute retrieve_relevant_chunks
  execute generate_bot_message
```

**Evaluation Tools**: CLI-based evaluation for testing guardrail effectiveness against topical rails, fact-checking, moderation, and hallucination detection:

```bash
nemoguardrails evaluate --config path/to/config
```

## Use Cases

**Customer Service Bots**: Enforce brand-safe responses, restrict conversations to supported product topics, prevent disclosure of internal policies, and mask customer PII in logs and responses.

**Healthcare Assistants**: Apply strict guardrails to prevent medical advice beyond approved scope, detect and refuse harmful health-related queries, and ensure responses cite verified medical sources through fact-checking rails.

**Financial Services Chatbots**: Block unauthorized investment recommendations, detect and mask financial PII (account numbers, Social Security Numbers), enforce regulatory compliance in generated responses, and restrict discussions to approved financial products.

**Enterprise Knowledge Assistants**: Control access to sensitive information based on user roles through custom actions that validate permissions, filter retrieved documents through retrieval rails, and enforce topic boundaries aligned with departmental scope.

**Educational Platforms**: Restrict content to age-appropriate topics, prevent generation of harmful or inappropriate material, and guide conversations toward constructive educational outcomes through dialog rails.

**Content Moderation Systems**: Layer multiple output rails for toxicity detection, fact verification, and topic adherence to ensure all generated content meets platform community standards before publication.

## API Reference Summary

**RailsConfig**: Configuration container for loading and validating guardrail settings.

```python
# Load from directory
config = RailsConfig.from_path("path/to/config/")

# Load from inline content
config = RailsConfig.from_content(
    colang_content="...",
    yaml_content="...",
)

# Load from dictionary
config = RailsConfig.from_content(
    colang_content="...",
    config={"models": [{"type": "main", "engine": "openai", "model": "gpt-4"}]}
)
```

**LLMRails**: Central class for message processing through the rail pipeline.

```python
rails = LLMRails(config)

# Synchronous generation
response = rails.generate(messages=[{"role": "user", "content": "Hello"}])

# Asynchronous generation
response = await rails.generate_async(messages=[{"role": "user", "content": "Hello"}])

# Generation with options
response = rails.generate(messages=messages, options=generation_options)

# Register custom actions
rails.register_action(my_custom_action)

# Register action parameters (global values accessible to all actions)
rails.register_action_param("api_key", "your-key-here")
```

**GenerationOptions**: Control which rails execute per request.

```python
from nemoguardrails.rails.llm.options import GenerationOptions

options = GenerationOptions(
    rails={
        "input": True,
        "output": False,
        "dialog": True,
        "retrieval": True
    }
)
```

**@action Decorator**: Register Python functions as invocable actions.

```python
from nemoguardrails.actions import action

@action(is_system_action=True, name="check_permission")
async def check_permission(user_id: str, context: dict = None, llm=None):
    return {"allowed": True, "reason": "User is authorized"}
```

**Server CLI**: Launch the guardrails HTTP server.

```bash
nemoguardrails server --config path/to/config --port 8000
```

**Evaluation CLI**: Run built-in evaluation suites.

```bash
nemoguardrails evaluate --config path/to/config
```

## Configuration and Customization

**Model Configuration** (`config.yml`): Define the LLM engine, model, and Colang version:

```yaml
models:
  - type: main
    engine: openai
    model: gpt-4

# Optional: specify Colang version
colang_version: "2.x"
```

**Rail Activation**: Enable specific rail flows in the YAML configuration:

```yaml
rails:
  input:
    flows:
      - check jailbreak
      - check input sensitive data
      - check toxicity
  output:
    flows:
      - self check facts
      - self check hallucination
      - check output sensitive data
  retrieval:
    flows:
      - check retrieval sensitive data
```

**GuardrailsAI Validators**: Configure third-party validators with parameters:

```yaml
rails:
  config:
    guardrails_ai:
      validators:
        - name: guardrails_pii
          parameters:
            entities: ["phone_number", "email", "ssn", "credit_card"]
          metadata: {}
        - name: competitor_check
          parameters:
            competitors: ["Apple", "Google", "Microsoft"]
          metadata: {}
        - name: valid_length
          parameters:
            min: 10
            max: 500
          metadata: {}
  input:
    flows:
      - guardrailsai check input $validator="guardrails_pii"
      - guardrailsai check input $validator="competitor_check"
  output:
    flows:
      - guardrailsai check output $validator="valid_length"
```

**Custom Actions** (`actions.py`): Define Python functions that Colang flows invoke:

```python
from nemoguardrails.actions import action
import aiohttp

@action(is_system_action=True, name="fetch_weather")
async def fetch_weather(location: str, context: dict = None):
    async with aiohttp.ClientSession() as session:
        async with session.get(f"https://api.weather.com/{location}") as resp:
            data = await resp.json()
    return {
        "condition": data["condition"],
        "temperature": data["temperature"],
        "error": False
    }
```

**Context Passing**: Inject runtime context into the conversation for actions to consume:

```python
messages = [
    {"role": "context", "content": {
        "user_id": "user123",
        "session_id": "sess_abc"
    }},
    {"role": "user", "content": "What is my account balance?"}
]

response = rails.generate(messages=messages)
```

## Integration Patterns

**LangChain RunnableRails**: Wrap guardrails around LangChain chains using the `RunnableRails` class, which implements the LangChain Runnable interface for seamless composition:

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from nemoguardrails.integrations.langchain.runnable_rails import RunnableRails

guardrails = RunnableRails(config)

prompt = ChatPromptTemplate.from_template("Tell me about {topic}")
model = ChatOpenAI()
output_parser = StrOutputParser()

# Insert guardrails between prompt and model
chain = prompt | (guardrails | model) | output_parser
result = chain.invoke({"topic": "machine learning"})
```

**LangChain Streaming**: RunnableRails supports both synchronous and asynchronous streaming:

```python
# Synchronous streaming
for chunk in guardrails.stream("What is machine learning?"):
    print(chunk, end="", flush=True)

# Asynchronous streaming
async for chunk in guardrails.astream("What is machine learning?"):
    print(chunk, end="", flush=True)
```

**LangChain Conditional Branching**: Combine guardrails with LangChain's branching logic:

```python
from langchain_core.runnables import RunnableBranch

branch = RunnableBranch(
    (lambda x: "technical" in x["topic"], guardrails | technical_chain),
    (lambda x: "creative" in x["topic"], creative_chain),
    guardrails | general_chain,
)
```

**FastAPI Server Deployment**: Launch the HTTP server with OpenAI-compatible endpoints:

```bash
nemoguardrails server --config path/to/config --port 8000
```

The server exposes `/v1/chat/completions` for direct integration with applications expecting the OpenAI API format. Docker deployment is supported through the included Dockerfile.

**OpenTelemetry Tracing**: Enable distributed tracing with the `tracing` extra for observability into rail execution:

```bash
pip install "nemoguardrails[tracing]"
```

## Examples

**Basic Greeting with Topic Control**:

```python
from nemoguardrails import LLMRails, RailsConfig

colang_content = """
define user express greeting
  "hello"
  "hi"
  "hey there"

define user ask about politics
  "what do you think about the election"
  "who should I vote for"

define flow greeting
  user express greeting
  bot express greeting
    "Hello! I'm here to help with product questions."

define flow block politics
  user ask about politics
  bot refuse topic
    "I'm not able to discuss political topics. Can I help with something product-related?"
"""

yaml_content = """
models:
  - type: main
    engine: openai
    model: gpt-4
"""

config = RailsConfig.from_content(
    colang_content=colang_content,
    yaml_content=yaml_content
)

rails = LLMRails(config)

response = rails.generate(
    messages=[{"role": "user", "content": "Hi there!"}]
)
print(response["content"])
# "Hello! I'm here to help with product questions."
```

**Custom Action with Permission Checking**:

```python
from nemoguardrails import LLMRails, RailsConfig
from nemoguardrails.actions import action

@action(is_system_action=True, name="validate_user_tier")
async def validate_user_tier(user_id: str, context: dict = None):
    tier_map = {"user123": "premium", "user456": "free"}
    return tier_map.get(user_id, "free")

colang_content = """
define flow premium feature access
  user asks for premium feature
  $user_id = context.user_id
  $tier = execute validate_user_tier(user_id=$user_id)
  if $tier == "free"
    bot refuse access
      "This feature requires a paid subscription."
    stop
  bot provide premium feature
"""

config = RailsConfig.from_content(
    colang_content=colang_content,
    config={"models": [{"type": "main", "engine": "openai", "model": "gpt-4"}]}
)

rails = LLMRails(config)
rails.register_action(validate_user_tier)

messages = [
    {"role": "context", "content": {"user_id": "user456"}},
    {"role": "user", "content": "I want advanced analytics"}
]

response = rails.generate(messages=messages)
print(response["content"])
# "This feature requires a paid subscription."
```

**Comprehensive Input/Output Guardrails**:

```yaml
models:
  - type: main
    engine: openai
    model: gpt-4

rails:
  config:
    sensitive_data_detection:
      input:
        entities:
          - PERSON
          - EMAIL_ADDRESS
          - PHONE_NUMBER
          - CREDIT_CARD
      output:
        entities:
          - PERSON
          - EMAIL_ADDRESS
    guardrails_ai:
      validators:
        - name: toxic_language
          parameters:
            threshold: 0.5
            validation_method: "sentence"
        - name: restricttotopic
          parameters:
            valid_topics: ["technology", "support", "billing"]

  input:
    flows:
      - jailbreak detection heuristics
      - mask sensitive data on input
      - check toxicity

  output:
    flows:
      - self check facts
      - mask sensitive data on output
      - guardrailsai check output $validator="restricttotopic"

  retrieval:
    flows:
      - mask sensitive data on retrieval
```

**Colang 2.0 with LLM-Driven Flow**:

```colang
import core
import llm

flow main
  """You are a car dealership assistant. You help customers learn about
  vehicles in our inventory. Politely decline to discuss other topics.

  Last user question is: "{{ question }}"
  Generate the output in the following format:

  bot say "<<the response>>"
  """
  $question = await user said something
  ...
```

## Limitations and Considerations

**Latency Overhead**: Each active rail adds processing time to the request-response cycle. Stacking multiple input, output, and retrieval rails compounds latency. Applications with strict response time requirements should benchmark their specific rail configuration and consider disabling non-critical rails for latency-sensitive paths using `GenerationOptions`.

**LLM Dependency for Rail Evaluation**: Several built-in rails (self-check facts, self-check hallucination, jailbreak detection) rely on additional LLM calls to evaluate content. This increases both latency and cost, as each guardrail check may invoke the model one or more times beyond the primary generation call.

**Colang Learning Curve**: While Colang syntax is readable, mastering flow design, action integration, context management, and the differences between Colang 1.0 and 2.0 requires dedicated learning. Complex guardrail logic can become difficult to debug when flows interact in unexpected ways.

**Model-Dependent Effectiveness**: The quality of guardrail enforcement depends on the underlying LLM's capabilities. Weaker models may produce less reliable results for self-check rails, fact verification, and intent classification. Guardrail configurations tuned for one model may require adjustment when switching providers.

**No Guarantee of Complete Safety**: Guardrails reduce but do not eliminate the risk of harmful outputs. Determined adversaries may find prompts that bypass heuristic-based jailbreak detection. The toolkit is best used as one layer in a defense-in-depth security strategy rather than a standalone safety solution.

**Python-Only SDK**: The toolkit is Python-only. Applications built in other languages must interact with NeMo Guardrails through the HTTP server API rather than embedding the SDK directly.

**Hardware Requirements**: While the library itself runs on CPU with minimal resources (1 CPU, 4GB RAM), practical deployments require network access to external LLM APIs. On-premise model deployments may require GPU infrastructure depending on the model serving backend.

**Colang 2.0 Maturity**: Colang 2.0 introduces significant enhancements (Jinja2 templating, async operations) but is newer than Colang 1.0. Some documentation and community examples still reference Colang 1.0 patterns, which can cause confusion when following tutorials.

## Changelog Highlights

- **v0.20.0**: Latest release with continued improvements to Colang 2.0, enhanced rail configuration options, and expanded validator support.
- **Colang 2.0 Introduction**: Added enhanced flow syntax with Jinja2 templating, async operations, and the `...` operator for LLM-driven flow continuation.
- **RunnableRails**: LangChain integration providing synchronous and asynchronous streaming, conditional branching, and composable chain insertion.
- **GuardrailsAI Integration**: Support for third-party validators including PII detection, competitor mention blocking, topic restriction, toxic language detection, and response length validation.
- **YARA-Based Jailbreak Detection**: Pattern-matching engine for detecting known jailbreak prompt structures via the `jailbreak` extra.
- **OpenTelemetry Tracing**: Distributed tracing support for monitoring rail execution in production environments via the `tracing` extra.
- **Presidio Integration**: Sensitive data detection and masking across input, output, and retrieval rails with configurable entity types.
- **GenerationOptions**: Per-request control over which rails execute, enabling selective rail activation and deactivation at runtime.
- **Multi-Config Server**: FastAPI server supporting multiple guardrail configurations with OpenAI-compatible `/v1/chat/completions` endpoint.

## Citations

- [1] NeMo Guardrails GitHub Repository - https://github.com/NVIDIA/NeMo-Guardrails
- [2] NeMo Guardrails Documentation - https://docs.nvidia.com/nemo-guardrails/index.html
- [3] Rebedea, T., et al. "NeMo Guardrails: A Toolkit for Controllable and Safe LLM Applications with Programmable Rails." EMNLP 2023 System Demonstrations. https://aclanthology.org/2023.emnlp-demo.40/
- [4] Colang Language Reference - https://github.com/NVIDIA/NeMo-Guardrails/tree/develop/docs/colang-2
- [5] LangChain RunnableRails Integration - https://github.com/NVIDIA/NeMo-Guardrails/blob/develop/docs/user-guides/langchain/runnable-rails.md
