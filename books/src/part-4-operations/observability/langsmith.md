# LangSmith

> LLM observability platform for tracing and evaluation with dashboard

| Field | Value |
|-------|-------|
| Group | Observability |
| Type | API/SDK/UI |
| Open Source | No |
| GitHub | N/A |
| Stars | N/A |
| Documentation | [Official Docs](https://docs.langchain.com/langsmith) |

## Overview

LangSmith is a framework-agnostic platform for developing, debugging, and deploying AI agents and Large Language Model (LLM) applications. Built by the LangChain team, it provides end-to-end observability, evaluation, prompt management, and deployment capabilities through a unified web dashboard and programmatic Software Development Kits (SDKs) in Python, TypeScript, and Java.

Unlike open-source alternatives that focus on a single concern (tracing or evaluation), LangSmith combines the entire LLM operations lifecycle into one commercial platform. Developers instrument their applications with lightweight SDK wrappers and decorators, and all execution data flows into the LangSmith dashboard where it can be inspected, evaluated against datasets, and used to iterate on prompts. The platform is framework-agnostic, supporting OpenAI, Anthropic, CrewAI, Vercel AI SDK, Pydantic AI, LangChain, LangGraph, and any custom LLM integration.

LangSmith operates as a managed cloud service at smith.langchain.com, with self-hosted and hybrid deployment options available for organizations with compliance or data residency requirements. The platform maintains Health Insurance Portability and Accountability Act (HIPAA), Service Organization Control 2 (SOC 2) Type 2, and General Data Protection Regulation (GDPR) compliance.

## Core Concepts

**Traces** are the top-level records of a complete request flowing through an LLM application. A trace captures the entire execution path from input to final output, providing a hierarchical view of every operation that occurred during processing. Each trace belongs to a project within a workspace.

**Runs** are the individual operations within a trace. A run represents a single unit of work such as an LLM call, a tool invocation, a retrieval step, or a custom function execution. Runs nest hierarchically within traces, forming parent-child relationships that reflect the call structure of the application. Each run captures its inputs, outputs, timing, token usage, and metadata.

**Projects** organize traces into logical groupings. By default, traces flow into a default project, but developers can route traces to specific projects using the `LANGSMITH_PROJECT` environment variable. Projects enable separation between development, staging, and production environments.

**Datasets** are collections of input-output pairs used for evaluation. Each dataset contains examples with inputs and optional reference outputs (ground truth). Datasets serve as the foundation for systematic evaluation, enabling repeatable experiments across different model configurations, prompts, or application versions.

**Experiments** are evaluation runs that execute a target function against a dataset and score the results using evaluators. Experiments produce metrics that can be compared across runs, enabling data-driven decisions about which configuration performs best.

**Evaluators** are functions that score the outputs of a target function. LangSmith supports custom code evaluators, LLM-as-judge evaluators (where another LLM grades the output), and pre-built evaluators for common patterns like correctness, conciseness, and hallucination detection.

**Prompts** are versioned templates managed through the LangSmith Prompt Hub. Prompts can be created, iterated, shared, and deployed collaboratively. Each prompt revision is tracked, enabling teams to roll back to previous versions and audit changes over time.

**Feedback** captures human or automated quality signals attached to individual runs. Feedback can be scores, labels, or free-text comments. It feeds into evaluation metrics and can trigger automation rules for quality monitoring.

**Annotation Queues** organize runs that require human review. Queues enable systematic human evaluation workflows where domain experts review, label, and provide feedback on application outputs at scale.

## Architecture

LangSmith follows a client-server architecture where lightweight SDK instrumentation in the application sends telemetry data to the LangSmith backend for storage, visualization, and analysis.

1. **SDK instrumentation** wraps LLM client calls and application functions with tracing decorators or wrappers. The SDK captures inputs, outputs, timing, token counts, and metadata with minimal overhead.
2. **Trace collection** sends run data asynchronously to the LangSmith API. The SDK batches and transmits traces in the background to avoid impacting application latency.
3. **Backend storage** persists trace data, datasets, experiments, prompts, and feedback in the LangSmith platform. Data is organized by workspace, project, and time range.
4. **Dashboard visualization** provides the web interface for exploring traces, comparing experiments, managing prompts, configuring automation rules, and reviewing annotation queues.
5. **Evaluation engine** executes target functions against datasets, applies evaluators, and produces experiment results with aggregate and per-example metrics.

The architecture supports three deployment topologies:

- **Managed cloud**: LangSmith hosts everything; traces are sent to `api.smith.langchain.com`
- **Self-hosted**: The entire LangSmith stack runs within an organization's own infrastructure, keeping all data on-premise
- **Hybrid**: The application and trace data remain on-premise while certain management features use the cloud control plane

LangSmith integrates with the broader LangChain ecosystem but does not require it. Any application that can make HTTP calls or use the LangSmith SDK can send traces, regardless of whether it uses LangChain, LangGraph, or any other framework.

## Key Features and Functionality

**Observability and Tracing** provides full visibility into LLM application execution. Every LLM call, tool invocation, retrieval step, and custom function is captured as a run within a trace. The dashboard renders traces as interactive hierarchical trees, showing inputs, outputs, latency, token usage, cost, and error states at each level. Developers can filter, search, and drill into traces to diagnose issues.

**Evaluation Framework** enables systematic measurement of application quality. Developers define datasets with input-output pairs, write evaluator functions (custom code or LLM-as-judge), and run experiments that produce comparable metrics. The framework supports row-level evaluators that score individual examples and summary evaluators that produce aggregate statistics across the entire dataset.

**Prompt Management** through the Prompt Hub provides versioned prompt storage with collaboration features. Prompts can be created in the visual Playground, tested against different models and parameters, shared across teams, and pulled into application code programmatically. Every revision is tracked for auditability.

**Studio** is a visual interface for designing, testing, and refining LLM applications interactively. It provides a playground for experimenting with prompts, models, and parameters without writing code, enabling rapid iteration on application behavior.

**Agent Builder** offers a no-code visual interface for designing and deploying AI agents. It allows non-technical users to construct agent workflows, define tool usage, and deploy agents to production without programming.

**Online Evaluation and Automation** rules monitor production traces in real time. Rules can trigger LLM-as-judge evaluators, flag anomalous traces, route runs to annotation queues, or send alerts based on configurable conditions. This enables continuous quality monitoring without manual review of every trace.

**Annotation Queues** support systematic human evaluation. Runs matching certain criteria are routed to queues where human reviewers provide feedback, labels, and corrections. This human-in-the-loop workflow feeds back into dataset creation and model improvement.

**Cost and Token Tracking** aggregates token usage and estimated costs across all traced runs. Developers can monitor spending by project, model, or time period, enabling budget management and cost optimization.

## Use Cases

- **Debugging LLM applications**: Trace execution paths to identify where an application produces incorrect or unexpected outputs, inspecting each step from input through retrieval, prompting, and response generation
- **Regression testing**: Run evaluation experiments against golden datasets before deploying changes, comparing new experiment results against baseline metrics to catch quality regressions
- **Prompt engineering**: Iterate on prompts using the Playground and Prompt Hub, testing variations against datasets and comparing results across experiments to find the optimal prompt formulation
- **Production monitoring**: Attach automation rules to production projects that evaluate a sample of traces with LLM-as-judge evaluators, flagging quality degradation or anomalous behavior for human review
- **Human evaluation workflows**: Route production traces to annotation queues where domain experts review outputs, provide feedback, and curate examples for future evaluation datasets
- **Multi-model comparison**: Run the same dataset against different models or configurations in separate experiments, then compare results side-by-side to select the best-performing option
- **Agent observability**: Trace multi-step agent workflows built with LangGraph, CrewAI, or custom frameworks, visualizing tool calls, reasoning steps, and decision points across the entire execution

## API Reference Summary

**Client Initialization**:

- `Client(api_key=None)` -- Create a LangSmith client (Python); reads `LANGSMITH_API_KEY` from environment if not provided
- `new Client({ apiKey })` -- Create a LangSmith client (TypeScript)

**Tracing**:

- `@traceable` -- Python decorator that creates a traced run for the decorated function
- `@traceable(run_type="tool", name="...")` -- Decorator with explicit run type and display name
- `traceable(fn, { name, run_type })` -- TypeScript function wrapper for creating traced runs
- `wrap_openai(client)` -- Python wrapper that instruments an OpenAI client for automatic tracing
- `wrapOpenAI(client)` -- TypeScript wrapper for OpenAI client instrumentation

**Datasets**:

- `client.create_dataset(name, description)` -- Create a new evaluation dataset
- `client.create_examples(dataset_id, examples)` -- Add input-output examples to a dataset
- `client.clone_public_dataset(url)` -- Clone a publicly shared dataset into the workspace

**Evaluation**:

- `client.evaluate(target, data, evaluators, experiment_prefix)` -- Run an evaluation experiment against a dataset with specified evaluators
- `evaluate(target, { data, evaluators, experimentPrefix })` -- TypeScript evaluation function
- Custom evaluator signature: `def evaluator(inputs, outputs, reference_outputs) -> bool | float | dict`

**Prompts**:

- `client.list_prompts(query, is_public)` -- List prompts with optional filtering
- `client.delete_prompt(name)` -- Delete a prompt by name
- `client.like_prompt(handle)` -- Like a public prompt
- `client.unlike_prompt(handle)` -- Remove a like from a prompt

**REST API**:

- `POST /api/v1/datasets/upload-experiment` -- Upload externally-run experiment results to a dataset
- Base URL: `https://api.smith.langchain.com` (managed cloud)

## Configuration and Customization

LangSmith configuration is primarily driven by environment variables and SDK parameters.

### Environment Variables

```bash
# Required for tracing
export LANGSMITH_TRACING=true
export LANGSMITH_API_KEY="lsv2_..."

# Optional configuration
export LANGSMITH_PROJECT="my-project"          # Route traces to a named project
export LANGSMITH_WORKSPACE_ID="<workspace-id>" # Target a specific workspace
export LANGSMITH_ENDPOINT="https://api.smith.langchain.com"  # API endpoint (override for self-hosted)
export LANGSMITH_OTEL_ENABLED=true             # Enable OpenTelemetry integration
```

### Trace Sampling and Filtering

For high-throughput production applications, tracing can be configured to sample a percentage of requests rather than tracing every call. This reduces overhead and storage costs while still providing statistical visibility into application behavior.

### Project Organization

Traces are organized into projects. Use separate projects for development, staging, and production to isolate concerns:

```bash
# Development
export LANGSMITH_PROJECT="my-app-dev"

# Production
export LANGSMITH_PROJECT="my-app-prod"
```

### Evaluator Configuration

Custom evaluators can be defined as simple functions returning boolean, numeric, or dictionary results:

```python
def correctness(inputs: dict, outputs: dict, reference_outputs: dict) -> bool:
    return outputs["answer"].strip().lower() == reference_outputs["answer"].strip().lower()

def relevance_score(inputs: dict, outputs: dict, reference_outputs: dict) -> float:
    # Return a score between 0 and 1
    return 0.85
```

LLM-as-judge evaluators delegate scoring to another LLM:

```python
from openevals import create_llm_as_judge, CORRECTNESS_PROMPT

correctness_evaluator = create_llm_as_judge(
    prompt=CORRECTNESS_PROMPT,
    model="gpt-4o-mini",
    feedback_key="correctness",
)
```

## Integration Patterns

**OpenAI Integration**: The `wrap_openai` wrapper instruments the OpenAI client to automatically capture all chat completion and embedding calls as traced runs. This is the lowest-friction integration path:

```python
import openai
from langsmith.wrappers import wrap_openai

client = wrap_openai(openai.Client())

# All calls through this client are automatically traced
response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Hello"}],
)
```

**LangChain and LangGraph Integration**: When LangSmith tracing is enabled via environment variables, LangChain and LangGraph automatically send traces without any additional instrumentation. Every chain invocation, agent step, and tool call is captured:

```python
from langchain_openai import ChatOpenAI
from langgraph.graph import StateGraph

# With LANGSMITH_TRACING=true, all LangChain/LangGraph
# operations are automatically traced
llm = ChatOpenAI(model="gpt-4o-mini")
```

**Anthropic Integration**: Use the `@traceable` decorator to wrap functions that call the Anthropic SDK:

```python
import anthropic
from langsmith import traceable

client = anthropic.Anthropic()

@traceable(name="Claude Call")
def ask_claude(question: str) -> str:
    message = client.messages.create(
        model="claude-sonnet-4-20250514",
        max_tokens=1024,
        messages=[{"role": "user", "content": question}],
    )
    return message.content[0].text
```

**Vercel AI SDK Integration**: In TypeScript applications using the Vercel AI SDK, enable tracing with the OpenTelemetry (OTel) flag:

```bash
export LANGSMITH_TRACING=true
export LANGSMITH_API_KEY="lsv2_..."
export LANGSMITH_OTEL_ENABLED=true
```

**Framework-Agnostic Custom Tracing**: Any function can be traced regardless of the LLM framework used:

```python
from langsmith import traceable

@traceable(run_type="retriever", name="Vector Search")
def search_documents(query: str) -> list[str]:
    # Custom retrieval logic using any vector store
    return ["Document 1 content", "Document 2 content"]

@traceable(run_type="chain", name="RAG Pipeline")
def rag_pipeline(question: str) -> str:
    docs = search_documents(question)
    # Custom LLM call using any provider
    return generate_answer(question, docs)
```

## Examples

**End-to-end Retrieval-Augmented Generation (RAG) application with tracing**:

```python
from openai import OpenAI
from langsmith.wrappers import wrap_openai
from langsmith import traceable

client = wrap_openai(OpenAI())

@traceable(run_type="retriever")
def retriever(query: str) -> list[str]:
    return ["Harrison worked at Kensho"]

@traceable
def rag(question: str) -> str:
    docs = retriever(question)
    system_message = (
        "Answer the user's question using only the provided information below:\n"
        + "\n".join(docs)
    )
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {"role": "system", "content": system_message},
            {"role": "user", "content": question},
        ],
    )
    return response.choices[0].message.content

if __name__ == "__main__":
    print(rag("Where did Harrison work?"))
```

**Creating a dataset and running an evaluation experiment**:

```python
from langsmith import Client
from langsmith.wrappers import wrap_openai
import openai

ls_client = Client()
openai_client = wrap_openai(openai.OpenAI())

# Create dataset with examples
dataset = ls_client.create_dataset("QA Evaluation", description="Question-answer pairs")
ls_client.create_examples(
    dataset_id=dataset.id,
    examples=[
        {
            "inputs": {"question": "What is LangSmith?"},
            "outputs": {"answer": "A platform for observing and evaluating LLM applications"},
        },
        {
            "inputs": {"question": "What is LangChain?"},
            "outputs": {"answer": "A framework for building LLM applications"},
        },
    ],
)

# Define target function
def target(inputs: dict) -> dict:
    response = openai_client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": inputs["question"]}],
    )
    return {"answer": response.choices[0].message.content}

# Define evaluators
def correctness(inputs: dict, outputs: dict, reference_outputs: dict) -> bool:
    prompt = f"""Grade the answer.
Question: {inputs['question']}
Reference: {reference_outputs['answer']}
Predicted: {outputs['answer']}
Respond with CORRECT or INCORRECT."""
    response = openai_client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": prompt}],
    )
    return response.choices[0].message.content.strip() == "CORRECT"

def conciseness(outputs: dict, reference_outputs: dict) -> bool:
    return len(outputs["answer"]) < 2 * len(reference_outputs["answer"])

# Run evaluation
results = ls_client.evaluate(
    target,
    data="QA Evaluation",
    evaluators=[correctness, conciseness],
    experiment_prefix="gpt-4o-mini-eval",
)
```

**TypeScript tracing with tool calls**:

```typescript
import OpenAI from "openai";
import { wrapOpenAI, traceable } from "langsmith/wrappers";

const client = wrapOpenAI(new OpenAI());

const retrieveContext = traceable(
    async (question: string): Promise<string> => {
        return "LangSmith is an LLM observability platform by LangChain.";
    },
    { name: "Retrieve Context", run_type: "retriever" }
);

const chatPipeline = traceable(
    async (question: string): Promise<string | null> => {
        const context = await retrieveContext(question);
        const response = await client.chat.completions.create({
            model: "gpt-4o-mini",
            messages: [
                {
                    role: "system",
                    content: `Answer based on this context: ${context}`,
                },
                { role: "user", content: question },
            ],
        });
        return response.choices[0].message?.content;
    },
    { name: "Chat Pipeline" }
);

(async () => {
    console.log(await chatPipeline("What is LangSmith?"));
})();
```

**Programmatic prompt management**:

```python
from langsmith import Client

client = Client()

# List all prompts
prompts = client.list_prompts()

# List private prompts matching a query
prompts = client.list_prompts(query="summarize", is_public=False)

# Delete a prompt
client.delete_prompt("old-summarizer")

# Like a public prompt
client.like_prompt("team/production-prompt")
```

## Limitations and Considerations

- **Closed-source platform**: LangSmith is a proprietary commercial product. Unlike open-source alternatives such as Langfuse or Arize Phoenix, the source code is not available for inspection or modification. Organizations must trust the vendor for data handling and platform reliability.
- **Vendor lock-in**: Applications instrumented with LangSmith SDK decorators and wrappers create a dependency on the LangSmith platform. While the `@traceable` decorator is lightweight, switching to a different observability platform requires re-instrumenting the application.
- **Cost at scale**: For high-throughput production applications generating millions of traces, storage and processing costs can become significant. Trace sampling may be necessary, which reduces observability coverage.
- **Network dependency**: In managed cloud mode, all trace data is transmitted over the network to LangSmith servers. This introduces latency sensitivity for the background trace transmission and requires network connectivity. Self-hosted deployments mitigate this but add operational complexity.
- **Evaluation dataset management**: Datasets are stored within LangSmith and managed through the SDK or UI. There is no native integration with version control systems for dataset versioning, though datasets can be exported and imported programmatically.
- **LangChain ecosystem affinity**: While LangSmith is framework-agnostic, the deepest integrations and most seamless experience are with LangChain and LangGraph. Applications using other frameworks require manual instrumentation with decorators and wrappers.
- **Self-hosted complexity**: Self-hosted deployments require managing the full LangSmith infrastructure stack, including databases, API servers, and the web dashboard. This demands operational expertise beyond what the managed cloud offering requires.

## Changelog Highlights

LangSmith has evolved from a tracing-focused companion to LangChain into a comprehensive LLM operations platform. Key milestones include the launch of the evaluation framework with datasets and experiments, the Prompt Hub for collaborative prompt management, Studio for visual application design, Agent Builder for no-code agent creation, OpenTelemetry integration for broader ecosystem compatibility, and Agent Server for production deployment of stateful agent workflows. The platform continues to expand its framework-agnostic integrations and compliance certifications.

## Citations

- [1] [LangSmith Documentation](https://docs.langchain.com/langsmith)
- [2] [LangSmith Observability Quickstart](https://docs.langchain.com/langsmith/observability-quickstart)
- [3] [LangSmith Evaluation Quickstart](https://docs.langchain.com/langsmith/evaluation-quickstart)
- [4] [LangSmith Trace with OpenAI](https://docs.langchain.com/langsmith/trace-openai)
- [5] [LangSmith Prompt Management](https://docs.langchain.com/langsmith/manage-prompts-programmatically)
- [6] [LangSmith Upload Experiments API](https://docs.langchain.com/langsmith/upload-existing-experiments)

## Discovery Signals

> Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search.

### Keywords

- LangSmith
- LangChain observability
- LangGraph tracing
- traces
- runs
- projects
- datasets
- experiments
- evaluators
- llm-as-judge
- prompt hub
- annotation queues
- feedback
- @traceable
- wrap_openai
- studio
- agent builder
- automation rules
- managed cloud
- self-hosted
- HIPAA
- SOC 2
- GDPR
- regression testing
- agent observability

### Verb-Noun Tasks

- Trace a LangChain chain or LangGraph agent automatically
- Wrap an OpenAI client for zero-friction tracing
- Decorate any Python function with @traceable
- Run evaluation experiments against a golden dataset
- Compare experiment results across models or prompts
- Manage versioned prompts in the Prompt Hub
- Build agents visually with Agent Builder
- Route production traces to annotation queues
- Configure online LLM-as-judge automation rules
- Aggregate token cost by project and model
- Deploy stateful agents via Agent Server
- Set the LANGSMITH_PROJECT environment variable to organize traces

### User Intent Phrases

- I'm already using LangChain — what's the easiest way to add observability?
- How do I trace LangGraph agents end-to-end?
- I need a commercial LLM observability platform with SOC 2 and HIPAA.
- How do I run regression tests on my LLM app before deploying changes?
- I want to compare GPT-4o vs Claude side-by-side on the same dataset.
- How do I instrument an OpenAI client to send traces automatically?
- Where can I manage prompt versions collaboratively across my team?
- I want a no-code agent builder backed by tracing.
- How do I trigger LLM-as-judge evaluators on production traffic automatically?
- I need to route flagged traces to human annotators.
- What's the LangChain team's official observability product?

### Problem Statements

- LangChain apps emit complex execution graphs that are hard to debug without tracing.
- Prompt iterations are scattered across notebooks and PRs.
- We cannot tell whether a code change regressed answer quality.
- Production LLM failures have no audit trail.
- Closed-source platforms create vendor lock-in we must weigh against managed convenience.
- Self-hosting observability adds infra burden we want to avoid.

### When to Pick This

- Pick this when your stack is LangChain/LangGraph-heavy and you want zero-instrumentation tracing.
- Pick this over Langfuse when you want a fully managed commercial platform with HIPAA/SOC 2 out of the box.
- Pick this over Arize Phoenix when you prefer one closed-source platform combining tracing, evals, prompts, and Studio.
- Pick this over Helicone when you need an evaluation framework with datasets, experiments, and feedback (not only API proxy logging).
- Pick this over Weights & Biases when LLM agent debugging — not ML experiment tracking — is the priority.
- Pick this when you need a no-code Agent Builder or Studio for non-engineers.

### Related Terms and Aliases

- LangChain observability
- LCEL tracing
- Prompt Hub
- LangSmith Studio
- LangSmith Agent Builder
- LangSmith Agent Server
- LANGSMITH_TRACING
- LANGSMITH_API_KEY
- smith.langchain.com
- openevals
- LLM evaluation framework
- automation rules

