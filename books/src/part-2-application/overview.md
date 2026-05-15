# Part II — Application Development

This part covers the application layer — the frameworks, protocols, and runtimes developers use to compose LLM-powered products on top of the foundation layer. Six groups span agent construction, agent-to-world interoperability, isolated execution surfaces, and the building blocks that make agentic apps reliable.

## What's in this part

- **Agent Frameworks** — libraries for building autonomous AI agents (LangChain, LangGraph, AutoGen, CrewAI, ADK, Semantic Kernel, smolagents, Pydantic AI)
- **Agent Protocols** — open standards for tool-and-agent interoperability (MCP, A2A)
- **Agent Runtimes & Sandboxes** — isolated execution environments for agent code and tools (E2B)
- **Browser Automation** — managed headless browser platforms for web-acting agents (Browserbase)
- **Structured Generation** — constrained output and schema-typed extraction (DSPy, Outlines, Instructor, BAML)
- **Memory Systems** — persistent memory and context management for agents (Mem0, Zep, Letta)

## How to navigate this part

The eight **Agent Frameworks** are the most-debated category in the modern AI stack; the chapters there cover what each framework's distinctive philosophy is (LangGraph's checkpoint-graphs, CrewAI's role-based crews, AutoGen's conversational patterns, ADK's hierarchical agent trees, etc.). Pick the framework that matches your team's mental model — the underlying capabilities are largely equivalent.

**Agent Protocols** (MCP, A2A) and **Agent Runtimes & Sandboxes** (E2B) are the connective tissue around the frameworks. MCP standardizes how agents reach external tools and data; A2A standardizes how agents reach other agents; E2B provides the safe Linux sandbox where an agent can actually execute the code it writes. **Browser Automation** (Browserbase) is the same idea for the browser surface — a managed Chromium that agents can drive.

**Structured Generation** and **Memory Systems** are framework-agnostic capabilities that pair with any of the above. Use structured generation when you need the LLM's output to conform to a typed schema. Use memory systems when an agent needs context that survives across sessions.
