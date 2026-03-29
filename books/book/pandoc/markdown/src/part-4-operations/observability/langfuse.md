[Header 1 ("langfuse", [], []) [Str "Langfuse"], BlockQuote [Para [Str "Open-source LLM engineering platform with observability and prompt management"]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.5)), (AlignDefault, (ColWidth 0.5))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Observability & LLM Ops"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "API/SDK/UI"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Yes"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "https://github.com/langfuse/langfuse"] ("https://github.com/langfuse/langfuse", "")]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "22154"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Documentation"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "Official Docs"] ("https://langfuse.com/docs", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "Langfuse is an open-source LLM engineering platform that provides comprehensive observability, prompt management, and evaluation capabilities for Large Language Model (LLM) applications. Designed for teams that need to collaboratively debug, analyze, and iterate on their LLM-powered systems, Langfuse captures detailed traces of every request flowing through an application, including LLM calls, retrieval operations, embedding generation, tool executions, and arbitrary API interactions. ", Str "[", Str "1", Str "]"], Para [Str "The platform is built on OpenTelemetry standards, reducing vendor lock-in while providing LLM-specific instrumentation that general-purpose observability tools lack. Langfuse natively understands token usage, model parameters, prompt/completion pairs, and evaluation scores. It is available as a managed cloud service (with EU and US regions) or as a fully self-hostable deployment under the MIT license, making it suitable for organizations with strict data residency and privacy requirements. ", Str "[", Str "1", Str "]", Str "[", Str "2", Str "]"], Para [Str "Langfuse spans three primary capability areas: ", Strong [Str "Observability"], Str " for tracing and debugging LLM application execution flows, ", Strong [Str "Prompt Management"], Str " for versioning, testing, and deploying prompts without code changes, and ", Strong [Str "Evaluation"], Str " for assessing output quality through automated, human, and model-based scoring methods. ", Str "[", Str "1", Str "]"], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], Header 3 ("traces", ["unnumbered", "unlisted"], []) [Str "Traces"], Para [Str "A trace represents the complete lifecycle of a single request as it flows through an LLM application. Each trace captures the full execution path, from initial input to final output, with timing, cost, and metadata at every step. Traces are the fundamental unit of observability in Langfuse and serve as the container for all nested observations. ", Str "[", Str "3", Str "]"], Header 3 ("observations", ["unnumbered", "unlisted"], []) [Str "Observations"], Para [Str "Observations are the building blocks within a trace. They represent individual operations such as LLM calls, retrieval steps, tool executions, or custom logic. Observations can be nested to reflect the hierarchical structure of an application. There are two primary observation types:"], BulletList [[Plain [Strong [Str "Spans"], Str ": Generic operations representing any unit of work (data processing, API calls, retrieval steps)"]], [Plain [Strong [Str "Generations"], Str ": LLM-specific operations that capture model name, token usage, prompt/completion pairs, and cost information ", Str "[", Str "3", Str "]"]]], Header 3 ("sessions", ["unnumbered", "unlisted"], []) [Str "Sessions"], Para [Str "Sessions group multiple traces together to represent multi-turn conversations or related interactions from the same user. This enables tracking of conversation flows across multiple requests, making it possible to analyze user journeys and multi-step agent workflows. ", Str "[", Str "3", Str "]"], Header 3 ("scores", ["unnumbered", "unlisted"], []) [Str "Scores"], Para [Str "Scores are evaluation results attached to traces or observations. They come in three types: ", Strong [Str "numeric"], Str " (continuous values like 0-1 quality ratings), ", Strong [Str "boolean"], Str " (pass/fail assessments), and ", Strong [Str "categorical"], Str " (classification labels). Scores can originate from automated LLM-as-a-judge evaluators, user feedback collected through the application frontend, manual human annotation, or programmatic custom metrics. ", Str "[", Str "5", Str "]"], Header 3 ("datasets", ["unnumbered", "unlisted"], []) [Str "Datasets"], Para [Str "Datasets are collections of input and expected output pairs used for systematic testing. Each dataset item contains an input (any structured object), an optional expected output for comparison, and optional metadata. Datasets can be populated from production traces where application performance was suboptimal, enabling a feedback loop from production to development. ", Str "[", Str "7", Str "]"], Header 3 ("prompts", ["unnumbered", "unlisted"], []) [Str "Prompts"], Para [Str "Prompts in Langfuse are versioned, managed artifacts that can be deployed to production via labels without code changes. They come in two types: ", Strong [Str "text prompts"], Str " (single string templates) and ", Strong [Str "chat prompts"], Str " (arrays of message objects with roles). Both support variable interpolation using ", Code ("", [], []) "{{variable}}", Str " syntax. ", Str "[", Str "6", Str "]"], Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Header 3 ("system-components", ["unnumbered", "unlisted"], []) [Str "System Components"], Para [Str "Langfuse consists of two primary application containers backed by a multi-database storage layer:"], CodeBlock ("", [""], []) "                    ┌─────────────────────┐
                    │    Langfuse Web      │
                    │  (UI + REST API)     │
                    └────────┬────────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
     ┌────────▼───┐  ┌──────▼─────┐  ┌────▼────────┐
     │ PostgreSQL  │  │ ClickHouse │  │ Redis/Valkey│
     │(operational)│  │  (OLAP)    │  │  (cache)    │
     └─────────────┘  └────────────┘  └─────────────┘
                             │
                    ┌────────▼────────────┐
                    │   Langfuse Worker   │
                    │ (async processing)  │
                    └─────────────────────┘
                             │
                    ┌────────▼────────────┐
                    │   S3/Blob Storage   │
                    │  (event persistence)│
                    └─────────────────────┘
", BulletList [[Plain [Strong [Str "Langfuse Web"], Str ": Serves the dashboard UI and all REST API endpoints"]], [Plain [Strong [Str "Langfuse Worker"], Str ": Handles asynchronous event processing, LLM-as-a-judge evaluations, batch exports, and background migrations"]], [Plain [Strong [Str "PostgreSQL"], Str ": Stores operational data (projects, API keys, prompt definitions, dataset configurations)"]], [Plain [Strong [Str "ClickHouse"], Str ": High-performance OLAP database optimized for storing and querying traces, observations, and scores at scale"]], [Plain [Strong [Str "Redis/Valkey"], Str ": Provides in-memory caching for API key validation, prompt caching, and job queue management"]], [Plain [Strong [Str "S3/Blob Storage"], Str ": Persists incoming trace events and stores multi-modal attachments, providing recoverability if databases are temporarily unavailable ", Str "[", Str "8", Str "]"]]], Header 3 ("data-flow", ["unnumbered", "unlisted"], []) [Str "Data Flow"], Para [Str "Trace events are queued locally in the SDK and flushed in batches asynchronously, ensuring that application response times are not affected by observability overhead. On the server side, incoming events are persisted to S3 before being processed into ClickHouse, preventing data loss during traffic spikes. API key caching in Redis reduces database load, and prompt caching accelerates SDK responses for frequently fetched prompts. ", Str "[", Str "3", Str "]", Str "[", Str "8", Str "]"], Header 3 ("opentelemetry-foundation", ["unnumbered", "unlisted"], []) [Str "OpenTelemetry Foundation"], Para [Str "Langfuse is built on OpenTelemetry, the industry-standard observability framework. The Python SDK uses the ", Code ("", [], []) "@observe", Str " decorator which creates OpenTelemetry spans under the hood. The JavaScript/TypeScript SDK integrates directly with the OpenTelemetry ", Code ("", [], []) "NodeSDK", Str " through a custom ", Code ("", [], []) "LangfuseSpanProcessor", Str ". This architecture enables interoperability with existing OpenTelemetry-instrumented services and reduces vendor lock-in. ", Str "[", Str "2", Str "]"], Header 2 ("key-features-and-functionality", ["unnumbered", "unlisted"], []) [Str "Key Features and Functionality"], Header 3 ("tracing-and-observability", ["unnumbered", "unlisted"], []) [Str "Tracing and Observability"], Para [Str "Langfuse provides comprehensive tracing that captures the full execution flow of LLM applications. Traces include execution timelines for latency debugging, cost and token usage dashboards, and agent graph visualization for complex workflows:"], CodeBlock ("", ["python"], []) "from langfuse import observe, get_client

langfuse = get_client()

@observe()
def rag_pipeline(query: str):
    documents = retrieve_documents(query)
    response = generate_answer(query, documents)
    return response

@observe()
def retrieve_documents(query: str):
    # Automatically captured as a child span
    embeddings = embed_query(query)
    return vector_search(embeddings)

@observe(as_type=\"generation\")
def generate_answer(query: str, context: list):
    # Captured as a generation with LLM-specific metadata
    with langfuse.start_as_current_observation(
        as_type=\"generation\",
        name=\"answer-generation\",
        model=\"gpt-4o\",
        input=[{\"role\": \"user\", \"content\": query}]
    ) as generation:
        answer = call_openai(query, context)
        generation.update(
            output=answer,
            usage_details={\"input_tokens\": 150, \"output_tokens\": 80}
        )
    return answer
", Para [Str "[", Str "3", Str "]", Str "[", Str "4", Str "]"], Header 3 ("user-and-session-tracking", ["unnumbered", "unlisted"], []) [Str "User and Session Tracking"], Para [Str "Associate traces with users and sessions for multi-turn conversation analysis:"], CodeBlock ("", ["python"], []) "from langfuse import observe, propagate_attributes

@observe()
def handle_request(user_id: str, session_id: str, query: str):
    with propagate_attributes(
        user_id=user_id,
        session_id=session_id,
        metadata={\"source\": \"api\"},
        tags=[\"production\", \"v2\"]
    ):
        result = process_query(query)
        return result
", Para [Str "[", Str "4", Str "]"], Header 3 ("prompt-management", ["unnumbered", "unlisted"], []) [Str "Prompt Management"], Para [Str "Create, version, and deploy prompts through the UI, SDK, or REST API. Prompts are deployed to production via labels, enabling non-code prompt updates:"], CodeBlock ("", ["python"], []) "from langfuse import get_client

langfuse = get_client()

# Fetch the production version of a prompt
prompt = langfuse.get_prompt(\"greeting-prompt\", label=\"production\")

# Compile with variables
compiled = prompt.compile(name=\"Alice\", topic=\"weather\")

# Use the compiled prompt with your LLM
response = call_llm(compiled)
", CodeBlock ("", ["python"], []) "# Create a new prompt version programmatically
langfuse.create_prompt(
    name=\"greeting-prompt\",
    prompt=\"Hello {{name}}! Let me help you with {{topic}}.\",
    config={\"temperature\": 0.7, \"model\": \"gpt-4o\"},
    labels=[\"staging\"]
)
", Para [Str "Creating a prompt with an existing name automatically generates a new version rather than overwriting. The ", Code ("", [], []) "production", Str " label is used to designate the version that should be fetched by default in production environments. Prompt performance can be tracked by linking prompts to traces, enabling analysis of quality metrics across prompt versions. ", Str "[", Str "6", Str "]"], Header 3 ("llm-playground", ["unnumbered", "unlisted"], []) [Str "LLM Playground"], Para [Str "The built-in LLM Playground allows interactive testing of prompts with different models, parameters, and variable values directly in the Langfuse UI. This enables rapid iteration without writing code or redeploying the application. ", Str "[", Str "1", Str "]"], Header 3 ("llm-as-a-judge-evaluation", ["unnumbered", "unlisted"], []) [Str "LLM-as-a-Judge Evaluation"], Para [Str "Configure automated evaluators that use an LLM to assess the quality of your application's outputs. Evaluators can target individual observations (completing in seconds), full traces (completing in minutes), or experiment batches:"], CodeBlock ("", ["python"], []) "from langfuse import get_client

langfuse = get_client()

# Fetch a dataset for evaluation
dataset = langfuse.get_dataset(\"qa-test-cases\")

def task(*, item, **kwargs):
    question = item.input
    response = call_llm(question)
    return response

# Run experiment with automatic LLM-as-a-judge evaluation
result = dataset.run_experiment(
    name=\"GPT-4o QA Evaluation\",
    description=\"Testing QA quality with LLM-as-a-judge\",
    task=task
)
langfuse.flush()
", Para [Str "Langfuse provides managed evaluator templates for common dimensions (hallucination detection, context relevance, toxicity, helpfulness) and supports custom evaluator prompts with configurable scoring ranges. Evaluator results include both numerical scores and reasoning explanations. ", Str "[", Str "5", Str "]"], Header 3 ("annotation-queues", ["unnumbered", "unlisted"], []) [Str "Annotation Queues"], Para [Str "Annotation queues streamline the process of human review for large batches of traces, sessions, and observations. Teams create queues with specific scoring criteria, and reviewers work through items systematically, providing human baseline scores that complement automated evaluations. ", Str "[", Str "5", Str "]"], Header 3 ("datasets-and-experiments", ["unnumbered", "unlisted"], []) [Str "Datasets and Experiments"], Para [Str "Datasets enable systematic, reproducible testing of LLM applications. Items can be created manually, populated from production traces, or added via the SDK:"], CodeBlock ("", ["python"], []) "langfuse = get_client()

# Create a dataset
langfuse.create_dataset(
    name=\"evaluation/geography-qa\",
    description=\"Geography question-answer pairs\",
    metadata={\"domain\": \"geography\"}
)

# Add items to the dataset
langfuse.create_dataset_item(
    dataset_name=\"evaluation/geography-qa\",
    input={\"question\": \"What is the capital of France?\"},
    expected_output={\"answer\": \"Paris\"},
    metadata={\"difficulty\": \"easy\"}
)
", Para [Str "Dataset names support folder-like organization using forward slashes (e.g., ", Code ("", [], []) "evaluation/geography-qa", Str "). Optional JSON Schema validation can enforce data quality on dataset items. Each modification to a dataset item generates a new version with timestamp tracking. ", Str "[", Str "7", Str "]"], Header 3 ("cost-and-token-tracking", ["unnumbered", "unlisted"], []) [Str "Cost and Token Tracking"], Para [Str "Langfuse automatically tracks token usage and cost across all LLM calls, providing dashboards for monitoring spend by model, user, session, or time period. This is captured natively through generation observations. ", Str "[", Str "3", Str "]"], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], Header 3 ("rag-application-debugging", ["unnumbered", "unlisted"], []) [Str "RAG Application Debugging"], Para [Str "Trace the full Retrieval-Augmented Generation pipeline from query embedding through vector search, document retrieval, and final generation. Identify bottlenecks in retrieval quality, measure latency at each stage, and evaluate answer quality against expected outputs using datasets."], Header 3 ("agent-workflow-monitoring", ["unnumbered", "unlisted"], []) [Str "Agent Workflow Monitoring"], Para [Str "Visualize complex agent execution graphs with tool calls, reasoning steps, and decision branches. Track multi-step agent workflows across sessions, monitor tool usage patterns, and identify failure modes in autonomous agent behavior."], Header 3 ("prompt-engineering-and-optimization", ["unnumbered", "unlisted"], []) [Str "Prompt Engineering and Optimization"], Para [Str "Use the prompt management system to A/B test prompt variations. Deploy new prompt versions via labels, track performance metrics per version, and use the LLM Playground for rapid iteration before promoting changes to production."], Header 3 ("production-quality-monitoring", ["unnumbered", "unlisted"], []) [Str "Production Quality Monitoring"], Para [Str "Configure LLM-as-a-judge evaluators to continuously assess output quality on live production traces. Set up annotation queues for human review of flagged outputs. Combine automated and human scores to build a comprehensive quality picture."], Header 3 ("cost-optimization", ["unnumbered", "unlisted"], []) [Str "Cost Optimization"], Para [Str "Analyze token usage and cost dashboards to identify expensive operations. Compare model performance across tiers (e.g., GPT-4o versus GPT-4o-mini) using datasets and experiments to find the optimal cost-quality tradeoff."], Header 3 ("multi-turn-conversation-analysis", ["unnumbered", "unlisted"], []) [Str "Multi-Turn Conversation Analysis"], Para [Str "Group related traces into sessions to analyze complete conversation flows. Track user satisfaction across multi-turn interactions, identify conversation abandonment patterns, and measure cumulative cost per conversation."], Header 2 ("api-reference-summary", ["unnumbered", "unlisted"], []) [Str "API Reference Summary"], Header 3 ("rest-api-endpoints", ["unnumbered", "unlisted"], []) [Str "REST API Endpoints"], Para [Str "Langfuse exposes a REST API authenticated via HTTP Basic Auth using the public and secret key pair:"], CodeBlock ("", ["bash"], []) "# Authentication format
curl -u \"pk-lf-...:sk-lf-...\" \"https://cloud.langfuse.com/api/public/...\"
", Para [Str "Key endpoints:"], BulletList [[Plain [Code ("", [], []) "POST /api/public/ingestion", Str " -- Ingest trace events (primary SDK endpoint)"]], [Plain [Code ("", [], []) "GET /api/public/traces", Str " -- List traces with filtering"]], [Plain [Code ("", [], []) "GET /api/public/traces/{traceId}", Str " -- Retrieve a specific trace"]], [Plain [Code ("", [], []) "GET /api/public/observations", Str " -- List observations"]], [Plain [Code ("", [], []) "POST /api/public/scores", Str " -- Create a score on a trace or observation"]], [Plain [Code ("", [], []) "GET /api/public/scores", Str " -- List scores with filtering"]], [Plain [Code ("", [], []) "GET /api/public/v2/prompts", Str " -- List all prompts"]], [Plain [Code ("", [], []) "GET /api/public/v2/prompts/{promptName}", Str " -- Get a specific prompt"]], [Plain [Code ("", [], []) "POST /api/public/v2/prompts", Str " -- Create a new prompt version"]], [Plain [Code ("", [], []) "GET /api/public/v2/datasets", Str " -- List datasets"]], [Plain [Code ("", [], []) "POST /api/public/v2/datasets", Str " -- Create a dataset"]], [Plain [Code ("", [], []) "POST /api/public/v2/dataset-items", Str " -- Create a dataset item"]], [Plain [Code ("", [], []) "GET /api/public/v2/datasets/{datasetName}/runs", Str " -- List experiment runs ", Str "[", Str "6", Str "]", Str "[", Str "7", Str "]"]]], Header 3 ("sdk-methods-python", ["unnumbered", "unlisted"], []) [Str "SDK Methods (Python)"], BulletList [[Plain [Code ("", [], []) "get_client()", Str " -- Access the globally initialized Langfuse client"]], [Plain [Code ("", [], []) "@observe()", Str " -- Decorator for automatic trace/span creation"]], [Plain [Code ("", [], []) "langfuse.get_prompt(name, label)", Str " -- Fetch a managed prompt"]], [Plain [Code ("", [], []) "langfuse.create_prompt(name, prompt, config, labels)", Str " -- Create a prompt version"]], [Plain [Code ("", [], []) "langfuse.create_dataset(name, description, metadata)", Str " -- Create a dataset"]], [Plain [Code ("", [], []) "langfuse.create_dataset_item(dataset_name, input, expected_output)", Str " -- Add dataset item"]], [Plain [Code ("", [], []) "langfuse.get_dataset(name)", Str " -- Fetch a dataset"]], [Plain [Code ("", [], []) "dataset.run_experiment(name, task)", Str " -- Run an experiment against a dataset"]], [Plain [Code ("", [], []) "langfuse.update_current_trace(input, output)", Str " -- Update the active trace"]], [Plain [Code ("", [], []) "langfuse.start_as_current_observation(as_type, name, model)", Str " -- Create a nested observation"]], [Plain [Code ("", [], []) "langfuse.flush()", Str " -- Flush all pending events to the server"]], [Plain [Code ("", [], []) "propagate_attributes(user_id, session_id, metadata, tags)", Str " -- Set trace-level attributes ", Str "[", Str "4", Str "]"]]], Header 3 ("sdk-methods-javascripttypescript", ["unnumbered", "unlisted"], []) [Str "SDK Methods (JavaScript/TypeScript)"], BulletList [[Plain [Code ("", [], []) "LangfuseSpanProcessor", Str " -- OpenTelemetry span processor for Langfuse"]], [Plain [Code ("", [], []) "startActiveObservation(name, callback)", Str " -- Create a traced observation with automatic context"]], [Plain [Code ("", [], []) "startObservation(name, attributes, options)", Str " -- Create a manual observation"]], [Plain [Code ("", [], []) "propagateAttributes(attributes, callback)", Str " -- Propagate trace attributes to children"]], [Plain [Code ("", [], []) "updateActiveTrace(attributes)", Str " -- Update the current trace"]], [Plain [Code ("", [], []) "observeOpenAI(client)", Str " -- Wrap OpenAI client for automatic tracing ", Str "[", Str "2", Str "]"]]], Header 2 ("configuration-and-customization", ["unnumbered", "unlisted"], []) [Str "Configuration and Customization"], Header 3 ("environment-variables", ["unnumbered", "unlisted"], []) [Str "Environment Variables"], BulletList [[Plain [Strong [Code ("", [], []) "LANGFUSE_SECRET_KEY"], Str " -- Secret key for API authentication (required)"]], [Plain [Strong [Code ("", [], []) "LANGFUSE_PUBLIC_KEY"], Str " -- Public key for API authentication (required)"]], [Plain [Strong [Code ("", [], []) "LANGFUSE_BASE_URL"], Str " -- API base URL; defaults to ", Code ("", [], []) "https://cloud.langfuse.com"]], [Plain [Strong [Code ("", [], []) "LANGFUSE_OBSERVE_DECORATOR_IO_CAPTURE_ENABLED"], Str " -- Toggle input/output capture globally (Python)"]]], Header 3 ("decorator-configuration-python", ["unnumbered", "unlisted"], []) [Str "Decorator Configuration (Python)"], BulletList [[Plain [Strong [Code ("", [], []) "name"], Str " -- Custom identifier for the observation (defaults to function name)"]], [Plain [Strong [Code ("", [], []) "as_type"], Str " -- Observation type: ", Code ("", [], []) "\"span\"", Str " (default) or ", Code ("", [], []) "\"generation\"", Str " for LLM calls"]], [Plain [Strong [Code ("", [], []) "capture_input"], Str " -- Whether to record function arguments (default ", Code ("", [], []) "True", Str ")"]], [Plain [Strong [Code ("", [], []) "capture_output"], Str " -- Whether to record function return values (default ", Code ("", [], []) "True", Str ")"]]], CodeBlock ("", ["python"], []) "@observe(name=\"llm-call\", as_type=\"generation\", capture_input=True, capture_output=True)
def my_llm_function(prompt: str):
    return call_llm(prompt)
", Para [Str "[", Str "4", Str "]"], Header 3 ("trace-attributes", ["unnumbered", "unlisted"], []) [Str "Trace Attributes"], BulletList [[Plain [Strong [Code ("", [], []) "user_id"], Str " -- Associate traces with individual users for user-level analytics"]], [Plain [Strong [Code ("", [], []) "session_id"], Str " -- Group traces into sessions for multi-turn conversation tracking"]], [Plain [Strong [Code ("", [], []) "metadata"], Str " -- Arbitrary key-value pairs for custom filtering and organization"]], [Plain [Strong [Code ("", [], []) "tags"], Str " -- String labels for categorizing and filtering traces"]], [Plain [Strong [Code ("", [], []) "environments"], Str " -- Separate traces across application stages (development, staging, production) ", Str "[", Str "3", Str "]"]]], Header 3 ("self-hosted-configuration", ["unnumbered", "unlisted"], []) [Str "Self-Hosted Configuration"], Para [Str "Self-hosted deployments support configuration of authentication/SSO, encryption at rest, data masking, custom base paths, transactional email, and OpenTelemetry export for infrastructure-level observability. Enterprise editions add UI customization and instance management APIs. ", Str "[", Str "8", Str "]"], Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Header 3 ("openai-sdk-drop-in-replacement", ["unnumbered", "unlisted"], []) [Str "OpenAI SDK (Drop-in Replacement)"], Para [Str "Langfuse provides drop-in wrapper functions for the OpenAI SDK that automatically trace all LLM calls without changing application code:"], CodeBlock ("", ["python"], []) "from langfuse.openai import openai

# All OpenAI calls are automatically traced
response = openai.chat.completions.create(
    model=\"gpt-4o\",
    messages=[{\"role\": \"user\", \"content\": \"Hello!\"}]
)
", CodeBlock ("", ["typescript"], []) "import { OpenAI } from \"openai\";
import { observeOpenAI } from \"@langfuse/openai\";

const openai = observeOpenAI(new OpenAI());
// All calls are automatically traced
const response = await openai.chat.completions.create({
  model: \"gpt-4o\",
  messages: [{ role: \"user\", content: \"Hello!\" }]
});
", Para [Str "[", Str "2", Str "]"], Header 3 ("langchain", ["unnumbered", "unlisted"], []) [Str "LangChain"], Para [Str "Langfuse integrates with LangChain through a callback handler that captures chain and agent execution:"], CodeBlock ("", ["python"], []) "from langfuse.callback import CallbackHandler

langfuse_handler = CallbackHandler()

# Pass to any LangChain chain or agent
result = chain.invoke(
    {\"input\": \"What is the weather?\"},
    config={\"callbacks\": [langfuse_handler]}
)
", Para [Str "[", Str "1", Str "]"], Header 3 ("llamaindex", ["unnumbered", "unlisted"], []) [Str "LlamaIndex"], Para [Str "Langfuse provides a callback handler for LlamaIndex to capture query engine and retrieval operations:"], CodeBlock ("", ["python"], []) "from langfuse import get_client

langfuse = get_client()

# LlamaIndex integration via Settings
from llama_index.core import Settings
Settings.callback_manager = langfuse.get_llama_index_handler()
", Para [Str "[", Str "1", Str "]"], Header 3 ("vercel-ai-sdk", ["unnumbered", "unlisted"], []) [Str "Vercel AI SDK"], Para [Str "Enable OpenTelemetry tracing in the Vercel AI SDK to send spans to Langfuse:"], CodeBlock ("", ["typescript"], []) "import { generateText } from \"ai\";
import { openai } from \"@ai-sdk/openai\";

const result = await generateText({
  model: openai(\"gpt-4.1\"),
  prompt: \"Write a short story about a cat.\",
  experimental_telemetry: {
    isEnabled: true,
    functionId: \"my-function\",
    metadata: {
      sessionId: \"123\",
      userId: \"456\",
      tags: [\"production\"],
    },
  },
});
", Para [Str "[", Str "2", Str "]"], Header 3 ("llm-gateways-litellm-portkey", ["unnumbered", "unlisted"], []) [Str "LLM Gateways (LiteLLM, Portkey)"], Para [Str "Langfuse integrates with LLM gateway/proxy layers that provide unified access to multiple LLM providers. Gateways can forward trace data to Langfuse for centralized observability across all provider calls. ", Str "[", Str "1", Str "]"], Header 3 ("opentelemetry-native", ["unnumbered", "unlisted"], []) [Str "OpenTelemetry Native"], Para [Str "Any application instrumented with OpenTelemetry can send traces to Langfuse through the OpenTelemetry protocol (OTLP), enabling integration from any programming language that supports OpenTelemetry. ", Str "[", Str "2", Str "]"], Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Header 3 ("full-rag-pipeline-with-tracing", ["unnumbered", "unlisted"], []) [Str "Full RAG Pipeline with Tracing"], CodeBlock ("", ["python"], []) "from langfuse import observe, get_client, propagate_attributes

langfuse = get_client()

@observe()
def rag_pipeline(user_id: str, session_id: str, query: str):
    with propagate_attributes(user_id=user_id, session_id=session_id):
        documents = retrieve(query)
        answer = generate(query, documents)
        langfuse.update_current_trace(
            input={\"query\": query},
            output={\"answer\": answer}
        )
        return answer

@observe()
def retrieve(query: str):
    embedding = embed(query)
    results = vector_search(embedding, top_k=5)
    return results

@observe(as_type=\"generation\")
def generate(query: str, context: list):
    with langfuse.start_as_current_observation(
        as_type=\"generation\",
        name=\"answer-llm\",
        model=\"gpt-4o\",
        input=[
            {\"role\": \"system\", \"content\": \"Answer based on the provided context.\"},
            {\"role\": \"user\", \"content\": f\"Context: {context}\\n\\nQuestion: {query}\"}
        ]
    ) as generation:
        answer = call_openai(query, context)
        generation.update(
            output=answer,
            usage_details={\"input_tokens\": 500, \"output_tokens\": 150}
        )
    return answer

rag_pipeline(\"user_42\", \"session_abc\", \"What are the benefits of RAG?\")
langfuse.flush()
", Header 3 ("prompt-management-workflow", ["unnumbered", "unlisted"], []) [Str "Prompt Management Workflow"], CodeBlock ("", ["python"], []) "from langfuse import get_client

langfuse = get_client()

# Create initial prompt version
langfuse.create_prompt(
    name=\"qa-system-prompt\",
    type=\"chat\",
    prompt=[
        {\"role\": \"system\", \"content\": \"You are a helpful assistant. Answer based on the context provided.\"},
        {\"role\": \"user\", \"content\": \"Context: {{context}}\\n\\nQuestion: {{question}}\"}
    ],
    config={\"temperature\": 0.3, \"model\": \"gpt-4o\"},
    labels=[\"staging\"]
)

# Fetch and use in production
prompt = langfuse.get_prompt(\"qa-system-prompt\", label=\"production\")
messages = prompt.compile(context=\"Langfuse is an LLM platform.\", question=\"What is Langfuse?\")

# Call LLM with managed prompt
response = call_llm(messages)
", Header 3 ("dataset-driven-experiment", ["unnumbered", "unlisted"], []) [Str "Dataset-Driven Experiment"], CodeBlock ("", ["python"], []) "from langfuse import get_client
from openai import OpenAI

langfuse = get_client()
openai_client = OpenAI()

# Create a dataset
langfuse.create_dataset(name=\"qa-golden-set\")
langfuse.create_dataset_item(
    dataset_name=\"qa-golden-set\",
    input={\"question\": \"What is the capital of France?\"},
    expected_output={\"answer\": \"Paris\"}
)
langfuse.create_dataset_item(
    dataset_name=\"qa-golden-set\",
    input={\"question\": \"What is the largest ocean?\"},
    expected_output={\"answer\": \"Pacific Ocean\"}
)

# Define the task function
def qa_task(*, item, **kwargs):
    response = openai_client.chat.completions.create(
        model=\"gpt-4o\",
        messages=[{\"role\": \"user\", \"content\": item.input[\"question\"]}]
    )
    return response.choices[0].message.content

# Run the experiment
dataset = langfuse.get_dataset(\"qa-golden-set\")
result = dataset.run_experiment(
    name=\"GPT-4o Baseline\",
    description=\"Baseline QA evaluation with GPT-4o\",
    task=qa_task
)
langfuse.flush()
", Header 3 ("rest-api-prompt-management", ["unnumbered", "unlisted"], []) [Str "REST API Prompt Management"], CodeBlock ("", ["bash"], []) "# List all prompts
curl -u \"pk-lf-...:sk-lf-...\" \\
  \"https://cloud.langfuse.com/api/public/v2/prompts\"

# Get a specific prompt by name
curl -u \"pk-lf-...:sk-lf-...\" \\
  \"https://cloud.langfuse.com/api/public/v2/prompts/greeting-prompt\"

# Create a new prompt version
curl -X POST \\
  -u \"pk-lf-...:sk-lf-...\" \\
  -H \"Content-Type: application/json\" \\
  -d '{
    \"name\": \"greeting-prompt\",
    \"prompt\": \"Hello {{name}}! Let me help you with {{topic}}.\",
    \"config\": {\"temperature\": 0.7},
    \"labels\": [\"production\"]
  }' \\
  \"https://cloud.langfuse.com/api/public/v2/prompts\"
", Header 2 ("limitations-and-considerations", ["unnumbered", "unlisted"], []) [Str "Limitations and Considerations"], BulletList [[Plain [Strong [Str "Asynchronous ingestion"], Str ": Trace events are queued and flushed in batches; in short-lived processes (serverless functions, scripts), ", Code ("", [], []) "langfuse.flush()", Str " must be called explicitly before shutdown to avoid data loss ", Str "[", Str "3", Str "]"]], [Plain [Strong [Str "ClickHouse dependency"], Str ": Self-hosted deployments require ClickHouse in addition to PostgreSQL, adding operational complexity compared to single-database platforms ", Str "[", Str "8", Str "]"]], [Plain [Strong [Str "LLM-as-a-judge cost"], Str ": Automated model-based evaluations incur additional LLM API costs for the judge model; high-capability models (GPT-4o, Claude Sonnet) are recommended for scoring accuracy, increasing evaluation expenses ", Str "[", Str "5", Str "]"]], [Plain [Strong [Str "Evaluation latency"], Str ": Trace-level LLM-as-a-judge evaluations can take minutes to complete due to the complexity of assembling full trace context for the judge model ", Str "[", Str "5", Str "]"]], [Plain [Strong [Str "OpenTelemetry complexity"], Str ": The JavaScript/TypeScript SDK requires explicit OpenTelemetry setup (NodeSDK, span processors), which adds initialization boilerplate compared to the Python decorator approach ", Str "[", Str "2", Str "]"]], [Plain [Strong [Str "Self-hosting infrastructure"], Str ": Production-grade self-hosted deployments require PostgreSQL, ClickHouse, Redis/Valkey, and S3-compatible storage, representing a significant infrastructure footprint ", Str "[", Str "8", Str "]"]], [Plain [Strong [Str "Prompt type immutability"], Str ": Once a prompt is created as either text or chat type, the type cannot be changed; a new prompt with a different name must be created instead ", Str "[", Str "6", Str "]"]]], Header 2 ("changelog-highlights", ["unnumbered", "unlisted"], []) [Str "Changelog Highlights"], BulletList [[Plain [Strong [Str "All product features open-sourced"], Str " (MIT license): LLM-as-a-judge evaluations, annotation queues, prompt experiments, and the LLM Playground are all available for self-hosted deployments ", Str "[", Str "1", Str "]"]], [Plain [Strong [Str "Langfuse v3 stable release"], Str ": New architecture with ClickHouse for OLAP queries, S3-based event persistence, and the worker container for scalable async processing ", Str "[", Str "8", Str "]"]], [Plain [Strong [Str "OpenTelemetry foundation"], Str ": SDK rebuilt on OpenTelemetry standards for cross-language interoperability and reduced vendor lock-in ", Str "[", Str "2", Str "]"]], [Plain [Strong [Str "Python SDK v3"], Str ": Generally available with ", Code ("", [], []) "@observe", Str " decorator, ", Code ("", [], []) "get_client()", Str " global access, and ", Code ("", [], []) "start_as_current_observation", Str " context manager ", Str "[", Str "4", Str "]"]], [Plain [Strong [Str "JavaScript/TypeScript SDK v4"], Str ": OpenTelemetry-native with ", Code ("", [], []) "LangfuseSpanProcessor", Str ", ", Code ("", [], []) "startActiveObservation", Str ", and ", Code ("", [], []) "observeOpenAI", Str " wrapper ", Str "[", Str "2", Str "]"]], [Plain [Strong [Str "Prompt experiments"], Str ": Run experiments against datasets directly within Langfuse for systematic prompt evaluation ", Str "[", Str "7", Str "]"]], [Plain [Strong [Str "Managed evaluators"], Str ": Pre-built LLM-as-a-judge templates for hallucination, context relevance, toxicity, and helpfulness ", Str "[", Str "5", Str "]"]], [Plain [Strong [Str "50+ framework integrations"], Str ": Including OpenAI, LangChain, LlamaIndex, Vercel AI SDK, LiteLLM, and OpenTelemetry native support ", Str "[", Str "1", Str "]"]], [Plain [Strong [Str "Dataset schema validation"], Str ": Optional JSON Schema enforcement on dataset items for data quality ", Str "[", Str "7", Str "]"]], [Plain [Strong [Str "Batch exports"], Str ": Export large volumes of trace data as CSV/JSON from the UI ", Str "[", Str "8", Str "]"]]], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " Langfuse Documentation - ", Link ("", [], []) [Str "https://langfuse.com/docs"] ("https://langfuse.com/docs", "")]], [Plain [Str "[", Str "2", Str "]", Str " Get Started Guide - ", Link ("", [], []) [Str "https://langfuse.com/docs/get-started"] ("https://langfuse.com/docs/get-started", "")]], [Plain [Str "[", Str "3", Str "]", Str " Tracing Overview - ", Link ("", [], []) [Str "https://langfuse.com/docs/tracing"] ("https://langfuse.com/docs/tracing", "")]], [Plain [Str "[", Str "4", Str "]", Str " Python SDK Decorators - ", Link ("", [], []) [Str "https://langfuse.com/docs/sdk/python/decorators"] ("https://langfuse.com/docs/sdk/python/decorators", "")]], [Plain [Str "[", Str "5", Str "]", Str " Evaluation Overview - ", Link ("", [], []) [Str "https://langfuse.com/docs/scores/overview"] ("https://langfuse.com/docs/scores/overview", "")]], [Plain [Str "[", Str "6", Str "]", Str " Prompt Management - ", Link ("", [], []) [Str "https://langfuse.com/docs/prompts/get-started"] ("https://langfuse.com/docs/prompts/get-started", "")]], [Plain [Str "[", Str "7", Str "]", Str " Datasets Overview - ", Link ("", [], []) [Str "https://langfuse.com/docs/datasets/overview"] ("https://langfuse.com/docs/datasets/overview", "")]], [Plain [Str "[", Str "8", Str "]", Str " Self-Hosting Guide - ", Link ("", [], []) [Str "https://langfuse.com/docs/deployment/self-host"] ("https://langfuse.com/docs/deployment/self-host", "")]]]]