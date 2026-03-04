[Header 1 ("cerebras", [], []) [Str "Cerebras"], BlockQuote [Para [Str "AI inference API on custom wafer-scale engine hardware"]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, ColWidthDefault), (AlignDefault, ColWidthDefault)] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GPU Compute & Cloud Platforms"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "API/Infra"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "No"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "N/A"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "N/A"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Documentation"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "Official Docs"] ("https://inference-docs.cerebras.ai/", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "Cerebras is an AI inference platform built on custom wafer-scale engine hardware (CS-3 chips). Rather than relying on traditional GPU clusters, Cerebras uses its proprietary chip architecture to deliver high-throughput, low-latency Large Language Model (LLM) inference. The platform exposes a cloud API that is OpenAI-compatible, allowing developers to switch from OpenAI endpoints with minimal code changes. Cerebras targets use cases where inference speed is critical, serving both general-purpose chat completions and advanced capabilities such as reasoning models, structured outputs, and function calling."], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], BulletList [[Plain [Strong [Str "Wafer-Scale Engine (WSE):"], Str " Cerebras hardware is based on wafer-scale chips (CS-3), where an entire silicon wafer acts as a single processor. This eliminates the inter-chip communication bottleneck found in traditional GPU clusters, enabling faster sequential token generation."]], [Plain [Strong [Str "OpenAI-Compatible API:"], Str " The chat completions endpoint follows the same request and response schema as the OpenAI API, making migration straightforward for applications already built against that interface."]], [Plain [Strong [Str "Service Tiers:"], Str " Cerebras offers configurable service tiers (priority, default, auto, flex) that control request scheduling and queue behavior, allowing users to trade off between latency guarantees and cost."]], [Plain [Strong [Str "Reasoning Effort:"], Str " For supported models (such as gpt-oss-120b), a ", Code ("", [], []) "reasoning_effort", Str " parameter (low, medium, high) controls how much compute the model allocates to chain-of-thought reasoning before producing a final answer."]], [Plain [Strong [Str "Prompt Caching:"], Str " Repeated prompt prefixes can be cached server-side, reducing time-to-first-token for workloads that share common system prompts or context across requests."]]], Header 2 ("installation-and-setup", ["unnumbered", "unlisted"], []) [Str "Installation and Setup"], Header 3 ("python-sdk", ["unnumbered", "unlisted"], []) [Str "Python SDK"], Para [Str "Install the Python SDK via pip:"], CodeBlock ("", ["bash"], []) "pip install cerebras-cloud-sdk
", Para [Str "Set the API key as an environment variable:"], CodeBlock ("", ["bash"], []) "export CEREBRAS_API_KEY=\"your_api_key_here\"
", Header 3 ("javascript-sdk", ["unnumbered", "unlisted"], []) [Str "JavaScript SDK"], Para [Str "Install the JavaScript SDK via npm:"], CodeBlock ("", ["bash"], []) "npm install @cerebras/cerebras_cloud_sdk
", Header 3 ("rest--curl", ["unnumbered", "unlisted"], []) [Str "REST / curl"], Para [Str "No SDK installation is required. Authenticate by passing the API key as a Bearer token in the Authorization header."], CodeBlock ("", ["bash"], []) "curl -X POST https://api.cerebras.ai/v1/chat/completions \\
  -H \"Authorization: Bearer $CEREBRAS_API_KEY\" \\
  -H \"Content-Type: application/json\" \\
  -d '{
    \"model\": \"gpt-oss-120b\",
    \"messages\": [{\"role\": \"user\", \"content\": \"Hello!\"}]
  }'
", Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Para [Str "Cerebras operates as a managed cloud inference service. The architecture has three layers:"], OrderedList (1, DefaultStyle, DefaultDelim) [[Plain [Strong [Str "Client Layer:"], Str " Applications interact with the platform through the Python SDK (", Code ("", [], []) "cerebras.cloud.sdk", Str "), the JavaScript SDK (", Code ("", [], []) "@cerebras/cerebras_cloud_sdk", Str "), or direct REST calls against the ", Code ("", [], []) "https://api.cerebras.ai/v1/", Str " base URL."]], [Plain [Strong [Str "API Gateway:"], Str " Handles authentication (Bearer token), request validation, service tier routing, and queue management. The gateway exposes an OpenAI-compatible interface so that existing tooling and libraries designed for OpenAI work without modification."]], [Plain [Strong [Str "Inference Engine:"], Str " Requests are dispatched to CS-3 wafer-scale engine hardware for model execution. The hardware architecture eliminates multi-chip communication overhead, producing low-latency token generation. Prompt caching is handled at this layer, reusing previously computed key-value states for shared prompt prefixes."]]], Para [Str "The response includes detailed ", Code ("", [], []) "time_info", Str " metrics (", Code ("", [], []) "queue_time", Str ", ", Code ("", [], []) "prompt_time", Str ", ", Code ("", [], []) "completion_time", Str ", ", Code ("", [], []) "total_time", Str ") that expose the performance characteristics of each layer."], Header 2 ("key-features-and-functionality", ["unnumbered", "unlisted"], []) [Str "Key Features and Functionality"], BulletList [[Plain [Strong [Str "High-Speed Inference:"], Str " Wafer-scale hardware delivers fast token generation compared to traditional GPU-based inference platforms, particularly for long-context and high-throughput workloads."]], [Plain [Strong [Str "Prompt Caching:"], Str " Server-side caching of prompt prefixes reduces redundant computation when multiple requests share common context (such as system prompts or few-shot examples)."]], [Plain [Strong [Str "Structured Outputs:"], Str " The ", Code ("", [], []) "response_format", Str " parameter supports ", Code ("", [], []) "json_object", Str " and ", Code ("", [], []) "json_schema", Str " modes, enforcing that model output conforms to a user-defined JSON schema."]], [Plain [Strong [Str "Streaming:"], Str " Real-time token streaming via Server-Sent Events (SSE) when the ", Code ("", [], []) "stream", Str " parameter is set to true, enabling progressive rendering in user interfaces."]], [Plain [Strong [Str "Tool Use and Function Calling:"], Str " The ", Code ("", [], []) "tools", Str " parameter accepts function definitions that the model can invoke. Parallel tool calls are supported via ", Code ("", [], []) "parallel_tool_calls", Str ", allowing the model to request multiple function executions in a single response turn."]], [Plain [Strong [Str "Reasoning Models:"], Str " The gpt-oss-120b model supports configurable reasoning effort (low, medium, high) through the ", Code ("", [], []) "reasoning_effort", Str " parameter, controlling the depth of chain-of-thought processing."]], [Plain [Strong [Str "Predicted Outputs:"], Str " The ", Code ("", [], []) "prediction", Str " parameter allows clients to supply an expected output, enabling the engine to accelerate generation when the prediction is close to the actual response."]], [Plain [Strong [Str "Clear Thinking:"], Str " The zai-glm-4.7 model supports a ", Code ("", [], []) "clear_thinking", Str " parameter for controlling the visibility of internal reasoning steps."]]], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], BulletList [[Plain [Strong [Str "Low-Latency Chat Applications:"], Str " The speed of wafer-scale inference makes Cerebras suitable for real-time conversational interfaces where response latency directly affects user experience."]], [Plain [Strong [Str "Batch Inference Pipelines:"], Str " The flex service tier and prompt caching enable cost-effective processing of large volumes of requests that share common prefixes."]], [Plain [Strong [Str "Structured Data Extraction:"], Str " JSON schema-enforced outputs are useful for extracting structured information from unstructured text (such as entity extraction, form parsing, or data normalization)."]], [Plain [Strong [Str "Agentic Tool Use:"], Str " Function calling with parallel execution supports agentic workflows where the model orchestrates multiple external tool invocations per turn."]], [Plain [Strong [Str "Reasoning-Heavy Tasks:"], Str " The configurable reasoning effort on gpt-oss-120b allows tuning the cost-accuracy tradeoff for tasks that benefit from extended chain-of-thought (such as math, coding, and multi-step analysis)."]]], Header 2 ("api-reference-summary", ["unnumbered", "unlisted"], []) [Str "API Reference Summary"], Header 3 ("endpoint", ["unnumbered", "unlisted"], []) [Str "Endpoint"], CodeBlock ("", [""], []) "POST https://api.cerebras.ai/v1/chat/completions
", Header 3 ("authentication", ["unnumbered", "unlisted"], []) [Str "Authentication"], CodeBlock ("", [""], []) "Authorization: Bearer <CEREBRAS_API_KEY>
", Header 3 ("available-models", ["unnumbered", "unlisted"], []) [Str "Available Models"], Table ("", [], []) (Caption Nothing []) [(AlignDefault, ColWidthDefault), (AlignDefault, ColWidthDefault)] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Model"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Notes"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "llama3.1-8b"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "General use"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "qwen-3-235b-a22b-instruct-2507"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Preview"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "gpt-oss-120b"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Reasoning"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "zai-glm-4.7"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Preview"]]]])] (TableFoot ("", [], []) []), Header 3 ("request-parameters", ["unnumbered", "unlisted"], []) [Str "Request Parameters"], Para [Strong [Str "Required:"]], BulletList [[Plain [Code ("", [], []) "model", Str " (string): The model identifier."]], [Plain [Code ("", [], []) "messages", Str " (array): An array of message objects, each with ", Code ("", [], []) "role", Str " (system, user, assistant) and ", Code ("", [], []) "content", Str " (string)."]]], Para [Strong [Str "Response Control:"]], BulletList [[Plain [Code ("", [], []) "max_completion_tokens", Str " (integer): Maximum number of tokens to generate."]], [Plain [Code ("", [], []) "temperature", Str " (float, 0 to 1.5): Sampling temperature."]], [Plain [Code ("", [], []) "top_p", Str " (float): Nucleus sampling threshold."]], [Plain [Code ("", [], []) "stream", Str " (boolean): Enable streaming responses."]], [Plain [Code ("", [], []) "stop", Str " (string or array): Up to 4 stop sequences."]]], Para [Strong [Str "Advanced:"]], BulletList [[Plain [Code ("", [], []) "reasoning_effort", Str " (string: low, medium, high): Chain-of-thought depth for gpt-oss-120b."]], [Plain [Code ("", [], []) "response_format", Str " (object): One of ", Code ("", [], []) "text", Str ", ", Code ("", [], []) "json_object", Str ", or ", Code ("", [], []) "json_schema", Str " with a schema definition."]], [Plain [Code ("", [], []) "tools", Str " (array): Function definitions for tool use."]], [Plain [Code ("", [], []) "tool_choice", Str " (string or object): Control which tools the model may call."]], [Plain [Code ("", [], []) "parallel_tool_calls", Str " (boolean): Allow multiple tool calls in a single response."]], [Plain [Code ("", [], []) "seed", Str " (integer): Deterministic sampling seed."]], [Plain [Code ("", [], []) "logprobs", Str " (boolean): Return log probabilities of output tokens."]], [Plain [Code ("", [], []) "top_logprobs", Str " (integer): Number of top log probabilities to return per token."]], [Plain [Code ("", [], []) "prediction", Str " (object): Predicted output for accelerated generation."]], [Plain [Code ("", [], []) "clear_thinking", Str " (boolean): Control reasoning visibility for zai-glm-4.7."]], [Plain [Code ("", [], []) "user", Str " (string): End-user identifier for tracking."]]], Para [Strong [Str "Service:"]], BulletList [[Plain [Code ("", [], []) "service_tier", Str " (string: priority, default, auto, flex): Request scheduling tier."]], [Plain [Code ("", [], []) "queue_threshold", Str " (integer): Maximum acceptable queue time before rejection."]]], Header 3 ("response-structure", ["unnumbered", "unlisted"], []) [Str "Response Structure"], BulletList [[Plain [Code ("", [], []) "choices", Str " (array): Array of completion choices, each containing ", Code ("", [], []) "message", Str " (with ", Code ("", [], []) "role", Str " and ", Code ("", [], []) "content", Str "), ", Code ("", [], []) "finish_reason", Str ", and optionally ", Code ("", [], []) "tool_calls", Str "."]], [Plain [Code ("", [], []) "usage", Str " (object): Token usage metrics (", Code ("", [], []) "prompt_tokens", Str ", ", Code ("", [], []) "completion_tokens", Str ", ", Code ("", [], []) "total_tokens", Str ")."]], [Plain [Code ("", [], []) "time_info", Str " (object): Timing breakdown with ", Code ("", [], []) "queue_time", Str ", ", Code ("", [], []) "prompt_time", Str ", ", Code ("", [], []) "completion_time", Str ", and ", Code ("", [], []) "total_time", Str "."]]], Header 2 ("configuration-and-customization", ["unnumbered", "unlisted"], []) [Str "Configuration and Customization"], Header 3 ("environment-variables", ["unnumbered", "unlisted"], []) [Str "Environment Variables"], Table ("", [], []) (Caption Nothing []) [(AlignDefault, ColWidthDefault), (AlignDefault, ColWidthDefault)] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Variable"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Description"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "CEREBRAS_API_KEY"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "API key for authentication"]]]])] (TableFoot ("", [], []) []), Header 3 ("service-tier-selection", ["unnumbered", "unlisted"], []) [Str "Service Tier Selection"], BulletList [[Plain [Strong [Str "priority:"], Str " Lowest latency, highest cost. Requests are processed immediately."]], [Plain [Strong [Str "default:"], Str " Standard scheduling with balanced latency and cost."]], [Plain [Strong [Str "auto:"], Str " Platform selects the optimal tier based on current load."]], [Plain [Strong [Str "flex:"], Str " Lowest cost, requests may be queued during peak periods. Suitable for batch workloads."]]], Para [Str "The ", Code ("", [], []) "queue_threshold", Str " parameter (in seconds) can be combined with any service tier to reject requests that would wait longer than the specified threshold."], Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Header 3 ("drop-in-replacement-for-openai", ["unnumbered", "unlisted"], []) [Str "Drop-In Replacement for OpenAI"], Para [Str "Because the API is OpenAI-compatible, applications using the OpenAI Python SDK can switch to Cerebras by changing the base URL and API key:"], CodeBlock ("", ["python"], []) "from openai import OpenAI

client = OpenAI(
    base_url=\"https://api.cerebras.ai/v1\",
    api_key=\"your_cerebras_api_key\"
)

response = client.chat.completions.create(
    model=\"gpt-oss-120b\",
    messages=[{\"role\": \"user\", \"content\": \"Explain wafer-scale computing.\"}]
)
", Header 3 ("framework-compatibility", ["unnumbered", "unlisted"], []) [Str "Framework Compatibility"], Para [Str "Any framework or library that supports OpenAI-compatible endpoints (such as LangChain, LlamaIndex, or custom orchestrators) can be pointed at the Cerebras API by configuring the base URL."], Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Header 3 ("basic-chat-completion-python-sdk", ["unnumbered", "unlisted"], []) [Str "Basic Chat Completion (Python SDK)"], CodeBlock ("", ["python"], []) "from cerebras.cloud.sdk import Cerebras

client = Cerebras(api_key=\"your_key\")

chat_completion = client.chat.completions.create(
    model=\"gpt-oss-120b\",
    messages=[{\"role\": \"user\", \"content\": \"Hello!\"}]
)

print(chat_completion.choices[0].message.content)
", Header 3 ("streaming-response", ["unnumbered", "unlisted"], []) [Str "Streaming Response"], CodeBlock ("", ["python"], []) "from cerebras.cloud.sdk import Cerebras

client = Cerebras(api_key=\"your_key\")

stream = client.chat.completions.create(
    model=\"llama3.1-8b\",
    messages=[{\"role\": \"user\", \"content\": \"Write a short poem.\"}],
    stream=True
)

for chunk in stream:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end=\"\")
", Header 3 ("structured-output-with-json-schema", ["unnumbered", "unlisted"], []) [Str "Structured Output with JSON Schema"], CodeBlock ("", ["python"], []) "from cerebras.cloud.sdk import Cerebras

client = Cerebras(api_key=\"your_key\")

response = client.chat.completions.create(
    model=\"gpt-oss-120b\",
    messages=[
        {\"role\": \"user\", \"content\": \"Extract the name and age from: John is 30 years old.\"}
    ],
    response_format={
        \"type\": \"json_schema\",
        \"json_schema\": {
            \"name\": \"person_info\",
            \"schema\": {
                \"type\": \"object\",
                \"properties\": {
                    \"name\": {\"type\": \"string\"},
                    \"age\": {\"type\": \"integer\"}
                },
                \"required\": [\"name\", \"age\"]
            }
        }
    }
)

print(response.choices[0].message.content)
", Header 3 ("function-calling", ["unnumbered", "unlisted"], []) [Str "Function Calling"], CodeBlock ("", ["python"], []) "from cerebras.cloud.sdk import Cerebras

client = Cerebras(api_key=\"your_key\")

tools = [
    {
        \"type\": \"function\",
        \"function\": {
            \"name\": \"get_weather\",
            \"description\": \"Get the current weather for a location\",
            \"parameters\": {
                \"type\": \"object\",
                \"properties\": {
                    \"location\": {
                        \"type\": \"string\",
                        \"description\": \"City name\"
                    }
                },
                \"required\": [\"location\"]
            }
        }
    }
]

response = client.chat.completions.create(
    model=\"gpt-oss-120b\",
    messages=[{\"role\": \"user\", \"content\": \"What is the weather in San Francisco?\"}],
    tools=tools,
    tool_choice=\"auto\"
)

print(response.choices[0].message.tool_calls)
", Header 3 ("reasoning-with-configurable-effort", ["unnumbered", "unlisted"], []) [Str "Reasoning with Configurable Effort"], CodeBlock ("", ["python"], []) "from cerebras.cloud.sdk import Cerebras

client = Cerebras(api_key=\"your_key\")

response = client.chat.completions.create(
    model=\"gpt-oss-120b\",
    messages=[
        {\"role\": \"user\", \"content\": \"Prove that the square root of 2 is irrational.\"}
    ],
    reasoning_effort=\"high\"
)

print(response.choices[0].message.content)
", Header 2 ("limitations-and-considerations", ["unnumbered", "unlisted"], []) [Str "Limitations and Considerations"], BulletList [[Plain [Strong [Str "Closed Source Hardware and Software:"], Str " The wafer-scale engine and inference runtime are proprietary. Users cannot self-host or inspect the inference pipeline."]], [Plain [Strong [Str "Model Selection:"], Str " The available model catalog is limited compared to general-purpose GPU cloud platforms. Only a small set of models are supported at any given time, and some are in preview status."]], [Plain [Strong [Str "No Fine-Tuning:"], Str " The platform provides inference only. Users cannot fine-tune or train custom models on Cerebras hardware through the cloud API."]], [Plain [Strong [Str "Regional Availability:"], Str " As a specialized hardware platform, availability may be constrained by data center locations and capacity."]], [Plain [Strong [Str "Preview Model Stability:"], Str " Models marked as preview (qwen-3-235b-a22b-instruct-2507, zai-glm-4.7) may change behavior or be removed without notice."]], [Plain [Strong [Str "Parameter Restrictions:"], Str " Some advanced parameters are model-specific (", Code ("", [], []) "reasoning_effort", Str " works only with gpt-oss-120b; ", Code ("", [], []) "clear_thinking", Str " works only with zai-glm-4.7), requiring conditional logic when supporting multiple models."]]], Header 2 ("changelog-highlights", ["unnumbered", "unlisted"], []) [Str "Changelog Highlights"], Para [Str "Cerebras does not publish a public changelog for the inference API. Model availability and parameter support should be verified against the current documentation, as the platform is actively evolving."], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " Cerebras Inference Documentation - ", Link ("", [], []) [Str "https://inference-docs.cerebras.ai/"] ("https://inference-docs.cerebras.ai/", "")]], [Plain [Str "[", Str "2", Str "]", Str " Cerebras API Reference - ", Link ("", [], []) [Str "https://inference-docs.cerebras.ai/api-reference/chat-completions"] ("https://inference-docs.cerebras.ai/api-reference/chat-completions", "")]]]]