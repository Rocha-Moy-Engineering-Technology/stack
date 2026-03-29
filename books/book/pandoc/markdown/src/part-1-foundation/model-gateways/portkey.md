[Header 1 ("portkey", [], []) [Str "Portkey"], BlockQuote [Para [Str "AI gateway for managing 200+ LLMs with observability and guardrails"]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.5)), (AlignDefault, (ColWidth 0.5))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "API Gateways & Model Routing"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "API/SDK"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Yes"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "Portkey-AI/gateway"] ("https://github.com/Portkey-AI/gateway", "")]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "10675"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Documentation"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "Official Docs"] ("https://portkey.ai/docs/introduction/what-is-portkey", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "Portkey is a unified AI gateway that provides a single interface for routing requests to 250+ Large Language Model (LLM) providers. It operates as a proxy layer between applications and LLM APIs, adding observability, guardrails, caching, fallback routing, and load balancing without requiring changes to the underlying model calls. The gateway runs on globally distributed edge workers, adding approximately 20-40ms of latency compared to direct API calls. ", Str "[", Str "1", Str "]"], Para [Str "The platform processes over 25 million requests daily with 99.99% uptime and handles millions of requests per minute at scale. Integration takes approximately 2 minutes through native SDKs (Python, Node.js), a REST API, or drop-in compatibility with the OpenAI SDK by changing the base URL. Portkey holds ISO 27001, SOC 2, GDPR, and HIPAA certifications, with AES-256 encryption for data in transit and at rest. ", Str "[", Str "1", Str "]"], Para [Str "Portkey ships as both an open-source gateway (free, self-hosted) and a managed cloud service. The managed service includes a free tier of 10,000 requests per month, with paid plans for higher volumes. Enterprise customers can deploy Portkey in a private cloud configuration. ", Str "[", Str "1", Str "]"], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], Para [Strong [Str "Gateway Configs"], Str " are JSON objects that define how Portkey processes requests. A config specifies the routing strategy, target providers, caching behavior, retry logic, guardrails, and request timeouts. Configs are the central orchestration mechanism for all gateway features and can be stored server-side (referenced by ID) or passed inline with each request. ", Str "[", Str "2", Str "]"], Para [Strong [Str "Model Catalog"], Str " (formerly Virtual Keys) provides a centralized system for managing provider credentials. Instead of embedding API keys in application code, credentials are stored securely in Portkey and referenced using the ", Code ("", [], []) "@provider-slug/model-name", Str " syntax. The Model Catalog supports organization-level credential sharing across workspaces, fine-grained budgets, rate limits, and model allow-lists. ", Str "[", Str "3", Str "]"], Para [Strong [Str "Targets"], Str " are the downstream LLM providers or model endpoints that receive routed requests. Each target in a config specifies a provider, credentials, optional model overrides, and weight (for load balancing). Targets can be nested to create complex routing trees with multiple fallback layers. ", Str "[", Str "2", Str "]"], Para [Strong [Str "Strategy Modes"], Str " define how Portkey distributes requests across targets. The four modes are ", Code ("", [], []) "single", Str " (one provider), ", Code ("", [], []) "loadbalance", Str " (weighted distribution), ", Code ("", [], []) "fallback", Str " (sequential failover), and ", Code ("", [], []) "conditional", Str " (rule-based routing). ", Str "[", Str "2", Str "]"], Para [Strong [Str "Guardrails"], Str " are real-time validators that check inputs before they reach the LLM and outputs before they reach the user. Guardrails can block requests, log violations, trigger fallbacks, or build evaluation datasets. Over 20 deterministic checks are available alongside LLM-based and third-party guardrail integrations. ", Str "[", Str "4", Str "]"], Para [Strong [Str "Observability"], Str " is an OpenTelemetry-compliant monitoring suite that automatically captures all requests, responses, costs, latencies, and token usage. The suite includes logs, distributed tracing, analytics dashboards with 21+ metrics, custom metadata tagging, and feedback integration. ", Str "[", Str "5", Str "]"], Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Para [Str "Portkey operates as an edge-deployed proxy between client applications and LLM providers:"], CodeBlock ("", [""], []) "Application (SDK / REST)
    |
    v
Portkey Gateway (Edge Workers)
    |--- Guardrails (input validation)
    |--- Routing (fallback / loadbalance / conditional)
    |--- Caching (simple / semantic)
    |--- Retry Logic
    |--- Observability (logs, traces, metrics)
    |
    v
LLM Providers (OpenAI, Anthropic, Azure, Bedrock, 250+ others)
", Para [Strong [Str "Edge Infrastructure"], Str ": The gateway runs on globally distributed edge workers, minimizing latency by processing requests close to their origin. The edge layer handles routing decisions, cache lookups, guardrail evaluation, and retry logic before forwarding to the target provider. ", Str "[", Str "1", Str "]"], Para [Strong [Str "Config-Driven Orchestration"], Str ": All gateway behavior is defined through JSON config objects. Configs can be stored server-side and referenced by ID (", Code ("", [], []) "pc-xxx", Str ") or passed inline. Server-side configs enable runtime changes without code deployments. ", Str "[", Str "2", Str "]"], Para [Strong [Str "OpenAI-Compatible Interface"], Str ": Portkey exposes an API surface compatible with the OpenAI Chat Completions format (", Code ("", [], []) "/v1/chat/completions", Str "). Applications using the OpenAI SDK can switch to Portkey by changing the base URL and adding Portkey headers, with no changes to the request body. ", Str "[", Str "1", Str "]"], Para [Strong [Str "Credential Isolation"], Str ": Provider API keys are stored in Portkey's vault (Model Catalog) and never exposed in application code. Requests reference credentials through the ", Code ("", [], []) "@provider-slug", Str " syntax, and Portkey injects the actual key at the edge layer before forwarding to the provider. ", Str "[", Str "3", Str "]"], Header 2 ("key-features-and-functionality", ["unnumbered", "unlisted"], []) [Str "Key Features and Functionality"], Header 3 ("fallback-routing", ["unnumbered", "unlisted"], []) [Str "Fallback Routing"], Para [Str "Automatically switch to backup providers when the primary fails:"], CodeBlock ("", ["json"], []) "{
    \"strategy\": {
        \"mode\": \"fallback\"
    },
    \"targets\": [
        {
            \"virtual_key\": \"openai-key\",
            \"override_params\": {\"model\": \"gpt-4o\"}
        },
        {
            \"virtual_key\": \"anthropic-key\",
            \"override_params\": {\"model\": \"claude-sonnet-4-6\"}
        }
    ]
}
", Para [Str "Fallbacks can be triggered by specific HTTP status codes using ", Code ("", [], []) "on_status_codes", Str " at the target level. ", Str "[", Str "2", Str "]"], Header 3 ("load-balancing", ["unnumbered", "unlisted"], []) [Str "Load Balancing"], Para [Str "Distribute requests across providers or API keys using weighted targets:"], CodeBlock ("", ["json"], []) "{
    \"strategy\": {
        \"mode\": \"loadbalance\"
    },
    \"targets\": [
        {
            \"virtual_key\": \"openai-key-1\",
            \"weight\": 0.7,
            \"override_params\": {\"model\": \"gpt-4o\"}
        },
        {
            \"virtual_key\": \"anthropic-key\",
            \"weight\": 0.3,
            \"override_params\": {\"model\": \"claude-sonnet-4-6\"}
        }
    ]
}
", Para [Str "Weights control traffic distribution probability and are normalized automatically. ", Str "[", Str "2", Str "]"], Header 3 ("conditional-routing", ["unnumbered", "unlisted"], []) [Str "Conditional Routing"], Para [Str "Route requests based on custom criteria using query conditions:"], CodeBlock ("", ["json"], []) "{
    \"strategy\": {
        \"mode\": \"conditional\",
        \"conditions\": [
            {
                \"query\": {\"metadata.tier\": \"premium\"},
                \"then\": \"target-gpt4\"
            }
        ],
        \"default\": \"target-gpt4o-mini\"
    },
    \"targets\": [...]
}
", Header 3 ("caching", ["unnumbered", "unlisted"], []) [Str "Caching"], Para [Str "Reduce latency and costs with simple (exact match) or semantic (similarity-based) caching:"], CodeBlock ("", ["json"], []) "{
    \"cache\": {
        \"mode\": \"semantic\",
        \"max_age\": 3600
    }
}
", BulletList [[Plain [Strong [Str "Simple cache"], Str ": Exact match on request body; fastest lookup"]], [Plain [Strong [Str "Semantic cache"], Str ": Similarity-based matching; returns cached responses for semantically equivalent prompts ", Str "[", Str "2", Str "]"]]], Header 3 ("automatic-retries", ["unnumbered", "unlisted"], []) [Str "Automatic Retries"], Para [Str "Retry failed requests with configurable attempts and status code filters:"], CodeBlock ("", ["json"], []) "{
    \"retry\": {
        \"attempts\": 3,
        \"on_status_codes\": [429, 500, 502, 503, 504],
        \"use_retry_after_headers\": true
    }
}
", Para [Str "The ", Code ("", [], []) "use_retry_after_headers", Str " option respects provider-sent ", Code ("", [], []) "Retry-After", Str " headers for rate-limited requests. ", Str "[", Str "2", Str "]"], Header 3 ("circuit-breaker", ["unnumbered", "unlisted"], []) [Str "Circuit Breaker"], Para [Str "Prevent cascading failures by temporarily disabling unhealthy targets:"], CodeBlock ("", ["json"], []) "{
    \"cb_config\": {
        \"failure_threshold\": 5,
        \"cooldown_interval\": 60000,
        \"failure_status_codes\": [500, 502, 503]
    }
}
", Para [Str "When a target exceeds the ", Code ("", [], []) "failure_threshold", Str ", the circuit opens and requests are routed to other targets for the duration of the ", Code ("", [], []) "cooldown_interval", Str " (minimum 30 seconds). ", Str "[", Str "2", Str "]"], Header 3 ("guardrails", ["unnumbered", "unlisted"], []) [Str "Guardrails"], Para [Str "Validate inputs and outputs with deterministic, LLM-based, or third-party checks:"], CodeBlock ("", ["json"], []) "{
    \"input_guardrails\": [\"guardrail-id-xxx\"],
    \"output_guardrails\": [\"guardrail-id-yyy\"]
}
", Para [Str "Guardrail actions include synchronous blocking (status 446 on failure), asynchronous logging (non-blocking), sequential or parallel execution, and feedback collection for evaluation datasets. Built-in checks cover regex matching, JSON schema validation, code detection (SQL, Python, TypeScript), prompt injection scanning, and gibberish detection. Third-party integrations with Aporia, SydeLabs, and Pillar Security are available. ", Str "[", Str "4", Str "]"], Header 3 ("observability", ["unnumbered", "unlisted"], []) [Str "Observability"], Para [Str "All requests are automatically logged with cost, latency, token usage, and provider metadata. Features include:"], BulletList [[Plain [Strong [Str "Logs"], Str ": Full request and response capture for all multimodal interactions"]], [Plain [Strong [Str "Traces"], Str ": Distributed tracing across the lifecycle of each request"]], [Plain [Strong [Str "Analytics"], Str ": 21+ metrics on dashboards for trend analysis"]], [Plain [Strong [Str "Custom Metadata"], Str ": Tag requests with arbitrary key-value pairs for grouping and filtering"]], [Plain [Strong [Str "Feedback"], Str ": Attach feedback values and weights to close observability loops"]], [Plain [Strong [Str "Budget Limits"], Str ": Configure cost limits per provider API key ", Str "[", Str "5", Str "]"]]], Header 3 ("model-context-protocol-mcp", ["unnumbered", "unlisted"], []) [Str "Model Context Protocol (MCP)"], Para [Str "Connect external tools and data sources to LLM requests through MCP support, enabling agents to access databases, file systems, and APIs through a standardized protocol. ", Str "[", Str "6", Str "]"], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], Para [Strong [Str "Multi-Provider Resilience"], Str ": Route production traffic through Portkey with fallback configs to ensure continuity when a single provider experiences downtime. An application can fall back from OpenAI to Anthropic to Azure OpenAI without any code changes."], Para [Strong [Str "Cost Optimization"], Str ": Use load balancing to distribute traffic across cheaper model tiers for routine queries while routing complex queries to frontier models via conditional routing. Semantic caching further reduces costs by serving cached responses for repeated or similar prompts."], Para [Strong [Str "Compliance and Security"], Str ": Store all provider credentials in the Model Catalog, enforce guardrails on inputs and outputs to prevent prompt injection and data leakage, and enable audit logging through the observability suite. The HIPAA, SOC 2, and GDPR certifications support regulated industry deployments."], Para [Strong [Str "A/B Testing and Canary Deployments"], Str ": Use weighted load balancing to gradually shift traffic from an existing model to a new model, monitoring performance and cost metrics through the analytics dashboard before full rollout."], Para [Strong [Str "Agent Observability"], Str ": Trace multi-step agent workflows across multiple LLM calls, tool invocations, and retrieval steps. Custom metadata tags enable grouping traces by user session, agent type, or business workflow."], Para [Strong [Str "Rate Limit Management"], Str ": Distribute requests across multiple API keys for the same provider using load balancing, effectively multiplying rate limits without application-level key rotation logic."], Header 2 ("api-reference-summary", ["unnumbered", "unlisted"], []) [Str "API Reference Summary"], Header 3 ("endpoints", ["unnumbered", "unlisted"], []) [Str "Endpoints"], Para [Str "Portkey mirrors the OpenAI-compatible API surface:"], BulletList [[Plain [Code ("", [], []) "POST /v1/chat/completions", Str " -- Chat completions (text generation)"]], [Plain [Code ("", [], []) "POST /v1/completions", Str " -- Legacy completions"]], [Plain [Code ("", [], []) "POST /v1/embeddings", Str " -- Vector embeddings"]], [Plain [Code ("", [], []) "POST /v1/images/generations", Str " -- Image generation"]], [Plain [Code ("", [], []) "POST /v1/audio/speech", Str " -- Text-to-speech"]], [Plain [Code ("", [], []) "POST /v1/audio/transcriptions", Str " -- Speech-to-text"]]], Header 3 ("request-headers", ["unnumbered", "unlisted"], []) [Str "Request Headers"], BulletList [[Plain [Code ("", [], []) "x-portkey-api-key", Str " -- Portkey API key (required)"]], [Plain [Code ("", [], []) "x-portkey-config", Str " -- Gateway config ID or inline JSON"]], [Plain [Code ("", [], []) "x-portkey-virtual-key", Str " -- Legacy virtual key reference"]], [Plain [Code ("", [], []) "x-portkey-metadata", Str " -- Custom metadata as JSON string"]], [Plain [Code ("", [], []) "x-portkey-trace-id", Str " -- Custom trace identifier"]], [Plain [Code ("", [], []) "x-portkey-cache-namespace", Str " -- Cache namespace for isolation"]], [Plain [Code ("", [], []) "Authorization", Str " -- Provider API key (when not using Model Catalog)"]]], Header 3 ("response-additions", ["unnumbered", "unlisted"], []) [Str "Response Additions"], Para [Str "Portkey responses include standard OpenAI-format fields plus:"], BulletList [[Plain [Code ("", [], []) "hook_results", Str " -- Guardrail check results (when synchronous guardrails are configured)"]], [Plain [Str "Status 246: Guardrails failed but request continues"]], [Plain [Str "Status 446: Guardrails failed and request is denied"]]], Header 2 ("configuration-and-customization", ["unnumbered", "unlisted"], []) [Str "Configuration and Customization"], Header 3 ("complete-config-structure", ["unnumbered", "unlisted"], []) [Str "Complete Config Structure"], CodeBlock ("", ["json"], []) "{
    \"strategy\": {
        \"mode\": \"fallback | loadbalance | conditional | single\",
        \"conditions\": [],
        \"default\": \"target-id\",
        \"on_status_codes\": [429, 500]
    },
    \"targets\": [
        {
            \"provider\": \"openai\",
            \"api_key\": \"sk-...\",
            \"virtual_key\": \"key-id\",
            \"custom_host\": \"http://private-llm/v1\",
            \"weight\": 0.7,
            \"override_params\": {\"model\": \"gpt-4o\", \"temperature\": 0.5},
            \"forward_headers\": [\"Authorization\"],
            \"on_status_codes\": [500, 502]
        }
    ],
    \"cache\": {
        \"mode\": \"simple | semantic\",
        \"max_age\": 3600
    },
    \"retry\": {
        \"attempts\": 3,
        \"on_status_codes\": [429, 500, 502, 503, 504],
        \"use_retry_after_headers\": true
    },
    \"cb_config\": {
        \"failure_threshold\": 5,
        \"cooldown_interval\": 60000,
        \"failure_status_codes\": [500, 502, 503]
    },
    \"request_timeout\": 30000,
    \"input_guardrails\": [\"guardrail-id\"],
    \"output_guardrails\": [\"guardrail-id\"],
    \"strict_open_ai_compliance\": true,
    \"forward_headers\": [\"X-Custom-Header\"]
}
", Header 3 ("cloud-provider-parameters", ["unnumbered", "unlisted"], []) [Str "Cloud Provider Parameters"], Para [Str "Configs support direct cloud provider authentication:"], BulletList [[Plain [Strong [Str "Azure OpenAI"], Str ": ", Code ("", [], []) "azure_region", Str ", ", Code ("", [], []) "azure_deployment_name", Str ", ", Code ("", [], []) "azure_api_version", Str ", ", Code ("", [], []) "azure_endpoint_name"]], [Plain [Strong [Str "AWS Bedrock"], Str ": ", Code ("", [], []) "aws_access_key_id", Str ", ", Code ("", [], []) "aws_secret_access_key", Str ", ", Code ("", [], []) "aws_region", Str ", ", Code ("", [], []) "aws_session_token"]], [Plain [Strong [Str "Google Vertex AI"], Str ": ", Code ("", [], []) "vertex_project_id", Str ", ", Code ("", [], []) "vertex_region", Str ", ", Code ("", [], []) "vertex_service_account_json"]]], Header 3 ("config-application-methods", ["unnumbered", "unlisted"], []) [Str "Config Application Methods"], Para [Str "Configs can be applied through multiple channels:"], BulletList [[Plain [Str "Portkey SDK ", Code ("", [], []) "config", Str " parameter (inline object or stored config ID)"]], [Plain [Str "OpenAI SDK via ", Code ("", [], []) "x-portkey-config", Str " header"]], [Plain [Str "REST API via ", Code ("", [], []) "x-portkey-config", Str " header"]], [Plain [Str "Default config attached to a Portkey API key in the dashboard"]]], Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Header 3 ("with-openai-sdk-python", ["unnumbered", "unlisted"], []) [Str "With OpenAI SDK (Python)"], CodeBlock ("", ["python"], []) "from openai import OpenAI
from portkey_ai import createHeaders

client = OpenAI(
    api_key=\"dummy\",
    base_url=\"https://api.portkey.ai/v1\",
    default_headers=createHeaders(
        api_key=\"YOUR_PORTKEY_API_KEY\",
        virtual_key=\"YOUR_OPENAI_VIRTUAL_KEY\"
    )
)

response = client.chat.completions.create(
    model=\"gpt-4o\",
    messages=[{\"role\": \"user\", \"content\": \"Hello!\"}]
)
", Header 3 ("with-langchain", ["unnumbered", "unlisted"], []) [Str "With LangChain"], CodeBlock ("", ["python"], []) "from langchain_openai import ChatOpenAI
from portkey_ai import createHeaders

llm = ChatOpenAI(
    api_key=\"dummy\",
    base_url=\"https://api.portkey.ai/v1\",
    default_headers=createHeaders(
        api_key=\"YOUR_PORTKEY_API_KEY\",
        virtual_key=\"YOUR_OPENAI_VIRTUAL_KEY\"
    ),
    model=\"gpt-4o\"
)

response = llm.invoke(\"What is the meaning of life?\")
", Header 3 ("with-observability-tracing-logfire", ["unnumbered", "unlisted"], []) [Str "With Observability Tracing (Logfire)"], CodeBlock ("", ["python"], []) "import logfire
import os
from portkey_ai import createHeaders
from openai import OpenAI

os.environ[\"OTEL_EXPORTER_OTLP_ENDPOINT\"] = \"https://api.portkey.ai/v1/logs/otel\"
os.environ[\"OTEL_EXPORTER_OTLP_HEADERS\"] = \"x-portkey-api-key=YOUR_PORTKEY_API_KEY\"

logfire.configure(service_name=\"my-llm-app\", send_to_logfire=False)

client = OpenAI(
    api_key=\"YOUR_OPENAI_API_KEY\",
    base_url=\"https://api.portkey.ai/v1\",
    default_headers=createHeaders(
        api_key=\"YOUR_PORTKEY_API_KEY\",
    )
)

logfire.instrument_openai(client)
", Header 3 ("with-private-or-self-hosted-models", ["unnumbered", "unlisted"], []) [Str "With Private or Self-Hosted Models"], CodeBlock ("", ["json"], []) "{
    \"strategy\": {
        \"mode\": \"fallback\"
    },
    \"targets\": [
        {
            \"provider\": \"openai\",
            \"custom_host\": \"http://my-private-llm:8080/v1\",
            \"forward_headers\": [\"Authorization\"]
        },
        {
            \"virtual_key\": \"openai-fallback-key\"
        }
    ]
}
", Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Header 3 ("production-ready-config-with-fallback-caching-and-retries", ["unnumbered", "unlisted"], []) [Str "Production-Ready Config with Fallback, Caching, and Retries"], CodeBlock ("", ["python"], []) "from portkey_ai import Portkey

config = {
    \"strategy\": {
        \"mode\": \"fallback\"
    },
    \"targets\": [
        {
            \"virtual_key\": \"openai-prod\",
            \"override_params\": {\"model\": \"gpt-4o\"},
            \"weight\": 1.0
        },
        {
            \"virtual_key\": \"anthropic-prod\",
            \"override_params\": {\"model\": \"claude-sonnet-4-6\"}
        }
    ],
    \"cache\": {
        \"mode\": \"semantic\",
        \"max_age\": 3600
    },
    \"retry\": {
        \"attempts\": 3,
        \"on_status_codes\": [429, 500, 502, 503, 504]
    },
    \"request_timeout\": 30000
}

portkey = Portkey(
    api_key=\"PORTKEY_API_KEY\",
    config=config
)

response = portkey.chat.completions.create(
    messages=[{\"role\": \"user\", \"content\": \"Summarize this document.\"}]
)
", Header 3 ("weighted-load-balancing-across-providers", ["unnumbered", "unlisted"], []) [Str "Weighted Load Balancing Across Providers"], CodeBlock ("", ["python"], []) "from portkey_ai import Portkey

config = {
    \"strategy\": {
        \"mode\": \"loadbalance\"
    },
    \"targets\": [
        {
            \"virtual_key\": \"openai-key-1\",
            \"weight\": 0.5,
            \"override_params\": {\"model\": \"gpt-4o\"}
        },
        {
            \"virtual_key\": \"openai-key-2\",
            \"weight\": 0.3,
            \"override_params\": {\"model\": \"gpt-4o\"}
        },
        {
            \"virtual_key\": \"anthropic-key\",
            \"weight\": 0.2,
            \"override_params\": {\"model\": \"claude-sonnet-4-6\"}
        }
    ]
}

portkey = Portkey(api_key=\"PORTKEY_API_KEY\", config=config)

response = portkey.chat.completions.create(
    messages=[{\"role\": \"user\", \"content\": \"Hello!\"}]
)
", Header 3 ("embedding-request-with-guardrails", ["unnumbered", "unlisted"], []) [Str "Embedding Request with Guardrails"], CodeBlock ("", ["python"], []) "from portkey_ai import Portkey

portkey = Portkey(
    api_key=\"PORTKEY_API_KEY\",
    config=\"pc-xxx\"  # Config with embedding guardrails
)

response = portkey.embeddings.create(
    input=\"Your text string goes here\",
    model=\"text-embedding-3-small\"
)
", Header 3 ("custom-metadata-for-observability", ["unnumbered", "unlisted"], []) [Str "Custom Metadata for Observability"], CodeBlock ("", ["python"], []) "from portkey_ai import Portkey

portkey = Portkey(
    api_key=\"PORTKEY_API_KEY\",
)

response = portkey.with_options(
    metadata={\"user_id\": \"user-123\", \"session\": \"abc\", \"environment\": \"production\"}
).chat.completions.create(
    model=\"@openai-prod/gpt-4o\",
    messages=[{\"role\": \"user\", \"content\": \"Help me debug this error.\"}]
)
", Header 2 ("limitations-and-considerations", ["unnumbered", "unlisted"], []) [Str "Limitations and Considerations"], Para [Strong [Str "Added Latency"], Str ": The edge proxy adds 20-40ms of latency to every request compared to direct provider API calls. For latency-critical applications where every millisecond matters, this overhead should be evaluated against the benefits of routing and observability."], Para [Strong [Str "Vendor Lock-In on Managed Features"], Str ": While the open-source gateway handles routing, caching, and retries, advanced features like the analytics dashboard, guardrails management UI, Model Catalog, and budget controls require the managed Portkey service."], Para [Strong [Str "Semantic Cache Accuracy"], Str ": Semantic caching relies on similarity matching, which may return cached responses for prompts that are similar but not semantically equivalent. Applications requiring deterministic responses should use simple (exact-match) caching or disable caching entirely."], Para [Strong [Str "Guardrail Latency"], Str ": Synchronous guardrails add processing time to each request. Applications with tight latency requirements should consider running guardrails asynchronously (logging only) or limiting the number of active checks per request."], Para [Strong [Str "Provider Feature Parity"], Str ": Not all provider-specific features are exposed through Portkey's unified interface. Advanced or recently released provider capabilities may require direct API access until Portkey adds support."], Para [Strong [Str "Virtual Key Deprecation"], Str ": Virtual Keys have been migrated to the Model Catalog system. Existing implementations using Virtual Keys continue to work but should migrate to the ", Code ("", [], []) "@provider-slug/model-name", Str " syntax for new projects."], Para [Strong [Str "Free Tier Limits"], Str ": The managed service free tier is limited to 10,000 requests per month, which is sufficient for development but requires a paid plan for production workloads."], Header 2 ("changelog-highlights", ["unnumbered", "unlisted"], []) [Str "Changelog Highlights"], BulletList [[Plain [Strong [Str "Model Catalog"], Str ": Replaced Virtual Keys with organization-level credential management, fine-grained budgets, rate limits, and model allow-lists"]], [Plain [Strong [Str "Guardrails on the Gateway"], Str ": Real-time input/output validation with 20+ deterministic checks, LLM-based detection, and third-party integrations (Aporia, SydeLabs, Pillar Security)"]], [Plain [Strong [Str "Conditional Routing"], Str ": Query-based routing rules for directing traffic based on custom criteria"]], [Plain [Strong [Str "Circuit Breaker"], Str ": Per-strategy failure handling with configurable thresholds and cooldown intervals"]], [Plain [Strong [Str "MCP Support"], Str ": Model Context Protocol integration for connecting external tools and data sources"]], [Plain [Strong [Str "gRPC Transport (Beta)"], Str ": Reduced-latency transport option alongside REST"]], [Plain [Strong [Str "Semantic Caching"], Str ": Similarity-based cache matching for semantically equivalent prompts"]], [Plain [Strong [Str "250+ Provider Support"], Str ": Expanded from initial provider set to over 250 supported LLM providers and models"]]], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " What is Portkey - ", Link ("", [], []) [Str "https://portkey.ai/docs/introduction/what-is-portkey"] ("https://portkey.ai/docs/introduction/what-is-portkey", "")]], [Plain [Str "[", Str "2", Str "]", Str " Gateway Configs - ", Link ("", [], []) [Str "https://portkey.ai/docs/product/ai-gateway/configs"] ("https://portkey.ai/docs/product/ai-gateway/configs", "")]], [Plain [Str "[", Str "3", Str "]", Str " Virtual Keys / Model Catalog - ", Link ("", [], []) [Str "https://portkey.ai/docs/product/ai-gateway/virtual-keys"] ("https://portkey.ai/docs/product/ai-gateway/virtual-keys", "")]], [Plain [Str "[", Str "4", Str "]", Str " Guardrails - ", Link ("", [], []) [Str "https://portkey.ai/docs/product/guardrails"] ("https://portkey.ai/docs/product/guardrails", "")]], [Plain [Str "[", Str "5", Str "]", Str " Observability - ", Link ("", [], []) [Str "https://portkey.ai/docs/product/observability"] ("https://portkey.ai/docs/product/observability", "")]], [Plain [Str "[", Str "6", Str "]", Str " AI Gateway Overview - ", Link ("", [], []) [Str "https://portkey.ai/docs/product/ai-gateway"] ("https://portkey.ai/docs/product/ai-gateway", "")]], [Plain [Str "[", Str "7", Str "]", Str " Config Object Schema - ", Link ("", [], []) [Str "https://portkey.ai/docs/api-reference/inference-api/config-object"] ("https://portkey.ai/docs/api-reference/inference-api/config-object", "")]], [Plain [Str "[", Str "8", Str "]", Str " Supported LLM Providers - ", Link ("", [], []) [Str "https://portkey.ai/docs/integrations/llms"] ("https://portkey.ai/docs/integrations/llms", "")]], [Plain [Str "[", Str "9", Str "]", Str " GitHub Repository - ", Link ("", [], []) [Str "https://github.com/Portkey-AI/gateway"] ("https://github.com/Portkey-AI/gateway", "")]]]]