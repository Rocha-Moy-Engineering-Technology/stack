[Header 1 ("aws-bedrock", [], []) [Str "AWS Bedrock"], BlockQuote [Para [Str "AWS managed service for foundation model access from multiple vendors via single API"]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.5)), (AlignDefault, (ColWidth 0.5))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Managed AI Platforms"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "API/SDK/Infra"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "No"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "N/A"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "N/A"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Documentation"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "Official Docs"] ("https://docs.aws.amazon.com/bedrock/", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "Amazon Bedrock is a fully managed service that provides secure, enterprise-grade access to over 100 foundation models from leading AI companies through a single API. It enables organizations to build and scale generative AI applications without managing infrastructure, offering model inference, customization, retrieval-augmented generation (RAG), agentic workflows, and safety guardrails as integrated capabilities."], Para [Str "Bedrock abstracts away the complexity of hosting and serving large language models. Users select from a catalog of models spanning multiple providers -- including Amazon, Anthropic, OpenAI, DeepSeek, Moonshot AI, MiniMax, GLM, Qwen, and NVIDIA -- and interact with them through unified APIs. The service handles provisioning, scaling, and security, allowing teams to focus on application logic rather than infrastructure management."], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], BulletList [[Plain [Strong [Str "Foundation Models"], Str ": Pre-trained large language models available through the Bedrock model catalog, spanning text generation, code generation, and multimodal tasks."]], [Plain [Strong [Str "Model Inference"], Str ": Submitting prompts to a selected model and receiving generated responses via API calls."]], [Plain [Strong [Str "Model Customization"], Str ": Fine-tuning foundation models on proprietary data to improve performance for domain-specific tasks."]], [Plain [Strong [Str "Knowledge Bases"], Str ": Managed RAG infrastructure that integrates proprietary data sources so models can ground responses in organizational knowledge."]], [Plain [Strong [Str "Agents"], Str ": Agentic application framework enabling models to plan, invoke tools, and execute multi-step workflows autonomously."]], [Plain [Strong [Str "Guardrails"], Str ": Configurable safety and compliance controls that filter model inputs and outputs according to organizational policies."]], [Plain [Strong [Str "Prompt Caching"], Str ": Server-side caching mechanism (1-hour duration for Claude models) that reduces latency and cost for repeated or similar prompts."]]], Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Para [Str "Bedrock operates as a managed API layer between client applications and foundation model infrastructure. The architecture consists of three tiers:"], OrderedList (1, DefaultStyle, DefaultDelim) [[Plain [Strong [Str "Client Tier"], Str ": Applications interact with Bedrock through the ", Code ("", [], []) "bedrock-runtime", Str " API endpoint using boto3, the OpenAI-compatible endpoint, or the AWS CLI."]], [Plain [Strong [Str "Service Tier"], Str ": Bedrock routes requests to the appropriate model provider, applies guardrails, manages prompt caching, and orchestrates agent workflows."]], [Plain [Strong [Str "Model Tier"], Str ": Foundation models from multiple providers run on AWS-managed GPU infrastructure, isolated per customer with no cross-tenant data exposure."]]], Para [Str "All data remains within the customer's AWS account and selected region. Model providers do not receive access to customer data, and customer inputs are not used to train or improve third-party models."], Header 2 ("key-features-and-functionality", ["unnumbered", "unlisted"], []) [Str "Key Features and Functionality"], BulletList [[Plain [Strong [Str "100+ Foundation Models"], Str ": Single API access to models from Amazon (Nova 2 Pro), Anthropic (Claude Opus 4.6), OpenAI (GPT-OSS-20B), DeepSeek (V3.2), Moonshot AI (Kimi K2.5), MiniMax (M2.1), GLM (4.7, 4.7 Flash), Qwen (3 Coder Next), and NVIDIA (Nemotron 3 Nano)."]], [Plain [Strong [Str "Multiple API Formats"], Str ": Converse API (recommended), Invoke API (Bedrock-native), Chat Completions API (OpenAI-compatible), and Responses API (OpenAI-compatible)."]], [Plain [Strong [Str "Knowledge Bases for RAG"], Str ": Managed vector storage and retrieval pipeline for grounding model responses in proprietary documents."]], [Plain [Strong [Str "Agentic Applications"], Str ": Built-in agent framework with tool calling, multi-step planning, and external system integration."]], [Plain [Strong [Str "Guardrails"], Str ": Content filtering, topic restrictions, personally identifiable information (PII) redaction, and custom policy enforcement."]], [Plain [Strong [Str "Server-Side Tools"], Str ": Tool execution via OpenAI-compatible endpoints, reducing client-side orchestration complexity."]], [Plain [Strong [Str "Prompt Caching"], Str ": 1-hour server-side cache for Claude models, lowering latency and cost on repeated prompt patterns."]], [Plain [Strong [Str "NVIDIA Nemotron 3 Nano"], Str ": 256k context window with native tool calling support."]], [Plain [Strong [Str "Open-Weight Models"], Str ": Six open-weight models available (DeepSeek V3.2, MiniMax M2.1, GLM 4.7, GLM 4.7 Flash, Kimi K2.5, Qwen3 Coder Next)."]], [Plain [Strong [Str "Model Customization"], Str ": Fine-tuning and continued pre-training on proprietary datasets."]]], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], BulletList [[Plain [Strong [Str "Enterprise Chat and Assistants"], Str ": Deploy conversational AI powered by best-in-class models with organizational data grounding through Knowledge Bases."]], [Plain [Strong [Str "Code Generation and Review"], Str ": Use code-specialized models (Qwen3 Coder Next, Claude Opus 4.6) for automated code generation, explanation, and review."]], [Plain [Strong [Str "Document Processing"], Str ": Extract, summarize, and transform information from large document corpora using long-context models."]], [Plain [Strong [Str "Agentic Workflows"], Str ": Build autonomous agents that plan multi-step tasks, invoke external APIs, query databases, and produce structured outputs."]], [Plain [Strong [Str "Content Moderation"], Str ": Apply Guardrails to filter harmful or non-compliant content in user-facing applications."]], [Plain [Strong [Str "RAG Applications"], Str ": Combine Knowledge Bases with foundation models to answer questions grounded in proprietary enterprise data."]]], Header 2 ("api-reference-summary", ["unnumbered", "unlisted"], []) [Str "API Reference Summary"], Header 3 ("converse-api-recommended", ["unnumbered", "unlisted"], []) [Str "Converse API (Recommended)"], Para [Str "The Converse API provides a unified interface across all Bedrock models with consistent request and response formats."], CodeBlock ("", ["python"], []) "import boto3

client = boto3.client('bedrock-runtime', region_name='us-east-1')
response = client.converse(
    modelId='anthropic.claude-opus-4-6-v1',
    messages=[
        {
            'role': 'user',
            'content': [{'text': 'Hello'}]
        }
    ]
)
print(response['output']['message']['content'][0]['text'])
", Header 3 ("invoke-api-bedrock-native", ["unnumbered", "unlisted"], []) [Str "Invoke API (Bedrock-Native)"], Para [Str "The Invoke API provides direct model invocation with provider-specific request body formats."], CodeBlock ("", ["python"], []) "import json
import boto3

client = boto3.client('bedrock-runtime', region_name='us-east-1')
response = client.invoke_model(
    modelId='anthropic.claude-opus-4-6-v1',
    body=json.dumps({
        'anthropic_version': 'bedrock-2023-05-31',
        'messages': [{'role': 'user', 'content': 'Hello'}],
        'max_tokens': 1024
    })
)
result = json.loads(response['body'].read())
print(result['content'][0]['text'])
", Header 3 ("chat-completions-api-openai-compatible", ["unnumbered", "unlisted"], []) [Str "Chat Completions API (OpenAI-Compatible)"], Para [Str "The OpenAI-compatible endpoint allows existing OpenAI SDK integrations to target Bedrock models with minimal code changes."], CodeBlock ("", ["python"], []) "from openai import OpenAI

client = OpenAI()
response = client.chat.completions.create(
    model=\"openai.gpt-oss-120b\",
    messages=[{\"role\": \"user\", \"content\": \"Hello\"}]
)
print(response.choices[0].message.content)
", Header 3 ("responses-api-openai-compatible", ["unnumbered", "unlisted"], []) [Str "Responses API (OpenAI-Compatible)"], Para [Str "The Responses API provides an OpenAI-compatible endpoint supporting server-side tool execution and structured responses."], Header 2 ("configuration-and-customization", ["unnumbered", "unlisted"], []) [Str "Configuration and Customization"], Header 3 ("iam-permissions", ["unnumbered", "unlisted"], []) [Str "IAM Permissions"], Para [Str "Bedrock requires IAM policies granting access to specific actions and model resources:"], CodeBlock ("", ["json"], []) "{
    \"Version\": \"2012-10-17\",
    \"Statement\": [
        {
            \"Effect\": \"Allow\",
            \"Action\": [
                \"bedrock:InvokeModel\",
                \"bedrock:Converse\",
                \"bedrock:ListFoundationModels\"
            ],
            \"Resource\": \"arn:aws:bedrock:us-east-1::foundation-model/*\"
        }
    ]
}
", Header 3 ("region-selection", ["unnumbered", "unlisted"], []) [Str "Region Selection"], Para [Str "Bedrock availability and model catalogs vary by AWS region. US East (N. Virginia, ", Code ("", [], []) "us-east-1", Str ") and US West (Oregon, ", Code ("", [], []) "us-west-2", Str ") offer the broadest model selection."], Header 3 ("model-access", ["unnumbered", "unlisted"], []) [Str "Model Access"], Para [Str "Foundation models must be explicitly enabled in the Bedrock console before they can be invoked via API. Navigate to ", Strong [Str "Model access"], Str " in the Bedrock console and request access to desired models."], Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Header 3 ("direct-api-integration", ["unnumbered", "unlisted"], []) [Str "Direct API Integration"], Para [Str "The simplest pattern: application code calls Bedrock APIs directly using boto3 or the OpenAI-compatible endpoint."], CodeBlock ("", ["python"], []) "import boto3

client = boto3.client('bedrock-runtime', region_name='us-east-1')
response = client.converse(
    modelId='amazon.nova-2-pro-v1',
    messages=[{'role': 'user', 'content': [{'text': 'Summarize this document.'}]}]
)
", Header 3 ("rag-with-knowledge-bases", ["unnumbered", "unlisted"], []) [Str "RAG with Knowledge Bases"], Para [Str "Combine document ingestion, vector search, and model inference through Bedrock Knowledge Bases:"], CodeBlock ("", ["python"], []) "import boto3

agent_client = boto3.client('bedrock-agent-runtime', region_name='us-east-1')
response = agent_client.retrieve_and_generate(
    input={'text': 'What is our refund policy?'},
    retrieveAndGenerateConfiguration={
        'type': 'KNOWLEDGE_BASE',
        'knowledgeBaseConfiguration': {
            'knowledgeBaseId': 'YOUR_KB_ID',
            'modelArn': 'arn:aws:bedrock:us-east-1::foundation-model/anthropic.claude-opus-4-6-v1'
        }
    }
)
", Header 3 ("openai-sdk-migration", ["unnumbered", "unlisted"], []) [Str "OpenAI SDK Migration"], Para [Str "Existing applications using the OpenAI SDK can target Bedrock by configuring the base URL and authentication:"], CodeBlock ("", ["python"], []) "from openai import OpenAI

client = OpenAI(
    base_url=\"https://bedrock-runtime.us-east-1.amazonaws.com/openai/v1\",
)
", Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Header 3 ("multi-turn-conversation", ["unnumbered", "unlisted"], []) [Str "Multi-Turn Conversation"], CodeBlock ("", ["python"], []) "import boto3

client = boto3.client('bedrock-runtime', region_name='us-east-1')

messages = []
messages.append({'role': 'user', 'content': [{'text': 'What is machine learning?'}]})

response = client.converse(
    modelId='anthropic.claude-opus-4-6-v1',
    messages=messages
)

assistant_message = response['output']['message']
messages.append(assistant_message)

messages.append({'role': 'user', 'content': [{'text': 'Give me a concrete example.'}]})

response = client.converse(
    modelId='anthropic.claude-opus-4-6-v1',
    messages=messages
)
print(response['output']['message']['content'][0]['text'])
", Header 3 ("streaming-response", ["unnumbered", "unlisted"], []) [Str "Streaming Response"], CodeBlock ("", ["python"], []) "import boto3

client = boto3.client('bedrock-runtime', region_name='us-east-1')
response = client.converse_stream(
    modelId='anthropic.claude-opus-4-6-v1',
    messages=[{'role': 'user', 'content': [{'text': 'Write a short essay on AI safety.'}]}]
)

for event in response['stream']:
    if 'contentBlockDelta' in event:
        print(event['contentBlockDelta']['delta']['text'], end='')
", Header 3 ("system-prompt-with-temperature", ["unnumbered", "unlisted"], []) [Str "System Prompt with Temperature"], CodeBlock ("", ["python"], []) "import boto3

client = boto3.client('bedrock-runtime', region_name='us-east-1')
response = client.converse(
    modelId='anthropic.claude-opus-4-6-v1',
    system=[{'text': 'You are a helpful coding assistant. Respond with code examples.'}],
    messages=[{'role': 'user', 'content': [{'text': 'Write a Python function to parse CSV files.'}]}],
    inferenceConfig={'temperature': 0.2, 'maxTokens': 2048}
)
print(response['output']['message']['content'][0]['text'])
", Header 2 ("limitations-and-considerations", ["unnumbered", "unlisted"], []) [Str "Limitations and Considerations"], BulletList [[Plain [Strong [Str "Regional Availability"], Str ": Not all models are available in every AWS region. Model catalogs vary by region."]], [Plain [Strong [Str "Model Access Approval"], Str ": Foundation models require explicit access requests before use; approval is not instant for all models."]], [Plain [Strong [Str "Vendor Lock-In"], Str ": While the OpenAI-compatible API reduces switching cost, Knowledge Bases, Agents, and Guardrails are AWS-specific services."]], [Plain [Strong [Str "Prompt Caching Scope"], Str ": 1-hour caching is currently limited to Claude models; other providers may have different or no caching behavior."]], [Plain [Strong [Str "Rate Limits"], Str ": Throughput quotas are per-account and per-model; high-volume applications may require provisioned throughput."]], [Plain [Strong [Str "Customization Constraints"], Str ": Fine-tuning is available only for a subset of models, not the full catalog."]], [Plain [Strong [Str "Closed Source"], Str ": The service itself is proprietary; no self-hosting option exists."]]], Header 2 ("changelog-highlights", ["unnumbered", "unlisted"], []) [Str "Changelog Highlights"], BulletList [[Plain [Strong [Str "2025 Q4"], Str ": Added six open-weight models (DeepSeek V3.2, MiniMax M2.1, GLM 4.7, GLM 4.7 Flash, Kimi K2.5, Qwen3 Coder Next)."]], [Plain [Strong [Str "2025 Q3"], Str ": NVIDIA Nemotron 3 Nano added with 256k context and native tool calling."]], [Plain [Strong [Str "2025 Q2"], Str ": OpenAI-compatible Chat Completions and Responses APIs launched."]], [Plain [Strong [Str "2025 Q1"], Str ": Amazon Nova 2 Pro model family released."]], [Plain [Strong [Str "2024"], Str ": Guardrails general availability, Knowledge Bases general availability, Agents general availability, prompt caching for Claude models."]]], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " AWS Bedrock Documentation - https://docs.aws.amazon.com/bedrock/"]], [Plain [Str "[", Str "2", Str "]", Str " AWS Bedrock User Guide - https://docs.aws.amazon.com/bedrock/latest/userguide/what-is-bedrock.html"]]], Header 2 ("discovery-signals", ["unnumbered", "unlisted"], []) [Str "Discovery Signals"], BlockQuote [Para [Str "Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search."]], Header 3 ("keywords", ["unnumbered", "unlisted"], []) [Str "Keywords"], BulletList [[Plain [Str "aws bedrock"]], [Plain [Str "amazon bedrock"]], [Plain [Str "bedrock-runtime"]], [Plain [Str "foundation models"]], [Plain [Str "model catalog"]], [Plain [Str "converse api"]], [Plain [Str "invoke_model"]], [Plain [Str "chat completions api"]], [Plain [Str "responses api"]], [Plain [Str "knowledge bases"]], [Plain [Str "bedrock agents"]], [Plain [Str "guardrails"]], [Plain [Str "prompt caching"]], [Plain [Str "claude on bedrock"]], [Plain [Str "amazon nova"]], [Plain [Str "deepseek on bedrock"]], [Plain [Str "nemotron"]], [Plain [Str "kimi on bedrock"]], [Plain [Str "glm on bedrock"]], [Plain [Str "qwen on bedrock"]], [Plain [Str "minimax on bedrock"]], [Plain [Str "bedrock fine-tuning"]], [Plain [Str "provisioned throughput"]], [Plain [Str "iam bedrock policy"]], [Plain [Str "region us-east-1"]], [Plain [Str "bedrock-agent-runtime"]], [Plain [Str "retrieve_and_generate"]], [Plain [Str "open-weight models on aws"]]], Header 3 ("verb-noun-tasks", ["unnumbered", "unlisted"], []) [Str "Verb-Noun Tasks"], BulletList [[Plain [Str "Call ", Code ("", [], []) "converse()", Str " to chat across multiple model providers"]], [Plain [Str "Use ", Code ("", [], []) "invoke_model", Str " for provider-native request bodies"]], [Plain [Str "Migrate an OpenAI SDK app to Bedrock by swapping the base URL"]], [Plain [Str "Stream responses with ", Code ("", [], []) "converse_stream"]], [Plain [Str "Build a RAG pipeline using Bedrock Knowledge Bases"]], [Plain [Str "Wire up multi-step Bedrock Agents with tool calling"]], [Plain [Str "Apply Guardrails to filter inputs and outputs"]], [Plain [Str "Enable prompt caching for repeated Claude prompt prefixes"]], [Plain [Str "Request model access in the Bedrock console"]], [Plain [Str "Fine-tune a foundation model on proprietary data"]], [Plain [Str "Configure IAM policies for ", Code ("", [], []) "bedrock:InvokeModel", Str " and ", Code ("", [], []) "bedrock:Converse"]], [Plain [Str "Run ", Code ("", [], []) "retrieve_and_generate", Str " for grounded RAG answers"]]], Header 3 ("user-intent-phrases", ["unnumbered", "unlisted"], []) [Str "User Intent Phrases"], BulletList [[Plain [Str "How do I call Claude or DeepSeek through AWS?"]], [Plain [Str "I want one API for 100+ foundation models without managing GPUs."]], [Plain [Str "How do I build RAG on AWS with managed vector storage?"]], [Plain [Str "How do I configure content filters on LLM inputs and outputs?"]], [Plain [Str "How do I fine-tune a foundation model on private data within my AWS account?"]], [Plain [Str "How do I get prompt caching with Claude on AWS?"]], [Plain [Str "How do I migrate from the OpenAI SDK to Bedrock with minimal code changes?"]], [Plain [Str "How do I build a multi-step agent on AWS that calls my internal APIs?"]], [Plain [Str "How do I grant IAM access to specific Bedrock models?"]], [Plain [Str "Where do I request model access for Anthropic on Bedrock?"]]], Header 3 ("problem-statements", ["unnumbered", "unlisted"], []) [Str "Problem Statements"], BulletList [[Plain [Str "Our data, identity, and billing are on AWS and we need LLM access without leaving the account boundary."]], [Plain [Str "We need regulated workloads where customer data never leaves our AWS region."]], [Plain [Str "We need managed RAG with no operational vector-database work."]], [Plain [Str "We want a unified API surface across providers without managing keys per vendor."]], [Plain [Str "We need content moderation as a built-in product, not a third-party integration."]], [Plain [Str "We need provisioned throughput for predictable inference latency at scale."]]], Header 3 ("when-to-pick-this", ["unnumbered", "unlisted"], []) [Str "When to Pick This"], BulletList [[Plain [Str "Pick this when your data, identity, and billing are already on AWS (vs Vertex AI when you are on GCP)."]], [Plain [Str "Pick this when you need managed Knowledge Bases and Agents as first-class AWS services (vs assembling RAG yourself on Modal/RunPod)."]], [Plain [Str "Pick this when Guardrails, PII redaction, and topic restriction must run inside the same managed surface."]], [Plain [Str "Pick this when access to Anthropic Claude alongside Amazon Nova, DeepSeek, Kimi, GLM, MiniMax, Qwen, and Nemotron through one API matters."]], [Plain [Str "Pick this when an OpenAI-compatible endpoint reduces migration cost from existing OpenAI apps."]], [Plain [Str "Pick this when provisioned throughput and per-account model isolation are required for compliance."]], [Plain [Str "Pick this when fine-tuning on a subset of models inside the same AWS account is part of the workflow."]]], Header 3 ("related-terms-and-aliases", ["unnumbered", "unlisted"], []) [Str "Related Terms and Aliases"], BulletList [[Plain [Str "aws gen ai"]], [Plain [Str "managed foundation models"]], [Plain [Str "aws llm api"]], [Plain [Str "bedrock knowledge base"]], [Plain [Str "bedrock guardrails"]], [Plain [Str "bedrock agents"]], [Plain [Str "aws ai platform"]], [Plain [Str "enterprise llm gateway aws"]], [Plain [Str "amazon nova"]], [Plain [Str "aws prompt caching"]]]]