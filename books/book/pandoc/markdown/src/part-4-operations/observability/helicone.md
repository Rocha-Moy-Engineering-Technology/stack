[Header 1 ("helicone", [], []) [Str "Helicone"], BlockQuote [Para [Str "AI gateway and LLM observability platform with routing"]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.5)), (AlignDefault, (ColWidth 0.5))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Observability & LLM Ops"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "API/UI"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Yes"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "https://github.com/Helicone/helicone"] ("https://github.com/Helicone/helicone", "")]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "5126"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Documentation"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "Official Docs"] ("https://docs.helicone.ai/", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "Helicone is an open-source AI gateway and Large Language Model (LLM) observability platform that provides a unified, OpenAI-compatible API for accessing over 100 models across multiple providers. Backed by Y Combinator (W23) and licensed under Apache 2.0, Helicone sits between your application and LLM providers, acting as both a proxy and an observability layer. Every request that passes through the gateway is automatically logged, enabling cost tracking, latency monitoring, session tracing, and analytics without requiring changes to application logic beyond a base URL swap."], Para [Str "Helicone distinguishes itself from pure observability platforms by combining gateway functionality (routing, caching, rate limiting, retries, fallbacks) with a full-featured analytics dashboard. It supports a Bring Your Own Keys (BYOK) model where developers supply their own provider API keys, as well as a managed credits system with zero markup on provider pricing. The platform can be used as a hosted service at ", Code ("", [], []) "helicone.ai", Str " or self-hosted via Docker Compose, Kubernetes, or manual installation."], Para [Strong [Str "Primary value propositions:"]], BulletList [[Plain [Str "Single integration point for 100+ models from OpenAI, Anthropic, Google, Groq, Vertex AI, AWS Bedrock, and others"]], [Plain [Str "Automatic logging and observability with sub-second dashboard latency"]], [Plain [Str "Edge-deployed caching on Cloudflare for cost reduction and faster responses"]], [Plain [Str "Production-grade features (fallbacks, retries, rate limiting) controlled entirely through HTTP headers"]], [Plain [Str "SOC 2 and General Data Protection Regulation (GDPR) compliant for enterprise deployments"]]], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], Para [Strong [Str "AI Gateway."], Str " The gateway is a reverse proxy that intercepts LLM API calls, logs them, and forwards them to the target provider. Requests use the standard OpenAI SDK format but point to ", Code ("", [], []) "gateway.helicone.ai", Str " (or ", Code ("", [], []) "oai.helicone.ai", Str " for OpenAI-specific traffic) instead of the provider endpoint. The target provider is specified through the ", Code ("", [], []) "Helicone-Target-Url", Str " header."], Para [Strong [Str "Header-Driven Configuration."], Str " Helicone's features are activated and configured entirely through HTTP headers on each request. Caching, rate limiting, retries, custom properties, session tracking, and security features are all toggled by including the appropriate ", Code ("", [], []) "Helicone-*", Str " headers. This design means no SDK lock-in and no configuration files to manage."], Para [Strong [Str "Sessions and Traces."], Str " Sessions group related requests (LLM calls, tool calls, vector database queries) into a single workflow view. Sessions use three headers: ", Code ("", [], []) "Helicone-Session-Id", Str " for the unique session identifier, ", Code ("", [], []) "Helicone-Session-Path", Str " for parent-child hierarchy using forward-slash syntax, and ", Code ("", [], []) "Helicone-Session-Name", Str " for human-readable categorization."], Para [Strong [Str "Custom Properties."], Str " Arbitrary key-value metadata attached to requests via ", Code ("", [], []) "Helicone-Property-[Name]", Str " headers. Properties enable segmentation by environment, feature, user tier, conversation ID, or any business dimension relevant to cost analysis and debugging."], Para [Strong [Str "Proxy Mode vs. Async Mode."], Str " Helicone supports two integration methods. Proxy mode routes requests through the gateway, providing access to all gateway features (caching, fallbacks, rate limiting). Async mode uses OpenLLMetry to log events without placing Helicone in the critical path, ensuring that Helicone issues cannot cause application outages."], Para [Strong [Str "Prompt Management."], Str " Prompts can be versioned and deployed through the gateway using ", Code ("", [], []) "Helicone-Prompt-Id", Str " headers, enabling iteration on prompts without code deployments."], Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Para [Str "Helicone's architecture comprises five core services:"], CodeBlock ("", [""], []) "                    +------------------+
                    |    Client App    |
                    +--------+---------+
                             |
                    OpenAI-compatible API
                             |
                    +--------v---------+
                    |  Cloudflare Edge |
                    |    (Workers)     |
                    |  - Proxy/Logging |
                    |  - Caching       |
                    |  - Rate Limiting |
                    +--------+---------+
                             |
               +-------------+-------------+
               |                           |
      +--------v---------+       +--------v---------+
      |   LLM Providers  |       |    Jawn Server   |
      | OpenAI, Anthropic|       |  (Express/Tsoa)  |
      | Google, Groq ... |       |  - Log Ingestion |
      +------------------+       |  - API Queries   |
                                 +--------+---------+
                                          |
                              +-----------+-----------+
                              |                       |
                     +--------v-------+     +---------v--------+
                     |   Supabase     |     |   ClickHouse     |
                     | - Auth         |     | - Analytics DB   |
                     | - User Data    |     | - Request Logs   |
                     +----------------+     | - Minio (Object) |
                                            +------------------+
                                                     |
                                            +--------v---------+
                                            |   Web Dashboard  |
                                            |    (Next.js)     |
                                            +------------------+
", Para [Strong [Str "Web (Next.js)."], Str " The frontend dashboard for viewing request logs, analytics, sessions, prompt management, and configuration."], Para [Strong [Str "Worker (Cloudflare Workers)."], Str " The edge proxy layer that intercepts requests, applies gateway features (caching, rate limiting, retries, fallbacks), and forwards to LLM providers. Being deployed on Cloudflare's edge network provides low-latency processing globally."], Para [Strong [Str "Jawn (Express/Tsoa)."], Str " The backend server responsible for log ingestion, REST API endpoints, and async event processing. Handles the write path from Workers and the read path from the dashboard and API consumers."], Para [Strong [Str "Supabase."], Str " Provides database storage for user accounts, organizations, API keys, and authentication. Handles the Identity and Access Management (IAM) layer."], Para [Strong [Str "ClickHouse with Minio."], Str " The analytics database optimized for high-volume log queries. ClickHouse handles request log storage and aggregation, while Minio provides S3-compatible object storage for request/response bodies."], Para [Str "The technology stack is predominantly TypeScript (91.3%), with additional MDX for documentation, Python, SQL, and shell scripting."], Header 2 ("key-features-and-functionality", ["unnumbered", "unlisted"], []) [Str "Key Features and Functionality"], Header 3 ("request-logging-and-observability", ["unnumbered", "unlisted"], []) [Str "Request Logging and Observability"], Para [Str "Every request through the gateway is automatically logged with full metadata: model, tokens (prompt and completion), cost, latency, status code, request body, response body, and headers. Logs appear in the dashboard within seconds and support filtering by any field."], Header 3 ("edge-caching", ["unnumbered", "unlisted"], []) [Str "Edge Caching"], Para [Str "Responses are cached on Cloudflare's edge network. Enable with a single header:"], CodeBlock ("", ["python"], []) "response = client.chat.completions.create(
    model=\"gpt-4o-mini\",
    messages=[{\"role\": \"user\", \"content\": \"What is 2+2?\"}],
    extra_headers={
        \"Helicone-Cache-Enabled\": \"true\",
        \"Cache-Control\": \"max-age=3600\",
    },
)
", Para [Str "Cache configuration options:"], BulletList [[Plain [Code ("", [], []) "Cache-Control", Str ": Standard HTTP cache duration (default: 7 days)"]], [Plain [Code ("", [], []) "Helicone-Cache-Bucket-Max-Size", Str ": Number of different responses stored for identical requests (1-20, default: 1); useful for non-deterministic prompts"]], [Plain [Code ("", [], []) "Helicone-Cache-Seed", Str ": Namespace for user- or context-specific caches (e.g., ", Code ("", [], []) "user-123", Str ")"]], [Plain [Code ("", [], []) "Helicone-Cache-Ignore-Keys", Str ": Exclude specific JSON fields from cache key generation"]]], Para [Str "Cache keys are computed by hashing the seed, request URL, request body, relevant headers, and bucket index. Response headers indicate cache status: ", Code ("", [], []) "Helicone-Cache: HIT", Str " or ", Code ("", [], []) "Helicone-Cache: MISS", Str "."], Header 3 ("rate-limiting", ["unnumbered", "unlisted"], []) [Str "Rate Limiting"], Para [Str "Control request quotas per user, organization, or custom segment:"], CodeBlock ("", ["python"], []) "response = client.chat.completions.create(
    model=\"gpt-4o-mini\",
    messages=[{\"role\": \"user\", \"content\": \"Hello\"}],
    extra_headers={
        \"Helicone-RateLimit-Policy\": \"10;w=1000;u=cents;s=user\",
    },
)
", Para [Str "Policy format: ", Code ("", [], []) "[quota];w=[time_window];u=[unit];s=[segment]"], BulletList [[Plain [Code ("", [], []) "quota", Str ": Maximum allowed in the window"]], [Plain [Code ("", [], []) "w", Str ": Time window in seconds"]], [Plain [Code ("", [], []) "u", Str ": Unit of measurement (", Code ("", [], []) "cents", Str ", ", Code ("", [], []) "requests", Str ", ", Code ("", [], []) "tokens", Str ")"]], [Plain [Code ("", [], []) "s", Str ": Segmentation key (", Code ("", [], []) "user", Str ", a custom property name)"]]], Para [Str "Response headers include ", Code ("", [], []) "Helicone-RateLimit-Limit", Str ", ", Code ("", [], []) "Helicone-RateLimit-Remaining", Str ", and ", Code ("", [], []) "Helicone-RateLimit-Policy", Str "."], Header 3 ("automatic-retries", ["unnumbered", "unlisted"], []) [Str "Automatic Retries"], Para [Str "Enable exponential backoff retries for failed requests:"], CodeBlock ("", ["python"], []) "response = client.chat.completions.create(
    model=\"gpt-4o-mini\",
    messages=[{\"role\": \"user\", \"content\": \"Hello\"}],
    extra_headers={
        \"Helicone-Retry-Enabled\": \"true\",
        \"helicone-retry-num\": \"3\",
        \"helicone-retry-factor\": \"2\",
    },
)
", Header 3 ("provider-fallbacks", ["unnumbered", "unlisted"], []) [Str "Provider Fallbacks"], Para [Str "When the primary provider fails, Helicone automatically routes to a fallback model. The ", Code ("", [], []) "Helicone-Fallback-Index", Str " response header indicates which fallback was used."], Header 3 ("session-tracking", ["unnumbered", "unlisted"], []) [Str "Session Tracking"], Para [Str "Group related requests into workflow traces:"], CodeBlock ("", ["typescript"], []) "import { randomUUID } from \"crypto\";

const sessionId = randomUUID();

// First call in session
const step1 = await client.chat.completions.create(
  {
    model: \"gpt-4o-mini\",
    messages: [{ role: \"user\", content: \"Research topic X\" }],
  },
  {
    headers: {
      \"Helicone-Session-Id\": sessionId,
      \"Helicone-Session-Path\": \"/research\",
      \"Helicone-Session-Name\": \"Research Pipeline\",
    },
  }
);

// Child call in session
const step2 = await client.chat.completions.create(
  {
    model: \"gpt-4o-mini\",
    messages: [{ role: \"user\", content: \"Summarize findings\" }],
  },
  {
    headers: {
      \"Helicone-Session-Id\": sessionId,
      \"Helicone-Session-Path\": \"/research/summarize\",
      \"Helicone-Session-Name\": \"Research Pipeline\",
    },
  }
);
", Para [Str "Path hierarchy examples:"], BulletList [[Plain [Str "Workflow: ", Code ("", [], []) "/task/research/web_search"]], [Plain [Str "Conversation: ", Code ("", [], []) "/session/question_1/answer_1"]], [Plain [Str "Pipeline: ", Code ("", [], []) "/process/extract/transform/load"]]], Header 3 ("user-tracking", ["unnumbered", "unlisted"], []) [Str "User Tracking"], Para [Str "Track per-user costs, usage patterns, and behavior:"], CodeBlock ("", ["python"], []) "response = client.chat.completions.create(
    model=\"gpt-4o-mini\",
    messages=[{\"role\": \"user\", \"content\": \"Hello\"}],
    extra_headers={
        \"Helicone-User-Id\": \"user-12345\",
        \"Helicone-Property-UserTier\": \"premium\",
        \"Helicone-Property-UserType\": \"business\",
    },
)
", Para [Str "The platform automatically tracks daily/weekly/monthly active users, session analytics, usage patterns, and user lifecycle stages."], Header 3 ("content-moderation-and-security", ["unnumbered", "unlisted"], []) [Str "Content Moderation and Security"], BulletList [[Plain [Code ("", [], []) "Helicone-Moderations-Enabled", Str ": Activates OpenAI moderation on requests"]], [Plain [Code ("", [], []) "Helicone-LLM-Security-Enabled", Str ": Protects against prompt injection attacks"]]], Header 3 ("prompt-management", ["unnumbered", "unlisted"], []) [Str "Prompt Management"], Para [Str "Version and deploy prompts using ", Code ("", [], []) "Helicone-Prompt-Id", Str " headers. Prompts can be iterated on through the dashboard and deployed to production without code changes."], Header 3 ("token-overflow-handling", ["unnumbered", "unlisted"], []) [Str "Token Overflow Handling"], Para [Str "The ", Code ("", [], []) "Helicone-Token-Limit-Exception-Handler", Str " header manages context window overflow with strategies: ", Code ("", [], []) "truncate", Str ", ", Code ("", [], []) "middle-out", Str ", or ", Code ("", [], []) "fallback", Str "."], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], Para [Strong [Str "Multi-Provider LLM Applications."], Str " Teams using models from multiple providers (OpenAI for chat, Anthropic for analysis, Google for embeddings) can consolidate all traffic through a single gateway, gaining unified cost tracking and the ability to switch providers by changing a single model parameter."], Para [Strong [Str "Cost Optimization for Development Teams."], Str " Caching identical requests during development and testing avoids repeated charges. Custom properties segmented by environment (", Code ("", [], []) "staging", Str ", ", Code ("", [], []) "production", Str ") enable precise cost attribution."], Para [Strong [Str "AI Agent Observability."], Str " Sessions trace multi-step agent workflows end-to-end, revealing where agents spend time, which tool calls are most expensive, and where failures occur in complex chains."], Para [Strong [Str "Per-User Cost Management in SaaS."], Str " User ID tracking combined with rate limiting enables precise unit economics: cost per user, cost per feature, and enforcement of usage quotas by subscription tier."], Para [Strong [Str "High-Availability Production Systems."], Str " Automatic fallbacks ensure continued service when a provider experiences an outage. Retries with exponential backoff handle transient failures without application-level retry logic."], Para [Strong [Str "Compliance and Auditing."], Str " Full request/response logging with data retention policies (up to forever on Enterprise) provides an audit trail for regulated industries. GDPR-compliant EU endpoints are available at ", Code ("", [], []) "eu.api.helicone.ai", Str "."], Header 2 ("api-reference-summary", ["unnumbered", "unlisted"], []) [Str "API Reference Summary"], Header 3 ("gateway-endpoint", ["unnumbered", "unlisted"], []) [Str "Gateway Endpoint"], Para [Str "All LLM requests are routed through the gateway:"], CodeBlock ("", [""], []) "POST https://gateway.helicone.ai/{provider-path}
POST https://ai-gateway.helicone.ai/{openai-path}
POST https://oai.helicone.ai/{openai-path}
", Para [Str "The gateway accepts standard provider request formats. Set ", Code ("", [], []) "Helicone-Target-Url", Str " to specify the downstream provider when using ", Code ("", [], []) "gateway.helicone.ai", Str "."], Header 3 ("query-api", ["unnumbered", "unlisted"], []) [Str "Query API"], Para [Str "Retrieve logged requests programmatically:"], CodeBlock ("", [""], []) "POST https://api.helicone.ai/v1/request/query
", Para [Str "EU endpoint: ", Code ("", [], []) "https://eu.api.helicone.ai/v1/request/query"], Para [Strong [Str "Authentication:"], Str " Bearer token via ", Code ("", [], []) "Authorization", Str " header."], Para [Strong [Str "Request body:"]], CodeBlock ("", ["json"], []) "{
  \"filter\": {
    \"operator\": \"and\",
    \"left\": {
      \"request\": {
        \"model\": { \"contains\": \"gpt-4\" }
      }
    },
    \"right\": {
      \"request_response_rmt\": {
        \"cost\": { \"gt\": 0.01 }
      }
    }
  },
  \"limit\": 100,
  \"offset\": 0,
  \"sort\": { \"created_at\": \"desc\" }
}
", Para [Strong [Str "Supported filter operators:"]], BulletList [[Plain [Str "Text: ", Code ("", [], []) "equals", Str ", ", Code ("", [], []) "not-equals", Str ", ", Code ("", [], []) "contains", Str ", ", Code ("", [], []) "not-contains", Str ", ", Code ("", [], []) "like", Str ", ", Code ("", [], []) "ilike"]], [Plain [Str "Numeric: ", Code ("", [], []) "equals", Str ", ", Code ("", [], []) "not-equals", Str ", ", Code ("", [], []) "gt", Str ", ", Code ("", [], []) "gte", Str ", ", Code ("", [], []) "lt", Str ", ", Code ("", [], []) "lte"]], [Plain [Str "Timestamp: ", Code ("", [], []) "equals", Str ", ", Code ("", [], []) "gt", Str ", ", Code ("", [], []) "gte", Str ", ", Code ("", [], []) "lt", Str ", ", Code ("", [], []) "lte"]], [Plain [Str "Boolean: ", Code ("", [], []) "equals"]]], Para [Strong [Str "Filterable fields:"], Str " ", Code ("", [], []) "request.user_id", Str ", ", Code ("", [], []) "request.model", Str ", ", Code ("", [], []) "request.prompt", Str ", ", Code ("", [], []) "request.created_at", Str ", ", Code ("", [], []) "response.status", Str ", ", Code ("", [], []) "response.model", Str ", ", Code ("", [], []) "feedback.rating", Str ", and ", Code ("", [], []) "request_response_rmt", Str " fields (latency, cost, provider, tokens, cache status). Custom property filters must be wrapped in a ", Code ("", [], []) "request_response_rmt", Str " object."], Para [Strong [Str "Response fields per request:"], Str " ", Code ("", [], []) "request_id", Str ", ", Code ("", [], []) "request_created_at", Str ", ", Code ("", [], []) "request_body", Str ", ", Code ("", [], []) "request_user_id", Str ", ", Code ("", [], []) "request_model", Str ", ", Code ("", [], []) "response_status", Str ", ", Code ("", [], []) "response_body", Str ", ", Code ("", [], []) "total_tokens", Str ", ", Code ("", [], []) "prompt_tokens", Str ", ", Code ("", [], []) "completion_tokens", Str ", ", Code ("", [], []) "cost", Str ", ", Code ("", [], []) "delay_ms", Str ", ", Code ("", [], []) "model", Str ", ", Code ("", [], []) "properties", Str "."], Header 3 ("user-query-api", ["unnumbered", "unlisted"], []) [Str "User Query API"], CodeBlock ("", [""], []) "POST https://api.helicone.ai/v1/user/query
", Para [Str "Returns aggregated user metrics and analytics."], Header 3 ("header-directory-complete-reference", ["unnumbered", "unlisted"], []) [Str "Header Directory (Complete Reference)"], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.47058823529411764)), (AlignDefault, (ColWidth 0.5294117647058824))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Header"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Purpose"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "Helicone-Auth"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Authentication (required): ", Code ("", [], []) "Bearer <API_KEY>"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "Helicone-Target-URL"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Downstream provider URL"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "Helicone-Request-Id"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Custom request UUID"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "Helicone-User-Id"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "User tracking identifier"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "Helicone-Session-Id"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Session grouping identifier"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "Helicone-Session-Path"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Session hierarchy path"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "Helicone-Session-Name"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Session display name"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "Helicone-Model-Override"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Override model for cost calculation"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "Helicone-Prompt-Id"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Prompt version tracking"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "Helicone-Property-[Name]"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Custom metadata properties"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "Helicone-RateLimit-Policy"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Rate limiting configuration"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "Helicone-Cache-Enabled"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Toggle edge caching"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "Helicone-Cache-Seed"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Cache namespace"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "Helicone-Cache-Bucket-Max-Size"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Max cached variants (1-20)"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "Helicone-Cache-Ignore-Keys"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Fields to exclude from cache key"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "Helicone-Retry-Enabled"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Toggle automatic retries"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "helicone-retry-num"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Maximum retry attempts"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "helicone-retry-factor"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Exponential backoff factor"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "Helicone-Omit-Response"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Exclude response body from logs"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "Helicone-Omit-Request"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Exclude request body from logs"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "Helicone-Token-Limit-Exception-Handler"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Overflow strategy: ", Code ("", [], []) "truncate", Str ", ", Code ("", [], []) "middle-out", Str ", ", Code ("", [], []) "fallback"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "Helicone-Moderations-Enabled"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Enable OpenAI moderation"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "Helicone-LLM-Security-Enabled"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Enable prompt injection protection"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "Helicone-Stream-Force-Format"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Fix stream formatting for incompatible libraries"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "Helicone-Posthog-Key"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "PostHog integration API key"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "Helicone-Posthog-Host"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "PostHog integration host"]]]])] (TableFoot ("", [], []) []), Header 2 ("configuration-and-customization", ["unnumbered", "unlisted"], []) [Str "Configuration and Customization"], Header 3 ("gateway-mode-configuration", ["unnumbered", "unlisted"], []) [Str "Gateway Mode Configuration"], Para [Str "When using the generic gateway endpoint (", Code ("", [], []) "gateway.helicone.ai", Str "), specify the target provider:"], CodeBlock ("", ["python"], []) "client = OpenAI(
    base_url=\"https://gateway.helicone.ai/v1\",
    api_key=\"sk-provider-key\",
    default_headers={
        \"Helicone-Auth\": f\"Bearer {os.getenv('HELICONE_API_KEY')}\",
        \"Helicone-Target-Url\": \"https://api.openai.com\",
    },
)
", Para [Str "For non-OpenAI providers (e.g., Groq):"], CodeBlock ("", ["python"], []) "client = OpenAI(
    base_url=\"https://gateway.helicone.ai/openai/v1\",
    api_key=\"sk-groq-key\",
    default_headers={
        \"Helicone-Auth\": f\"Bearer {os.getenv('HELICONE_API_KEY')}\",
        \"Helicone-Target-Url\": \"https://api.groq.com\",
    },
)
", Header 3 ("custom-properties-for-environment-segmentation", ["unnumbered", "unlisted"], []) [Str "Custom Properties for Environment Segmentation"], CodeBlock ("", ["python"], []) "client = OpenAI(
    base_url=\"https://ai-gateway.helicone.ai\",
    api_key=os.getenv(\"HELICONE_API_KEY\"),
    default_headers={
        \"Helicone-Property-Environment\": \"production\",
        \"Helicone-Property-App\": \"chatbot-v2\",
        \"Helicone-Property-Team\": \"ml-engineering\",
    },
)
", Header 3 ("async-logger-configuration", ["unnumbered", "unlisted"], []) [Str "Async Logger Configuration"], CodeBlock ("", ["python"], []) "from helicone_async import HeliconeAsyncLogger

logger = HeliconeAsyncLogger(api_key=\"sk-helicone-...\")
logger.init()

# Set session-level properties
logger.set_properties({
    \"Helicone-Session-Id\": \"session-abc\",
    \"Helicone-Session-Path\": \"/pipeline/step1\",
    \"Helicone-Session-Name\": \"Data Processing\",
    \"Helicone-Property-Pipeline\": \"etl-v3\",
})

# Disable/enable logging dynamically
logger.disable_logging()  # Pause logging
logger.enable_logging()   # Resume logging
", Header 3 ("data-privacy-controls", ["unnumbered", "unlisted"], []) [Str "Data Privacy Controls"], Para [Str "Exclude sensitive content from logs:"], CodeBlock ("", ["python"], []) "response = client.chat.completions.create(
    model=\"gpt-4o-mini\",
    messages=[{\"role\": \"user\", \"content\": \"Process this PII data...\"}],
    extra_headers={
        \"Helicone-Omit-Request\": \"true\",
        \"Helicone-Omit-Response\": \"true\",
    },
)
", Header 3 ("eu-data-residency", ["unnumbered", "unlisted"], []) [Str "EU Data Residency"], Para [Str "For GDPR compliance, use EU-specific endpoints:"], BulletList [[Plain [Str "API queries: ", Code ("", [], []) "https://eu.api.helicone.ai"]], [Plain [Str "Gateway: EU-specific gateway endpoints available"]]], Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Header 3 ("openai-sdk-direct-gateway", ["unnumbered", "unlisted"], []) [Str "OpenAI SDK (Direct Gateway)"], Para [Str "The simplest integration swaps the base URL:"], CodeBlock ("", ["python"], []) "from openai import OpenAI

client = OpenAI(
    base_url=\"https://ai-gateway.helicone.ai\",
    api_key=os.getenv(\"HELICONE_API_KEY\"),
)
", Header 3 ("langchain-integration", ["unnumbered", "unlisted"], []) [Str "LangChain Integration"], CodeBlock ("", ["python"], []) "from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    model=\"gpt-4o-mini\",
    base_url=\"https://gateway.helicone.ai/v1\",
    api_key=\"sk-openai-key\",
    default_headers={
        \"Helicone-Auth\": f\"Bearer {os.getenv('HELICONE_API_KEY')}\",
        \"Helicone-Target-Url\": \"https://api.openai.com\",
    },
)
", Header 3 ("vercel-ai-sdk", ["unnumbered", "unlisted"], []) [Str "Vercel AI SDK"], CodeBlock ("", ["typescript"], []) "import { createOpenAI } from \"@ai-sdk/openai\";

const openai = createOpenAI({
  baseURL: \"https://gateway.helicone.ai/v1\",
  headers: {
    \"Helicone-Auth\": `Bearer ${process.env.HELICONE_API_KEY}`,
    \"Helicone-Target-Url\": \"https://api.openai.com\",
  },
});
", Header 3 ("llamaindex-integration", ["unnumbered", "unlisted"], []) [Str "LlamaIndex Integration"], CodeBlock ("", ["python"], []) "from llama_index.llms.openai import OpenAI

llm = OpenAI(
    model=\"gpt-4o-mini\",
    api_base=\"https://gateway.helicone.ai/v1\",
    additional_kwargs={
        \"headers\": {
            \"Helicone-Auth\": f\"Bearer {os.getenv('HELICONE_API_KEY')}\",
            \"Helicone-Target-Url\": \"https://api.openai.com\",
        }
    },
)
", Header 3 ("posthog-analytics-integration", ["unnumbered", "unlisted"], []) [Str "PostHog Analytics Integration"], Para [Str "Forward LLM analytics to PostHog for product analytics correlation:"], CodeBlock ("", ["python"], []) "response = client.chat.completions.create(
    model=\"gpt-4o-mini\",
    messages=[{\"role\": \"user\", \"content\": \"Hello\"}],
    extra_headers={
        \"Helicone-Posthog-Key\": os.getenv(\"POSTHOG_API_KEY\"),
        \"Helicone-Posthog-Host\": \"https://app.posthog.com\",
    },
)
", Header 3 ("supported-providers", ["unnumbered", "unlisted"], []) [Str "Supported Providers"], Para [Strong [Str "Inference providers:"], Str " OpenAI, Azure OpenAI, Anthropic, AWS Bedrock, Google Gemini, Google Vertex AI, Groq, TogetherAI, Anyscale, DeepInfra, Fireworks, Hyperbolic."], Para [Strong [Str "Framework integrations:"], Str " LangChain, LlamaIndex, LangGraph, Vercel AI SDK, CrewAI, Semantic Kernel, Open WebUI."], Para [Str "Unapproved domains can also be proxied through the gateway with a rate limit of 10,000 requests per day."], Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Header 3 ("example-1-cost-optimized-development-with-caching", ["unnumbered", "unlisted"], []) [Str "Example 1: Cost-Optimized Development with Caching"], CodeBlock ("", ["python"], []) "import os
from openai import OpenAI

client = OpenAI(
    base_url=\"https://ai-gateway.helicone.ai\",
    api_key=os.getenv(\"HELICONE_API_KEY\"),
    default_headers={
        \"Helicone-Cache-Enabled\": \"true\",
        \"Cache-Control\": \"max-age=86400\",
        \"Helicone-Property-Environment\": \"development\",
    },
)

# First call: cache MISS, billed by provider
response = client.chat.completions.create(
    model=\"gpt-4o-mini\",
    messages=[{\"role\": \"user\", \"content\": \"Explain the CAP theorem\"}],
)

# Second identical call: cache HIT, zero cost, instant response
response = client.chat.completions.create(
    model=\"gpt-4o-mini\",
    messages=[{\"role\": \"user\", \"content\": \"Explain the CAP theorem\"}],
)
", Header 3 ("example-2-multi-step-agent-session-with-user-tracking", ["unnumbered", "unlisted"], []) [Str "Example 2: Multi-Step Agent Session with User Tracking"], CodeBlock ("", ["typescript"], []) "import OpenAI from \"openai\";
import { randomUUID } from \"crypto\";

const client = new OpenAI({
  baseURL: \"https://ai-gateway.helicone.ai\",
  apiKey: process.env.HELICONE_API_KEY,
});

const sessionId = randomUUID();
const userId = \"user-456\";

// Step 1: Plan
const plan = await client.chat.completions.create(
  {
    model: \"gpt-4o-mini\",
    messages: [{ role: \"user\", content: \"Plan a trip to Tokyo\" }],
  },
  {
    headers: {
      \"Helicone-Session-Id\": sessionId,
      \"Helicone-Session-Path\": \"/trip-planner/plan\",
      \"Helicone-Session-Name\": \"Trip Planning Agent\",
      \"Helicone-User-Id\": userId,
      \"Helicone-Property-Feature\": \"trip-planner\",
    },
  }
);

// Step 2: Research hotels (child of plan)
const hotels = await client.chat.completions.create(
  {
    model: \"gpt-4o-mini\",
    messages: [
      { role: \"user\", content: \"Find hotels in Shinjuku under $200/night\" },
    ],
  },
  {
    headers: {
      \"Helicone-Session-Id\": sessionId,
      \"Helicone-Session-Path\": \"/trip-planner/plan/hotels\",
      \"Helicone-Session-Name\": \"Trip Planning Agent\",
      \"Helicone-User-Id\": userId,
      \"Helicone-Property-Feature\": \"trip-planner\",
    },
  }
);

// Step 3: Summarize itinerary
const summary = await client.chat.completions.create(
  {
    model: \"gpt-4o-mini\",
    messages: [{ role: \"user\", content: \"Summarize the complete itinerary\" }],
  },
  {
    headers: {
      \"Helicone-Session-Id\": sessionId,
      \"Helicone-Session-Path\": \"/trip-planner/summarize\",
      \"Helicone-Session-Name\": \"Trip Planning Agent\",
      \"Helicone-User-Id\": userId,
      \"Helicone-Property-Feature\": \"trip-planner\",
    },
  }
);
", Header 3 ("example-3-rate-limited-saas-with-per-user-quotas", ["unnumbered", "unlisted"], []) [Str "Example 3: Rate-Limited SaaS with Per-User Quotas"], CodeBlock ("", ["python"], []) "import os
from openai import OpenAI

def create_client_for_user(user_id: str, tier: str) -> OpenAI:
    # 100 cents per 3600 seconds for free tier, 1000 for premium
    quota = \"100\" if tier == \"free\" else \"1000\"
    return OpenAI(
        base_url=\"https://ai-gateway.helicone.ai\",
        api_key=os.getenv(\"HELICONE_API_KEY\"),
        default_headers={
            \"Helicone-User-Id\": user_id,
            \"Helicone-Property-UserTier\": tier,
            \"Helicone-RateLimit-Policy\": f\"{quota};w=3600;u=cents;s=user\",
        },
    )

client = create_client_for_user(\"user-789\", \"free\")

response = client.chat.completions.create(
    model=\"gpt-4o-mini\",
    messages=[{\"role\": \"user\", \"content\": \"Hello!\"}],
)
", Header 3 ("example-4-querying-logged-data-via-rest-api", ["unnumbered", "unlisted"], []) [Str "Example 4: Querying Logged Data via REST API"], CodeBlock ("", ["python"], []) "import requests
import os

response = requests.post(
    \"https://api.helicone.ai/v1/request/query\",
    headers={
        \"Authorization\": f\"Bearer {os.getenv('HELICONE_API_KEY')}\",
        \"Content-Type\": \"application/json\",
    },
    json={
        \"filter\": {
            \"operator\": \"and\",
            \"left\": {
                \"request\": {
                    \"model\": {\"contains\": \"gpt-4\"}
                }
            },
            \"right\": {
                \"request_response_rmt\": {
                    \"cost\": {\"gt\": 0.05}
                }
            },
        },
        \"limit\": 50,
        \"offset\": 0,
        \"sort\": {\"created_at\": \"desc\"},
    },
)

data = response.json()
for req in data[\"data\"]:
    print(f\"Model: {req['model']}, Cost: ${req['cost']:.4f}, \"
          f\"Latency: {req['delay_ms']}ms, Tokens: {req['total_tokens']}\")
", Header 3 ("example-5-async-logging-with-session-properties", ["unnumbered", "unlisted"], []) [Str "Example 5: Async Logging with Session Properties"], CodeBlock ("", ["python"], []) "from helicone_async import HeliconeAsyncLogger
from openai import OpenAI

logger = HeliconeAsyncLogger(api_key=\"sk-helicone-...\")
logger.init()

client = OpenAI(api_key=\"sk-openai-...\")

# Set properties for the next group of calls
logger.set_properties({
    \"Helicone-Session-Id\": \"batch-job-001\",
    \"Helicone-Session-Path\": \"/batch/process\",
    \"Helicone-Session-Name\": \"Nightly Batch\",
    \"Helicone-Property-Pipeline\": \"content-generation\",
})

response = client.chat.completions.create(
    model=\"gpt-4o-mini\",
    messages=[{\"role\": \"user\", \"content\": \"Generate product description\"}],
)

# Temporarily disable logging for internal calls
logger.disable_logging()
# ... internal calls not logged ...
logger.enable_logging()
", Header 2 ("limitations-and-considerations", ["unnumbered", "unlisted"], []) [Str "Limitations and Considerations"], Para [Strong [Str "Latency Overhead."], Str " Proxy mode adds a small latency overhead since requests route through Cloudflare Workers before reaching the provider. For latency-sensitive applications, the async integration avoids this by logging out of band."], Para [Strong [Str "Free Tier Constraints."], Str " The free (Hobby) tier is limited to 10,000 requests per month, 1 GB storage, 7-day data retention, and 10 logs per minute ingestion. Production workloads will likely require the Pro tier ($79/month) or higher."], Para [Strong [Str "Data Retention."], Str " Retention is tier-dependent: 7 days (Free), 1 month (Pro), 3 months (Team), and unlimited (Enterprise). Historical data beyond the retention window is permanently deleted."], Para [Strong [Str "Rate Limit on Unapproved Domains."], Str " Providers not in Helicone's approved list can still be proxied, but are subject to a 10,000 requests per day limit."], Para [Strong [Str "Custom Property Query Caveat."], Str " When querying the REST API, custom property filters must be wrapped in a ", Code ("", [], []) "request_response_rmt", Str " object. Omitting this wrapper silently returns empty results, which can be a source of debugging confusion."], Para [Strong [Str "Self-Hosting Complexity."], Str " While open-source, the self-hosted deployment requires running five services (Next.js, Cloudflare Workers, Express, Supabase, ClickHouse with Minio). Docker Compose simplifies local development, but production self-hosting demands significant infrastructure expertise."], Para [Strong [Str "Feature Parity Between Modes."], Str " The async (OpenLLMetry) integration does not support gateway features like caching, rate limiting, retries, and fallbacks. These features require proxy mode since they operate at the request routing layer."], Para [Strong [Str "No Native SDK."], Str " Helicone relies on the OpenAI SDK with base URL modification rather than providing a dedicated client SDK. While this reduces vendor lock-in, it means feature configuration is done entirely through HTTP headers rather than typed SDK methods."], Header 2 ("changelog-highlights", ["unnumbered", "unlisted"], []) [Str "Changelog Highlights"], BulletList [[Plain [Strong [Str "November 2025"], Str " -- Claude Sonnet 4 and Claude Sonnet 4.5 models on the AI Gateway support 1M context window by default"]], [Plain [Strong [Str "August 2025"], Str " -- Reasoning effort control added to Playground with adjustable reasoning parameters and a \"minimal\" option for faster responses"]], [Plain [Strong [Str "August 2025"], Str " -- GPT-5 model family pricing support across multiple providers"]], [Plain [Strong [Str "August 2025"], Str " -- Pricing support for GPT-OSS models (Fireworks, Groq, OpenRouter) and Claude Opus 4.1"]], [Plain [Strong [Str "July 2025"], Str " -- Prompt Management V2 launched with composability, version control, and instant deployment through the AI Gateway via prompt IDs"]], [Plain [Strong [Str "July 2025"], Str " -- Automatic timezone detection and country-based request filtering powered by Cloudflare's edge network"]]], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "Helicone Documentation: ", Link ("", [], []) [Str "https://docs.helicone.ai/"] ("https://docs.helicone.ai/", "")]], [Plain [Str "Helicone GitHub Repository: ", Link ("", [], []) [Str "https://github.com/Helicone/helicone"] ("https://github.com/Helicone/helicone", "")]], [Plain [Str "Helicone Changelog: ", Link ("", [], []) [Str "https://www.helicone.ai/changelog"] ("https://www.helicone.ai/changelog", "")]], [Plain [Str "Helicone Pricing: ", Link ("", [], []) [Str "https://www.helicone.ai/pricing"] ("https://www.helicone.ai/pricing", "")]], [Plain [Str "Helicone Header Directory: ", Link ("", [], []) [Str "https://docs.helicone.ai/helicone-headers/header-directory"] ("https://docs.helicone.ai/helicone-headers/header-directory", "")]], [Plain [Str "Helicone Gateway Integration: ", Link ("", [], []) [Str "https://docs.helicone.ai/getting-started/integration-method/gateway"] ("https://docs.helicone.ai/getting-started/integration-method/gateway", "")]], [Plain [Str "Helicone OpenLLMetry Integration: ", Link ("", [], []) [Str "https://docs.helicone.ai/getting-started/integration-method/openllmetry"] ("https://docs.helicone.ai/getting-started/integration-method/openllmetry", "")]], [Plain [Str "Helicone REST API Reference: ", Link ("", [], []) [Str "https://docs.helicone.ai/rest/request/post-v1requestquery"] ("https://docs.helicone.ai/rest/request/post-v1requestquery", "")]]]]