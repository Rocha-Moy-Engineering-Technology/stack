[Header 1 ("groq", [], []) [Str "Groq"], BlockQuote [Para [Str "Hosted API for fast LLM inference on custom LPU hardware"]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.5)), (AlignDefault, (ColWidth 0.5))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Name"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Groq"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Hosted Inference APIs"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "API/Infra"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "No"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "N/A"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "N/A"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Docs"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "console.groq.com/docs"] ("https://console.groq.com/docs/overview", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "Groq is an inference platform built on custom Language Processing Unit (LPU) hardware, designed to deliver fast Large Language Model (LLM) inference through an OpenAI-compatible API. Unlike GPU-based inference providers, Groq uses purpose-built Application-Specific Integrated Circuit (ASIC) silicon optimized for sequential token generation, producing deterministic low-latency inference. The platform exposes a REST API at ", Code ("", [], []) "https://api.groq.com/openai/v1", Str " and provides official Software Development Kits (SDKs) for Python and JavaScript/TypeScript. Groq hosts a curated set of open-weight models spanning text generation, reasoning, speech-to-text, text-to-speech, vision, and content moderation. The platform also offers agentic AI systems (Compound and Compound Mini) with built-in tool orchestration, a Responses API for advanced agentic workflows, and Model Context Protocol (MCP) support for connecting to external tool servers. ", Str "[", Str "1", Str "]"], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], BulletList [[Plain [Strong [Str "Language Processing Unit (LPU)"], Str ": Groq's custom ASIC hardware architecture, purpose-built for sequential inference workloads rather than the parallel matrix operations GPUs are optimized for. The LPU architecture delivers deterministic, low-latency token generation. Models reside in LPU memory continuously, eliminating cold-start latency."]], [Plain [Strong [Str "OpenAI-Compatible API"], Str ": Groq's API follows the OpenAI chat completions interface, meaning existing code targeting the OpenAI SDK can be redirected to Groq by changing the base URL and API key with minimal modification. Known incompatibilities include lack of support for ", Code ("", [], []) "logprobs", Str ", ", Code ("", [], []) "logit_bias", Str ", ", Code ("", [], []) "top_logprobs", Str ", ", Code ("", [], []) "messages[].name", Str ", and the N parameter (must equal 1). A temperature value of 0 is converted to ", Code ("", [], []) "1e-8", Str ". ", Str "[", Str "12", Str "]"]], [Plain [Strong [Str "Service Tiers"], Str ": Groq offers three processing tiers -- Performance Tier with dedicated compute resources and guaranteed availability, Flex Processing for cost-optimized high-throughput workloads with 10x higher rate limits but no availability guarantee, and Batch Processing for asynchronous bulk inference at 50% lower cost with 24-hour to 7-day completion windows. ", Str "[", Str "6", Str "]", Str "[", Str "8", Str "]"]], [Plain [Strong [Str "Prompt Caching"], Str ": Groq automatically caches prompt prefixes from recent requests. When a subsequent request shares the same prefix, cached computation is reused, reducing both latency and cost by 50% for cached token portions. Cached tokens do not count toward rate limits. Caches expire after 2 hours without use. Minimum cacheable prompt length varies by model (128 to 1024 tokens). ", Str "[", Str "7", Str "]"]], [Plain [Strong [Str "Groq Compound"], Str ": Agentic AI systems (Compound and Compound Mini) with built-in tools including web search, code execution, Wolfram Alpha integration, and parallel browser automation (up to 10 pages simultaneously). These handle tool orchestration autonomously in a single API call. ", Str "[", Str "14", Str "]"]], [Plain [Strong [Str "Responses API (Beta)"], Str ": An OpenAI-compatible Responses API supporting text and image inputs, function calling, built-in tools, MCP integration, structured outputs, and reasoning. Does not yet support stateful conversations (", Code ("", [], []) "previous_response_id", Str " is unavailable). ", Str "[", Str "10", Str "]"]], [Plain [Strong [Str "Model Context Protocol (MCP)"], Str ": Server-side remote tool calling via MCP servers. Groq discovers tools from MCP servers, passes definitions to the model, executes tool calls, and returns results -- all within a single API request. ", Str "[", Str "11", Str "]"]], [Plain [Strong [Str "LoRA Inference"], Str ": Enterprise-only support for Low-Rank Adaptation (LoRA) adapters, allowing serving fine-tuned model variants without hosting separate full model copies. Adapters must be trained externally and uploaded to Groq. Currently limited to ", Code ("", [], []) "llama-3.1-8b-instant", Str " base model. ", Str "[", Str "9", Str "]"]]], Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Para [Str "Groq's architecture consists of three layers:"], OrderedList (1, DefaultStyle, DefaultDelim) [[Plain [Strong [Str "Hardware Layer"], Str ": Custom LPU chips arranged in GroqRack systems. Each LPU handles inference deterministically, meaning the same input produces identical timing characteristics across runs. This contrasts with GPU inference, where batching and scheduling introduce variable latency."]], [Plain [Strong [Str "API Gateway Layer"], Str ": An OpenAI-compatible REST API that routes requests to model-specific inference endpoints. The gateway handles authentication, rate limiting, prompt caching, service tier selection, and MCP tool orchestration."]], [Plain [Strong [Str "Model Serving Layer"], Str ": Pre-loaded open-weight models served directly from LPU memory. Models are not loaded on-demand; they reside in hardware memory continuously, eliminating cold-start latency. Inference speeds range from 200 to 1,000+ tokens per second depending on model size."]]], Header 2 ("key-features", ["unnumbered", "unlisted"], []) [Str "Key Features"], BulletList [[Plain [Strong [Str "Text Generation (Chat Completions)"], Str ": Standard chat completions endpoint supporting streaming, asynchronous calls, stop sequences, temperature control (0.0 to 2.0, default 0.5), top-p sampling, and max completion tokens. ", Str "[", Str "2", Str "]"]], [Plain [Strong [Str "Reasoning"], Str ": Dedicated reasoning capabilities via GPT-OSS models (with ", Code ("", [], []) "include_reasoning", Str " parameter and ", Code ("", [], []) "low", Str "/", Code ("", [], []) "medium", Str "/", Code ("", [], []) "high", Str " effort levels) and Qwen3-32B (with ", Code ("", [], []) "reasoning_format", Str " parameter supporting ", Code ("", [], []) "parsed", Str ", ", Code ("", [], []) "raw", Str ", or ", Code ("", [], []) "hidden", Str " modes). Recommended temperature 0.5-0.7. Cannot use ", Code ("", [], []) "raw", Str " format with JSON mode or tool use. ", Str "[", Str "4", Str "]"]], [Plain [Strong [Str "Speech-to-Text"], Str ": Transcription and translation via Whisper model variants. Supports FLAC, MP3, MP4, MPEG, MPGA, M4A, OGG, WAV, and WebM formats. File size limits: 25 MB (free tier), 100 MB (dev tier). Audio is downsampled to 16KHz mono. Response formats include JSON, verbose_json (with timestamps and quality metadata), and plain text. ", Str "[", Str "5", Str "]"]], [Plain [Strong [Str "Text-to-Speech"], Str ": Audio generation via Orpheus models (", Code ("", [], []) "canopylabs/orpheus-v1-english", Str " and ", Code ("", [], []) "canopylabs/orpheus-arabic-saudi", Str ") with vocal direction controls (e.g., ", Code ("", [], []) "[cheerful]", Str " tags). Output defaults to WAV format. ", Str "[", Str "13", Str "]"]], [Plain [Strong [Str "Vision"], Str ": Image analysis through multimodal models (Llama 4 Scout 17B). Supports URL-based images (up to 20 MB) and base64-encoded images (up to 4 MB). Maximum 5 images per request, 33 megapixel resolution ceiling. Includes Optical Character Recognition (OCR) capabilities. ", Str "[", Str "3", Str "]"]], [Plain [Strong [Str "Tool Use"], Str ": Function calling support with three patterns -- built-in tools (web search, code execution, browser automation, Wolfram Alpha) executed on Groq infrastructure; remote MCP tools via third-party servers; and local tool calling with custom function definitions. Supports parallel tool calls on most models. ", Str "[", Str "15", Str "]"]], [Plain [Strong [Str "Structured Outputs"], Str ": Two modes -- Strict mode (", Code ("", [], []) "strict: true", Str ") with constrained decoding guaranteeing 100% schema adherence (GPT-OSS models only), and Best-effort mode (", Code ("", [], []) "strict: false", Str ") available across more models with retry-based validation. Also supports basic JSON Object Mode for models without full structured output support. ", Str "[", Str "16", Str "]"]], [Plain [Strong [Str "Content Moderation"], Str ": GPT-OSS-Safeguard 20B for bring-your-own-policy trust and safety workflows with reasoning explanations. Llama Prompt Guard 2 (22M and 86M parameter variants) for prompt injection detection. Llama Guard 4 12B for multimodal content moderation using MLCommons Taxonomy. ", Str "[", Str "17", Str "]"]], [Plain [Strong [Str "Batch Processing"], Str ": Asynchronous batch API supporting chat completions, audio transcription, and audio translation. Up to 50,000 requests per file, 200 MB maximum. 50% cost discount versus synchronous pricing. Completion windows from 24 hours to 7 days. Results retained for 30 days. ", Str "[", Str "8", Str "]"]]], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], BulletList [[Plain [Strong [Str "Low-Latency Chatbots"], Str ": Applications requiring sub-second response times for interactive conversation, where Groq's LPU latency advantage over GPU inference is most pronounced. At 300-1,000+ tokens per second, multi-turn conversations feel instantaneous."]], [Plain [Strong [Str "Agentic Workflows"], Str ": Tool-augmented agents that combine text generation with function calling (web search, code execution, MCP tools). Fast inference reduces end-to-end agent loop time -- a typical multi-tool workflow requiring 3-5 inference calls completes in seconds rather than minutes."]], [Plain [Strong [Str "Real-Time Speech Processing"], Str ": Transcription pipelines using Whisper models at 189-216x real-time speed factor for live audio streams or recorded media."]], [Plain [Strong [Str "High-Throughput Document Processing"], Str ": Batch processing tier for summarization, extraction, or classification across large document corpora at 50% reduced cost."]], [Plain [Strong [Str "Content Moderation Pipelines"], Str ": Automated safety screening with custom policies using GPT-OSS-Safeguard, prompt injection detection via Llama Prompt Guard, or multimodal content analysis with Llama Guard 4."]], [Plain [Strong [Str "Structured Data Extraction"], Str ": Vision OCR combined with structured outputs for extracting typed data from documents and images with guaranteed schema compliance."]]], Header 2 ("api-reference", ["unnumbered", "unlisted"], []) [Str "API Reference"], Header 3 ("endpoints", ["unnumbered", "unlisted"], []) [Str "Endpoints"], Para [Strong [Str "Chat Completions"], Str ": ", Code ("", [], []) "POST /openai/v1/chat/completions", Str " -- Creates a model response for a chat conversation. ", Str "[", Str "2", Str "]"], Para [Strong [Str "Responses (Beta)"], Str ": ", Code ("", [], []) "POST /openai/v1/responses", Str " -- Advanced API for agentic workflows with built-in tool support and MCP integration. ", Str "[", Str "10", Str "]"], Para [Strong [Str "Transcription"], Str ": ", Code ("", [], []) "POST /openai/v1/audio/transcriptions", Str " -- Transcribes audio into the input language. ", Str "[", Str "5", Str "]"], Para [Strong [Str "Translation"], Str ": ", Code ("", [], []) "POST /openai/v1/audio/translations", Str " -- Translates audio into English. ", Str "[", Str "5", Str "]"], Para [Strong [Str "Speech"], Str ": ", Code ("", [], []) "POST /openai/v1/audio/speech", Str " -- Generates audio from input text. ", Str "[", Str "13", Str "]"], Para [Strong [Str "Models"], Str ": ", Code ("", [], []) "GET /openai/v1/models", Str " -- Lists available models. ", Code ("", [], []) "GET /openai/v1/models/{model}", Str " -- Retrieves model details."], Para [Strong [Str "Batches"], Str ": ", Code ("", [], []) "POST /openai/v1/batches", Str " -- Creates a batch job. ", Code ("", [], []) "GET /openai/v1/batches/{batch_id}", Str " -- Retrieves batch status. ", Code ("", [], []) "POST /openai/v1/batches/{batch_id}/cancel", Str " -- Cancels a batch. ", Str "[", Str "8", Str "]"], Para [Strong [Str "Files"], Str ": ", Code ("", [], []) "POST /openai/v1/files", Str " -- Uploads a file (100 MB max, JSONL). ", Code ("", [], []) "GET /openai/v1/files", Str " -- Lists files. ", Code ("", [], []) "GET /openai/v1/files/{file_id}/content", Str " -- Downloads file content."], Para [Strong [Str "Fine-Tuning (Beta)"], Str ": ", Code ("", [], []) "POST /v1/fine_tunings", Str " -- Registers a LoRA adapter. ", Code ("", [], []) "GET /v1/fine_tunings", Str " -- Lists adapters. Enterprise only. ", Str "[", Str "9", Str "]"], Header 3 ("chat-completions-parameters", ["unnumbered", "unlisted"], []) [Str "Chat Completions Parameters"], BulletList [[Plain [Code ("", [], []) "messages", Str " (required): Array of message objects with ", Code ("", [], []) "role", Str " (system, user, assistant, tool) and ", Code ("", [], []) "content", Str " fields."]], [Plain [Code ("", [], []) "model", Str " (required): Model identifier string (e.g., ", Code ("", [], []) "llama-3.3-70b-versatile", Str ")."]], [Plain [Code ("", [], []) "temperature", Str ": Sampling temperature, default ", Code ("", [], []) "0.5", Str ". Range ", Code ("", [], []) "0.0", Str " to ", Code ("", [], []) "2.0", Str "."]], [Plain [Code ("", [], []) "max_completion_tokens", Str ": Maximum tokens in the generated response."]], [Plain [Code ("", [], []) "top_p", Str ": Nucleus sampling threshold."]], [Plain [Code ("", [], []) "stop", Str ": String or array of strings where the model stops generating."]], [Plain [Code ("", [], []) "stream", Str ": Boolean to enable Server-Sent Events (SSE) streaming of partial responses."]], [Plain [Code ("", [], []) "tools", Str ": Array of tool definitions in JSON Schema format for function calling."]], [Plain [Code ("", [], []) "response_format", Str ": Structured output specification (", Code ("", [], []) "json_schema", Str " or ", Code ("", [], []) "json_object", Str ")."]], [Plain [Code ("", [], []) "service_tier", Str ": Processing tier selection (", Code ("", [], []) "flex", Str " for high-throughput)."]]], Header 3 ("available-models", ["unnumbered", "unlisted"], []) [Str "Available Models"], Para [Strong [Str "Text Generation (Production)"], Str ":"], BulletList [[Plain [Code ("", [], []) "llama-3.1-8b-instant", Str " -- 8B parameter Llama 3.1, 131,072 context window, 560 tps, $0.05/$0.08 per million tokens"]], [Plain [Code ("", [], []) "llama-3.3-70b-versatile", Str " -- 70B parameter Llama 3.3, 131,072 context window, 280 tps, $0.59/$0.79 per million tokens"]], [Plain [Code ("", [], []) "openai/gpt-oss-20b", Str " -- 20B parameter GPT-OSS, 131,072 context window, 1,000 tps, $0.075/$0.30 per million tokens"]], [Plain [Code ("", [], []) "openai/gpt-oss-120b", Str " -- 120B parameter GPT-OSS, 131,072 context window, 500+ tps"]], [Plain [Code ("", [], []) "openai/gpt-oss-safeguard-20b", Str " -- 20B parameter safety model, 131,072 context window, ", Str "~", Str "1,000 tps"]]], Para [Strong [Str "Agentic Systems"], Str ":"], BulletList [[Plain [Code ("", [], []) "groq/compound", Str " -- Agentic AI with built-in tools, 131,072 context, 8,192 max completion, ", Str "~", Str "450 tps"]], [Plain [Code ("", [], []) "groq/compound-mini", Str " -- Lightweight agentic AI, 131,072 context, 8,192 max completion, ", Str "~", Str "450 tps"]]], Para [Strong [Str "Speech-to-Text"], Str ":"], BulletList [[Plain [Code ("", [], []) "whisper-large-v3", Str " -- Full Whisper v3, $0.111/hour, 10.3% word error rate, 189x real-time, supports translation"]], [Plain [Code ("", [], []) "whisper-large-v3-turbo", Str " -- Optimized Whisper v3, $0.04/hour, 12% word error rate, 216x real-time"]]], Para [Strong [Str "Text-to-Speech"], Str ":"], BulletList [[Plain [Code ("", [], []) "canopylabs/orpheus-v1-english", Str " -- Expressive English TTS with vocal direction controls"]], [Plain [Code ("", [], []) "canopylabs/orpheus-arabic-saudi", Str " -- Saudi Arabic dialect synthesis"]]], Para [Strong [Str "Safety"], Str ":"], BulletList [[Plain [Code ("", [], []) "llama-guard-4-12b", Str " -- Multimodal content moderation, 128K context"]], [Plain [Code ("", [], []) "meta-llama/llama-prompt-guard-2-86m", Str " -- Prompt injection detection (86M params)"]], [Plain [Code ("", [], []) "meta-llama/llama-prompt-guard-2-22m", Str " -- Prompt injection detection (22M params)"]]], Para [Strong [Str "Preview Models"], Str " (evaluation only, may be discontinued without notice):"], BulletList [[Plain [Code ("", [], []) "meta-llama/llama-4-scout-17b-16e-instruct", Str " -- 17B x 16 expert MoE, 750 tps, vision support"]], [Plain [Code ("", [], []) "moonshotai/kimi-k2-instruct-0905", Str " -- 262,144 context, 200 tps, $1.00/$3.00 per million tokens"]], [Plain [Code ("", [], []) "qwen/qwen3-32b", Str " -- 128K context, 400 tps, reasoning support, $0.29/$0.59 per million tokens"]]], Header 2 ("configuration", ["unnumbered", "unlisted"], []) [Str "Configuration"], Header 3 ("client-configuration-python", ["unnumbered", "unlisted"], []) [Str "Client Configuration (Python)"], CodeBlock ("", ["python"], []) "from groq import Groq

# Reads GROQ_API_KEY from environment automatically
client = Groq()

# Explicit configuration
client = Groq(
    api_key=\"gsk_your_api_key_here\",
    base_url=\"https://api.groq.com/openai/v1\"
)

# Async client
from groq import AsyncGroq
async_client = AsyncGroq()
", Header 3 ("client-configuration-javascripttypescript", ["unnumbered", "unlisted"], []) [Str "Client Configuration (JavaScript/TypeScript)"], CodeBlock ("", ["typescript"], []) "import Groq from \"groq-sdk\";

// Reads GROQ_API_KEY from environment automatically
const client = new Groq();

// Explicit configuration
const client = new Groq({
    apiKey: \"gsk_your_api_key_here\"
});
", Header 3 ("environment-variables", ["unnumbered", "unlisted"], []) [Str "Environment Variables"], BulletList [[Plain [Code ("", [], []) "GROQ_API_KEY", Str ": Required. API key for authentication obtained from ", Code ("", [], []) "https://console.groq.com", Str "."]]], Header 3 ("rate-limits", ["unnumbered", "unlisted"], []) [Str "Rate Limits"], Para [Str "Rate limits are enforced at the organization level across six dimensions: requests per minute (RPM), requests per day (RPD), tokens per minute (TPM), tokens per day (TPD), audio seconds per hour (ASH), and audio seconds per day (ASD). The API returns HTTP 429 when any limit is exceeded, with ", Code ("", [], []) "retry-after", Str " and ", Code ("", [], []) "x-ratelimit-remaining-*", Str " headers. Cached tokens do not count toward rate limits. ", Str "[", Str "6", Str "]"], Para [Strong [Str "Free tier examples"], Str ":"], BulletList [[Plain [Code ("", [], []) "llama-3.1-8b-instant", Str ": 30 RPM, 14,400 RPD, 6,000 TPM, 500,000 TPD"]], [Plain [Code ("", [], []) "llama-3.3-70b-versatile", Str ": 30 RPM, 1,000 RPD, 12,000 TPM, 100,000 TPD"]], [Plain [Code ("", [], []) "whisper-large-v3", Str ": 20 RPM, 2,000 RPD"]]], Para [Str "Higher limits are available on the Developer plan and for enterprise workloads."], Header 3 ("inference-metrics", ["unnumbered", "unlisted"], []) [Str "Inference Metrics"], Para [Str "To include detailed performance metrics in API responses, set the header ", Code ("", [], []) "Groq-Beta: inference-metrics", Str ". Response metadata includes completion time, prompt processing time, queue time, and total request duration. ", Str "[", Str "10", Str "]"], Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Header 3 ("openai-sdk-compatibility", ["unnumbered", "unlisted"], []) [Str "OpenAI SDK Compatibility"], Para [Str "Groq can be used as a drop-in replacement with the OpenAI Python SDK by overriding the base URL:"], CodeBlock ("", ["python"], []) "from openai import OpenAI

client = OpenAI(
    api_key=\"gsk_your_api_key_here\",
    base_url=\"https://api.groq.com/openai/v1\"
)

response = client.chat.completions.create(
    messages=[{\"role\": \"user\", \"content\": \"Hello\"}],
    model=\"llama-3.3-70b-versatile\"
)
", Para [Str "Unsupported OpenAI parameters: ", Code ("", [], []) "logprobs", Str ", ", Code ("", [], []) "logit_bias", Str ", ", Code ("", [], []) "top_logprobs", Str ", ", Code ("", [], []) "messages[].name", Str ", N > 1, and ", Code ("", [], []) "vtt", Str "/", Code ("", [], []) "srt", Str " text completion formats. ", Str "[", Str "12", Str "]"], Header 3 ("restcurl", ["unnumbered", "unlisted"], []) [Str "REST/curl"], CodeBlock ("", ["bash"], []) "curl -X POST \"https://api.groq.com/openai/v1/chat/completions\" \\
  -H \"Authorization: Bearer $GROQ_API_KEY\" \\
  -H \"Content-Type: application/json\" \\
  -d '{
    \"messages\": [{\"role\": \"user\", \"content\": \"Hello\"}],
    \"model\": \"llama-3.3-70b-versatile\"
  }'
", Header 3 ("flex-processing", ["unnumbered", "unlisted"], []) [Str "Flex Processing"], Para [Str "Add ", Code ("", [], []) "\"service_tier\": \"flex\"", Str " to the request body for 10x higher rate limits (paid plans only). Requests may fail with HTTP 498 when capacity is unavailable; implement jittered backoff and retries. ", Str "[", Str "6", Str "]"], Header 3 ("prompt-caching-optimization", ["unnumbered", "unlisted"], []) [Str "Prompt Caching Optimization"], Para [Str "Place static content (system prompts, tool definitions, few-shot examples) at the beginning of messages and dynamic content (user queries, session data) at the end to maximize cache hit rate. Monitor cache performance via the ", Code ("", [], []) "prompt_tokens_details.cached_tokens", Str " field in API responses. ", Str "[", Str "7", Str "]"], Header 3 ("remote-mcp-integration", ["unnumbered", "unlisted"], []) [Str "Remote MCP Integration"], CodeBlock ("", ["javascript"], []) "const response = await client.responses.create({
    model: \"openai/gpt-oss-120b\",
    input: \"What models are trending on Huggingface?\",
    tools: [{
        type: \"mcp\",
        server_label: \"Huggingface\",
        server_url: \"https://huggingface.co/mcp\",
        require_approval: \"never\"
    }]
});
", Para [Str "Multiple MCP servers can be combined in a single request. Only connect to trusted servers, as MCP servers have access to all data in the model's context including messages, system prompts, and conversation history. ", Str "[", Str "11", Str "]"], Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Header 3 ("basic-chat-completion", ["unnumbered", "unlisted"], []) [Str "Basic Chat Completion"], CodeBlock ("", ["python"], []) "from groq import Groq

client = Groq()
chat_completion = client.chat.completions.create(
    messages=[
        {\"role\": \"system\", \"content\": \"You are a helpful assistant.\"},
        {\"role\": \"user\", \"content\": \"Explain the importance of fast language models\"}
    ],
    model=\"llama-3.3-70b-versatile\"
)
print(chat_completion.choices[0].message.content)
", Header 3 ("streaming-response", ["unnumbered", "unlisted"], []) [Str "Streaming Response"], CodeBlock ("", ["python"], []) "from groq import Groq

client = Groq()
stream = client.chat.completions.create(
    messages=[
        {\"role\": \"system\", \"content\": \"You are a helpful assistant.\"},
        {\"role\": \"user\", \"content\": \"Explain quantum computing in simple terms.\"}
    ],
    model=\"llama-3.3-70b-versatile\",
    temperature=0.5,
    max_completion_tokens=1024,
    top_p=1,
    stream=True
)
for chunk in stream:
    content = chunk.choices[0].delta.content
    if content:
        print(content, end=\"\")
", Header 3 ("async-chat-completion", ["unnumbered", "unlisted"], []) [Str "Async Chat Completion"], CodeBlock ("", ["python"], []) "import asyncio
from groq import AsyncGroq

async def main():
    client = AsyncGroq()
    chat_completion = await client.chat.completions.create(
        messages=[
            {\"role\": \"user\", \"content\": \"Explain the importance of fast language models\"}
        ],
        model=\"llama-3.3-70b-versatile\"
    )
    print(chat_completion.choices[0].message.content)

asyncio.run(main())
", Header 3 ("structured-output-with-json-schema-strict-mode", ["unnumbered", "unlisted"], []) [Str "Structured Output with JSON Schema (Strict Mode)"], CodeBlock ("", ["python"], []) "from groq import Groq

client = Groq()
response = client.chat.completions.create(
    messages=[{\"role\": \"user\", \"content\": \"List three programming languages with their use cases.\"}],
    model=\"openai/gpt-oss-20b\",
    response_format={
        \"type\": \"json_schema\",
        \"json_schema\": {
            \"name\": \"languages\",
            \"strict\": True,
            \"schema\": {
                \"type\": \"object\",
                \"properties\": {
                    \"languages\": {
                        \"type\": \"array\",
                        \"items\": {
                            \"type\": \"object\",
                            \"properties\": {
                                \"name\": {\"type\": \"string\"},
                                \"use_case\": {\"type\": \"string\"}
                            },
                            \"required\": [\"name\", \"use_case\"],
                            \"additionalProperties\": False
                        }
                    }
                },
                \"required\": [\"languages\"],
                \"additionalProperties\": False
            }
        }
    }
)
", Header 3 ("tool-use-function-calling", ["unnumbered", "unlisted"], []) [Str "Tool Use (Function Calling)"], CodeBlock ("", ["python"], []) "from groq import Groq
import json

client = Groq()

tools = [{
    \"type\": \"function\",
    \"function\": {
        \"name\": \"get_weather\",
        \"description\": \"Get current weather for a location\",
        \"parameters\": {
            \"type\": \"object\",
            \"properties\": {
                \"location\": {\"type\": \"string\", \"description\": \"City and state\"},
                \"unit\": {\"type\": \"string\", \"enum\": [\"celsius\", \"fahrenheit\"]}
            },
            \"required\": [\"location\"]
        }
    }
}]

response = client.chat.completions.create(
    messages=[{\"role\": \"user\", \"content\": \"What's the weather in San Francisco?\"}],
    model=\"llama-3.3-70b-versatile\",
    tools=tools,
    tool_choice=\"auto\"
)

# Handle tool call
tool_call = response.choices[0].message.tool_calls[0]
# Execute function, then send result back
follow_up = client.chat.completions.create(
    messages=[
        {\"role\": \"user\", \"content\": \"What's the weather in San Francisco?\"},
        response.choices[0].message,
        {
            \"role\": \"tool\",
            \"tool_call_id\": tool_call.id,
            \"content\": json.dumps({\"temperature\": 72, \"condition\": \"sunny\"})
        }
    ],
    model=\"llama-3.3-70b-versatile\",
    tools=tools
)
", Header 3 ("speech-to-text-transcription", ["unnumbered", "unlisted"], []) [Str "Speech-to-Text Transcription"], CodeBlock ("", ["python"], []) "from groq import Groq

client = Groq()
with open(\"audio.mp3\", \"rb\") as audio_file:
    transcription = client.audio.transcriptions.create(
        file=audio_file,
        model=\"whisper-large-v3-turbo\",
        response_format=\"verbose_json\",
        timestamp_granularities=[\"word\", \"segment\"]
    )
print(transcription.text)
", Header 3 ("reasoning-with-gpt-oss", ["unnumbered", "unlisted"], []) [Str "Reasoning with GPT-OSS"], CodeBlock ("", ["python"], []) "from groq import Groq

client = Groq()
response = client.chat.completions.create(
    messages=[{\"role\": \"user\", \"content\": \"Solve this step by step: If a train travels 120km in 2 hours, then slows to cover 80km in 2 more hours, what is the average speed for the entire trip?\"}],
    model=\"openai/gpt-oss-20b\",
    reasoning_effort=\"high\",
    include_reasoning=True,
    temperature=0.6
)
", Header 3 ("batch-processing", ["unnumbered", "unlisted"], []) [Str "Batch Processing"], CodeBlock ("", ["python"], []) "from groq import Groq

client = Groq()

# Step 1: Upload JSONL file
with open(\"batch_requests.jsonl\", \"rb\") as f:
    file = client.files.create(file=f, purpose=\"batch\")

# Step 2: Create batch
batch = client.batches.create(
    input_file_id=file.id,
    endpoint=\"/v1/chat/completions\",
    completion_window=\"24h\"
)

# Step 3: Poll for completion
import time
while batch.status not in (\"completed\", \"failed\", \"expired\"):
    time.sleep(30)
    batch = client.batches.retrieve(batch.id)

# Step 4: Download results
if batch.output_file_id:
    content = client.files.content(batch.output_file_id)
", Header 3 ("javascript-streaming", ["unnumbered", "unlisted"], []) [Str "JavaScript Streaming"], CodeBlock ("", ["javascript"], []) "import Groq from \"groq-sdk\";

const client = new Groq();

const stream = await client.chat.completions.create({
    messages: [
        {role: \"system\", content: \"You are a helpful assistant.\"},
        {role: \"user\", content: \"Explain the importance of fast language models\"}
    ],
    model: \"llama-3.3-70b-versatile\",
    temperature: 0.5,
    max_completion_tokens: 1024,
    stream: true
});

for await (const chunk of stream) {
    process.stdout.write(chunk.choices[0]?.delta?.content || \"\");
}
", Header 2 ("limitations", ["unnumbered", "unlisted"], []) [Str "Limitations"], BulletList [[Plain [Strong [Str "Model Selection"], Str ": Groq hosts a curated subset of open-weight models. Custom model uploads or arbitrary model hosting is not supported outside of enterprise-only LoRA adapters."]], [Plain [Strong [Str "Closed-Source Hardware"], Str ": The LPU architecture is proprietary. There is no self-hosted or on-premises deployment option; all inference runs on Groq's managed infrastructure."]], [Plain [Strong [Str "Rate Limits"], Str ": Each tier has distinct rate limits. Free tier limits are restrictive (e.g., 30 RPM, 1,000 RPD for 70B models). Higher limits require paid plans. Flex tier provides 10x limits but without availability guarantees."]], [Plain [Strong [Str "Context Window Constraints"], Str ": Maximum context windows vary by model, with most production models at 131,072 tokens. Context windows are fixed per model and cannot be extended."]], [Plain [Strong [Str "No Fine-Tuning Service"], Str ": Groq does not offer a fine-tuning API. LoRA adapters must be trained externally, are enterprise-only, and currently support only ", Code ("", [], []) "llama-3.1-8b-instant", Str " as a base model with ranks limited to 8, 16, 32, or 64."]], [Plain [Strong [Str "Responses API Limitations"], Str ": The beta Responses API does not yet support stateful conversations (", Code ("", [], []) "previous_response_id", Str "), ", Code ("", [], []) "store", Str ", ", Code ("", [], []) "truncation", Str ", or ", Code ("", [], []) "prompt_cache_key", Str "."]], [Plain [Strong [Str "Structured Outputs Constraints"], Str ": Strict mode (guaranteed schema adherence) is available only on GPT-OSS models. Streaming and tool use are unsupported with structured outputs."]], [Plain [Strong [Str "OpenAI SDK Gaps"], Str ": Several OpenAI parameters are unsupported: ", Code ("", [], []) "logprobs", Str ", ", Code ("", [], []) "logit_bias", Str ", ", Code ("", [], []) "top_logprobs", Str ", ", Code ("", [], []) "messages[].name", Str ", N > 1, and ", Code ("", [], []) "vtt", Str "/", Code ("", [], []) "srt", Str " formats."]], [Plain [Strong [Str "Reasoning Restrictions"], Str ": Cannot use ", Code ("", [], []) "raw", Str " reasoning format when JSON mode or tool use are enabled. System prompts should be avoided with reasoning models."]], [Plain [Strong [Str "Regional Availability"], Str ": Infrastructure is concentrated in specific data center regions. LoRA inference is not available for regional/sovereign endpoints."]]], Header 2 ("changelog", ["unnumbered", "unlisted"], []) [Str "Changelog"], BulletList [[Plain [Strong [Str "December 2025"], Str ": MCP Connectors (Beta) for Google Workspace (Gmail, Calendar, Drive) with OAuth 2.0 authentication. ", Str "[", Str "14", Str "]"]], [Plain [Strong [Str "October 2025"], Str ": GPT-OSS-Safeguard 20B safety model with bring-your-own-policy content moderation at ", Str "~", Str "1,000 tps. Prompt caching extended to GPT-OSS 120B. SDK updates: Python v0.33.0, TypeScript v0.34.0. ", Str "[", Str "14", Str "]"]], [Plain [Strong [Str "September 2025"], Str ": Remote MCP support (Beta) for external tool servers. Groq Compound and Compound Mini reach General Availability with web search, code execution, Wolfram Alpha, and parallel browser automation. Kimi K2 Instruct with 256K context. Prompt caching launched for GPT-OSS 20B and Kimi K2. ", Str "[", Str "14", Str "]"]], [Plain [Strong [Str "August 2025"], Str ": GPT-OSS 20B (1,000+ tps) and GPT-OSS 120B (500+ tps) launched. Responses API (Beta) introduced. Automatic prompt caching feature launched. ", Str "[", Str "14", Str "]"]], [Plain [Strong [Str "July 2025"], Str ": Structured Outputs with JSON Schema support. Kimi 2 Instruct (1T parameter MoE). ", Str "[", Str "14", Str "]"]], [Plain [Strong [Str "June 2025"], Str ": Qwen3-32B with reasoning support and 128K context. SDK reasoning field additions. ", Str "[", Str "14", Str "]"]], [Plain [Strong [Str "May 2025"], Str ": Llama Prompt Guard 2 (22M/86M) for prompt injection detection. Llama Guard 4 12B multimodal moderation. Compound Beta search settings with domain filtering. ", Str "[", Str "14", Str "]"]], [Plain [Strong [Str "April 2025"], Str ": Llama 4 Scout and Maverick models with vision support. Compound Beta and Compound Beta Mini agentic systems. Gemma-7b-it and Mixtral-8x7b-32768 deprecated. ", Str "[", Str "14", Str "]"]]], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " Groq Documentation Overview - https://console.groq.com/docs/overview"]], [Plain [Str "[", Str "2", Str "]", Str " Groq Text Chat API - https://console.groq.com/docs/text-chat"]], [Plain [Str "[", Str "3", Str "]", Str " Groq Vision - https://console.groq.com/docs/vision"]], [Plain [Str "[", Str "4", Str "]", Str " Groq Reasoning - https://console.groq.com/docs/reasoning"]], [Plain [Str "[", Str "5", Str "]", Str " Groq Speech-to-Text - https://console.groq.com/docs/speech-to-text"]], [Plain [Str "[", Str "6", Str "]", Str " Groq Rate Limits - https://console.groq.com/docs/rate-limits"]], [Plain [Str "[", Str "7", Str "]", Str " Groq Prompt Caching - https://console.groq.com/docs/prompt-caching"]], [Plain [Str "[", Str "8", Str "]", Str " Groq Batch Processing - https://console.groq.com/docs/batch"]], [Plain [Str "[", Str "9", Str "]", Str " Groq LoRA Inference - https://console.groq.com/docs/lora"]], [Plain [Str "[", Str "10", Str "]", Str " Groq Responses API - https://console.groq.com/docs/responses-api"]], [Plain [Str "[", Str "11", Str "]", Str " Groq MCP Support - https://console.groq.com/docs/mcp"]], [Plain [Str "[", Str "12", Str "]", Str " Groq OpenAI Compatibility - https://console.groq.com/docs/openai"]], [Plain [Str "[", Str "13", Str "]", Str " Groq Text-to-Speech - https://console.groq.com/docs/text-to-speech"]], [Plain [Str "[", Str "14", Str "]", Str " Groq Changelog - https://console.groq.com/docs/changelog"]], [Plain [Str "[", Str "15", Str "]", Str " Groq Tool Use - https://console.groq.com/docs/tool-use"]], [Plain [Str "[", Str "16", Str "]", Str " Groq Structured Outputs - https://console.groq.com/docs/structured-outputs"]], [Plain [Str "[", Str "17", Str "]", Str " Groq Content Moderation - https://console.groq.com/docs/content-moderation"]], [Plain [Str "[", Str "18", Str "]", Str " Groq API Reference - https://console.groq.com/docs/api-reference"]], [Plain [Str "[", Str "19", Str "]", Str " Groq Flex Processing - https://console.groq.com/docs/flex-processing"]], [Plain [Str "[", Str "20", Str "]", Str " Groq Error Codes - https://console.groq.com/docs/errors"]]], Header 2 ("discovery-signals", ["unnumbered", "unlisted"], []) [Str "Discovery Signals"], BlockQuote [Para [Str "Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search."]], Header 3 ("keywords", ["unnumbered", "unlisted"], []) [Str "Keywords"], Para [Str "Groq, LPU, Language Processing Unit, GroqRack, hosted inference API, OpenAI-compatible, Llama 3.1, Llama 3.3, GPT-OSS, Whisper, Orpheus TTS, Llama Guard, Groq Compound, Compound Mini, Responses API, MCP, Model Context Protocol, prompt caching, flex processing, batch processing, LoRA adapter, structured outputs, function calling, reasoning effort, vision OCR, content moderation, fast inference, deterministic latency"], Header 3 ("verb-noun-tasks", ["unnumbered", "unlisted"], []) [Str "Verb-Noun Tasks"], BulletList [[Plain [Str "Call the Groq chat completions API with the Python or TypeScript SDK"]], [Plain [Str "Redirect OpenAI SDK requests to ", Code ("", [], []) "https://api.groq.com/openai/v1"]], [Plain [Str "Stream tokens from Llama 3.3 70B at 280+ tokens/sec"]], [Plain [Str "Transcribe audio with whisper-large-v3-turbo at 216x real-time"]], [Plain [Str "Synthesize speech with Orpheus models and ", Code ("", [], []) "[cheerful]", Str " vocal direction tags"]], [Plain [Str "Run agentic workflows with Groq Compound's built-in web search and code execution"]], [Plain [Str "Connect remote MCP servers via the Responses API"]], [Plain [Str "Enforce JSON schemas in strict mode on GPT-OSS models"]], [Plain [Str "Submit batch jobs at 50% discount with 24h-7d completion windows"]], [Plain [Str "Activate ", Code ("", [], []) "service_tier=flex", Str " for 10x rate-limit headroom"]], [Plain [Str "Run prompt-injection detection with Llama Prompt Guard 2"]], [Plain [Str "Moderate multimodal content with Llama Guard 4 12B"]], [Plain [Str "Place static content first to maximize prompt cache hit rate"]]], Header 3 ("user-intent-phrases", ["unnumbered", "unlisted"], []) [Str "User Intent Phrases"], BulletList [[Plain [Str "\"I want the lowest-latency hosted LLM API\""]], [Plain [Str "\"How do I run Llama 3.3 70B at production speed?\""]], [Plain [Str "\"What's the fastest Whisper transcription endpoint?\""]], [Plain [Str "\"I need an OpenAI-compatible API on custom ASIC silicon\""]], [Plain [Str "\"How do I use MCP tools through a hosted inference API?\""]], [Plain [Str "\"How do I do agentic workflows with built-in web search?\""]], [Plain [Str "\"I need cheap batch inference with a 24-hour window\""]], [Plain [Str "\"How do I detect prompt injection attacks on user input?\""]], [Plain [Str "\"Where can I get hosted text-to-speech with emotion controls?\""]], [Plain [Str "\"How do I serve LoRA-fine-tuned Llama variants without managing GPUs?\""]], [Plain [Str "\"What's the difference between Groq's LPU and a GPU?\""]]], Header 3 ("problem-statements", ["unnumbered", "unlisted"], []) [Str "Problem Statements"], BulletList [[Plain [Str "GPU-based chat completions have variable, unpredictable latency"]], [Plain [Str "Real-time voice agents need sub-second total response time"]], [Plain [Str "Multi-tool agent loops are too slow when each call takes seconds"]], [Plain [Str "Self-hosting Whisper for live transcription pipelines is operationally heavy"]], [Plain [Str "Free-tier rate limits on OpenAI throttle high-throughput experimentation"]], [Plain [Str "Repeated system prompts inflate prompt-token costs across requests"]], [Plain [Str "Prompt injection and unsafe content need lightweight detection layers"]]], Header 3 ("when-to-pick-this", ["unnumbered", "unlisted"], []) [Str "When to Pick This"], BulletList [[Plain [Str "Pick this when deterministic low-latency token generation is paramount"]], [Plain [Str "Pick this over Cerebras when you need Whisper STT, Orpheus TTS, and vision in one API"]], [Plain [Str "Pick this when agentic workflows benefit from built-in tools (Compound) with no orchestration code"]], [Plain [Str "Pick this when remote MCP connectors are part of your tool strategy"]], [Plain [Str "Pick this when speech pipelines need 189-216x real-time transcription"]], [Plain [Str "Pick this when content moderation (Llama Guard, Prompt Guard) sits in the same stack"]], [Plain [Str "Pick this over self-hosted vLLM when you don't want to operate GPU clusters"]], [Plain [Str "Skip this when you need open weights, on-prem deployment, or custom model uploads"]]], Header 3 ("related-terms-and-aliases", ["unnumbered", "unlisted"], []) [Str "Related Terms and Aliases"], BulletList [[Plain [Str "Groq Cloud, Groq Inc., Groq API"]], [Plain [Str "LPU, Language Processing Unit, GroqChip"]], [Plain [Str "\"Fast Llama API\", \"fast inference API\""]], [Plain [Str "Compound, Compound Mini (Groq agentic systems)"]], [Plain [Str "Whisper Turbo, Whisper v3"]], [Plain [Str "Alternative to Cerebras, Together AI, Fireworks AI, SambaNova, Replicate"]], [Plain [Str "ASIC inference, deterministic inference"]]]]