[Header 1 ("groq", [], []) [Str "Groq"], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.20512820512820512)), (AlignDefault, (ColWidth 0.7948717948717948))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Strong [Str "Group"]]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GPU Compute & Cloud Platforms"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Strong [Str "Type"]]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "API/Infra"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Strong [Str "Open Source"]]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "No"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Strong [Str "GitHub"]]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "N/A"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Strong [Str "Stars"]]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "N/A"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Strong [Str "Docs"]]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "Official Docs"] ("https://console.groq.com/docs/overview", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "Groq is an inference platform built on custom Language Processing Unit (LPU) hardware, designed to deliver fast Large Language Model (LLM) inference through an OpenAI-compatible API. Unlike GPU-based inference providers, Groq uses purpose-built silicon optimized for sequential token generation, which results in significantly lower latency per token. The platform exposes a REST API at ", Code ("", [], []) "https://api.groq.com/openai/v1", Str " and provides official Software Development Kits (SDKs) for Python and JavaScript/TypeScript. Groq hosts a curated set of open-weight models spanning text generation, speech-to-text, vision, and content moderation. ", Str "[", Str "1", Str "]"], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], BulletList [[Plain [Strong [Str "Language Processing Unit (LPU)"], Str ": Groq's custom Application-Specific Integrated Circuit (ASIC) hardware architecture, purpose-built for sequential inference workloads rather than the parallel matrix operations GPUs are optimized for. The LPU architecture delivers deterministic, low-latency token generation."]], [Plain [Strong [Str "OpenAI-Compatible API"], Str ": Groq's API follows the OpenAI chat completions interface, meaning existing code targeting the OpenAI SDK can be redirected to Groq by changing the base URL and API key with minimal modification."]], [Plain [Strong [Str "Service Tiers"], Str ": Groq offers three processing tiers -- Performance Tier with dedicated compute resources, Flex Processing for cost-optimized workloads, and Batch Processing for asynchronous bulk inference jobs."]], [Plain [Strong [Str "Prompt Caching"], Str ": Groq caches prompt prefixes so that repeated requests sharing the same system prompt or conversation prefix skip redundant computation, reducing both latency and cost."]], [Plain [Strong [Str "LoRA Inference"], Str ": Support for Low-Rank Adaptation (LoRA) adapters allows serving fine-tuned model variants without hosting separate full model copies."]]], Header 2 ("installation-and-setup", ["unnumbered", "unlisted"], []) [Str "Installation and Setup"], Header 3 ("python-sdk", ["unnumbered", "unlisted"], []) [Str "Python SDK"], CodeBlock ("", ["bash"], []) "pip install groq
", Header 3 ("javascripttypescript-sdk", ["unnumbered", "unlisted"], []) [Str "JavaScript/TypeScript SDK"], CodeBlock ("", ["bash"], []) "npm install groq-sdk
", Header 3 ("authentication", ["unnumbered", "unlisted"], []) [Str "Authentication"], Para [Str "Groq uses API key authentication via the ", Code ("", [], []) "GROQ_API_KEY", Str " environment variable:"], CodeBlock ("", ["bash"], []) "export GROQ_API_KEY=\"gsk_your_api_key_here\"
", Para [Str "The API key is obtained from the Groq Console at ", Code ("", [], []) "https://console.groq.com", Str ". The Python and JavaScript SDKs automatically read ", Code ("", [], []) "GROQ_API_KEY", Str " from the environment when no key is explicitly passed to the client constructor. ", Str "[", Str "1", Str "]"], Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Para [Str "Groq's architecture consists of three layers:"], OrderedList (1, DefaultStyle, DefaultDelim) [[Plain [Strong [Str "Hardware Layer"], Str ": Custom LPU chips arranged in GroqRack systems. Each LPU handles inference deterministically, meaning the same input produces identical timing characteristics across runs. This contrasts with GPU inference, where batching and scheduling introduce variable latency."]], [Plain [Strong [Str "API Gateway Layer"], Str ": An OpenAI-compatible REST API that routes requests to model-specific inference endpoints. The gateway handles authentication, rate limiting, prompt caching, and service tier selection."]], [Plain [Strong [Str "Model Serving Layer"], Str ": Pre-loaded open-weight models (Llama, Gemma, Whisper, and others) served directly from LPU memory. Models are not loaded on-demand; they reside in hardware memory continuously, eliminating cold-start latency."]]], Header 2 ("key-features", ["unnumbered", "unlisted"], []) [Str "Key Features"], BulletList [[Plain [Strong [Str "Text Generation (Chat Completions)"], Str ": Standard chat completions endpoint supporting streaming, asynchronous calls, stop sequences, temperature control, and top-p sampling. ", Str "[", Str "2", Str "]"]], [Plain [Strong [Str "Speech-to-Text"], Str ": Transcription and translation via Whisper model variants (whisper-large-v3, whisper-large-v3-turbo, distil-whisper-large-v3-en)."]], [Plain [Strong [Str "Vision"], Str ": Optical Character Recognition (OCR) and image recognition capabilities through multimodal model endpoints."]], [Plain [Strong [Str "Tool Use"], Str ": Function calling support including web search, browser automation, code execution, and Wolfram Alpha integration."]], [Plain [Strong [Str "Reasoning"], Str ": Dedicated reasoning capabilities for multi-step problem solving."]], [Plain [Strong [Str "Structured Outputs"], Str ": JSON schema validation on model outputs, ensuring responses conform to a developer-specified schema."]], [Plain [Strong [Str "Content Moderation"], Str ": Safety classification via dedicated models such as llama-guard-3-8b."]], [Plain [Strong [Str "Batch Processing"], Str ": Asynchronous batch API for submitting large volumes of requests to be processed without real-time latency requirements."]]], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], BulletList [[Plain [Strong [Str "Low-Latency Chatbots"], Str ": Applications requiring sub-second response times for interactive conversation, where Groq's LPU latency advantage over GPU inference is most pronounced."]], [Plain [Strong [Str "Real-Time Speech Processing"], Str ": Transcription pipelines using Whisper models for live audio streams or recorded media."]], [Plain [Strong [Str "High-Throughput Document Processing"], Str ": Batch processing tier for summarization, extraction, or classification across large document corpora."]], [Plain [Strong [Str "Tool-Augmented Agents"], Str ": Agentic workflows that combine text generation with tool use (web search, code execution) where fast inference reduces end-to-end agent loop time."]], [Plain [Strong [Str "Content Moderation Pipelines"], Str ": Automated safety screening of user-generated content using llama-guard-3-8b before publishing or further processing."]]], Header 2 ("api-reference-summary", ["unnumbered", "unlisted"], []) [Str "API Reference Summary"], Header 3 ("chat-completions", ["unnumbered", "unlisted"], []) [Str "Chat Completions"], Para [Strong [Str "Endpoint"], Str ": ", Code ("", [], []) "POST /openai/v1/chat/completions"], Para [Strong [Str "Parameters"], Str ":"], BulletList [[Plain [Code ("", [], []) "messages", Str ": Array of message objects with ", Code ("", [], []) "role", Str " (system, user, assistant) and ", Code ("", [], []) "content", Str " fields."]], [Plain [Code ("", [], []) "model", Str ": Model identifier string (e.g., ", Code ("", [], []) "llama-3.3-70b-versatile", Str ")."]], [Plain [Code ("", [], []) "temperature", Str ": Sampling temperature, default ", Code ("", [], []) "0.5", Str ". Range ", Code ("", [], []) "0.0", Str " to ", Code ("", [], []) "2.0", Str "."]], [Plain [Code ("", [], []) "max_completion_tokens", Str ": Maximum tokens in the generated response."]], [Plain [Code ("", [], []) "top_p", Str ": Nucleus sampling threshold."]], [Plain [Code ("", [], []) "stop", Str ": String or array of strings where the model stops generating."]], [Plain [Code ("", [], []) "stream", Str ": Boolean to enable Server-Sent Events (SSE) streaming of partial responses."]]], Header 3 ("available-models", ["unnumbered", "unlisted"], []) [Str "Available Models"], Para [Strong [Str "Text Generation"], Str ":"], BulletList [[Plain [Code ("", [], []) "llama3-8b-8192", Str " -- 8B parameter Llama 3, 8192 context window"]], [Plain [Code ("", [], []) "llama3-70b-8192", Str " -- 70B parameter Llama 3, 8192 context window"]], [Plain [Code ("", [], []) "llama-3.1-8b-instant", Str " -- 8B parameter Llama 3.1, 131072 context window"]], [Plain [Code ("", [], []) "llama-3.3-70b-versatile", Str " -- 70B parameter Llama 3.3, general purpose"]], [Plain [Code ("", [], []) "gemma2-9b-it", Str " -- 9B parameter Gemma 2 instruction-tuned, 8192 context window"]], [Plain [Code ("", [], []) "openai/gpt-oss-20b", Str " -- 20B parameter open-source GPT variant"]]], Para [Strong [Str "Speech-to-Text"], Str ":"], BulletList [[Plain [Code ("", [], []) "whisper-large-v3", Str " -- Full Whisper v3"]], [Plain [Code ("", [], []) "whisper-large-v3-turbo", Str " -- Optimized Whisper v3"]], [Plain [Code ("", [], []) "distil-whisper-large-v3-en", Str " -- Distilled English-only Whisper v3"]]], Para [Strong [Str "Safety"], Str ":"], BulletList [[Plain [Code ("", [], []) "llama-guard-3-8b", Str " -- Content moderation classifier"]]], Header 2 ("configuration", ["unnumbered", "unlisted"], []) [Str "Configuration"], Header 3 ("client-configuration-python", ["unnumbered", "unlisted"], []) [Str "Client Configuration (Python)"], CodeBlock ("", ["python"], []) "from groq import Groq

# Reads GROQ_API_KEY from environment automatically
client = Groq()

# Explicit configuration
client = Groq(
    api_key=\"gsk_your_api_key_here\",
    base_url=\"https://api.groq.com/openai/v1\"
)
", Header 3 ("client-configuration-javascripttypescript", ["unnumbered", "unlisted"], []) [Str "Client Configuration (JavaScript/TypeScript)"], CodeBlock ("", ["typescript"], []) "import Groq from \"groq-sdk\";

// Reads GROQ_API_KEY from environment automatically
const client = new Groq();

// Explicit configuration
const client = new Groq({
    apiKey: \"gsk_your_api_key_here\"
});
", Header 3 ("environment-variables", ["unnumbered", "unlisted"], []) [Str "Environment Variables"], BulletList [[Plain [Code ("", [], []) "GROQ_API_KEY", Str ": Required. API key for authentication."]]], Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Header 3 ("openai-sdk-compatibility", ["unnumbered", "unlisted"], []) [Str "OpenAI SDK Compatibility"], Para [Str "Groq can be used as a drop-in replacement with the OpenAI Python SDK by overriding the base URL:"], CodeBlock ("", ["python"], []) "from openai import OpenAI

client = OpenAI(
    api_key=\"gsk_your_api_key_here\",
    base_url=\"https://api.groq.com/openai/v1\"
)

response = client.chat.completions.create(
    messages=[{\"role\": \"user\", \"content\": \"Hello\"}],
    model=\"llama-3.3-70b-versatile\"
)
", Header 3 ("restcurl", ["unnumbered", "unlisted"], []) [Str "REST/curl"], CodeBlock ("", ["bash"], []) "curl -X POST \"https://api.groq.com/openai/v1/chat/completions\" \\
  -H \"Authorization: Bearer $GROQ_API_KEY\" \\
  -H \"Content-Type: application/json\" \\
  -d '{
    \"messages\": [{\"role\": \"user\", \"content\": \"Hello\"}],
    \"model\": \"llama-3.3-70b-versatile\"
  }'
", Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Header 3 ("basic-chat-completion", ["unnumbered", "unlisted"], []) [Str "Basic Chat Completion"], CodeBlock ("", ["python"], []) "from groq import Groq

client = Groq()
chat_completion = client.chat.completions.create(
    messages=[{\"role\": \"user\", \"content\": \"Hello\"}],
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
    stream=True
)
for chunk in stream:
    content = chunk.choices[0].delta.content
    if content:
        print(content, end=\"\")
", Header 3 ("structured-output-with-json-schema", ["unnumbered", "unlisted"], []) [Str "Structured Output with JSON Schema"], CodeBlock ("", ["python"], []) "from groq import Groq

client = Groq()
response = client.chat.completions.create(
    messages=[{\"role\": \"user\", \"content\": \"List three programming languages.\"}],
    model=\"llama-3.3-70b-versatile\",
    response_format={
        \"type\": \"json_schema\",
        \"json_schema\": {
            \"name\": \"languages\",
            \"schema\": {
                \"type\": \"object\",
                \"properties\": {
                    \"languages\": {
                        \"type\": \"array\",
                        \"items\": {\"type\": \"string\"}
                    }
                },
                \"required\": [\"languages\"]
            }
        }
    }
)
", Header 3 ("speech-to-text-transcription", ["unnumbered", "unlisted"], []) [Str "Speech-to-Text Transcription"], CodeBlock ("", ["python"], []) "from groq import Groq

client = Groq()
with open(\"audio.mp3\", \"rb\") as audio_file:
    transcription = client.audio.transcriptions.create(
        file=audio_file,
        model=\"whisper-large-v3-turbo\"
    )
print(transcription.text)
", Header 2 ("limitations", ["unnumbered", "unlisted"], []) [Str "Limitations"], BulletList [[Plain [Strong [Str "Model Selection"], Str ": Groq hosts a curated subset of open-weight models. Custom model uploads or arbitrary model hosting is not supported outside of LoRA adapters."]], [Plain [Strong [Str "Closed-Source Hardware"], Str ": The LPU architecture is proprietary. There is no self-hosted or on-premises deployment option; all inference runs on Groq's managed infrastructure."]], [Plain [Strong [Str "Rate Limits"], Str ": Each service tier has distinct rate limits on requests per minute and tokens per minute. The Performance Tier provides the highest throughput but at higher cost."]], [Plain [Strong [Str "Context Window Constraints"], Str ": Maximum context windows vary by model, ranging from 8192 tokens (Llama 3 base, Gemma 2) to 131072 tokens (Llama 3.1 instant). These are fixed per model and cannot be extended."]], [Plain [Strong [Str "No Fine-Tuning"], Str ": Groq does not offer a fine-tuning API. LoRA adapters must be trained externally and uploaded for inference."]], [Plain [Strong [Str "Regional Availability"], Str ": Infrastructure is concentrated in specific data center regions, which may affect latency for geographically distant clients."]]], Header 2 ("changelog-highlights", ["unnumbered", "unlisted"], []) [Str "Changelog Highlights"], BulletList [[Plain [Str "Addition of Llama 3.3 70B Versatile model with improved general-purpose capabilities."]], [Plain [Str "Introduction of Batch Processing API for asynchronous bulk inference."]], [Plain [Str "Prompt caching support to reduce latency and cost for repeated prompt prefixes."]], [Plain [Str "LoRA inference support for serving fine-tuned model adapters."]], [Plain [Str "Structured output with JSON schema validation."]], [Plain [Str "Tool use expansion to include web search, browser automation, code execution, and Wolfram Alpha."]], [Plain [Str "Whisper model variants added for speech-to-text (full, turbo, distilled English)."]]], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " Groq Documentation - https://console.groq.com/docs/overview"]], [Plain [Str "[", Str "2", Str "]", Str " Groq Text Chat API - https://console.groq.com/docs/text-chat"]]]]