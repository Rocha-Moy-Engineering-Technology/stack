[Header 1 ("arize-phoenix", [], []) [Str "Arize Phoenix"], BlockQuote [Para [Str "Open-source AI observability for debugging and iterating on LLM applications"]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, ColWidthDefault), (AlignDefault, ColWidthDefault)] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Observability"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "SDK/UI"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Yes"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "Arize-AI/phoenix"] ("https://github.com/Arize-AI/phoenix", "")]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "9673"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Documentation"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "Official Docs"] ("https://arize.com/docs/phoenix", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "Arize Phoenix is an open-source AI observability and evaluation platform designed for experimentation, evaluation, and troubleshooting of Large Language Model (LLM) applications. Built on OpenTelemetry standards and powered by OpenInference instrumentation, Phoenix provides a vendor-agnostic and language-agnostic approach to capturing, analyzing, and improving AI application behavior. ", Str "[", Str "1", Str "]", Str "[", Str "2", Str "]"], Para [Str "The platform is organized around four core workflows: ", Strong [Str "tracing"], Str " captures the full execution path of LLM applications through distributed traces; ", Strong [Str "evaluation"], Str " measures output quality using LLM-based, code-based, and human evaluators; ", Strong [Str "prompt engineering"], Str " enables iterative prompt improvement with version control and playground experimentation; and ", Strong [Str "datasets and experiments"], Str " provide systematic comparison of application versions against consistent inputs and evaluation criteria. ", Str "[", Str "1", Str "]"], Para [Str "Phoenix runs as a standalone server with a web UI for visualization and analysis, backed by either SQLite (default) or PostgreSQL for persistent storage. It accepts traces via the OpenTelemetry Protocol (OTLP), making it compatible with any OpenTelemetry-instrumented application. SDKs are available for Python, TypeScript, and Java. ", Str "[", Str "1", Str "]", Str "[", Str "2", Str "]"], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], Para [Strong [Str "Traces"], Str " represent the complete execution path of a single request through an LLM application. A trace captures the sequence of operations from initial input to final output, including model calls, document retrieval, tool invocations, and custom logic. Phoenix accepts traces via OTLP over HTTP or gRPC. ", Str "[", Str "1", Str "]"], Para [Strong [Str "Spans"], Str " are the individual units within a trace, each representing a discrete operation. Phoenix defines nine span kinds to categorize operations: LLM (model API calls), EMBEDDING (vector generation), CHAIN (orchestration steps), RETRIEVER (data fetching), RERANKER (document relevance scoring), TOOL (external tool invocations), AGENT (reasoning blocks coordinating tools), GUARDRAIL (safety filtering), and EVALUATOR (output assessment). ", Str "[", Str "3", Str "]"], Para [Strong [Str "Projects"], Str " organize traces by application or environment. Each project collects its own set of traces, evaluations, and annotations, allowing teams to separate development, staging, and production data within a single Phoenix instance. ", Str "[", Str "1", Str "]"], Para [Strong [Str "Evaluations"], Str " are quality assessments attached to traces or spans. Phoenix supports three evaluation methods: LLM-based evaluators that use language models to judge output quality, code-based evaluators that apply deterministic checks (regex, exact match, distance metrics), and human annotations with ground truth labels. ", Str "[", Str "1", Str "]", Str "[", Str "4", Str "]"], Para [Strong [Str "Datasets"], Str " are versioned collections of input-output examples extracted from traces or uploaded directly. They serve as fixed benchmarks for running experiments and comparing application changes. ", Str "[", Str "1", Str "]"], Para [Strong [Str "Experiments"], Str " systematically compare application versions by running them against the same dataset with identical evaluators. Each experiment records task outputs and evaluator scores, enabling side-by-side comparison of different prompts, models, or retrieval configurations. ", Str "[", Str "5", Str "]"], Para [Strong [Str "Prompts"], Str " are managed artifacts with version control, supporting Mustache template syntax (", Code ("", [], []) "{{ variable }}", Str ") and f-string formatting. Each prompt version records its template content, model configuration, and associated metadata, enabling teams to track changes and deploy the best-performing version. ", Str "[", Str "6", Str "]"], Para [Strong [Str "Annotations"], Str " are metadata labels attached to spans, including scores, labels, and explanations from human reviewers or automated evaluators. Annotations provide the ground truth data used to measure and improve application quality. ", Str "[", Str "1", Str "]"], Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Para [Str "Phoenix follows a collector-server-UI architecture built on OpenTelemetry:"], CodeBlock ("", [""], []) "Application Code
    |
    | (OTLP over HTTP/gRPC)
    v
Phoenix Collector (port 4317 gRPC, port 6006 HTTP)
    |
    v
Storage Backend (SQLite or PostgreSQL)
    |
    v
Phoenix Web UI (port 6006)
    |
    v
REST API / Python Client / TypeScript Client
", Para [Strong [Str "Collector Layer"], Str ": Accepts traces via OTLP, the standard OpenTelemetry protocol. Any OpenTelemetry-compatible application can send traces to Phoenix without Phoenix-specific dependencies. The gRPC collector runs on port 4317 and HTTP on port 6006 by default. ", Str "[", Str "1", Str "]"], Para [Strong [Str "Storage Layer"], Str ": Phoenix stores traces, evaluations, datasets, and experiments in a SQL database. SQLite is the default for local development (stored in ", Code ("", [], []) "~/.phoenix/", Str "). PostgreSQL is supported for production deployments with higher concurrency and durability requirements. ", Str "[", Str "7", Str "]"], Para [Strong [Str "Web UI"], Str ": A browser-based interface for exploring traces, viewing span details, running evaluations, managing prompts, and comparing experiments. The UI provides filtering, search, and visualization of trace hierarchies with timing and token usage breakdowns. ", Str "[", Str "1", Str "]"], Para [Strong [Str "Client SDK"], Str ": The ", Code ("", [], []) "arize-phoenix-client", Str " package provides a REST API client for programmatic access to all Phoenix resources: projects, traces, spans, annotations, datasets, experiments, and prompts. Both synchronous (", Code ("", [], []) "Client", Str ") and asynchronous (", Code ("", [], []) "AsyncClient", Str ") implementations are available. ", Str "[", Str "8", Str "]"], Para [Strong [Str "OpenInference"], Str ": An open-source project maintained alongside Phoenix that defines semantic conventions for AI observability on top of OpenTelemetry. OpenInference provides auto-instrumentor packages for popular frameworks and the span attribute schema that Phoenix uses to parse and display trace data. ", Str "[", Str "1", Str "]"], Header 3 ("python-packages", ["unnumbered", "unlisted"], []) [Str "Python Packages"], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.5)), (AlignDefault, (ColWidth 0.5))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Package"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Purpose"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "arize-phoenix"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Full Phoenix server with UI"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "arize-phoenix-otel"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Lightweight OpenTelemetry wrapper with Phoenix defaults"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "arize-phoenix-client"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "REST API client for server interaction"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "arize-phoenix-evals"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Evaluation framework (LLM-based and code-based)"]]]])] (TableFoot ("", [], []) []), Header 3 ("typescript-packages", ["unnumbered", "unlisted"], []) [Str "TypeScript Packages"], Table ("", [], []) (Caption Nothing []) [(AlignDefault, ColWidthDefault), (AlignDefault, ColWidthDefault)] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Package"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Purpose"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "@arizeai/phoenix-otel"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "OpenTelemetry configuration for Phoenix"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "@arizeai/phoenix-client"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "REST API client"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "@arizeai/phoenix-evals"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Evaluation framework"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "@arizeai/openinference-core"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Core instrumentation utilities"]]]])] (TableFoot ("", [], []) []), Header 2 ("key-features-and-functionality", ["unnumbered", "unlisted"], []) [Str "Key Features and Functionality"], Header 3 ("distributed-tracing", ["unnumbered", "unlisted"], []) [Str "Distributed Tracing"], Para [Str "Phoenix captures the full execution path of LLM applications through OpenTelemetry-based distributed tracing. Auto-instrumentation packages automatically capture spans for supported frameworks without code changes. The trace view provides visibility into performance bottlenecks, token usage breakdown across LLM calls, runtime exceptions, retrieved documents with relevance scores, LLM parameters and prompt templates, and function call selections. ", Str "[", Str "1", Str "]"], CodeBlock ("", ["python"], []) "from phoenix.otel import register

tracer_provider = register(
    project_name=\"my-rag-app\",
    auto_instrument=True,
)
", Header 3 ("manual-instrumentation", ["unnumbered", "unlisted"], []) [Str "Manual Instrumentation"], Para [Str "For custom logic not covered by auto-instrumentation, Phoenix provides decorators and context managers:"], CodeBlock ("", ["python"], []) "from phoenix.otel import register

tracer_provider = register(project_name=\"my-app\")
tracer = tracer_provider.get_tracer(__name__)

@tracer.chain
def my_processing_step(input: str) -> str:
    return process(input)

@tracer.tool
def search_database(query: str) -> str:
    \"\"\"Search the product database.\"\"\"
    return results

@tracer.agent
def run_agent(input: str) -> str:
    return agent_response
", Para [Str "Context managers provide granular control over individual code blocks:"], CodeBlock ("", ["python"], []) "with tracer.start_as_current_span(\"custom-step\", openinference_span_kind=\"chain\") as span:
    span.set_input(\"input data\")
    result = do_work()
    span.set_output(result)
", Para [Str "LLM spans support rich metadata including message history, token counts, and model parameters:"], CodeBlock ("", ["python"], []) "@tracer.llm(
    process_input=process_input,
    process_output=process_output,
)
def invoke_llm(messages):
    return client.chat.completions.create(messages=messages)
", Header 3 ("context-attributes", ["unnumbered", "unlisted"], []) [Str "Context Attributes"], Para [Str "Attach metadata to traces using context managers that propagate to all child spans:"], CodeBlock ("", ["python"], []) "from openinference.instrumentation import using_attributes

with using_attributes(
    session_id=\"session-123\",
    user_id=\"user-456\",
    metadata={\"environment\": \"production\"},
    tags=[\"v2\", \"experiment-a\"],
    prompt_template=\"Summarize: {{ text }}\",
    prompt_template_version=\"v1.0\",
):
    result = my_llm_chain(input_text)
", Para [Str "Individual context managers are also available for setting specific attributes:"], CodeBlock ("", ["python"], []) "from openinference.instrumentation import using_session, using_user, using_metadata, using_tags

with using_session(session_id=\"session-123\"):
    with using_user(\"user-456\"):
        result = my_llm_chain(input_text)
", Header 3 ("evaluations", ["unnumbered", "unlisted"], []) [Str "Evaluations"], Para [Str "Phoenix provides a flexible evaluation framework for measuring output quality. Evaluators can be LLM-based (using language models as judges) or code-based (deterministic checks): ", Str "[", Str "4", Str "]"], CodeBlock ("", ["python"], []) "from phoenix.evals import create_classifier
from phoenix.evals.llm import LLM

llm = LLM(provider=\"openai\", model=\"gpt-4o\")

helpfulness_evaluator = create_classifier(
    name=\"helpfulness\",
    prompt_template=\"Rate the response as helpful or not:\\n\\nQuery: {input}\\nResponse: {output}\",
    llm=llm,
    choices={\"helpful\": 1.0, \"not_helpful\": 0.0},
)

scores = helpfulness_evaluator.evaluate({
    \"input\": \"How do I reset my password?\",
    \"output\": \"Go to Settings > Security > Reset Password.\",
})
", Para [Str "Run evaluations across DataFrames of traces:"], CodeBlock ("", ["python"], []) "from phoenix.evals.evaluators import async_evaluate_dataframe

results_df = await async_evaluate_dataframe(
    dataframe=trace_df,
    evaluators=[helpfulness_evaluator, relevance_evaluator],
)
", Header 3 ("datasets-and-experiments", ["unnumbered", "unlisted"], []) [Str "Datasets and Experiments"], Para [Str "Create datasets from traces or direct uploads, then run experiments to compare application versions: ", Str "[", Str "5", Str "]"], CodeBlock ("", ["python"], []) "from phoenix.client import Client
from phoenix.client.experiments import run_experiment

client = Client()

dataset = client.datasets.get_dataset(dataset=\"my-test-set\")

def task(input):
    return my_llm_chain(input[\"question\"])

def evaluate_correctness(output, expected):
    return output.strip().lower() == expected.strip().lower()

experiment = run_experiment(
    dataset,
    task=task,
    evaluators=[evaluate_correctness],
    experiment_name=\"v2-prompt-update\",
)
", Para [Str "Experiments support both synchronous and asynchronous task functions. The ", Code ("", [], []) "dry_run=True", Str " parameter allows testing the setup before full execution. Evaluators can be added or modified after an experiment completes using ", Code ("", [], []) "evaluate_experiment()", Str "."], Header 3 ("prompt-management", ["unnumbered", "unlisted"], []) [Str "Prompt Management"], Para [Str "Store, version, and retrieve prompts programmatically: ", Str "[", Str "6", Str "]"], CodeBlock ("", ["python"], []) "from phoenix.client import Client
from phoenix.client.types import PromptVersion

client = Client()

content = \"\"\"You're an expert educator in {{ topic }}.
Summarize the following article in bullet points:
{{ article }}\"\"\"

prompt = client.prompts.create(
    name=\"article-summarizer\",
    prompt_description=\"Summarize articles for beginners\",
    version=PromptVersion(
        [{\"role\": \"user\", \"content\": content}],
        model_name=\"gpt-4o-mini\",
    ),
)
", Para [Str "Retrieve prompts by name (latest version), specific version ID, or tag:"], CodeBlock ("", ["python"], []) "prompt = client.prompts.get(name=\"article-summarizer\")
", Header 3 ("prompt-playground", ["unnumbered", "unlisted"], []) [Str "Prompt Playground"], Para [Str "The web UI includes an interactive Prompt Playground for iterating on prompts with production data. Features include side-by-side model comparison, parameter adjustment (temperature, max tokens), span replay to test prompts against real inputs captured in traces, and multi-variant testing across model providers. ", Str "[", Str "1", Str "]"], Header 3 ("trace-suppression", ["unnumbered", "unlisted"], []) [Str "Trace Suppression"], Para [Str "Conditionally disable tracing for sensitive or high-frequency operations:"], CodeBlock ("", ["python"], []) "from phoenix.otel import suppress_tracing

with suppress_tracing():
    sensitive_result = call_internal_service()
", Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], Para [Strong [Str "RAG Application Debugging"], Str ": Trace the full retrieval-augmented generation pipeline to identify retrieval failures, irrelevant document selection, or hallucinations. Inspect retrieved documents with relevance scores, view embedding distances, and evaluate response quality against ground truth."], Para [Strong [Str "Agent Loop Analysis"], Str ": Visualize multi-step agent executions to identify infinite loops, unnecessary tool calls, or suboptimal tool selection patterns. Agent spans capture the reasoning chain including which tools were called, with what parameters, and in what order."], Para [Strong [Str "Production Monitoring"], Str ": Deploy Phoenix as a persistent server receiving traces from production applications. Set up automated evaluators to score incoming traces, track quality metrics over time, and set alerts on degradation patterns."], Para [Strong [Str "Model Comparison"], Str ": Use the Prompt Playground to compare responses across different models (GPT-4o, Claude, Gemini) with identical inputs. Run experiments against datasets to quantify performance differences with statistical rigor."], Para [Strong [Str "Prompt Iteration"], Str ": Capture production traces to build datasets of real-world inputs, then iterate on prompts using the Playground with span replay. Version each prompt change and run experiments to validate improvements before deployment."], Para [Strong [Str "Evaluation Pipeline Development"], Str ": Build custom evaluation pipelines combining LLM judges, code-based checks, and human annotations. Use benchmark datasets to calibrate evaluator accuracy before deploying them for automated scoring."], Para [Strong [Str "Cost Optimization"], Str ": Break down token usage across LLM calls within traces to identify expensive operations. Compare token consumption across prompt versions or model choices to optimize cost-performance tradeoffs."], Header 2 ("api-reference-summary", ["unnumbered", "unlisted"], []) [Str "API Reference Summary"], Header 3 ("python-client-arize-phoenix-client", ["unnumbered", "unlisted"], []) [Str "Python Client (", Code ("", [], []) "arize-phoenix-client", Str ")"], Para [Strong [Str "Client Initialization"], Str ":"], CodeBlock ("", ["python"], []) "from phoenix.client import Client

client = Client()                                    # Local server (localhost:6006)
client = Client(endpoint=\"https://your-phoenix.com\") # Remote server
", Para [Strong [Str "Prompts Resource"], Str ":"], BulletList [[Plain [Code ("", [], []) "client.prompts.create(name, version, prompt_description)", Str " -- Create a new prompt with initial version"]], [Plain [Code ("", [], []) "client.prompts.get(name, version_id, tag)", Str " -- Retrieve prompt by name, version, or tag"]]], Para [Strong [Str "Datasets Resource"], Str ":"], BulletList [[Plain [Code ("", [], []) "client.datasets.list()", Str " -- List all datasets"]], [Plain [Code ("", [], []) "client.datasets.get_dataset(dataset)", Str " -- Fetch dataset with examples"]], [Plain [Code ("", [], []) "client.datasets.create_dataset(name, inputs, outputs)", Str " -- Create dataset from dictionaries, DataFrames, or CSV"]], [Plain [Code ("", [], []) "client.datasets.to_dataframe(dataset)", Str " -- Convert dataset to pandas DataFrame"]]], Para [Strong [Str "Spans Resource"], Str ":"], BulletList [[Plain [Code ("", [], []) "client.spans.get_spans_dataframe(project_name, filter)", Str " -- Query spans as DataFrame with filtering"]], [Plain [Code ("", [], []) "client.spans.get_span_annotations_dataframe(project_name)", Str " -- Retrieve span annotations"]], [Plain [Code ("", [], []) "client.spans.log_span_annotations(annotations)", Str " -- Bulk log annotations"]]], Para [Strong [Str "Projects Resource"], Str ":"], BulletList [[Plain [Code ("", [], []) "client.projects.list()", Str " -- List all projects"]], [Plain [Code ("", [], []) "client.projects.create(name)", Str " -- Create a new project"]]], Para [Strong [Str "Experiments"], Str ":"], BulletList [[Plain [Code ("", [], []) "run_experiment(dataset, task, evaluators, experiment_name)", Str " -- Run experiment against dataset"]], [Plain [Code ("", [], []) "evaluate_experiment(experiment, evaluators)", Str " -- Add evaluators to completed experiment"]]], Header 3 ("rest-api-endpoints", ["unnumbered", "unlisted"], []) [Str "REST API Endpoints"], Para [Str "Phoenix exposes a REST API for spans, annotations, datasets, and experiments. The ", Code ("", [], []) "/v1/traces", Str " endpoint accepts OTLP trace data via HTTP POST. ", Str "[", Str "8", Str "]"], Header 3 ("opentelemetry-integration", ["unnumbered", "unlisted"], []) [Str "OpenTelemetry Integration"], Para [Str "Phoenix accepts standard OTLP traces on:"], BulletList [[Plain [Strong [Str "HTTP"], Str ": ", Code ("", [], []) "http://localhost:6006/v1/traces"]], [Plain [Strong [Str "gRPC"], Str ": ", Code ("", [], []) "localhost:4317"]]], Header 2 ("configuration-and-customization", ["unnumbered", "unlisted"], []) [Str "Configuration and Customization"], Header 3 ("server-environment-variables", ["unnumbered", "unlisted"], []) [Str "Server Environment Variables"], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.3125)), (AlignDefault, (ColWidth 0.28125)), (AlignDefault, (ColWidth 0.40625))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Variable"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Default"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Description"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "PHOENIX_PORT"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "6006"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Web server and HTTP collector port"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "PHOENIX_GRPC_PORT"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "4317"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "gRPC OTLP collector port"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "PHOENIX_HOST"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "0.0.0.0"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Server host binding"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "PHOENIX_WORKING_DIR"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "~", Str "/.phoenix/"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Data directory for SQLite and exports"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "PHOENIX_SQL_DATABASE_URL"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "SQLite"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Database connection string (PostgreSQL or SQLite)"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "PHOENIX_HOST_ROOT_PATH"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "(none)"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Root path prefix for reverse proxy deployments"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "PHOENIX_DEFAULT_RETENTION_POLICY_DAYS"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "0 (infinite)"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Trace retention period in days"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "PHOENIX_ENABLE_PROMETHEUS"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "false"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Enable Prometheus metrics on port 9090"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "PHOENIX_CSRF_TRUSTED_ORIGINS"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "(none)"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Comma-separated origins bypassing CSRF protection"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "PHOENIX_ALLOW_EXTERNAL_RESOURCES"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "true"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "External resource loading (set false for air-gapped)"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "PHOENIX_TELEMETRY_ENABLED"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "true"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Anonymous usage analytics (never collects trace data)"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "PHOENIX_DEFAULT_ADMIN_INITIAL_PASSWORD"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "admin"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Initial admin password when authentication is enabled"]]]])] (TableFoot ("", [], []) []), Header 3 ("postgresql-configuration", ["unnumbered", "unlisted"], []) [Str "PostgreSQL Configuration"], Para [Str "For production deployments, configure PostgreSQL either via connection string or individual variables: ", Str "[", Str "7", Str "]"], CodeBlock ("", ["bash"], []) "export PHOENIX_SQL_DATABASE_URL=\"postgresql://user:pass@localhost:5432/phoenix\"

# Or via individual variables:
export PHOENIX_POSTGRES_HOST=\"localhost\"
export PHOENIX_POSTGRES_PORT=\"5432\"
export PHOENIX_POSTGRES_USER=\"phoenix\"
export PHOENIX_POSTGRES_PASSWORD=\"secret\"
export PHOENIX_POSTGRES_DB=\"phoenix\"
export PHOENIX_SQL_DATABASE_SCHEMA=\"phoenix\"  # Optional schema
", Header 3 ("docker-compose-with-postgresql", ["unnumbered", "unlisted"], []) [Str "Docker Compose with PostgreSQL"], CodeBlock ("", ["yaml"], []) "version: \"3.8\"
services:
  phoenix:
    image: arizephoenix/phoenix:version-8.0.0
    ports:
      - \"6006:6006\"
      - \"4317:4317\"
    environment:
      - PHOENIX_SQL_DATABASE_URL=postgresql://phoenix:secret@db:5432/phoenix
    depends_on:
      - db
  db:
    image: postgres:16
    environment:
      - POSTGRES_USER=phoenix
      - POSTGRES_PASSWORD=secret
      - POSTGRES_DB=phoenix
    volumes:
      - pgdata:/var/lib/postgresql/data
volumes:
  pgdata:
", Header 3 ("authentication", ["unnumbered", "unlisted"], []) [Str "Authentication"], Para [Str "Phoenix supports authentication via OAuth2, Lightweight Directory Access Protocol (LDAP), and local accounts with role-based access controls. When authentication is enabled, all API interactions require an ", Code ("", [], []) "Authorization: Bearer <token>", Str " header or the ", Code ("", [], []) "PHOENIX_API_KEY", Str " environment variable. ", Str "[", Str "7", Str "]"], Header 3 ("client-configuration", ["unnumbered", "unlisted"], []) [Str "Client Configuration"], CodeBlock ("", ["python"], []) "import os

os.environ[\"PHOENIX_COLLECTOR_ENDPOINT\"] = \"https://your-phoenix-instance.com\"
os.environ[\"PHOENIX_API_KEY\"] = \"your-api-key\"
os.environ[\"PHOENIX_CLIENT_HEADERS\"] = \"api_key=your-api-key\"
", Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Header 3 ("with-openai", ["unnumbered", "unlisted"], []) [Str "With OpenAI"], CodeBlock ("", ["python"], []) "from openinference.instrumentation.openai import OpenAIInstrumentor
from phoenix.otel import register

tracer_provider = register(project_name=\"openai-app\")
OpenAIInstrumentor().instrument(tracer_provider=tracer_provider)

# All OpenAI calls are now automatically traced
from openai import OpenAI
client = OpenAI()
response = client.chat.completions.create(
    model=\"gpt-4o\",
    messages=[{\"role\": \"user\", \"content\": \"Hello\"}],
)
", Header 3 ("with-langchain", ["unnumbered", "unlisted"], []) [Str "With LangChain"], CodeBlock ("", ["python"], []) "from openinference.instrumentation.langchain import LangChainInstrumentor
from phoenix.otel import register

tracer_provider = register(project_name=\"langchain-app\")
LangChainInstrumentor().instrument(tracer_provider=tracer_provider)
", Header 3 ("with-llamaindex", ["unnumbered", "unlisted"], []) [Str "With LlamaIndex"], CodeBlock ("", ["python"], []) "from openinference.instrumentation.llama_index import LlamaIndexInstrumentor
from phoenix.otel import register

tracer_provider = register(project_name=\"llamaindex-app\")
LlamaIndexInstrumentor().instrument(tracer_provider=tracer_provider)
", Header 3 ("with-haystack", ["unnumbered", "unlisted"], []) [Str "With Haystack"], CodeBlock ("", ["python"], []) "from openinference.instrumentation.haystack import HaystackInstrumentor
from phoenix.otel import register

tracer_provider = register()
HaystackInstrumentor().instrument(tracer_provider=tracer_provider)
", Header 3 ("with-crewai", ["unnumbered", "unlisted"], []) [Str "With CrewAI"], CodeBlock ("", ["python"], []) "from openinference.instrumentation.crewai import CrewAIInstrumentor
from openinference.instrumentation.openai import OpenAIInstrumentor
from phoenix.otel import register

tracer_provider = register(project_name=\"crewai-app\", auto_instrument=True)
", Header 3 ("with-dspy", ["unnumbered", "unlisted"], []) [Str "With DSPy"], CodeBlock ("", ["python"], []) "from openinference.instrumentation.dspy import DSPyInstrumentor
from phoenix.otel import register

tracer_provider = register(project_name=\"dspy-app\")
DSPyInstrumentor().instrument(tracer_provider=tracer_provider)
", Header 3 ("with-evaluation-frameworks-ragas-deepeval-cleanlab", ["unnumbered", "unlisted"], []) [Str "With Evaluation Frameworks (Ragas, Deepeval, Cleanlab)"], Para [Str "Phoenix integrates with third-party evaluation frameworks. Trace evaluations from Ragas, Deepeval, or Cleanlab can be ingested as annotations on Phoenix spans, combining framework-specific metrics with Phoenix's trace visualization. ", Str "[", Str "1", Str "]"], Header 3 ("typescript--vercel-ai-sdk", ["unnumbered", "unlisted"], []) [Str "TypeScript / Vercel AI SDK"], CodeBlock ("", ["typescript"], []) "import { NodeTracerProvider } from \"@opentelemetry/sdk-trace-node\";
import { BatchSpanProcessor } from \"@opentelemetry/sdk-trace-base\";
import { OTLPTraceExporter } from \"@opentelemetry/exporter-trace-otlp-proto\";
import { resourceFromAttributes } from \"@opentelemetry/resources\";
import { ATTR_SERVICE_NAME } from \"@opentelemetry/semantic-conventions\";
import { SEMRESATTRS_PROJECT_NAME } from \"@arizeai/openinference-semantic-conventions\";

const provider = new NodeTracerProvider({
    resource: resourceFromAttributes({
        [ATTR_SERVICE_NAME]: \"my-app\",
        [SEMRESATTRS_PROJECT_NAME]: \"my-app\",
    }),
    spanProcessors: [
        new BatchSpanProcessor(
            new OTLPTraceExporter({
                url: `${process.env.PHOENIX_COLLECTOR_ENDPOINT}/v1/traces`,
            })
        ),
    ],
});

provider.register();
", Header 3 ("with-litellm-proxy", ["unnumbered", "unlisted"], []) [Str "With LiteLLM Proxy"], Para [Str "Phoenix integrates with LiteLLM as an observability backend. LiteLLM can be configured to send traces to Phoenix, providing visibility across all models routed through the proxy."], Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Header 3 ("end-to-end-rag-tracing", ["unnumbered", "unlisted"], []) [Str "End-to-End RAG Tracing"], CodeBlock ("", ["python"], []) "from openinference.instrumentation.openai import OpenAIInstrumentor
from phoenix.otel import register

tracer_provider = register(
    project_name=\"rag-pipeline\",
    endpoint=\"http://localhost:6006/v1/traces\",
    auto_instrument=True,
)
tracer = tracer_provider.get_tracer(__name__)

from openai import OpenAI
client = OpenAI()

@tracer.chain
def rag_pipeline(question: str) -> str:
    documents = retrieve_documents(question)
    context = \"\\n\".join(doc.text for doc in documents)
    response = client.chat.completions.create(
        model=\"gpt-4o\",
        messages=[
            {\"role\": \"system\", \"content\": f\"Answer using this context:\\n{context}\"},
            {\"role\": \"user\", \"content\": question},
        ],
    )
    return response.choices[0].message.content

@tracer.retriever
def retrieve_documents(query: str):
    # Vector search implementation
    return vector_store.similarity_search(query, k=5)
", Header 3 ("custom-llm-evaluator-with-benchmark-dataset", ["unnumbered", "unlisted"], []) [Str "Custom LLM Evaluator with Benchmark Dataset"], CodeBlock ("", ["python"], []) "from phoenix.client import Client
from phoenix.client.experiments import run_experiment
from phoenix.evals import create_classifier
from phoenix.evals.llm import LLM

client = Client()
llm = LLM(provider=\"openai\", model=\"gpt-4o-mini\")

tone_evaluator = create_classifier(
    name=\"tone\",
    llm=llm,
    prompt_template=\"Is the tone of this response professional?\\n\\nResponse: {output}\",
    choices={\"professional\": 1.0, \"unprofessional\": 0.0},
)

relevance_evaluator = create_classifier(
    name=\"relevance\",
    llm=llm,
    prompt_template=\"Is this response relevant to the query?\\n\\nQuery: {input}\\nResponse: {output}\",
    choices={\"relevant\": 1.0, \"irrelevant\": 0.0},
)

dataset = client.datasets.get_dataset(dataset=\"customer-support-test\")

experiment = run_experiment(
    dataset,
    task=my_support_agent,
    evaluators=[tone_evaluator, relevance_evaluator],
    experiment_name=\"support-agent-v3\",
)
", Header 3 ("prompt-versioning-workflow", ["unnumbered", "unlisted"], []) [Str "Prompt Versioning Workflow"], CodeBlock ("", ["python"], []) "from phoenix.client import Client
from phoenix.client.types import PromptVersion

client = Client()

# Create initial prompt version
v1_content = \"Summarize this article:\\n{{ article }}\"
client.prompts.create(
    name=\"summarizer\",
    version=PromptVersion(
        [{\"role\": \"user\", \"content\": v1_content}],
        model_name=\"gpt-4o-mini\",
    ),
)

# Create improved version
v2_content = \"\"\"You are an expert technical writer.
Summarize the following article in 3-5 bullet points for a technical audience:
{{ article }}\"\"\"

client.prompts.create(
    name=\"summarizer\",
    version=PromptVersion(
        [{\"role\": \"user\", \"content\": v2_content}],
        model_name=\"gpt-4o\",
    ),
)

# Retrieve latest version in production
prompt = client.prompts.get(name=\"summarizer\")
", Header 3 ("session-aware-tracing", ["unnumbered", "unlisted"], []) [Str "Session-Aware Tracing"], CodeBlock ("", ["python"], []) "from openinference.instrumentation import using_attributes
from phoenix.otel import register

register(project_name=\"chatbot\", auto_instrument=True)

def handle_message(session_id: str, user_id: str, message: str) -> str:
    with using_attributes(
        session_id=session_id,
        user_id=user_id,
        metadata={\"source\": \"web-chat\"},
        tags=[\"production\", \"v2\"],
    ):
        return chatbot.respond(message)
", Header 2 ("limitations-and-considerations", ["unnumbered", "unlisted"], []) [Str "Limitations and Considerations"], Para [Strong [Str "Product Distinction"], Str ": Arize offers two separate products -- Phoenix (open-source, self-hosted) and Arize AX (commercial cloud platform). These have different APIs, authentication methods, and endpoints. Documentation and SDK imports differ between the two; verify which product you are using before following guides. ", Str "[", Str "1", Str "]"], Para [Strong [Str "Storage Scalability"], Str ": The default SQLite backend is suitable for development and small-scale deployments. Production environments with high trace volumes or concurrent users should use PostgreSQL for better performance and reliability. ", Str "[", Str "7", Str "]"], Para [Strong [Str "Evaluation Latency"], Str ": LLM-based evaluators introduce latency and cost proportional to the number of spans evaluated. For high-throughput applications, consider sampling strategies or asynchronous evaluation pipelines rather than evaluating every trace."], Para [Strong [Str "Auto-Instrumentation Scope"], Str ": Auto-instrumentation captures framework-level operations but cannot instrument custom business logic. Applications with significant custom processing between framework calls require manual instrumentation with decorators or context managers to achieve complete trace coverage. ", Str "[", Str "3", Str "]"], Para [Strong [Str "Local Server Resources"], Str ": Running Phoenix locally with ", Code ("", [], []) "phoenix serve", Str " stores all data in ", Code ("", [], []) "~/.phoenix/", Str " using SQLite. For long-running production use, the working directory can grow substantially; configure ", Code ("", [], []) "PHOENIX_DEFAULT_RETENTION_POLICY_DAYS", Str " to manage data lifecycle. ", Str "[", Str "7", Str "]"], Para [Strong [Str "TypeScript Parity"], Str ": While Phoenix provides Python and TypeScript SDKs, the Python ecosystem has broader auto-instrumentation coverage and more mature evaluation tooling. TypeScript support relies on standard OpenTelemetry configuration rather than the streamlined ", Code ("", [], []) "register()", Str " API available in Python."], Para [Strong [Str "Cold Start"], Str ": The Phoenix server requires initialization time to load the database and start the web UI. In containerized environments, account for this startup delay in health check configurations."], Header 2 ("changelog-highlights", ["unnumbered", "unlisted"], []) [Str "Changelog Highlights"], BulletList [[Plain [Strong [Str "OpenInference Instrumentation"], Str ": Expanded auto-instrumentation support for OpenAI (including Agents SDK), Anthropic, CrewAI, Pydantic AI, Autogen, Mastra, and Vercel AI SDK"]], [Plain [Strong [Str "Prompt Management"], Str ": Version-controlled prompt storage with Mustache and f-string template support, plus SDK-based retrieval for production deployment"]], [Plain [Strong [Str "Experiments Framework"], Str ": Systematic comparison of application versions with dataset-driven evaluation and side-by-side result analysis"]], [Plain [Strong [Str "Nine Span Kinds"], Str ": Expanded from initial span types to include AGENT, GUARDRAIL, and EVALUATOR span categories"]], [Plain [Strong [Str "PostgreSQL Support"], Str ": Production-grade database backend as alternative to SQLite"]], [Plain [Strong [Str "Authentication and RBAC"], Str ": OAuth2, LDAP, and local account authentication with role-based access controls"]], [Plain [Strong [Str "Prometheus Metrics"], Str ": Optional metrics export for integration with existing monitoring infrastructure"]], [Plain [Strong [Str "Docker Non-Root Images"], Str ": Security-hardened container images (", Code ("", [], []) "phoenix:latest-nonroot", Str ") for production Kubernetes deployments ", Str "[", Str "1", Str "]", Str "[", Str "2", Str "]"]]], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " Phoenix Documentation - ", Link ("", [], []) [Str "https://arize.com/docs/phoenix"] ("https://arize.com/docs/phoenix", "")]], [Plain [Str "[", Str "2", Str "]", Str " Phoenix GitHub Repository - ", Link ("", [], []) [Str "https://github.com/Arize-AI/phoenix"] ("https://github.com/Arize-AI/phoenix", "")]], [Plain [Str "[", Str "3", Str "]", Str " Manual Instrumentation Guide - ", Link ("", [], []) [Str "https://arize.com/docs/phoenix/tracing/how-to-tracing/setup-tracing/instrument"] ("https://arize.com/docs/phoenix/tracing/how-to-tracing/setup-tracing/instrument", "")]], [Plain [Str "[", Str "4", Str "]", Str " Building Custom Evaluators - ", Link ("", [], []) [Str "https://arize.com/docs/phoenix/evaluation/concepts-evals/building-your-own-evals"] ("https://arize.com/docs/phoenix/evaluation/concepts-evals/building-your-own-evals", "")]], [Plain [Str "[", Str "5", Str "]", Str " Running Experiments - ", Link ("", [], []) [Str "https://arize.com/docs/phoenix/datasets-and-experiments/how-to-experiments/run-experiments"] ("https://arize.com/docs/phoenix/datasets-and-experiments/how-to-experiments/run-experiments", "")]], [Plain [Str "[", Str "6", Str "]", Str " Prompt Management - ", Link ("", [], []) [Str "https://arize.com/docs/phoenix/prompt-engineering/overview-prompts/prompt-management"] ("https://arize.com/docs/phoenix/prompt-engineering/overview-prompts/prompt-management", "")]], [Plain [Str "[", Str "7", Str "]", Str " Self-Hosting Configuration - ", Link ("", [], []) [Str "https://arize.com/docs/phoenix/self-hosting/configuration"] ("https://arize.com/docs/phoenix/self-hosting/configuration", "")]], [Plain [Str "[", Str "8", Str "]", Str " Python Client SDK - ", Link ("", [], []) [Str "https://arize.com/docs/phoenix/sdk-api-reference/python/arize-phoenix-client"] ("https://arize.com/docs/phoenix/sdk-api-reference/python/arize-phoenix-client", "")]]], Header 2 ("discovery-signals", ["unnumbered", "unlisted"], []) [Str "Discovery Signals"], BlockQuote [Para [Str "Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search."]], Header 3 ("keywords", ["unnumbered", "unlisted"], []) [Str "Keywords"], BulletList [[Plain [Str "Arize Phoenix"]], [Plain [Str "AI observability"]], [Plain [Str "OpenTelemetry"]], [Plain [Str "OpenInference"]], [Plain [Str "OTLP"]], [Plain [Str "distributed tracing"]], [Plain [Str "span kinds"]], [Plain [Str "LLM span"]], [Plain [Str "EMBEDDING span"]], [Plain [Str "RETRIEVER span"]], [Plain [Str "RERANKER span"]], [Plain [Str "AGENT span"]], [Plain [Str "GUARDRAIL span"]], [Plain [Str "EVALUATOR span"]], [Plain [Str "self-hosted"]], [Plain [Str "SQLite"]], [Plain [Str "PostgreSQL"]], [Plain [Str "prompt playground"]], [Plain [Str "prompt versioning"]], [Plain [Str "experiments"]], [Plain [Str "datasets"]], [Plain [Str "annotations"]], [Plain [Str "evaluators"]], [Plain [Str "create_classifier"]], [Plain [Str "span replay"]], [Plain [Str "arize-phoenix-otel"]], [Plain [Str "Arize AX"]]], Header 3 ("verb-noun-tasks", ["unnumbered", "unlisted"], []) [Str "Verb-Noun Tasks"], BulletList [[Plain [Str "Stand up a Phoenix server locally with ", Code ("", [], []) "phoenix serve"]], [Plain [Str "Send OTLP traces over HTTP or gRPC from any language"]], [Plain [Str "Auto-instrument OpenAI, LangChain, LlamaIndex, CrewAI, DSPy"]], [Plain [Str "Decorate functions with ", Code ("", [], []) "@tracer.chain", Str ", ", Code ("", [], []) "@tracer.tool", Str ", ", Code ("", [], []) "@tracer.agent"]], [Plain [Str "Create LLM-as-judge classifiers with ", Code ("", [], []) "create_classifier"]], [Plain [Str "Run experiments comparing prompt versions"]], [Plain [Str "Replay production spans against new prompts in the Playground"]], [Plain [Str "Store datasets in versioned form for benchmarks"]], [Plain [Str "Annotate spans with human ground-truth labels"]], [Plain [Str "Configure PostgreSQL backend for production deployments"]], [Plain [Str "Export traces from Ragas/Deepeval as annotations"]], [Plain [Str "Suppress tracing for sensitive operations"]]], Header 3 ("user-intent-phrases", ["unnumbered", "unlisted"], []) [Str "User Intent Phrases"], BulletList [[Plain [Str "I want OpenTelemetry-native LLM observability."]], [Plain [Str "How do I trace a RAG pipeline with retriever and reranker spans?"]], [Plain [Str "I need to debug an agent's tool-call loop."]], [Plain [Str "How do I run experiments to compare prompt versions?"]], [Plain [Str "I want a Prompt Playground that can replay real production spans."]], [Plain [Str "How do I self-host an open-source observability platform with SQLite or Postgres?"]], [Plain [Str "I want to send traces from CrewAI / DSPy / Haystack to one dashboard."]], [Plain [Str "How do I build LLM-as-judge evaluators in Python?"]], [Plain [Str "I need nine span kinds including AGENT, GUARDRAIL, and EVALUATOR."]], [Plain [Str "What's the difference between Phoenix and Arize AX?"]]], Header 3 ("problem-statements", ["unnumbered", "unlisted"], []) [Str "Problem Statements"], BulletList [[Plain [Str "Generic OpenTelemetry tools don't understand LLM semantics."]], [Plain [Str "Agents loop infinitely or call wrong tools and we can't see why."]], [Plain [Str "RAG retrieval quality is invisible without per-step relevance scores."]], [Plain [Str "Prompt iterations break in production with no replay capability."]], [Plain [Str "SQLite is fine for dev but won't scale; we need PostgreSQL."]], [Plain [Str "Auto-instrumentation misses our custom business logic."]]], Header 3 ("when-to-pick-this", ["unnumbered", "unlisted"], []) [Str "When to Pick This"], BulletList [[Plain [Str "Pick this when your observability stack is already OpenTelemetry/OTLP-centric and you want LLM-aware spans on top."]], [Plain [Str "Pick this over LangSmith when you want fully open-source, self-hostable, and framework-agnostic."]], [Plain [Str "Pick this over Langfuse when you want OpenInference semantic conventions and span-kind richness (AGENT, GUARDRAIL, EVALUATOR)."]], [Plain [Str "Pick this over Helicone when you need full tracing/evals/experiments (not gateway logging)."]], [Plain [Str "Pick this over Weights & Biases when LLM observability — not ML experiments — is the focus."]], [Plain [Str "Pick this when you want span replay against real production data in a Prompt Playground."]]], Header 3 ("related-terms-and-aliases", ["unnumbered", "unlisted"], []) [Str "Related Terms and Aliases"], BulletList [[Plain [Str "arize-phoenix-otel"]], [Plain [Str "arize-phoenix-client"]], [Plain [Str "arize-phoenix-evals"]], [Plain [Str "OpenInference instrumentation"]], [Plain [Str "nine span kinds"]], [Plain [Str "Arize AX (commercial counterpart)"]], [Plain [Str "phoenix serve"]], [Plain [Str "~", Str "/.phoenix/ working directory"]], [Plain [Str "using_attributes context manager"]], [Plain [Str "LLMInstrumentor classes"]]]]