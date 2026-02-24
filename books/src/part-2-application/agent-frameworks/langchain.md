# LangChain

> Open source framework for building LLM-powered applications with chains, agents, and retrieval-augmented generation, serving as the platform for agent engineering.

| Field         | Value                                                        |
|---------------|--------------------------------------------------------------|
| Group         | Agent Frameworks                                             |
| Type          | SDK                                                          |
| Open Source   | Yes                                                          |
| GitHub        | [langchain-ai/langchain](https://github.com/langchain-ai/langchain) |
| Stars         | 127162                                                       |
| Documentation | [Official Docs](https://docs.langchain.com/)                |

## Overview

LangChain is an open source framework for building applications powered by large language models. Positioned as "the platform for agent engineering," it provides composable abstractions for connecting LLMs to external data sources, tools, and APIs. The framework is trusted by organizations including Replit, Clay, Rippling, Cloudflare, and Workday, and is available in both Python and TypeScript.

LangChain spans three tiers of abstraction. At the highest level, the LangChain package provides pre-built agent architectures and integrations for rapid development. LangGraph offers low-level orchestration for teams that need advanced customization and stateful workflows. Deep Agents provide batteries-included functionality with conversation compression and subagent spawning for long-running, complex tasks.

## Core Concepts

**Chat Models** provide a standardized interface across LLM providers. Whether using OpenAI, Anthropic, Google, or other providers, the same abstraction layer handles prompt formatting, token counting, and response parsing, allowing applications to swap providers without rewriting business logic.

**Agents** are autonomous applications that decide which tools to invoke and in what order. LangChain agents support durable execution, streaming responses, human-in-the-loop approval workflows, and persistence of conversation state across sessions.

**Tools** are user-defined functions that agents can invoke to interact with the outside world. Functions are decorated with `@tool` to expose their name, description, and parameter schema to the agent, which decides when and how to call them.

**LangChain Expression Language (LCEL)** is the declarative composition system that replaced legacy chains. LCEL uses a pipe operator to compose runnables into sequences, enabling streaming, batching, and async execution with minimal boilerplate. Legacy chains (sequential LLM calls and tool invocations) are still supported but no longer the recommended pattern.

**Prompts** are constructed using template classes such as `ChatPromptTemplate`, `SystemMessage`, and `HumanMessage`. Templates support variable interpolation, few-shot examples, and dynamic prompt assembly based on runtime context.

**Output Parsers** structure raw LLM text into typed data. Parsers for JSON, Pydantic models, comma-separated lists, and custom formats transform unstructured completions into objects that downstream code can consume reliably.

**Document Loaders** ingest content from diverse sources including PDFs, web pages, databases, and file systems, converting them into LangChain's `Document` abstraction with content and metadata fields.

**Retrievers** search and return relevant documents for Retrieval-Augmented Generation (RAG). They abstract over vector stores, keyword search, and hybrid approaches, providing a uniform interface for context injection into prompts.

**Vector Stores** manage embeddings for semantic search. Integrations include FAISS, Chroma, Pinecone, Weaviate, and dozens of other backends, each accessible through the same query interface.

**Memory** manages conversation history across turns. Strategies include buffer memory (full history), summary memory (compressed history), and entity memory (tracked facts), each trading off context window usage against information retention.

**Callbacks** provide hooks for logging, monitoring, streaming, and custom side effects at every stage of chain or agent execution.

## Installation and Setup

Install the core package and provider-specific integrations:

```bash
pip install langchain
pip install langchain-openai       # OpenAI models
pip install langchain-anthropic    # Anthropic models
pip install langchain-community    # Community integrations
```

Set the API key for your chosen provider:

```bash
export OPENAI_API_KEY="sk-..."
# or
export ANTHROPIC_API_KEY="sk-ant-..."
```

Verify the installation:

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4o")
response = llm.invoke("Hello, world!")
print(response.content)
```

## Architecture

LangChain is organized into layered packages with increasing specificity:

```
langchain-core          Base abstractions (Runnables, ChatModels, Messages)
    |
langchain               Chains, agents, retrieval strategies
    |
langchain-community     Third-party integrations (vector stores, loaders, tools)
    |
Provider packages       langchain-openai, langchain-anthropic, langchain-google, etc.
```

**langchain-core** defines the fundamental interfaces: `Runnable`, `BaseChatModel`, `BaseRetriever`, `BaseTool`, and the message types. All higher-level packages depend on core but core depends on nothing else.

**langchain** builds on core to provide agent executors, chain constructors, retrieval pipelines, and the LCEL composition framework.

**langchain-community** houses third-party integrations that are maintained by the community. These include document loaders, vector stores, embedding providers, and tool wrappers.

**Provider packages** (such as `langchain-openai` and `langchain-anthropic`) are first-party integrations maintained alongside the providers themselves, offering tighter version coupling and faster updates.

## Key Features and Functionality

**Agent Construction**: Build agents with tool-calling capabilities in minimal code. Agents autonomously select and invoke tools based on user input and conversation context.

```python
from langchain.agents import create_agent

agent = create_agent(
    model="gpt-4o",
    tools=[get_weather],
    prompt="You are a helpful assistant."
)
result = agent.invoke({"input": "What's the weather in SF?"})
```

**LCEL Composition**: Declaratively compose processing pipelines using the pipe operator:

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI
from langchain_core.output_parsers import StrOutputParser

prompt = ChatPromptTemplate.from_template("Tell me a joke about {topic}")
model = ChatOpenAI(model="gpt-4o")
parser = StrOutputParser()

chain = prompt | model | parser
result = chain.invoke({"topic": "programming"})
```

**Retrieval-Augmented Generation (RAG)**: Combine document loaders, embedding models, vector stores, and retrievers to ground LLM responses in domain-specific knowledge.

**Streaming**: First-class streaming support across the entire stack. Agents, chains, and individual model calls can stream tokens as they are generated.

**Human-in-the-Loop**: Interrupt agent execution to request human approval before sensitive tool invocations, then resume execution with the decision.

**Persistence**: Save and restore agent state across sessions, enabling long-running workflows that survive process restarts.

**LangSmith Observability**: Integrated tracing provides detailed visibility into every step of chain and agent execution. LangSmith offers aggregate metrics, evaluation frameworks, prompt engineering tools, and deployment pipelines. The platform is HIPAA, SOC 2 Type 2, and GDPR compliant.

## Use Cases

**Conversational Assistants**: Customer support bots, internal knowledge assistants, and interactive help systems that maintain context across multi-turn conversations.

**RAG Applications**: Question-answering systems that retrieve relevant documents from a knowledge base before generating responses, reducing hallucination and grounding answers in source material.

**Data Analysis Agents**: Agents that query databases, execute code, and synthesize results in response to natural language questions about structured data.

**Document Processing Pipelines**: Automated extraction, summarization, and classification of documents from heterogeneous sources.

**Multi-Agent Workflows**: Complex task decomposition where specialized agents collaborate through LangGraph orchestration, each handling a distinct subtask.

**Tool-Augmented Applications**: Applications where LLMs interact with external APIs, calculators, search engines, and custom business logic through the tool abstraction.

## API Reference Summary

**Chat Models**: `ChatOpenAI`, `ChatAnthropic`, `ChatGoogleGenerativeAI` -- standardized `invoke()`, `stream()`, `batch()` methods across all providers.

**Prompts**: `ChatPromptTemplate.from_messages()`, `SystemMessage`, `HumanMessage`, `AIMessage` -- template construction and message type definitions.

**Agents**: `create_agent()`, `AgentExecutor` -- agent creation with tool binding and execution loop management.

**Tools**: `@tool` decorator, `StructuredTool.from_function()` -- function-to-tool conversion with automatic schema generation.

**Retrievers**: `VectorStoreRetriever`, `MultiQueryRetriever`, `EnsembleRetriever` -- document retrieval strategies with configurable search parameters.

**Output Parsers**: `JsonOutputParser`, `PydanticOutputParser`, `StrOutputParser` -- structured output extraction from LLM completions.

**Document Loaders**: `PyPDFLoader`, `WebBaseLoader`, `CSVLoader`, `DirectoryLoader` -- content ingestion from diverse sources.

**Vector Stores**: `FAISS`, `Chroma`, `Pinecone`, `Weaviate` -- embedding storage and similarity search backends.

## Configuration and Customization

**Model Parameters**: Configure temperature, max tokens, top-p, and other generation parameters per model instance:

```python
llm = ChatOpenAI(
    model="gpt-4o",
    temperature=0.0,
    max_tokens=1024,
)
```

**Custom Tools**: Define tools with typed parameters and descriptions:

```python
from langchain_core.tools import tool

@tool
def search_database(query: str, limit: int = 10) -> str:
    """Search the product database for items matching the query."""
    # Implementation here
    return results
```

**Custom Retrievers**: Implement the `BaseRetriever` interface to create domain-specific retrieval logic that integrates with any LangChain pipeline.

**Callback Handlers**: Attach custom handlers to monitor token usage, log intermediate steps, or implement rate limiting:

```python
from langchain_core.callbacks import BaseCallbackHandler

class TokenCounter(BaseCallbackHandler):
    def on_llm_end(self, response, **kwargs):
        # Track token usage
        pass
```

## Integration Patterns

**Provider Swapping**: The standardized chat model interface allows switching between OpenAI, Anthropic, Google, and other providers by changing a single import and model name, with no changes to application logic.

**LangSmith Tracing**: Enable observability by setting environment variables:

```bash
export LANGSMITH_TRACING=true
export LANGSMITH_API_KEY="lsv2_..."
```

**LangGraph Orchestration**: For workflows requiring cycles, conditional branching, or persistent state, LangGraph extends LangChain with graph-based agent orchestration.

**Third-Party Tool Integration**: LangChain's tool abstraction wraps external APIs (search engines, databases, SaaS platforms) into a format that agents can discover and invoke autonomously.

**Embedding Pipeline**: Combine document loaders, text splitters, embedding models, and vector stores into an ingestion pipeline that prepares knowledge bases for RAG.

## Examples

**Basic Chat with Memory**:

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.messages import HumanMessage

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant."),
    MessagesPlaceholder("history"),
    ("human", "{input}"),
])

chain = prompt | ChatOpenAI(model="gpt-4o")
```

**RAG Pipeline**:

```python
from langchain_community.document_loaders import PyPDFLoader
from langchain_openai import OpenAIEmbeddings
from langchain_community.vectorstores import FAISS
from langchain.text_splitter import RecursiveCharacterTextSplitter

loader = PyPDFLoader("document.pdf")
docs = loader.load()

splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=200)
splits = splitter.split_documents(docs)

vectorstore = FAISS.from_documents(splits, OpenAIEmbeddings())
retriever = vectorstore.as_retriever(search_kwargs={"k": 4})
```

**Tool-Calling Agent**:

```python
from langchain_core.tools import tool
from langchain_openai import ChatOpenAI

@tool
def get_weather(city: str) -> str:
    """Get the current weather for a city."""
    return f"72F and sunny in {city}"

llm = ChatOpenAI(model="gpt-4o").bind_tools([get_weather])
response = llm.invoke("What's the weather in San Francisco?")
```

## Limitations and Considerations

**Abstraction Overhead**: The multiple layers of abstraction can obscure what is happening at the LLM API level, making debugging more difficult for developers who need fine-grained control over request and response handling.

**Rapid API Changes**: LangChain's API surface has evolved significantly across versions. Code written against older versions (pre-0.2) may require substantial refactoring to align with current patterns, particularly the shift from legacy chains to LCEL.

**Package Fragmentation**: The split across langchain-core, langchain, langchain-community, and provider packages means dependency management requires attention. Version mismatches between packages can cause runtime errors.

**Latency**: The abstraction layers add overhead compared to direct API calls. For latency-sensitive applications, the additional processing time from routing through chains, parsers, and callbacks should be measured and evaluated.

**Memory Management**: Built-in memory strategies have limitations at scale. Applications with very long conversation histories or high-concurrency requirements may need custom memory implementations beyond what the framework provides out of the box.

**Learning Curve**: The breadth of the framework (agents, chains, LCEL, retrievers, vector stores, callbacks) presents a steep learning curve. Developers must understand which abstractions to use for their specific use case.

## Changelog Highlights

- **LangGraph introduction**: Low-level orchestration framework for stateful, multi-step agent workflows with cycles and conditional branching.
- **Deep Agents**: Batteries-included agents with conversation compression and subagent spawning for complex, long-running tasks.
- **LCEL adoption**: Declarative chain composition replaced legacy sequential chains as the recommended pattern.
- **Provider package split**: First-party provider integrations (OpenAI, Anthropic, Google) moved to dedicated packages for independent versioning.
- **LangSmith integration**: Observability platform with tracing, evaluation, and deployment capabilities, achieving HIPAA, SOC 2 Type 2, and GDPR compliance.

## Citations

- [1] LangChain Documentation - https://docs.langchain.com/
- [2] LangChain Python - https://docs.langchain.com/oss/python/langchain/overview
