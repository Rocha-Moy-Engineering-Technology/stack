[Header 1 ("langsmith", [], []) [Str "LangSmith"], BlockQuote [Para [Str "LLM observability platform for tracing and evaluation with dashboard"]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.5)), (AlignDefault, (ColWidth 0.5))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Observability & LLM Ops"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "API/SDK/UI"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "No"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "N/A"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "N/A"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Documentation"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "Official Docs"] ("https://docs.langchain.com/langsmith", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "LangSmith is a framework-agnostic platform for developing, debugging, and deploying AI agents and Large Language Model (LLM) applications. Built by the LangChain team, it provides end-to-end observability, evaluation, prompt management, and deployment capabilities through a unified web dashboard and programmatic Software Development Kits (SDKs) in Python, TypeScript, and Java."], Para [Str "Unlike open-source alternatives that focus on a single concern (tracing or evaluation), LangSmith combines the entire LLM operations lifecycle into one commercial platform. Developers instrument their applications with lightweight SDK wrappers and decorators, and all execution data flows into the LangSmith dashboard where it can be inspected, evaluated against datasets, and used to iterate on prompts. The platform is framework-agnostic, supporting OpenAI, Anthropic, CrewAI, Vercel AI SDK, Pydantic AI, LangChain, LangGraph, and any custom LLM integration."], Para [Str "LangSmith operates as a managed cloud service at smith.langchain.com, with self-hosted and hybrid deployment options available for organizations with compliance or data residency requirements. The platform maintains Health Insurance Portability and Accountability Act (HIPAA), Service Organization Control 2 (SOC 2) Type 2, and General Data Protection Regulation (GDPR) compliance."], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], Para [Strong [Str "Traces"], Str " are the top-level records of a complete request flowing through an LLM application. A trace captures the entire execution path from input to final output, providing a hierarchical view of every operation that occurred during processing. Each trace belongs to a project within a workspace."], Para [Strong [Str "Runs"], Str " are the individual operations within a trace. A run represents a single unit of work such as an LLM call, a tool invocation, a retrieval step, or a custom function execution. Runs nest hierarchically within traces, forming parent-child relationships that reflect the call structure of the application. Each run captures its inputs, outputs, timing, token usage, and metadata."], Para [Strong [Str "Projects"], Str " organize traces into logical groupings. By default, traces flow into a default project, but developers can route traces to specific projects using the ", Code ("", [], []) "LANGSMITH_PROJECT", Str " environment variable. Projects enable separation between development, staging, and production environments."], Para [Strong [Str "Datasets"], Str " are collections of input-output pairs used for evaluation. Each dataset contains examples with inputs and optional reference outputs (ground truth). Datasets serve as the foundation for systematic evaluation, enabling repeatable experiments across different model configurations, prompts, or application versions."], Para [Strong [Str "Experiments"], Str " are evaluation runs that execute a target function against a dataset and score the results using evaluators. Experiments produce metrics that can be compared across runs, enabling data-driven decisions about which configuration performs best."], Para [Strong [Str "Evaluators"], Str " are functions that score the outputs of a target function. LangSmith supports custom code evaluators, LLM-as-judge evaluators (where another LLM grades the output), and pre-built evaluators for common patterns like correctness, conciseness, and hallucination detection."], Para [Strong [Str "Prompts"], Str " are versioned templates managed through the LangSmith Prompt Hub. Prompts can be created, iterated, shared, and deployed collaboratively. Each prompt revision is tracked, enabling teams to roll back to previous versions and audit changes over time."], Para [Strong [Str "Feedback"], Str " captures human or automated quality signals attached to individual runs. Feedback can be scores, labels, or free-text comments. It feeds into evaluation metrics and can trigger automation rules for quality monitoring."], Para [Strong [Str "Annotation Queues"], Str " organize runs that require human review. Queues enable systematic human evaluation workflows where domain experts review, label, and provide feedback on application outputs at scale."], Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Para [Str "LangSmith follows a client-server architecture where lightweight SDK instrumentation in the application sends telemetry data to the LangSmith backend for storage, visualization, and analysis."], OrderedList (1, DefaultStyle, DefaultDelim) [[Plain [Strong [Str "SDK instrumentation"], Str " wraps LLM client calls and application functions with tracing decorators or wrappers. The SDK captures inputs, outputs, timing, token counts, and metadata with minimal overhead."]], [Plain [Strong [Str "Trace collection"], Str " sends run data asynchronously to the LangSmith API. The SDK batches and transmits traces in the background to avoid impacting application latency."]], [Plain [Strong [Str "Backend storage"], Str " persists trace data, datasets, experiments, prompts, and feedback in the LangSmith platform. Data is organized by workspace, project, and time range."]], [Plain [Strong [Str "Dashboard visualization"], Str " provides the web interface for exploring traces, comparing experiments, managing prompts, configuring automation rules, and reviewing annotation queues."]], [Plain [Strong [Str "Evaluation engine"], Str " executes target functions against datasets, applies evaluators, and produces experiment results with aggregate and per-example metrics."]]], Para [Str "The architecture supports three deployment topologies:"], BulletList [[Plain [Strong [Str "Managed cloud"], Str ": LangSmith hosts everything; traces are sent to ", Code ("", [], []) "api.smith.langchain.com"]], [Plain [Strong [Str "Self-hosted"], Str ": The entire LangSmith stack runs within an organization's own infrastructure, keeping all data on-premise"]], [Plain [Strong [Str "Hybrid"], Str ": The application and trace data remain on-premise while certain management features use the cloud control plane"]]], Para [Str "LangSmith integrates with the broader LangChain ecosystem but does not require it. Any application that can make HTTP calls or use the LangSmith SDK can send traces, regardless of whether it uses LangChain, LangGraph, or any other framework."], Header 2 ("key-features-and-functionality", ["unnumbered", "unlisted"], []) [Str "Key Features and Functionality"], Para [Strong [Str "Observability and Tracing"], Str " provides full visibility into LLM application execution. Every LLM call, tool invocation, retrieval step, and custom function is captured as a run within a trace. The dashboard renders traces as interactive hierarchical trees, showing inputs, outputs, latency, token usage, cost, and error states at each level. Developers can filter, search, and drill into traces to diagnose issues."], Para [Strong [Str "Evaluation Framework"], Str " enables systematic measurement of application quality. Developers define datasets with input-output pairs, write evaluator functions (custom code or LLM-as-judge), and run experiments that produce comparable metrics. The framework supports row-level evaluators that score individual examples and summary evaluators that produce aggregate statistics across the entire dataset."], Para [Strong [Str "Prompt Management"], Str " through the Prompt Hub provides versioned prompt storage with collaboration features. Prompts can be created in the visual Playground, tested against different models and parameters, shared across teams, and pulled into application code programmatically. Every revision is tracked for auditability."], Para [Strong [Str "Studio"], Str " is a visual interface for designing, testing, and refining LLM applications interactively. It provides a playground for experimenting with prompts, models, and parameters without writing code, enabling rapid iteration on application behavior."], Para [Strong [Str "Agent Builder"], Str " offers a no-code visual interface for designing and deploying AI agents. It allows non-technical users to construct agent workflows, define tool usage, and deploy agents to production without programming."], Para [Strong [Str "Online Evaluation and Automation"], Str " rules monitor production traces in real time. Rules can trigger LLM-as-judge evaluators, flag anomalous traces, route runs to annotation queues, or send alerts based on configurable conditions. This enables continuous quality monitoring without manual review of every trace."], Para [Strong [Str "Annotation Queues"], Str " support systematic human evaluation. Runs matching certain criteria are routed to queues where human reviewers provide feedback, labels, and corrections. This human-in-the-loop workflow feeds back into dataset creation and model improvement."], Para [Strong [Str "Cost and Token Tracking"], Str " aggregates token usage and estimated costs across all traced runs. Developers can monitor spending by project, model, or time period, enabling budget management and cost optimization."], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], BulletList [[Plain [Strong [Str "Debugging LLM applications"], Str ": Trace execution paths to identify where an application produces incorrect or unexpected outputs, inspecting each step from input through retrieval, prompting, and response generation"]], [Plain [Strong [Str "Regression testing"], Str ": Run evaluation experiments against golden datasets before deploying changes, comparing new experiment results against baseline metrics to catch quality regressions"]], [Plain [Strong [Str "Prompt engineering"], Str ": Iterate on prompts using the Playground and Prompt Hub, testing variations against datasets and comparing results across experiments to find the optimal prompt formulation"]], [Plain [Strong [Str "Production monitoring"], Str ": Attach automation rules to production projects that evaluate a sample of traces with LLM-as-judge evaluators, flagging quality degradation or anomalous behavior for human review"]], [Plain [Strong [Str "Human evaluation workflows"], Str ": Route production traces to annotation queues where domain experts review outputs, provide feedback, and curate examples for future evaluation datasets"]], [Plain [Strong [Str "Multi-model comparison"], Str ": Run the same dataset against different models or configurations in separate experiments, then compare results side-by-side to select the best-performing option"]], [Plain [Strong [Str "Agent observability"], Str ": Trace multi-step agent workflows built with LangGraph, CrewAI, or custom frameworks, visualizing tool calls, reasoning steps, and decision points across the entire execution"]]], Header 2 ("api-reference-summary", ["unnumbered", "unlisted"], []) [Str "API Reference Summary"], Para [Strong [Str "Client Initialization"], Str ":"], BulletList [[Plain [Code ("", [], []) "Client(api_key=None)", Str " -- Create a LangSmith client (Python); reads ", Code ("", [], []) "LANGSMITH_API_KEY", Str " from environment if not provided"]], [Plain [Code ("", [], []) "new Client({ apiKey })", Str " -- Create a LangSmith client (TypeScript)"]]], Para [Strong [Str "Tracing"], Str ":"], BulletList [[Plain [Code ("", [], []) "@traceable", Str " -- Python decorator that creates a traced run for the decorated function"]], [Plain [Code ("", [], []) "@traceable(run_type=\"tool\", name=\"...\")", Str " -- Decorator with explicit run type and display name"]], [Plain [Code ("", [], []) "traceable(fn, { name, run_type })", Str " -- TypeScript function wrapper for creating traced runs"]], [Plain [Code ("", [], []) "wrap_openai(client)", Str " -- Python wrapper that instruments an OpenAI client for automatic tracing"]], [Plain [Code ("", [], []) "wrapOpenAI(client)", Str " -- TypeScript wrapper for OpenAI client instrumentation"]]], Para [Strong [Str "Datasets"], Str ":"], BulletList [[Plain [Code ("", [], []) "client.create_dataset(name, description)", Str " -- Create a new evaluation dataset"]], [Plain [Code ("", [], []) "client.create_examples(dataset_id, examples)", Str " -- Add input-output examples to a dataset"]], [Plain [Code ("", [], []) "client.clone_public_dataset(url)", Str " -- Clone a publicly shared dataset into the workspace"]]], Para [Strong [Str "Evaluation"], Str ":"], BulletList [[Plain [Code ("", [], []) "client.evaluate(target, data, evaluators, experiment_prefix)", Str " -- Run an evaluation experiment against a dataset with specified evaluators"]], [Plain [Code ("", [], []) "evaluate(target, { data, evaluators, experimentPrefix })", Str " -- TypeScript evaluation function"]], [Plain [Str "Custom evaluator signature: ", Code ("", [], []) "def evaluator(inputs, outputs, reference_outputs) -> bool | float | dict"]]], Para [Strong [Str "Prompts"], Str ":"], BulletList [[Plain [Code ("", [], []) "client.list_prompts(query, is_public)", Str " -- List prompts with optional filtering"]], [Plain [Code ("", [], []) "client.delete_prompt(name)", Str " -- Delete a prompt by name"]], [Plain [Code ("", [], []) "client.like_prompt(handle)", Str " -- Like a public prompt"]], [Plain [Code ("", [], []) "client.unlike_prompt(handle)", Str " -- Remove a like from a prompt"]]], Para [Strong [Str "REST API"], Str ":"], BulletList [[Plain [Code ("", [], []) "POST /api/v1/datasets/upload-experiment", Str " -- Upload externally-run experiment results to a dataset"]], [Plain [Str "Base URL: ", Code ("", [], []) "https://api.smith.langchain.com", Str " (managed cloud)"]]], Header 2 ("configuration-and-customization", ["unnumbered", "unlisted"], []) [Str "Configuration and Customization"], Para [Str "LangSmith configuration is primarily driven by environment variables and SDK parameters."], Header 3 ("environment-variables", ["unnumbered", "unlisted"], []) [Str "Environment Variables"], CodeBlock ("", ["bash"], []) "# Required for tracing
export LANGSMITH_TRACING=true
export LANGSMITH_API_KEY=\"lsv2_...\"

# Optional configuration
export LANGSMITH_PROJECT=\"my-project\"          # Route traces to a named project
export LANGSMITH_WORKSPACE_ID=\"<workspace-id>\" # Target a specific workspace
export LANGSMITH_ENDPOINT=\"https://api.smith.langchain.com\"  # API endpoint (override for self-hosted)
export LANGSMITH_OTEL_ENABLED=true             # Enable OpenTelemetry integration
", Header 3 ("trace-sampling-and-filtering", ["unnumbered", "unlisted"], []) [Str "Trace Sampling and Filtering"], Para [Str "For high-throughput production applications, tracing can be configured to sample a percentage of requests rather than tracing every call. This reduces overhead and storage costs while still providing statistical visibility into application behavior."], Header 3 ("project-organization", ["unnumbered", "unlisted"], []) [Str "Project Organization"], Para [Str "Traces are organized into projects. Use separate projects for development, staging, and production to isolate concerns:"], CodeBlock ("", ["bash"], []) "# Development
export LANGSMITH_PROJECT=\"my-app-dev\"

# Production
export LANGSMITH_PROJECT=\"my-app-prod\"
", Header 3 ("evaluator-configuration", ["unnumbered", "unlisted"], []) [Str "Evaluator Configuration"], Para [Str "Custom evaluators can be defined as simple functions returning boolean, numeric, or dictionary results:"], CodeBlock ("", ["python"], []) "def correctness(inputs: dict, outputs: dict, reference_outputs: dict) -> bool:
    return outputs[\"answer\"].strip().lower() == reference_outputs[\"answer\"].strip().lower()

def relevance_score(inputs: dict, outputs: dict, reference_outputs: dict) -> float:
    # Return a score between 0 and 1
    return 0.85
", Para [Str "LLM-as-judge evaluators delegate scoring to another LLM:"], CodeBlock ("", ["python"], []) "from openevals import create_llm_as_judge, CORRECTNESS_PROMPT

correctness_evaluator = create_llm_as_judge(
    prompt=CORRECTNESS_PROMPT,
    model=\"gpt-4o-mini\",
    feedback_key=\"correctness\",
)
", Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Para [Strong [Str "OpenAI Integration"], Str ": The ", Code ("", [], []) "wrap_openai", Str " wrapper instruments the OpenAI client to automatically capture all chat completion and embedding calls as traced runs. This is the lowest-friction integration path:"], CodeBlock ("", ["python"], []) "import openai
from langsmith.wrappers import wrap_openai

client = wrap_openai(openai.Client())

# All calls through this client are automatically traced
response = client.chat.completions.create(
    model=\"gpt-4o-mini\",
    messages=[{\"role\": \"user\", \"content\": \"Hello\"}],
)
", Para [Strong [Str "LangChain and LangGraph Integration"], Str ": When LangSmith tracing is enabled via environment variables, LangChain and LangGraph automatically send traces without any additional instrumentation. Every chain invocation, agent step, and tool call is captured:"], CodeBlock ("", ["python"], []) "from langchain_openai import ChatOpenAI
from langgraph.graph import StateGraph

# With LANGSMITH_TRACING=true, all LangChain/LangGraph
# operations are automatically traced
llm = ChatOpenAI(model=\"gpt-4o-mini\")
", Para [Strong [Str "Anthropic Integration"], Str ": Use the ", Code ("", [], []) "@traceable", Str " decorator to wrap functions that call the Anthropic SDK:"], CodeBlock ("", ["python"], []) "import anthropic
from langsmith import traceable

client = anthropic.Anthropic()

@traceable(name=\"Claude Call\")
def ask_claude(question: str) -> str:
    message = client.messages.create(
        model=\"claude-sonnet-4-20250514\",
        max_tokens=1024,
        messages=[{\"role\": \"user\", \"content\": question}],
    )
    return message.content[0].text
", Para [Strong [Str "Vercel AI SDK Integration"], Str ": In TypeScript applications using the Vercel AI SDK, enable tracing with the OpenTelemetry (OTel) flag:"], CodeBlock ("", ["bash"], []) "export LANGSMITH_TRACING=true
export LANGSMITH_API_KEY=\"lsv2_...\"
export LANGSMITH_OTEL_ENABLED=true
", Para [Strong [Str "Framework-Agnostic Custom Tracing"], Str ": Any function can be traced regardless of the LLM framework used:"], CodeBlock ("", ["python"], []) "from langsmith import traceable

@traceable(run_type=\"retriever\", name=\"Vector Search\")
def search_documents(query: str) -> list[str]:
    # Custom retrieval logic using any vector store
    return [\"Document 1 content\", \"Document 2 content\"]

@traceable(run_type=\"chain\", name=\"RAG Pipeline\")
def rag_pipeline(question: str) -> str:
    docs = search_documents(question)
    # Custom LLM call using any provider
    return generate_answer(question, docs)
", Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Para [Strong [Str "End-to-end Retrieval-Augmented Generation (RAG) application with tracing"], Str ":"], CodeBlock ("", ["python"], []) "from openai import OpenAI
from langsmith.wrappers import wrap_openai
from langsmith import traceable

client = wrap_openai(OpenAI())

@traceable(run_type=\"retriever\")
def retriever(query: str) -> list[str]:
    return [\"Harrison worked at Kensho\"]

@traceable
def rag(question: str) -> str:
    docs = retriever(question)
    system_message = (
        \"Answer the user's question using only the provided information below:\\n\"
        + \"\\n\".join(docs)
    )
    response = client.chat.completions.create(
        model=\"gpt-4o-mini\",
        messages=[
            {\"role\": \"system\", \"content\": system_message},
            {\"role\": \"user\", \"content\": question},
        ],
    )
    return response.choices[0].message.content

if __name__ == \"__main__\":
    print(rag(\"Where did Harrison work?\"))
", Para [Strong [Str "Creating a dataset and running an evaluation experiment"], Str ":"], CodeBlock ("", ["python"], []) "from langsmith import Client
from langsmith.wrappers import wrap_openai
import openai

ls_client = Client()
openai_client = wrap_openai(openai.OpenAI())

# Create dataset with examples
dataset = ls_client.create_dataset(\"QA Evaluation\", description=\"Question-answer pairs\")
ls_client.create_examples(
    dataset_id=dataset.id,
    examples=[
        {
            \"inputs\": {\"question\": \"What is LangSmith?\"},
            \"outputs\": {\"answer\": \"A platform for observing and evaluating LLM applications\"},
        },
        {
            \"inputs\": {\"question\": \"What is LangChain?\"},
            \"outputs\": {\"answer\": \"A framework for building LLM applications\"},
        },
    ],
)

# Define target function
def target(inputs: dict) -> dict:
    response = openai_client.chat.completions.create(
        model=\"gpt-4o-mini\",
        messages=[{\"role\": \"user\", \"content\": inputs[\"question\"]}],
    )
    return {\"answer\": response.choices[0].message.content}

# Define evaluators
def correctness(inputs: dict, outputs: dict, reference_outputs: dict) -> bool:
    prompt = f\"\"\"Grade the answer.
Question: {inputs['question']}
Reference: {reference_outputs['answer']}
Predicted: {outputs['answer']}
Respond with CORRECT or INCORRECT.\"\"\"
    response = openai_client.chat.completions.create(
        model=\"gpt-4o-mini\",
        messages=[{\"role\": \"user\", \"content\": prompt}],
    )
    return response.choices[0].message.content.strip() == \"CORRECT\"

def conciseness(outputs: dict, reference_outputs: dict) -> bool:
    return len(outputs[\"answer\"]) < 2 * len(reference_outputs[\"answer\"])

# Run evaluation
results = ls_client.evaluate(
    target,
    data=\"QA Evaluation\",
    evaluators=[correctness, conciseness],
    experiment_prefix=\"gpt-4o-mini-eval\",
)
", Para [Strong [Str "TypeScript tracing with tool calls"], Str ":"], CodeBlock ("", ["typescript"], []) "import OpenAI from \"openai\";
import { wrapOpenAI, traceable } from \"langsmith/wrappers\";

const client = wrapOpenAI(new OpenAI());

const retrieveContext = traceable(
    async (question: string): Promise<string> => {
        return \"LangSmith is an LLM observability platform by LangChain.\";
    },
    { name: \"Retrieve Context\", run_type: \"retriever\" }
);

const chatPipeline = traceable(
    async (question: string): Promise<string | null> => {
        const context = await retrieveContext(question);
        const response = await client.chat.completions.create({
            model: \"gpt-4o-mini\",
            messages: [
                {
                    role: \"system\",
                    content: `Answer based on this context: ${context}`,
                },
                { role: \"user\", content: question },
            ],
        });
        return response.choices[0].message?.content;
    },
    { name: \"Chat Pipeline\" }
);

(async () => {
    console.log(await chatPipeline(\"What is LangSmith?\"));
})();
", Para [Strong [Str "Programmatic prompt management"], Str ":"], CodeBlock ("", ["python"], []) "from langsmith import Client

client = Client()

# List all prompts
prompts = client.list_prompts()

# List private prompts matching a query
prompts = client.list_prompts(query=\"summarize\", is_public=False)

# Delete a prompt
client.delete_prompt(\"old-summarizer\")

# Like a public prompt
client.like_prompt(\"team/production-prompt\")
", Header 2 ("limitations-and-considerations", ["unnumbered", "unlisted"], []) [Str "Limitations and Considerations"], BulletList [[Plain [Strong [Str "Closed-source platform"], Str ": LangSmith is a proprietary commercial product. Unlike open-source alternatives such as Langfuse or Arize Phoenix, the source code is not available for inspection or modification. Organizations must trust the vendor for data handling and platform reliability."]], [Plain [Strong [Str "Vendor lock-in"], Str ": Applications instrumented with LangSmith SDK decorators and wrappers create a dependency on the LangSmith platform. While the ", Code ("", [], []) "@traceable", Str " decorator is lightweight, switching to a different observability platform requires re-instrumenting the application."]], [Plain [Strong [Str "Cost at scale"], Str ": For high-throughput production applications generating millions of traces, storage and processing costs can become significant. Trace sampling may be necessary, which reduces observability coverage."]], [Plain [Strong [Str "Network dependency"], Str ": In managed cloud mode, all trace data is transmitted over the network to LangSmith servers. This introduces latency sensitivity for the background trace transmission and requires network connectivity. Self-hosted deployments mitigate this but add operational complexity."]], [Plain [Strong [Str "Evaluation dataset management"], Str ": Datasets are stored within LangSmith and managed through the SDK or UI. There is no native integration with version control systems for dataset versioning, though datasets can be exported and imported programmatically."]], [Plain [Strong [Str "LangChain ecosystem affinity"], Str ": While LangSmith is framework-agnostic, the deepest integrations and most seamless experience are with LangChain and LangGraph. Applications using other frameworks require manual instrumentation with decorators and wrappers."]], [Plain [Strong [Str "Self-hosted complexity"], Str ": Self-hosted deployments require managing the full LangSmith infrastructure stack, including databases, API servers, and the web dashboard. This demands operational expertise beyond what the managed cloud offering requires."]]], Header 2 ("changelog-highlights", ["unnumbered", "unlisted"], []) [Str "Changelog Highlights"], Para [Str "LangSmith has evolved from a tracing-focused companion to LangChain into a comprehensive LLM operations platform. Key milestones include the launch of the evaluation framework with datasets and experiments, the Prompt Hub for collaborative prompt management, Studio for visual application design, Agent Builder for no-code agent creation, OpenTelemetry integration for broader ecosystem compatibility, and Agent Server for production deployment of stateful agent workflows. The platform continues to expand its framework-agnostic integrations and compliance certifications."], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " ", Link ("", [], []) [Str "LangSmith Documentation"] ("https://docs.langchain.com/langsmith", "")]], [Plain [Str "[", Str "2", Str "]", Str " ", Link ("", [], []) [Str "LangSmith Observability Quickstart"] ("https://docs.langchain.com/langsmith/observability-quickstart", "")]], [Plain [Str "[", Str "3", Str "]", Str " ", Link ("", [], []) [Str "LangSmith Evaluation Quickstart"] ("https://docs.langchain.com/langsmith/evaluation-quickstart", "")]], [Plain [Str "[", Str "4", Str "]", Str " ", Link ("", [], []) [Str "LangSmith Trace with OpenAI"] ("https://docs.langchain.com/langsmith/trace-openai", "")]], [Plain [Str "[", Str "5", Str "]", Str " ", Link ("", [], []) [Str "LangSmith Prompt Management"] ("https://docs.langchain.com/langsmith/manage-prompts-programmatically", "")]], [Plain [Str "[", Str "6", Str "]", Str " ", Link ("", [], []) [Str "LangSmith Upload Experiments API"] ("https://docs.langchain.com/langsmith/upload-existing-experiments", "")]]]]