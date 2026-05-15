# Gemini

> Google hosted LLM API with GenAI SDKs for Python/JS/Go/Java

| Field | Value |
|-------|-------|
| Group | LLM Providers |
| Type | API/SDK |
| Open Source | No |
| GitHub | N/A |
| Stars | N/A |
| Documentation | [Official Docs](https://ai.google.dev/gemini-api/docs) |

## Overview

Gemini is Google's hosted generative AI platform providing multimodal large language models accessible via the GenAI SDKs. The platform supports text generation, image generation and understanding, video creation and analysis, document processing (up to 1,000 PDF pages), speech generation, audio understanding, function calling, structured outputs, and long context processing with million-token context windows. The API is available through SDKs for Python, JavaScript, Go, Java, C#, and Apps Script, plus direct REST API access. [1][2]

The platform positions itself as "the fastest path from prompt to production" with models ranging from the flagship Gemini 3.1 Pro to cost-efficient Gemini 2.5 Flash-Lite, plus specialized models for image generation (Nano Banana), video (Veo 3.1), music (Lyria), and robotics. [1]

## Core Concepts

### Models and Tiers

Gemini offers models across performance tiers:

- **Gemini 3.1 Pro**: Advanced intelligence with complex problem-solving and agentic capabilities
- **Gemini 3 Flash**: Frontier-class performance at a fraction of the cost
- **Gemini 3 Pro**: State-of-the-art reasoning with multimodal understanding
- **Gemini 2.5 Flash**: Best price-performance for low-latency, high-volume reasoning tasks
- **Gemini 2.5 Flash-Lite**: Fastest and most budget-friendly multimodal model
- **Gemini 2.5 Pro**: Most advanced model for complex tasks with deep reasoning [2]

### Thinking

Models have "thinking" enabled by default, allowing reasoning before generating responses. Control thinking depth via `ThinkingConfig` with levels like `low`, `medium`, `high` to trade off cost, latency, and intelligence. [3]

### Multimodal Inputs

The API natively supports text, images, video, audio, and documents as inputs in a single request. Combine modalities freely in the `contents` parameter. [3]

### Function Calling

Enables models to connect with external tools and APIs. The model determines when to call functions and provides structured parameters; your application executes the actual function. Supports parallel and compositional (chained) function calling. [4]

## Architecture

### API Endpoints

The Gemini API provides:

- **`generateContent`** -- Primary text/multimodal generation
- **`generateContentStream`** -- Streaming generation
- **`embedContent`** -- Vector embeddings
- **Live API** -- Real-time conversational agents with sub-second native audio streaming
- **Image generation** -- Via Nano Banana and Imagen 4 models
- **Video generation** -- Via Veo 3.1 with synchronized audio
- **Computer Use** -- UI automation by "seeing" screens and performing actions [1]

### Google AI Studio

Browser-based platform for prototyping and testing prompts before integrating with the API. Provides a visual interface for model selection, parameter tuning, and prompt iteration. [1]

### Model Versioning

Models use version naming: Stable (specific date), Preview, Latest, and Experimental releases. Pin specific versions for production consistency. [2]

## Key Features and Functionality

### Text Generation

```python
from google import genai

client = genai.Client()
response = client.models.generate_content(
    model="gemini-3-flash-preview",
    contents="How does AI work?"
)
print(response.text)
```

Configure with `GenerateContentConfig` for system instructions, temperature, top_p, top_k, and stop sequences. Warning: setting temperature below 1.0 may cause looping or degraded performance on Gemini 3 models. [3]

### System Instructions

```python
config = types.GenerateContentConfig(
    system_instruction="You are a cat. Your name is Neko."
)
response = client.models.generate_content(
    model="gemini-3-flash-preview",
    contents="What's your name?",
    config=config
)
```
[3]

### Streaming

```python
response = client.models.generate_content_stream(
    model="gemini-3-flash-preview",
    contents=["Explain how AI works"]
)
for chunk in response:
    print(chunk.text, end="")
```
[3]

### Multi-Turn Conversations

```python
chat = client.chats.create(model="gemini-3-flash-preview")
response = chat.send_message("I have 2 dogs in my house.")
response = chat.send_message("How many paws are in my house?")
```

The SDK manages conversation history automatically, sending the full history with each turn. [3]

### Multimodal Inputs

```python
from PIL import Image

image = Image.open("/path/to/photo.png")
response = client.models.generate_content(
    model="gemini-3-flash-preview",
    contents=[image, "Tell me about this instrument"]
)
```
[3]

### Function Calling

Four modes: **AUTO** (default, model decides), **ANY** (must call a function), **NONE** (no function calls), **VALIDATED** (preview, schema-enforced). Supports parallel function calling for independent operations and compositional calling for chained sequential operations. [4]

Best practices: use descriptive function names, keep below 10-20 active tools, use low temperature (0) for deterministic calls. [4]

### Long Context

Gemini 2.0 Flash features a 1M token context window. Gemini 3 models support advanced long-context processing for large documents, codebases, and extended conversations. [2]

### Specialized Capabilities

- **Nano Banana / Nano Banana Pro**: Image generation and editing
- **Veo 3.1**: Video generation with synchronized audio
- **Lyria**: Music generation with creative control
- **Imagen 4**: Text-to-image up to 2K resolution
- **Computer Use**: Automated UI tasks
- **Deep Research**: Autonomous multi-step research
- **Gemini Robotics**: Embodied reasoning for physical spaces [1][2]

## Use Cases

### Conversational AI

Build chatbots with multi-turn context using the chat API. System instructions guide persona and behavior. [3]

### Document Processing

Process up to 1,000 PDF pages in a single request. Combine with text prompts for summarization, extraction, and analysis. [1]

### Code Generation

Gemini 3.1 Pro excels at "vibe coding" and agentic coding tasks. Use function calling to integrate with development tools. [2]

### Multimodal Analysis

Analyze images, video, and audio alongside text in unified requests for comprehensive content understanding. [3]

### Agentic Applications

Build agents using function calling with compositional patterns. Gemini 3/2.5 models with thinking excel at chaining multiple tool calls. [4]

## API Reference Summary

### Key Methods

- `client.models.generate_content()` -- Generate content from prompt
- `client.models.generate_content_stream()` -- Stream content generation
- `client.chats.create()` -- Create chat session
- `chat.send_message()` -- Send message in chat
- `client.models.embed_content()` -- Generate embeddings

### Configuration Classes

- `GenerateContentConfig` -- System instructions, temperature, top_p, top_k, stop sequences
- `ThinkingConfig` -- Thinking level control (low/medium/high)
- `FunctionDeclaration` -- Function metadata for tool use [3][4]

## Configuration and Customization

### Generation Parameters

- **`temperature`** -- Randomness control (default 1.0 for Gemini 3; values below 1.0 may degrade)
- **`top_p`** -- Nucleus sampling diversity control
- **`top_k`** -- Vocabulary selection limit
- **`stopSequences`** -- Define generation stopping points
- **`system_instruction`** -- Guide model behavior
- **`thinking_config`** -- Control reasoning depth and cost [3]

### Function Calling Modes

- **AUTO**: Model decides text response vs function call
- **ANY**: Forces function call; respects `allowed_function_names`
- **NONE**: Disables function calling
- **VALIDATED**: Ensures schema adherence (preview) [4]

## Integration Patterns

### With Google Cloud (Vertex AI)

Gemini models are available via Vertex AI for enterprise deployments with VPC controls, IAM, and compliance features.

### With Agent Frameworks (LangChain, ADK)

Google's Agent Development Kit (ADK) provides native Gemini integration. LangChain and other frameworks support Gemini as an LLM backend.

### With RAG Systems (LlamaIndex, Haystack)

Use Gemini embeddings for vector search and Gemini generation for answer synthesis in retrieval-augmented generation pipelines.

### With Observability (Langfuse, Weights & Biases)

Log Gemini API calls for monitoring, evaluation, and debugging through observability platforms.

## Examples

### Function Calling with Weather

```python
from google import genai
from google.genai import types

# Define function declaration
weather_fn = types.FunctionDeclaration(
    name="get_weather",
    description="Get weather for a city",
    parameters={
        "type": "object",
        "properties": {
            "city": {"type": "string", "description": "City name"}
        },
        "required": ["city"]
    }
)

client = genai.Client()
response = client.models.generate_content(
    model="gemini-3-flash-preview",
    contents="What's the weather in Tokyo?",
    config=types.GenerateContentConfig(
        tools=[types.Tool(function_declarations=[weather_fn])]
    )
)
```
[4]

### Thinking Control

```python
response = client.models.generate_content(
    model="gemini-3-flash-preview",
    contents="Solve this complex math problem...",
    config=types.GenerateContentConfig(
        thinking_config=types.ThinkingConfig(thinking_level="high")
    )
)
```
[3]

## Limitations and Considerations

- **Temperature warning**: Setting below 1.0 on Gemini 3 models may cause looping or degraded performance [3]
- **Model lifecycle**: Preview models may change; pin specific versions for production [2]
- **Function calling accuracy**: Decreases with more than 10-20 active tools [4]
- **Deprecated models**: Gemini 2.0 Flash and Flash-Lite marked for shutdown [2]
- **Rate limits**: API key quotas apply; higher limits available through Google Cloud billing
- **Regional availability**: Some models and features are preview-only in certain regions

## Changelog Highlights

- **Gemini 3.1 Pro**: Latest frontier model with advanced agentic capabilities
- **Gemini 3 Flash/Pro**: New generation with thinking enabled by default
- **Nano Banana Pro**: Advanced image generation and editing
- **Veo 3.1**: Video generation with native synchronized audio
- **Lyria**: Music generation model
- **Imagen 4**: High-resolution text-to-image
- **Computer Use**: UI automation capability
- **Live API**: Real-time conversational agents with sub-second audio
- **VALIDATED mode**: Schema-enforced function calling (preview) [1][2]

## Citations

- [1] Gemini API Documentation - <https://ai.google.dev/gemini-api/docs>
- [2] Gemini Models Overview - <https://ai.google.dev/gemini-api/docs/models>
- [3] Text Generation Guide - <https://ai.google.dev/gemini-api/docs/text-generation>
- [4] Function Calling Guide - <https://ai.google.dev/gemini-api/docs/function-calling>
- [5] Quickstart - <https://ai.google.dev/gemini-api/docs/quickstart>

## Discovery Signals

> Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search.

### Keywords

Gemini, Gemini 3.1 Pro, Gemini 3 Pro, Gemini 3 Flash, Gemini 2.5 Pro, Gemini 2.5 Flash, Gemini 2.5 Flash-Lite, Google AI, Google AI Studio, Google GenAI SDK, generateContent, 1M context window, native multimodal, vision, video understanding, audio understanding, document processing, function calling, AUTO ANY NONE VALIDATED, code execution, grounding with Google Search, Imagen, Nano Banana, Veo, Lyria, structured outputs, ThinkingConfig, context caching, file search

### Verb-Noun Tasks

- Call Gemini via the generateContent endpoint
- Process a 1M-token context in a single request
- Analyze a video file directly without pre-extracting frames
- Reason over a 1,000-page PDF
- Transcribe and reason about audio in a single multimodal prompt
- Generate images with Nano Banana / Imagen
- Generate video with Veo 3.1
- Generate music with Lyria
- Configure function calling modes (AUTO, ANY, NONE, VALIDATED)
- Ground responses in fresh web data via Google Search
- Enable Code Execution for the model to run Python on your behalf
- Tune thinking depth with ThinkingConfig (low / medium / high)
- Cache a long system prefix with Context Caching
- Stream responses with Server-Sent Events

### User Intent Phrases

- "I need a model with a 1M-token context window"
- "How do I send a video to an LLM for analysis?"
- "What's the cheapest way to do long-context RAG?"
- "I want Google Search results inside my LLM response"
- "How do I generate images with Google's API?"
- "I want native video generation, not just text-to-video via a separate API"
- "How do I cache a long system prompt to avoid re-paying for it?"
- "I need to do speech recognition and reasoning in one model call"
- "Which LLM provider has the strongest multimodal coverage?"
- "How do I enable Code Execution mode in Gemini?"
- "Should I use Gemini 3 Pro or Gemini 2.5 Flash for cost-sensitive workloads?"
- "How do I tune thinking depth in Gemini?"

### Problem Statements

- 200K context isn't enough for our document set
- Stitching together separate models for text + vision + audio + video creates integration overhead
- Need fresh web facts inside model responses without building a separate search-then-prompt pipeline
- Long system prompts are too expensive to send on every request
- Video and PDF processing pipelines (frame extraction → model → reassemble) are brittle

### When to Pick This

- Pick this when you need 1M+ tokens of context in a single request
- Pick this when native video, audio, or PDF input is required and you don't want to pre-process the media
- Pick this when grounding responses in fresh Google Search results is a product requirement
- Pick this when you need integrated image (Nano Banana / Imagen), video (Veo), or music (Lyria) generation alongside text from one provider
- Pick this when you're already on Google Cloud (Vertex AI gives the same models with IAM, VPC, regional endpoints)
- Pick this when Context Caching makes a repeated-prefix workload economically viable
- Pick this when you want a single SDK across Python, JavaScript, Go, Java, and C#

### Related Terms and Aliases

- Google Gemini, Google AI, Google AI Studio, Google GenAI
- Vertex AI Gemini (Google Cloud–hosted variant of the same models)
- PaLM (predecessor; deprecated)
- Bard (consumer chat product; not the API)
- generateContent API
- "Google's LLM"
- Nano Banana = Gemini's image-generation model name
- Veo = video generation; Lyria = music generation
