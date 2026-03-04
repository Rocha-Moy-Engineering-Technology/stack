[Header 1 ("nemo-guardrails", [], []) [Str "NeMo Guardrails"], BlockQuote [Para [Str "NVIDIA toolkit for programmable safety guardrails in conversational AI"]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.5)), (AlignDefault, (ColWidth 0.5))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Guardrails & Safety"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "SDK"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Yes"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "https://github.com/NVIDIA/NeMo-Guardrails"] ("https://github.com/NVIDIA/NeMo-Guardrails", "")]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "5683"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Documentation"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "Official Docs"] ("https://docs.nvidia.com/nemo-guardrails/index.html", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "NeMo Guardrails is an open-source toolkit from NVIDIA for adding programmable guardrails to LLM-based conversational systems. It provides a framework for controlling model output, enforcing safety policies, and steering dialogue flow through a combination of a domain-specific language called Colang and YAML-based configuration. The toolkit intercepts user input and model output at multiple stages of the conversation pipeline, applying configurable validation, filtering, and transformation rules before responses reach the end user."], Para [Str "NeMo Guardrails operates independently of the underlying LLM provider. It supports OpenAI, NVIDIA-hosted models, LLaMa-2, Falcon, Vicuna, Mosaic, and other models through configurable engine backends. The toolkit integrates with LangChain as a composable Runnable, supports FastAPI-based server deployment, and provides built-in mechanisms for jailbreak detection, sensitive data masking, fact-checking, toxicity filtering, and topic restriction."], Para [Str "The project is licensed under Apache 2.0 and has accumulated over 5,600 GitHub stars. It was introduced in a 2023 EMNLP paper by Rebedea et al. and continues active development, with the latest release at v0.20.0."], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], Para [Strong [Str "Colang"], Str " is a Python-like domain-specific language for defining conversational guardrails. It models user intents, bot responses, and dialogue flows in a readable, declarative format. Two versions exist: Colang 1.0 (the default, simpler syntax for standard guardrails) and Colang 2.0 (enhanced capabilities for complex workflows with Jinja2 templating and async operations). Colang files use the ", Code ("", [], []) ".co", Str " extension."], Para [Strong [Str "Rails"], Str " are the guardrail mechanisms that intercept and process data at specific stages of the conversation pipeline. NeMo Guardrails defines five rail types, each operating at a different point in the request-response lifecycle: input rails, dialog rails, retrieval rails, execution rails, and output rails."], Para [Strong [Str "Flows"], Str " are sequences of steps defined in Colang that describe conversation patterns and guardrail logic. Flows match user intents, execute actions, evaluate conditions, and produce bot responses. They are the primary building block for both dialogue management and safety enforcement."], Para [Strong [Str "Actions"], Str " are Python functions that flows invoke to interact with external systems, perform computations, or apply validation logic. Actions can be asynchronous, access conversation context, and return structured data that flows consume for branching decisions."], Para [Strong [Str "Configuration"], Str " is split between YAML files (", Code ("", [], []) "config.yml", Str ") for model selection and rail activation, Colang files (", Code ("", [], []) ".co", Str ") for flow definitions, and Python files (", Code ("", [], []) "config.py", Str ", ", Code ("", [], []) "actions.py", Str ") for custom initialization and action registration."], Para [Strong [Str "LLMRails"], Str " is the central Python class that loads configuration, registers actions, and processes messages through the configured rail pipeline. It exposes both synchronous and asynchronous methods for generating responses."], Para [Strong [Str "RailsConfig"], Str " is the configuration container that loads and validates Colang content, YAML settings, and custom code. It can be constructed from a directory path or from inline content strings."], Header 2 ("installation-and-setup", ["unnumbered", "unlisted"], []) [Str "Installation and Setup"], Para [Str "NeMo Guardrails requires Python 3.10, 3.11, 3.12, or 3.13. A C++ compiler is needed for the ", Code ("", [], []) "annoy", Str " dependency."], Para [Str "Install the core package with NVIDIA model support:"], CodeBlock ("", ["bash"], []) "pip install \"nemoguardrails[nvidia]\"
", Para [Str "Available extras for additional functionality:"], CodeBlock ("", ["bash"], []) "pip install \"nemoguardrails[nvidia]\"       # NVIDIA-hosted model integration
pip install \"nemoguardrails[openai]\"       # OpenAI model support
pip install \"nemoguardrails[sdd]\"          # Sensitive data detection via Presidio
pip install \"nemoguardrails[eval]\"         # Evaluation tools
pip install \"nemoguardrails[tracing]\"      # OpenTelemetry support
pip install \"nemoguardrails[jailbreak]\"    # YARA-based jailbreak detection
pip install \"nemoguardrails[multilingual]\" # Language detection
pip install \"nemoguardrails[gcp]\"          # Google Cloud Platform services
pip install \"nemoguardrails[all]\"          # All extras
", Para [Str "Install from source using Poetry:"], CodeBlock ("", ["bash"], []) "git clone https://github.com/NVIDIA/NeMo-Guardrails.git
cd NeMo-Guardrails
poetry install --extras \"nvidia\"
", Para [Str "Set the API key for your chosen provider:"], CodeBlock ("", ["bash"], []) "export NVIDIA_API_KEY=\"nvapi-...\"   # NVIDIA models via build.nvidia.com
# or
export OPENAI_API_KEY=\"sk-...\"      # OpenAI models
", Para [Str "Verify the installation:"], CodeBlock ("", ["python"], []) "from nemoguardrails import LLMRails, RailsConfig

config = RailsConfig.from_content(
    colang_content=\"\"\"
define user express greeting
  \"hello\"
  \"hi\"

define flow greeting
  user express greeting
  bot express greeting
    \"Hello! How can I help you today?\"
\"\"\",
    yaml_content=\"\"\"
models:
  - type: main
    engine: openai
    model: gpt-4
\"\"\"
)

rails = LLMRails(config)
response = rails.generate(messages=[{\"role\": \"user\", \"content\": \"Hello!\"}])
print(response[\"content\"])
", Para [Str "Compiler setup for the ", Code ("", [], []) "annoy", Str " dependency on Linux/macOS:"], CodeBlock ("", ["bash"], []) "apt-get install gcc g++ python3-dev
", Para [Str "On Windows, install Microsoft C++ Build Tools version 14.0 or higher."], Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Para [Str "NeMo Guardrails processes every conversation turn through a pipeline of five rail types, each intercepting data at a specific stage:"], CodeBlock ("", [""], []) "User Input
    |
    v
[Input Rails]       -- Validate, reject, or alter user input
    |
    v
[Dialog Rails]      -- Influence LLM prompting and conversation flow
    |
    v
[Retrieval Rails]   -- Filter RAG chunks before LLM processing
    |
    v
[LLM Generation]    -- Core model inference
    |
    v
[Execution Rails]   -- Monitor custom action inputs/outputs
    |
    v
[Output Rails]      -- Validate or modify generated responses
    |
    v
Bot Response
", Para [Strong [Str "Input Rails"], Str " trigger when the system receives a new user message. They can reject messages outright (returning a canned refusal), modify the content (masking sensitive data), or pass the message through unchanged. Common input rails include jailbreak detection, toxicity checking, and Personally Identifiable Information (PII) filtering."], Para [Strong [Str "Dialog Rails"], Str " control how the LLM is prompted and how conversation flow proceeds. They match user intents against Colang flow definitions, determine which actions to execute, and select bot response templates. Dialog rails implement the conversational logic layer."], Para [Strong [Str "Retrieval Rails"], Str " trigger when the system retrieves context chunks from a knowledge base for Retrieval-Augmented Generation (RAG). They filter, mask, or reject retrieved documents before they are injected into the LLM prompt."], Para [Strong [Str "Execution Rails"], Str " monitor the inputs and outputs of custom actions invoked during flow execution. They provide a checkpoint for validating that action parameters and return values meet safety requirements."], Para [Strong [Str "Output Rails"], Str " trigger after the system generates a bot message. They validate the response against fact-checking rules, hallucination detectors, toxicity filters, topic restrictions, and sensitive data policies before delivering it to the user."], Para [Str "The standard configuration directory structure:"], CodeBlock ("", [""], []) "config/
  config.yml        -- Model selection, rail activation, parameters
  config.py         -- Custom initialization code (optional)
  actions.py        -- Custom action definitions (optional)
  rails.co          -- Colang flow definitions for guardrails
  prompts.co        -- Custom prompt templates (optional)
", Header 2 ("key-features-and-functionality", ["unnumbered", "unlisted"], []) [Str "Key Features and Functionality"], Para [Strong [Str "Jailbreak Detection"], Str ": Built-in heuristics and YARA-based pattern matching detect prompt injection and jailbreak attempts. Enable through input rail configuration:"], CodeBlock ("", ["yaml"], []) "rails:
  input:
    flows:
      - jailbreak detection heuristics
", Para [Strong [Str "Sensitive Data Detection"], Str ": Integration with Microsoft Presidio for detecting and masking PII across input, output, and retrieval rails. Supports configurable entity types:"], CodeBlock ("", ["yaml"], []) "rails:
  config:
    sensitive_data_detection:
      input:
        entities:
          - PERSON
          - EMAIL_ADDRESS
          - PHONE_NUMBER
          - CREDIT_CARD
      output:
        entities:
          - PERSON
          - EMAIL_ADDRESS
  input:
    flows:
      - mask sensitive data on input
  output:
    flows:
      - mask sensitive data on output
  retrieval:
    flows:
      - mask sensitive data on retrieval
", Para [Strong [Str "Fact-Checking and Hallucination Detection"], Str ": Output rails that verify generated responses against retrieved context, flagging or blocking responses that contain unsupported claims:"], CodeBlock ("", ["yaml"], []) "rails:
  output:
    flows:
      - self check facts
      - self check hallucination
", Para [Strong [Str "Toxicity Filtering"], Str ": Content moderation for both input and output using configurable thresholds:"], CodeBlock ("", ["yaml"], []) "rails:
  config:
    guardrails_ai:
      validators:
        - name: toxic_language
          parameters:
            threshold: 0.5
            validation_method: \"sentence\"
  input:
    flows:
      - check toxicity
", Para [Strong [Str "Topic Restriction"], Str ": Constrain conversations to approved topics and block off-topic requests:"], CodeBlock ("", ["yaml"], []) "rails:
  config:
    guardrails_ai:
      validators:
        - name: restricttotopic
          parameters:
            valid_topics: [\"technology\", \"science\", \"education\"]
  output:
    flows:
      - guardrailsai check output $validator=\"restricttotopic\"
", Para [Strong [Str "Selective Rail Execution"], Str ": Programmatically enable or disable specific rails per request using ", Code ("", [], []) "GenerationOptions", Str ":"], CodeBlock ("", ["python"], []) "from nemoguardrails.rails.llm.options import GenerationOptions

options = GenerationOptions(
    rails={
        \"input\": [\"self check input\", \"mask sensitive data on input\"],
        \"output\": [\"self check output\"],
        \"dialog\": True,
        \"retrieval\": False
    }
)

response = rails.generate(
    messages=[{\"role\": \"user\", \"content\": \"Hello\"}],
    options=options
)
", Para [Strong [Str "Knowledge Base Retrieval"], Str ": Built-in support for RAG workflows with retrieval rails that filter chunks before they reach the LLM:"], CodeBlock ("", ["colang"], []) "define extension flow generate bot message
  priority 100
  bot ...
  execute retrieve_relevant_chunks
  execute generate_bot_message
", Para [Strong [Str "Evaluation Tools"], Str ": CLI-based evaluation for testing guardrail effectiveness against topical rails, fact-checking, moderation, and hallucination detection:"], CodeBlock ("", ["bash"], []) "nemoguardrails evaluate --config path/to/config
", Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], Para [Strong [Str "Customer Service Bots"], Str ": Enforce brand-safe responses, restrict conversations to supported product topics, prevent disclosure of internal policies, and mask customer PII in logs and responses."], Para [Strong [Str "Healthcare Assistants"], Str ": Apply strict guardrails to prevent medical advice beyond approved scope, detect and refuse harmful health-related queries, and ensure responses cite verified medical sources through fact-checking rails."], Para [Strong [Str "Financial Services Chatbots"], Str ": Block unauthorized investment recommendations, detect and mask financial PII (account numbers, Social Security Numbers), enforce regulatory compliance in generated responses, and restrict discussions to approved financial products."], Para [Strong [Str "Enterprise Knowledge Assistants"], Str ": Control access to sensitive information based on user roles through custom actions that validate permissions, filter retrieved documents through retrieval rails, and enforce topic boundaries aligned with departmental scope."], Para [Strong [Str "Educational Platforms"], Str ": Restrict content to age-appropriate topics, prevent generation of harmful or inappropriate material, and guide conversations toward constructive educational outcomes through dialog rails."], Para [Strong [Str "Content Moderation Systems"], Str ": Layer multiple output rails for toxicity detection, fact verification, and topic adherence to ensure all generated content meets platform community standards before publication."], Header 2 ("api-reference-summary", ["unnumbered", "unlisted"], []) [Str "API Reference Summary"], Para [Strong [Str "RailsConfig"], Str ": Configuration container for loading and validating guardrail settings."], CodeBlock ("", ["python"], []) "# Load from directory
config = RailsConfig.from_path(\"path/to/config/\")

# Load from inline content
config = RailsConfig.from_content(
    colang_content=\"...\",
    yaml_content=\"...\",
)

# Load from dictionary
config = RailsConfig.from_content(
    colang_content=\"...\",
    config={\"models\": [{\"type\": \"main\", \"engine\": \"openai\", \"model\": \"gpt-4\"}]}
)
", Para [Strong [Str "LLMRails"], Str ": Central class for message processing through the rail pipeline."], CodeBlock ("", ["python"], []) "rails = LLMRails(config)

# Synchronous generation
response = rails.generate(messages=[{\"role\": \"user\", \"content\": \"Hello\"}])

# Asynchronous generation
response = await rails.generate_async(messages=[{\"role\": \"user\", \"content\": \"Hello\"}])

# Generation with options
response = rails.generate(messages=messages, options=generation_options)

# Register custom actions
rails.register_action(my_custom_action)

# Register action parameters (global values accessible to all actions)
rails.register_action_param(\"api_key\", \"your-key-here\")
", Para [Strong [Str "GenerationOptions"], Str ": Control which rails execute per request."], CodeBlock ("", ["python"], []) "from nemoguardrails.rails.llm.options import GenerationOptions

options = GenerationOptions(
    rails={
        \"input\": True,
        \"output\": False,
        \"dialog\": True,
        \"retrieval\": True
    }
)
", Para [Strong [Str "@action Decorator"], Str ": Register Python functions as invocable actions."], CodeBlock ("", ["python"], []) "from nemoguardrails.actions import action

@action(is_system_action=True, name=\"check_permission\")
async def check_permission(user_id: str, context: dict = None, llm=None):
    return {\"allowed\": True, \"reason\": \"User is authorized\"}
", Para [Strong [Str "Server CLI"], Str ": Launch the guardrails HTTP server."], CodeBlock ("", ["bash"], []) "nemoguardrails server --config path/to/config --port 8000
", Para [Strong [Str "Evaluation CLI"], Str ": Run built-in evaluation suites."], CodeBlock ("", ["bash"], []) "nemoguardrails evaluate --config path/to/config
", Header 2 ("configuration-and-customization", ["unnumbered", "unlisted"], []) [Str "Configuration and Customization"], Para [Strong [Str "Model Configuration"], Str " (", Code ("", [], []) "config.yml", Str "): Define the LLM engine, model, and Colang version:"], CodeBlock ("", ["yaml"], []) "models:
  - type: main
    engine: openai
    model: gpt-4

# Optional: specify Colang version
colang_version: \"2.x\"
", Para [Strong [Str "Rail Activation"], Str ": Enable specific rail flows in the YAML configuration:"], CodeBlock ("", ["yaml"], []) "rails:
  input:
    flows:
      - check jailbreak
      - check input sensitive data
      - check toxicity
  output:
    flows:
      - self check facts
      - self check hallucination
      - check output sensitive data
  retrieval:
    flows:
      - check retrieval sensitive data
", Para [Strong [Str "GuardrailsAI Validators"], Str ": Configure third-party validators with parameters:"], CodeBlock ("", ["yaml"], []) "rails:
  config:
    guardrails_ai:
      validators:
        - name: guardrails_pii
          parameters:
            entities: [\"phone_number\", \"email\", \"ssn\", \"credit_card\"]
          metadata: {}
        - name: competitor_check
          parameters:
            competitors: [\"Apple\", \"Google\", \"Microsoft\"]
          metadata: {}
        - name: valid_length
          parameters:
            min: 10
            max: 500
          metadata: {}
  input:
    flows:
      - guardrailsai check input $validator=\"guardrails_pii\"
      - guardrailsai check input $validator=\"competitor_check\"
  output:
    flows:
      - guardrailsai check output $validator=\"valid_length\"
", Para [Strong [Str "Custom Actions"], Str " (", Code ("", [], []) "actions.py", Str "): Define Python functions that Colang flows invoke:"], CodeBlock ("", ["python"], []) "from nemoguardrails.actions import action
import aiohttp

@action(is_system_action=True, name=\"fetch_weather\")
async def fetch_weather(location: str, context: dict = None):
    async with aiohttp.ClientSession() as session:
        async with session.get(f\"https://api.weather.com/{location}\") as resp:
            data = await resp.json()
    return {
        \"condition\": data[\"condition\"],
        \"temperature\": data[\"temperature\"],
        \"error\": False
    }
", Para [Strong [Str "Context Passing"], Str ": Inject runtime context into the conversation for actions to consume:"], CodeBlock ("", ["python"], []) "messages = [
    {\"role\": \"context\", \"content\": {
        \"user_id\": \"user123\",
        \"session_id\": \"sess_abc\"
    }},
    {\"role\": \"user\", \"content\": \"What is my account balance?\"}
]

response = rails.generate(messages=messages)
", Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Para [Strong [Str "LangChain RunnableRails"], Str ": Wrap guardrails around LangChain chains using the ", Code ("", [], []) "RunnableRails", Str " class, which implements the LangChain Runnable interface for seamless composition:"], CodeBlock ("", ["python"], []) "from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from nemoguardrails.integrations.langchain.runnable_rails import RunnableRails

guardrails = RunnableRails(config)

prompt = ChatPromptTemplate.from_template(\"Tell me about {topic}\")
model = ChatOpenAI()
output_parser = StrOutputParser()

# Insert guardrails between prompt and model
chain = prompt | (guardrails | model) | output_parser
result = chain.invoke({\"topic\": \"machine learning\"})
", Para [Strong [Str "LangChain Streaming"], Str ": RunnableRails supports both synchronous and asynchronous streaming:"], CodeBlock ("", ["python"], []) "# Synchronous streaming
for chunk in guardrails.stream(\"What is machine learning?\"):
    print(chunk, end=\"\", flush=True)

# Asynchronous streaming
async for chunk in guardrails.astream(\"What is machine learning?\"):
    print(chunk, end=\"\", flush=True)
", Para [Strong [Str "LangChain Conditional Branching"], Str ": Combine guardrails with LangChain's branching logic:"], CodeBlock ("", ["python"], []) "from langchain_core.runnables import RunnableBranch

branch = RunnableBranch(
    (lambda x: \"technical\" in x[\"topic\"], guardrails | technical_chain),
    (lambda x: \"creative\" in x[\"topic\"], creative_chain),
    guardrails | general_chain,
)
", Para [Strong [Str "FastAPI Server Deployment"], Str ": Launch the HTTP server with OpenAI-compatible endpoints:"], CodeBlock ("", ["bash"], []) "nemoguardrails server --config path/to/config --port 8000
", Para [Str "The server exposes ", Code ("", [], []) "/v1/chat/completions", Str " for direct integration with applications expecting the OpenAI API format. Docker deployment is supported through the included Dockerfile."], Para [Strong [Str "OpenTelemetry Tracing"], Str ": Enable distributed tracing with the ", Code ("", [], []) "tracing", Str " extra for observability into rail execution:"], CodeBlock ("", ["bash"], []) "pip install \"nemoguardrails[tracing]\"
", Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Para [Strong [Str "Basic Greeting with Topic Control"], Str ":"], CodeBlock ("", ["python"], []) "from nemoguardrails import LLMRails, RailsConfig

colang_content = \"\"\"
define user express greeting
  \"hello\"
  \"hi\"
  \"hey there\"

define user ask about politics
  \"what do you think about the election\"
  \"who should I vote for\"

define flow greeting
  user express greeting
  bot express greeting
    \"Hello! I'm here to help with product questions.\"

define flow block politics
  user ask about politics
  bot refuse topic
    \"I'm not able to discuss political topics. Can I help with something product-related?\"
\"\"\"

yaml_content = \"\"\"
models:
  - type: main
    engine: openai
    model: gpt-4
\"\"\"

config = RailsConfig.from_content(
    colang_content=colang_content,
    yaml_content=yaml_content
)

rails = LLMRails(config)

response = rails.generate(
    messages=[{\"role\": \"user\", \"content\": \"Hi there!\"}]
)
print(response[\"content\"])
# \"Hello! I'm here to help with product questions.\"
", Para [Strong [Str "Custom Action with Permission Checking"], Str ":"], CodeBlock ("", ["python"], []) "from nemoguardrails import LLMRails, RailsConfig
from nemoguardrails.actions import action

@action(is_system_action=True, name=\"validate_user_tier\")
async def validate_user_tier(user_id: str, context: dict = None):
    tier_map = {\"user123\": \"premium\", \"user456\": \"free\"}
    return tier_map.get(user_id, \"free\")

colang_content = \"\"\"
define flow premium feature access
  user asks for premium feature
  $user_id = context.user_id
  $tier = execute validate_user_tier(user_id=$user_id)
  if $tier == \"free\"
    bot refuse access
      \"This feature requires a paid subscription.\"
    stop
  bot provide premium feature
\"\"\"

config = RailsConfig.from_content(
    colang_content=colang_content,
    config={\"models\": [{\"type\": \"main\", \"engine\": \"openai\", \"model\": \"gpt-4\"}]}
)

rails = LLMRails(config)
rails.register_action(validate_user_tier)

messages = [
    {\"role\": \"context\", \"content\": {\"user_id\": \"user456\"}},
    {\"role\": \"user\", \"content\": \"I want advanced analytics\"}
]

response = rails.generate(messages=messages)
print(response[\"content\"])
# \"This feature requires a paid subscription.\"
", Para [Strong [Str "Comprehensive Input/Output Guardrails"], Str ":"], CodeBlock ("", ["yaml"], []) "models:
  - type: main
    engine: openai
    model: gpt-4

rails:
  config:
    sensitive_data_detection:
      input:
        entities:
          - PERSON
          - EMAIL_ADDRESS
          - PHONE_NUMBER
          - CREDIT_CARD
      output:
        entities:
          - PERSON
          - EMAIL_ADDRESS
    guardrails_ai:
      validators:
        - name: toxic_language
          parameters:
            threshold: 0.5
            validation_method: \"sentence\"
        - name: restricttotopic
          parameters:
            valid_topics: [\"technology\", \"support\", \"billing\"]

  input:
    flows:
      - jailbreak detection heuristics
      - mask sensitive data on input
      - check toxicity

  output:
    flows:
      - self check facts
      - mask sensitive data on output
      - guardrailsai check output $validator=\"restricttotopic\"

  retrieval:
    flows:
      - mask sensitive data on retrieval
", Para [Strong [Str "Colang 2.0 with LLM-Driven Flow"], Str ":"], CodeBlock ("", ["colang"], []) "import core
import llm

flow main
  \"\"\"You are a car dealership assistant. You help customers learn about
  vehicles in our inventory. Politely decline to discuss other topics.

  Last user question is: \"{{ question }}\"
  Generate the output in the following format:

  bot say \"<<the response>>\"
  \"\"\"
  $question = await user said something
  ...
", Header 2 ("limitations-and-considerations", ["unnumbered", "unlisted"], []) [Str "Limitations and Considerations"], Para [Strong [Str "Latency Overhead"], Str ": Each active rail adds processing time to the request-response cycle. Stacking multiple input, output, and retrieval rails compounds latency. Applications with strict response time requirements should benchmark their specific rail configuration and consider disabling non-critical rails for latency-sensitive paths using ", Code ("", [], []) "GenerationOptions", Str "."], Para [Strong [Str "LLM Dependency for Rail Evaluation"], Str ": Several built-in rails (self-check facts, self-check hallucination, jailbreak detection) rely on additional LLM calls to evaluate content. This increases both latency and cost, as each guardrail check may invoke the model one or more times beyond the primary generation call."], Para [Strong [Str "Colang Learning Curve"], Str ": While Colang syntax is readable, mastering flow design, action integration, context management, and the differences between Colang 1.0 and 2.0 requires dedicated learning. Complex guardrail logic can become difficult to debug when flows interact in unexpected ways."], Para [Strong [Str "Model-Dependent Effectiveness"], Str ": The quality of guardrail enforcement depends on the underlying LLM's capabilities. Weaker models may produce less reliable results for self-check rails, fact verification, and intent classification. Guardrail configurations tuned for one model may require adjustment when switching providers."], Para [Strong [Str "No Guarantee of Complete Safety"], Str ": Guardrails reduce but do not eliminate the risk of harmful outputs. Determined adversaries may find prompts that bypass heuristic-based jailbreak detection. The toolkit is best used as one layer in a defense-in-depth security strategy rather than a standalone safety solution."], Para [Strong [Str "Python-Only SDK"], Str ": The toolkit is Python-only. Applications built in other languages must interact with NeMo Guardrails through the HTTP server API rather than embedding the SDK directly."], Para [Strong [Str "Hardware Requirements"], Str ": While the library itself runs on CPU with minimal resources (1 CPU, 4GB RAM), practical deployments require network access to external LLM APIs. On-premise model deployments may require GPU infrastructure depending on the model serving backend."], Para [Strong [Str "Colang 2.0 Maturity"], Str ": Colang 2.0 introduces significant enhancements (Jinja2 templating, async operations) but is newer than Colang 1.0. Some documentation and community examples still reference Colang 1.0 patterns, which can cause confusion when following tutorials."], Header 2 ("changelog-highlights", ["unnumbered", "unlisted"], []) [Str "Changelog Highlights"], BulletList [[Plain [Strong [Str "v0.20.0"], Str ": Latest release with continued improvements to Colang 2.0, enhanced rail configuration options, and expanded validator support."]], [Plain [Strong [Str "Colang 2.0 Introduction"], Str ": Added enhanced flow syntax with Jinja2 templating, async operations, and the ", Code ("", [], []) "...", Str " operator for LLM-driven flow continuation."]], [Plain [Strong [Str "RunnableRails"], Str ": LangChain integration providing synchronous and asynchronous streaming, conditional branching, and composable chain insertion."]], [Plain [Strong [Str "GuardrailsAI Integration"], Str ": Support for third-party validators including PII detection, competitor mention blocking, topic restriction, toxic language detection, and response length validation."]], [Plain [Strong [Str "YARA-Based Jailbreak Detection"], Str ": Pattern-matching engine for detecting known jailbreak prompt structures via the ", Code ("", [], []) "jailbreak", Str " extra."]], [Plain [Strong [Str "OpenTelemetry Tracing"], Str ": Distributed tracing support for monitoring rail execution in production environments via the ", Code ("", [], []) "tracing", Str " extra."]], [Plain [Strong [Str "Presidio Integration"], Str ": Sensitive data detection and masking across input, output, and retrieval rails with configurable entity types."]], [Plain [Strong [Str "GenerationOptions"], Str ": Per-request control over which rails execute, enabling selective rail activation and deactivation at runtime."]], [Plain [Strong [Str "Multi-Config Server"], Str ": FastAPI server supporting multiple guardrail configurations with OpenAI-compatible ", Code ("", [], []) "/v1/chat/completions", Str " endpoint."]]], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " NeMo Guardrails GitHub Repository - https://github.com/NVIDIA/NeMo-Guardrails"]], [Plain [Str "[", Str "2", Str "]", Str " NeMo Guardrails Documentation - https://docs.nvidia.com/nemo-guardrails/index.html"]], [Plain [Str "[", Str "3", Str "]", Str " Rebedea, T., et al. \"NeMo Guardrails: A Toolkit for Controllable and Safe LLM Applications with Programmable Rails.\" EMNLP 2023 System Demonstrations. https://aclanthology.org/2023.emnlp-demo.40/"]], [Plain [Str "[", Str "4", Str "]", Str " Colang Language Reference - https://github.com/NVIDIA/NeMo-Guardrails/tree/develop/docs/colang-2"]], [Plain [Str "[", Str "5", Str "]", Str " LangChain RunnableRails Integration - https://github.com/NVIDIA/NeMo-Guardrails/blob/develop/docs/user-guides/langchain/runnable-rails.md"]]]]