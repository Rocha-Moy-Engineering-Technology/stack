# Claude

> Anthropic hosted LLM API with SDKs in Python/TypeScript/Java/Go/Ruby/C#/PHP plus Agent SDK

| Field | Value |
|-------|-------|
| Group | LLM Providers |
| Type | API/SDK |
| Open Source | No |
| GitHub | N/A |
| Stars | N/A |
| Documentation | [Official Docs](https://platform.claude.com/docs/en/home) |

## Overview

Claude is Anthropic's family of large language models accessible via the Messages API. The platform provides three model tiers -- Opus (most intelligent), Sonnet (balanced speed/intelligence), and Haiku (fastest) -- with capabilities spanning text generation, vision (image analysis), tool use, extended thinking, structured outputs, web search, code execution, computer use, and prompt caching. Models are available directly via the Claude API, AWS Bedrock, Google Vertex AI, and Azure AI. SDKs are provided for Python, TypeScript, Java, Go, Ruby, C#, and PHP. [1][2]

The platform is organized around five capability areas: model capabilities (reasoning, formatting), tools (web/environment actions), tool infrastructure (discovery, orchestration), context management (efficiency for long sessions), and files/assets management. [3]

## Core Concepts

### Messages API

The Messages API is Claude's primary interface. Requests include a model identifier, max_tokens, and a messages array with alternating `user` and `assistant` roles. Responses return content blocks (text, tool_use) with stop reasons and token usage. [1]

### Model Tiers

| Model | Description | Input Price | Output Price | Context |
|-------|-------------|-------------|--------------|---------|
| Claude Opus 4.6 | Most intelligent, best for agents and coding | $5/MTok | $25/MTok | 200K (1M beta) |
| Claude Sonnet 4.6 | Best speed/intelligence balance | $3/MTok | $15/MTok | 200K (1M beta) |
| Claude Haiku 4.5 | Fastest, near-frontier intelligence | $1/MTok | $5/MTok | 200K |

All models support extended thinking, text and image input, text output, and multilingual capabilities. Max output ranges from 64K-128K tokens. [2]

### Extended Thinking

Enhanced reasoning for complex tasks. Provides transparency into Claude's step-by-step thought process before the final answer. Available on all current models. [3]

### Adaptive Thinking

Lets Claude dynamically decide when and how much to think. The recommended thinking mode for Opus 4.6. Control thinking depth with the `effort` parameter. Available on Opus 4.6 and Sonnet 4.6. [3]

### Tool Use

Claude supports two tool types:
- **Client tools**: Execute on your systems (custom tools, computer use, text editor, bash)
- **Server tools**: Execute on Anthropic's servers (web search, web fetch, code execution, memory) [4]

## Architecture

### API Surface

Claude's API is organized into five areas:

- **Model capabilities** -- Extended thinking, adaptive thinking, structured outputs, citations, effort control, batch processing, PDF support, 1M token context window
- **Tools** -- Server-side (web search, web fetch, code execution, memory) and client-side (bash, computer use, text editor)
- **Tool infrastructure** -- Agent Skills, MCP connector, tool search, fine-grained tool streaming, programmatic tool calling
- **Context management** -- Prompt caching (5min and 1hr), compaction, context editing, token counting
- **Files and assets** -- Files API for PDFs, images, and text files [3]

### Authentication

Requests require `x-api-key` header with your API key and `anthropic-version` header (currently `2023-06-01`). [1]

### Multi-Platform Availability

Models are available on Claude API (direct), AWS Bedrock, Google Vertex AI, and Azure AI with platform-specific model IDs. Starting with Claude Sonnet 4.5, Bedrock and Vertex AI offer global endpoints (dynamic routing) and regional endpoints (geographic data routing). [2]

## Key Features and Functionality

### Tool Use

Define tools with name, description, and JSON schema input:

```python
response = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=1024,
    tools=[{
        "name": "get_weather",
        "description": "Get the current weather in a given location",
        "input_schema": {
            "type": "object",
            "properties": {
                "location": {
                    "type": "string",
                    "description": "City and state, e.g. San Francisco, CA"
                }
            },
            "required": ["location"]
        }
    }],
    messages=[{"role": "user", "content": "What's the weather in SF?"}],
)
```

When Claude decides to use a tool, the response has `stop_reason: "tool_use"`. Execute the tool, return results in a `tool_result` block, and Claude incorporates the data into a final response. Supports parallel tool calls and sequential chaining. [4]

### Strict Tool Use (Structured Outputs)

Add `strict: true` to tool definitions for guaranteed schema validation on tool inputs. Eliminates type mismatches and missing fields for production agents. [3][4]

### Server-Side Tools

- **Web search**: Augment knowledge with real-world web data
- **Web fetch**: Retrieve full content from web pages and PDFs
- **Code execution**: Run code in a sandboxed environment
- **Memory**: Store and retrieve information across conversations [3]

### Client-Side Tools

- **Bash**: Execute shell commands and scripts
- **Computer use**: Control interfaces via screenshots and mouse/keyboard
- **Text editor**: Create and edit text files [3]

### MCP Connector

Connect to remote Model Context Protocol (MCP) servers directly from the Messages API without implementing a separate MCP client. Convert MCP tool definitions by renaming `inputSchema` to `input_schema`. [4]

### Extended Thinking and Effort

Control reasoning depth with extended thinking for step-by-step transparency, or use the `effort` parameter (on Opus 4.6/4.5) to control token usage vs thoroughness tradeoff. [3]

### Prompt Caching

Two durations: 5-minute (standard) and 1-hour (extended). Automatic prompt caching simplifies to a single API parameter, caching the last cacheable block. Reduces costs and latency for repeated context. [3]

### Citations

Ground responses in source documents with detailed references to exact sentences and passages. Enables verifiable, trustworthy outputs. [3]

### Batch Processing

Process large volumes asynchronously at 50% cost reduction. [3]

### 1M Token Context Window

Available on Opus 4.6 and Sonnet 4.6 with the `context-1m-2025-08-07` beta header. Long context pricing applies to requests exceeding 200K tokens. [2]

## Use Cases

### Agentic Applications

Build agents using tool use with web search, code execution, and custom tools. Agent Skills extend capabilities with pre-built (PowerPoint, Excel, Word, PDF) or custom skills. [3]

### Document Analysis

Process PDFs, images, and text files. Vision capabilities analyze images sent as base64 or file references. Citations provide source attribution. [3]

### Code Generation and Review

Claude excels at coding tasks. Use bash tool and text editor tool for automated code generation, editing, and testing workflows. [3]

### Conversational AI

Multi-turn conversations with context management. Compaction automatically summarizes earlier conversation parts when approaching context limits. [3]

### Data Processing

Batch API processes large volumes at 50% cost savings. Structured outputs guarantee schema conformance for data extraction. [3]

## API Reference Summary

### Key Endpoints

- `POST /v1/messages` -- Create a message (primary generation endpoint)
- `POST /v1/messages/count_tokens` -- Count tokens before sending
- `POST /v1/messages/batches` -- Create batch processing job
- `POST /v1/files` -- Upload files (beta)

### Request Parameters

- `model` -- Model identifier (e.g., `claude-opus-4-6`)
- `max_tokens` -- Maximum output tokens (required)
- `messages` -- Array of user/assistant message objects
- `system` -- System prompt string
- `tools` -- Array of tool definitions
- `tool_choice` -- Control tool usage (auto/any/tool/none)
- `temperature` -- Randomness control (0-1)
- `stream` -- Enable Server-Sent Events streaming

### Response Properties

- `content` -- Array of content blocks (TextBlock, ToolUseBlock)
- `stop_reason` -- Why generation stopped (end_turn, tool_use, max_tokens, pause_turn)
- `usage` -- Token counts (input_tokens, output_tokens)
- `model` -- Model used [1][4]

## Configuration and Customization

### Generation Parameters

- **`model`** -- `claude-opus-4-6`, `claude-sonnet-4-6`, `claude-haiku-4-5`
- **`max_tokens`** -- Maximum output tokens (required; up to 128K for Opus 4.6)
- **`temperature`** -- Randomness (0 = deterministic, 1 = default)
- **`system`** -- System prompt for behavioral guidance
- **`stream`** -- Server-Sent Events streaming
- **`tool_choice`** -- auto, any, tool (specific), none
- **`metadata`** -- User ID for abuse detection [1]

### Platform-Specific IDs

| Platform | Opus 4.6 | Sonnet 4.6 | Haiku 4.5 |
|----------|----------|------------|-----------|
| Claude API | claude-opus-4-6 | claude-sonnet-4-6 | claude-haiku-4-5-20251001 |
| AWS Bedrock | anthropic.claude-opus-4-6-v1 | anthropic.claude-sonnet-4-6 | anthropic.claude-haiku-4-5-20251001-v1:0 |
| GCP Vertex AI | claude-opus-4-6 | claude-sonnet-4-6 | claude-haiku-4-5@20251001 |
[2]

### Beta Features

Enable with `anthropic-beta` header:
- `context-1m-2025-08-07` -- 1M token context window
- Computer use, MCP connector, Agent Skills, Files API [3]

## Integration Patterns

### With Agent Frameworks (LangChain, LangGraph, CrewAI)

Claude models integrate as LLM backends. Tool use maps to framework tool abstractions. MCP connector enables direct server tool access.

### With RAG Systems (LlamaIndex, Haystack)

Use citations feature for grounded responses. Prompt caching reduces cost for repeated retrieval context. 1M context window supports large document collections.

### With Observability (Langfuse, LangSmith)

Log API requests for monitoring and evaluation. Token counting endpoint enables pre-request cost estimation.

### With API Gateways (LiteLLM, Portkey)

Route through proxy layers for multi-provider failover. Supports OpenAI-compatible message format translation.

### With AWS/GCP/Azure

Available natively on Bedrock, Vertex AI, and Azure AI. Global and regional endpoints provide geographic data routing control.

## Examples

### Tool Use with Weather

```python
import anthropic

client = anthropic.Anthropic()

# Step 1: Send request with tools
response = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=1024,
    tools=[{
        "name": "get_weather",
        "description": "Get current weather for a location",
        "input_schema": {
            "type": "object",
            "properties": {
                "location": {"type": "string", "description": "City, State"}
            },
            "required": ["location"]
        }
    }],
    messages=[{"role": "user", "content": "Weather in San Francisco?"}],
)

# Step 2: Extract tool call, execute, return result
# Response has stop_reason="tool_use" with tool name and input
# Execute get_weather("San Francisco, CA")
# Return result in tool_result block for final response
```
[4]

### MCP Integration

```python
from mcp import ClientSession

async def get_claude_tools(mcp_session: ClientSession):
    mcp_tools = await mcp_session.list_tools()
    return [{
        "name": tool.name,
        "description": tool.description or "",
        "input_schema": tool.inputSchema,
    } for tool in mcp_tools.tools]

# Use converted tools with Claude
claude_tools = await get_claude_tools(mcp_session)
response = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=1024,
    tools=claude_tools,
    messages=[{"role": "user", "content": "Use the available tools"}],
)
```
[4]

## Limitations and Considerations

- **Max tokens required**: Every request must specify `max_tokens` (unlike some competitors)
- **Tool use token overhead**: Tool definitions add 313-346 system prompt tokens per request [4]
- **Pause turn handling**: Server-side tools have a 10-iteration limit; `pause_turn` stop reason requires continuation [4]
- **Context limits**: 200K tokens standard; 1M requires beta header and incurs long context pricing [2]
- **Model variability**: Sonnet may infer missing parameters rather than asking for clarification; Opus is more likely to ask [4]
- **Legacy deprecation**: Claude Haiku 3 deprecated, retiring April 2026 [2]
- **Rate limits**: Platform-specific quotas apply

## Changelog Highlights

- **Claude 4.6**: Opus 4.6 and Sonnet 4.6 with adaptive thinking, 1M context (beta)
- **Claude 4.5**: Opus 4.5 and Sonnet 4.5 with extended thinking
- **Claude 4.1**: Opus 4.1 with improved coding
- **Claude 4**: Initial Claude 4 generation (Opus 4, Sonnet 4)
- **Agent Skills**: Pre-built and custom skills for PowerPoint, Excel, Word, PDF
- **MCP Connector**: Direct MCP server access from Messages API
- **Server tools**: Web search, web fetch, code execution, memory
- **Structured outputs**: Strict tool use for guaranteed schema validation
- **Prompt caching**: 5-minute and 1-hour durations with automatic mode
- **Compaction**: Server-side context summarization
- **Tool search**: Scale to thousands of tools with dynamic discovery [2][3]

## Citations

- [1] Getting Started - <https://platform.claude.com/docs/en/get-started>
- [2] Models Overview - <https://platform.claude.com/docs/en/about-claude/models>
- [3] Features Overview - <https://platform.claude.com/docs/en/build-with-claude/overview>
- [4] Tool Use Overview - <https://platform.claude.com/docs/en/build-with-claude/tool-use/overview>
- [5] Claude Developer Platform - <https://platform.claude.com/docs/en/home>
