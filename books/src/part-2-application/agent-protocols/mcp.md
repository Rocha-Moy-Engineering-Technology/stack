# MCP

> Open standard and SDKs for connecting LLM applications to external tools and data sources

| Field | Value |
|-------|-------|
| Group | Agent Protocols |
| Type | SDK |
| Open Source | Yes |
| GitHub | [modelcontextprotocol/modelcontextprotocol](https://github.com/modelcontextprotocol/modelcontextprotocol) |
| Stars | 8108 |
| Documentation | [Official Docs](https://modelcontextprotocol.io/) |

## Overview

The Model Context Protocol (MCP) is an open standard for connecting AI applications to external systems. Using MCP, AI applications like Claude, ChatGPT, and IDE assistants can connect to data sources (local files, databases), tools (search engines, calculators, API clients), and workflows (specialized prompts and templates), enabling them to access information and perform tasks across heterogeneous backends through a single uniform protocol. [1]

MCP is described by its authors as a "USB-C port for AI applications": just as USB-C provides a standardized way to connect electronic devices to peripherals, MCP provides a standardized way for AI applications to connect to external systems. The protocol is supported across a wide range of clients (Claude, ChatGPT, Visual Studio Code, Cursor) and a growing ecosystem of community-built servers. [1]

## Core Concepts

### Hosts, Clients, and Servers

MCP defines three roles in its architecture:

- **Host**: The AI application the end user interacts with (e.g., Claude desktop, an IDE, a chatbot). The host orchestrates LLM invocations and integrates context from one or more MCP servers.
- **Client**: The MCP protocol implementation living inside the host that opens a connection to a specific server. A host can run multiple clients, one per connected server.
- **Server**: A process that exposes tools, resources, and prompts over MCP. Servers can run locally (stdio transport) or remotely (Streamable HTTP transport). [2]

### Primitives

MCP defines a small fixed set of primitives that clients and servers can offer each other. Server primitives:

- **Tools**: Executable functions an AI application can invoke (file operations, API calls, database queries).
- **Resources**: Data sources that supply contextual information to the model (file contents, database records, API responses).
- **Prompts**: Reusable templates that structure interactions with language models (system prompts, few-shot examples). [2]

Client primitives that servers can request:

- **Sampling**: Lets a server request a language-model completion from the client's host application, keeping the server model-agnostic.
- **Elicitation**: Lets a server request additional information or confirmation from the user.
- **Logging**: Lets a server send log messages to the client for debugging. [2]

Each primitive type has discovery methods (`tools/list`, `resources/list`, `prompts/list`), retrieval methods (`*/get`), and where applicable execution methods (`tools/call`). Listings are dynamic, so a server can change its exposed primitives at runtime and notify connected clients. [2]

### Data Layer and Transport Layer

The protocol separates two concerns:

- **Data layer**: A JSON-RPC 2.0 message protocol that defines lifecycle (`initialize`, capability negotiation, shutdown), primitive operations, and notifications. This is the part developers most often work with.
- **Transport layer**: The mechanism that moves JSON-RPC messages between client and server. Two transports are defined: `stdio` (process pipes for local servers) and `Streamable HTTP` (HTTP POST plus Server-Sent Events for remote servers, with OAuth-based auth support). [2]

The same data-layer messages flow over either transport unchanged.

## Architecture

A typical MCP exchange:

1. Host launches or connects to one or more servers, creating a client per server.
2. Each client and server perform an `initialize` handshake to negotiate protocol version and capabilities.
3. Client calls `tools/list`, `resources/list`, and `prompts/list` to discover what the server offers.
4. When the LLM decides to use a tool, the host translates that into a `tools/call` request to the relevant server.
5. The server may, during a tool call, invoke client-side primitives (e.g., `sampling/createMessage` to ask the host's LLM for a completion, or `elicitation/create` to prompt the user).
6. Servers can push notifications (e.g., `notifications/tools/list_changed`) when their exposed primitives change. [2]

The protocol is stateful: capability negotiation in `initialize` determines which features are available for the rest of the session. A subset of MCP can be made stateless using Streamable HTTP for scenarios that need request-level scaling. [2]

## Key Features and Functionality

### Tools

Tools are the most-used primitive: typed callable functions exposed by a server. Each tool has a name, description, JSON-schema input definition, and (optionally) output schema. Hosts present tools to the LLM as function-calling targets; the LLM emits a tool-use request and the host forwards it to the appropriate MCP server as `tools/call`. [2]

### Resources

Resources expose read-only data — URIs that the host or model can fetch as context (`resources/read`). Use cases include exposing a project's source files, a database schema, or the contents of an external knowledge base.

### Prompts

Prompts let a server ship pre-authored interaction templates (e.g., "review this PR following our team checklist"). The host surfaces them in a UI; the user picks one; the host substitutes parameters and sends the resulting message to the model.

### Sampling

Sampling inverts the relationship: a server asks the host for an LLM completion. This is useful when the server logic itself needs LLM reasoning (e.g., to summarize a retrieved document) but the server wants to remain model-independent rather than bundling its own LLM client. [2]

### Notifications

Servers and clients exchange JSON-RPC notifications (one-way, no response) for events: a tool list changed, a resource was updated, progress on a long-running call. This avoids polling and keeps state consistent. [2]

## SDKs

Official SDKs cover the major languages:

- **Python**: `mcp` package on PyPI
- **TypeScript**: `@modelcontextprotocol/sdk`
- **C#, Java, Kotlin, Go, Ruby, Swift**: official maintained packages
- **Rust**: community-maintained [3]

SDKs handle JSON-RPC framing, lifecycle negotiation, and primitive-handler registration so server authors mostly write business logic (tool implementations) and host authors write client-side glue.

## Use Cases

### IDE Assistants

VS Code, Cursor, and similar editors run MCP servers for file-system access, language-server queries, and custom team tooling. The assistant can read files, run tests, and call internal APIs through a uniform interface rather than bespoke IDE-specific integrations. [1]

### Enterprise Chatbots

A single assistant can connect to multiple internal databases, ticketing systems, and document stores through MCP servers, letting end-users run cross-system queries via chat. [1]

### Personal Agents

Agents can connect to Google Calendar, Notion, Gmail, and similar consumer services through publicly available MCP servers, acting as a more personalized assistant without each app shipping its own integration SDK. [1]

### Workflow Automation

Long-running multi-step workflows can be wrapped in MCP tools that an agent invokes via `tools/call`. The experimental Tasks primitive adds durable-execution semantics for results that are not immediately available. [2]

## Configuration

### Connection Setup (stdio, local server)

A host's configuration typically lists each server it should launch, with a command, arguments, and environment variables. Example shape:

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/dir"]
    }
  }
}
```

The host spawns the process, opens stdin/stdout pipes, and runs the MCP handshake.

### Connection Setup (Streamable HTTP, remote server)

Remote servers are addressed by URL. Auth typically uses OAuth 2.0 to obtain a bearer token, which the client attaches as an `Authorization` header on the HTTP POST that carries each JSON-RPC request. [2]

### Capability Negotiation

During `initialize`, client and server each declare which optional capabilities they support (e.g., a client may declare it supports `sampling`; a server may declare it supports `tools` and `prompts` but not `resources`). The intersection determines what subsequent requests are valid. [2]

## Integration Patterns

### With Agent Frameworks (LangChain, LangGraph, CrewAI, OpenAI Agents SDK)

Agent frameworks wrap MCP clients so that any MCP-exposed tool becomes a tool the agent can call. This means a single MCP server can serve agents written in any framework.

### With Hosted LLM Providers (Claude, ChatGPT)

Claude's Messages API supports MCP connectors directly, letting a remote MCP server's tools be passed to the model without writing a separate client. ChatGPT, VS Code, and Cursor offer similar built-in MCP support. [1]

### With Internal APIs

The most common production pattern is to wrap an existing REST/GraphQL service with a thin MCP server that exposes a curated subset of operations as tools. This avoids re-implementing the API in each agent framework and keeps authorization centralized.

## Examples

### Minimal Python Server

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("weather")

@mcp.tool()
def get_weather(location: str) -> str:
    """Return the current weather for a location."""
    # ... call your weather API ...
    return f"It's 72°F and sunny in {location}."

if __name__ == "__main__":
    mcp.run()
```

The `FastMCP` decorator pattern handles tool registration, JSON-schema generation, and the stdio transport. [3]

### Minimal TypeScript Client

```ts
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";

const client = new Client({ name: "demo", version: "0.1.0" });
await client.connect(new StdioClientTransport({
  command: "node",
  args: ["./server.js"],
}));
const tools = await client.listTools();
const result = await client.callTool({ name: "get_weather", arguments: { location: "SF" } });
```

## Limitations and Considerations

- **Stateful protocol**: Sessions need explicit lifecycle handling. Streamable HTTP supports a stateless mode but with feature trade-offs.
- **Authorization is transport-specific**: stdio assumes the host trusts the local process; Streamable HTTP relies on OAuth and bearer tokens. Tool-call authorization beyond connection-level auth (e.g., per-tool ACLs) is the host's responsibility.
- **Tool descriptions are LLM-facing**: Poorly written descriptions or schemas degrade tool-call accuracy. Treat them as prompts.
- **Sampling adds a callback loop**: When a server uses sampling, the host's LLM is invoked recursively — easy to create latency and cost spikes if used carelessly.
- **Ecosystem is young**: The protocol is stable but auxiliary tooling (debuggers, registries, marketplaces) is still maturing. The MCP Inspector is the canonical debugging tool. [4]

## Specification and Versioning

The specification is versioned by date (e.g., `2025-11-25`). The current spec is published at `modelcontextprotocol.io/specification/<date>`. Backwards-incompatible changes go through the SEP (Specification Enhancement Proposal) process. [5]

## Citations

- [1] What is the Model Context Protocol? - <https://modelcontextprotocol.io/docs/getting-started/intro>
- [2] Architecture Overview - <https://modelcontextprotocol.io/docs/learn/architecture>
- [3] SDKs - <https://modelcontextprotocol.io/docs/sdk>
- [4] MCP Inspector - <https://modelcontextprotocol.io/docs/tools/inspector>
- [5] MCP Specification - <https://modelcontextprotocol.io/specification/2025-11-25>

## Discovery Signals

> Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search.

### Keywords

- model context protocol
- MCP
- tool protocol
- JSON-RPC
- stdio transport
- streamable HTTP
- MCP server
- MCP client
- MCP host
- tools primitive
- resources primitive
- prompts primitive
- sampling
- elicitation
- FastMCP
- MCP Inspector
- USB-C for AI
- OAuth bearer token
- capability negotiation
- tool calling standard
- agent-to-tool boundary
- Claude MCP connector
- Cursor MCP
- VS Code MCP
- function calling interop

### Verb-Noun Tasks

- Expose a tool to LLMs via an MCP server
- Connect Claude or ChatGPT to a custom data source
- Wrap a REST API as an MCP server
- Discover available tools with `tools/list`
- Invoke a remote tool with `tools/call`
- Stream prompts and resources from a local filesystem
- Authenticate an HTTP MCP server with OAuth
- Implement sampling so a server can ask the host LLM for completions
- Elicit additional input from the user mid-tool-call
- Build a Python MCP server with FastMCP decorators
- Debug an MCP connection with MCP Inspector
- Negotiate capabilities during the `initialize` handshake

### User Intent Phrases

- How do I let Claude desktop access my local files and databases?
- What is the standard protocol for AI tool integration?
- How can I make one tool work across Claude, ChatGPT, and Cursor?
- I need to expose my internal API to an AI assistant
- How do I write a server that an IDE assistant can call?
- What is the difference between MCP tools, resources, and prompts?
- How do I stream tool output back to an LLM client?
- How can a tool ask the user a clarifying question mid-call?
- What replaces bespoke per-IDE integrations for AI features?
- How do I add OAuth to a remote MCP server?
- Show me a minimal MCP server in Python or TypeScript

### Problem Statements

- Every AI host needs its own bespoke tool integration today
- Custom function-calling glue does not transfer between Claude, ChatGPT, and IDEs
- Sampling adds a recursive LLM call loop that can spike cost and latency
- Tool descriptions double as prompts; poor schemas degrade accuracy
- stdio assumes trusted local processes; remote servers need OAuth
- Per-tool authorization beyond connection-level auth must be enforced by the host
- Tooling for debugging and registries is still maturing

### When to Pick This

- Pick MCP when your problem is the agent-to-tool boundary (not agent-to-agent — that is A2A)
- Pick MCP when you want one tool/data integration usable by every supporting host (Claude, ChatGPT, Cursor, VS Code)
- Pick MCP when an LLM needs to read files, query databases, or call internal APIs through a uniform protocol
- Pick MCP when you want to ship reusable prompt templates and read-only resources alongside tools
- Pick MCP when the server should remain model-agnostic but still leverage LLM reasoning via sampling
- Pick A2A instead when you need agents built by different vendors to delegate tasks to each other as opaque peers

### Related Terms and Aliases

- Model Context Protocol
- MCP connector
- Anthropic MCP
- JSON-RPC 2.0 over stdio
- Streamable HTTP transport
- Server-Sent Events transport
- SEP (Specification Enhancement Proposal)
- tool-use protocol
- function calling interop layer
