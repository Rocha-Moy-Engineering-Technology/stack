[Header 1 ("outlines", [], []) [Str "Outlines"], BlockQuote [Para [Str "Python library for structured Large Language Model (LLM) generation via JSON Schema, regex, and context-free grammars. Guarantees structured outputs during the generation process itself rather than through post-hoc parsing."]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.5)), (AlignDefault, (ColWidth 0.5))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Name"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Outlines"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Structured Generation"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "SDK"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Yes"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "dottxt-ai/outlines"] ("https://github.com/dottxt-ai/outlines", "")]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "13838"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Docs"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "dottxt-ai.github.io/outlines"] ("https://dottxt-ai.github.io/outlines/", "")]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "License"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Apache 2.0"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Language"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Python"]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "Outlines is a Python library developed by dottxt-ai that enables structured text generation from LLMs. Unlike approaches that generate free-form text and then attempt to parse it into a desired format, Outlines constrains the generation process at the token level using Finite-State Machines (FSMs) and specialized backends. This means every token produced by the model is guaranteed to conform to the specified structure, eliminating parsing failures, broken JSON, and malformed outputs entirely."], Para [Str "The library supports a range of structured output formats including JSON Schema, regular expressions, Context-Free Grammars (CFGs), native Python types, Pydantic models, and multiple-choice selection. It integrates with major LLM providers and inference engines, making it a versatile tool for any workflow that requires reliable, machine-readable output from language models."], Para [Str "Outlines is used in production by organizations including Amazon, Apple, Databricks, and Meta."], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], BulletList [[Plain [Strong [Str "Generation-Time Constraints"], Str ": Outlines applies structural constraints during the token generation process rather than after it. Each token is validated against the target schema before being emitted, ensuring 100% conformance without retry loops or post-processing."]], [Plain [Strong [Str "Finite-State Machine (FSM) Guided Decoding"], Str ": The library compiles output schemas (JSON Schema, regex patterns, grammars) into FSMs that mask invalid tokens at each generation step. Only tokens that maintain a valid path through the FSM are considered by the model's sampling procedure."]], [Plain [Strong [Str "Schema Compilation"], Str ": Schemas are compiled into their FSM representations once per session. Subsequent generation calls reuse the compiled representation, amortizing the compilation cost across multiple invocations."]], [Plain [Strong [Str "Backend Agnosticism"], Str ": Outlines decouples the structured generation logic from the model backend. The same schema definition works across OpenAI, Anthropic, vLLM, Hugging Face Transformers, Ollama, and Gemini without modification."]], [Plain [Strong [Str "Type-Safe Output"], Str ": When using Pydantic models or Python type annotations, the generated output is automatically deserialized into the corresponding typed object, providing immediate programmatic access without manual parsing."]]], Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Para [Str "Outlines is organized around three primary layers:"], BulletList [[Plain [Strong [Str "Schema Layer"], Str ": Accepts user-defined output specifications in the form of JSON Schema objects, regex patterns, CFGs, Pydantic models, Python type annotations, or enumerated choices. This layer validates and normalizes the schema definition."]], [Plain [Strong [Str "Compilation Layer"], Str ": Transforms the normalized schema into an FSM or equivalent constraint representation. Compilation happens once per unique schema within a session. The compiled artifact encodes all valid token sequences that satisfy the schema."]], [Plain [Strong [Str "Generation Layer"], Str ": Interfaces with the LLM backend to perform constrained decoding. At each generation step, the FSM state determines which tokens are valid continuations. Invalid tokens are masked (assigned zero probability) before sampling, ensuring the output always conforms to the schema."]]], Para [Str "The separation of these layers allows Outlines to support multiple backends through a common interface while keeping the constraint logic centralized and reusable."], Header 2 ("key-features-and-functionality", ["unnumbered", "unlisted"], []) [Str "Key Features and Functionality"], BulletList [[Plain [Strong [Str "JSON Schema Generation"], Str ": Define output structure using JSON Schema and receive guaranteed-valid JSON from any supported model. Supports nested objects, arrays, enums, optional fields, and all standard JSON Schema constructs."]], [Plain [Strong [Str "Regex-Constrained Generation"], Str ": Specify output format using regular expressions. Useful for dates, phone numbers, identifiers, and other pattern-based formats."]], [Plain [Strong [Str "Context-Free Grammar (CFG) Support"], Str ": Define output structure using formal grammars for complex, recursive structures that go beyond what regex can express."]], [Plain [Strong [Str "Pydantic Model Integration"], Str ": Pass a Pydantic model class directly and receive a fully instantiated, validated model object as output."]], [Plain [Strong [Str "Python Type Support"], Str ": Use native Python types (str, int, float, bool, lists, dicts) as output specifications for simple structured outputs."]], [Plain [Strong [Str "Multiple-Choice Selection"], Str ": Constrain the model to select from a predefined set of options, useful for classification and decision-making tasks."]], [Plain [Strong [Str "One-Time Compilation"], Str ": Schemas are compiled into FSMs once per session, making repeated generation calls with the same schema efficient."]], [Plain [Strong [Str "Multi-Provider Support"], Str ": Works with OpenAI, Anthropic, vLLM, Hugging Face Transformers, Ollama, and Gemini through a unified interface."]]], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], BulletList [[Plain [Strong [Str "Classification"], Str ": Constrain model output to a fixed set of labels for text classification, sentiment analysis, or intent detection tasks."]], [Plain [Strong [Str "Named Entity Recognition (NER)"], Str ": Extract structured entity data from unstructured text with guaranteed output format conformance."]], [Plain [Strong [Str "Knowledge Graph Construction"], Str ": Generate structured triples (subject, predicate, object) from text for knowledge graph population."]], [Plain [Strong [Str "Question Answering with Citations"], Str ": Produce answers that include structured citation references pointing back to source material."]], [Plain [Strong [Str "PDF and Document Processing"], Str ": Extract structured data from unstructured document content with reliable output formatting."]], [Plain [Strong [Str "ReAct Agents"], Str ": Generate structured action-observation-thought sequences for agent-based reasoning frameworks."]], [Plain [Strong [Str "Data Extraction Pipelines"], Str ": Convert unstructured text into structured records for database ingestion or downstream processing."]], [Plain [Strong [Str "Form Generation"], Str ": Produce structured form data from natural language descriptions."]]], Header 2 ("api-reference-summary", ["unnumbered", "unlisted"], []) [Str "API Reference Summary"], Header 3 ("model-initialization", ["unnumbered", "unlisted"], []) [Str "Model Initialization"], CodeBlock ("", ["python"], []) "import outlines

# OpenAI backend
model = outlines.models.openai(\"gpt-4o\")

# Transformers backend
model = outlines.models.transformers(\"mistralai/Mistral-7B-v0.1\")

# vLLM backend
model = outlines.models.vllm(\"mistralai/Mistral-7B-v0.1\")

# Ollama backend
model = outlines.models.ollama(\"llama3\")
", Header 3 ("json-schema-generation", ["unnumbered", "unlisted"], []) [Str "JSON Schema Generation"], CodeBlock ("", ["python"], []) "from pydantic import BaseModel
import outlines

class Customer(BaseModel):
    name: str
    age: int
    email: str

model = outlines.models.openai(\"gpt-4o\")
generator = outlines.generate.json(model, Customer)

result = generator(\"Alice needs help with her account.\")
# result is a Customer instance with guaranteed valid fields
", Header 3 ("regex-constrained-generation", ["unnumbered", "unlisted"], []) [Str "Regex-Constrained Generation"], CodeBlock ("", ["python"], []) "import outlines

model = outlines.models.openai(\"gpt-4o\")
date_pattern = r\"\\d{4}-\\d{2}-\\d{2}\"
generator = outlines.generate.regex(model, date_pattern)

result = generator(\"What is today's date?\")
# result matches the YYYY-MM-DD pattern
", Header 3 ("multiple-choice-selection", ["unnumbered", "unlisted"], []) [Str "Multiple-Choice Selection"], CodeBlock ("", ["python"], []) "import outlines

model = outlines.models.openai(\"gpt-4o\")
generator = outlines.generate.choice(model, [\"positive\", \"negative\", \"neutral\"])

result = generator(\"Classify the sentiment: 'I love this product!'\")
# result is one of \"positive\", \"negative\", or \"neutral\"
", Header 3 ("grammar-based-generation", ["unnumbered", "unlisted"], []) [Str "Grammar-Based Generation"], CodeBlock ("", ["python"], []) "import outlines

model = outlines.models.transformers(\"mistralai/Mistral-7B-v0.1\")
grammar = r\"\"\"
    start: expression
    expression: term ((\"+\"|\"-\") term)*
    term: NUMBER
    NUMBER: /[0-9]+/
\"\"\"
generator = outlines.generate.cfg(model, grammar)

result = generator(\"Generate a simple arithmetic expression.\")
", Header 3 ("text-generation-unconstrained", ["unnumbered", "unlisted"], []) [Str "Text Generation (Unconstrained)"], CodeBlock ("", ["python"], []) "import outlines

model = outlines.models.openai(\"gpt-4o\")
generator = outlines.generate.text(model)

result = generator(\"Tell me a story.\")
", Header 2 ("configuration-and-customization", ["unnumbered", "unlisted"], []) [Str "Configuration and Customization"], BulletList [[Plain [Strong [Str "Schema Compilation Caching"], Str ": Compiled FSMs are cached for the duration of the session. No explicit configuration is required; reusing the same generator object across calls leverages the cached compilation."]], [Plain [Strong [Str "Sampling Parameters"], Str ": Generation calls accept standard sampling parameters (temperature, top_p, max_tokens) through the underlying model backend configuration."]], [Plain [Strong [Str "Backend Selection"], Str ": The backend is determined by the model initialization call. Each backend may support additional configuration options specific to the provider (API keys, base URLs, device placement)."]]], Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Header 3 ("with-pydantic-for-validated-outputs", ["unnumbered", "unlisted"], []) [Str "With Pydantic for Validated Outputs"], CodeBlock ("", ["python"], []) "from pydantic import BaseModel, Field
import outlines

class Invoice(BaseModel):
    vendor: str
    amount: float = Field(ge=0)
    currency: str
    date: str

model = outlines.models.openai(\"gpt-4o\")
generator = outlines.generate.json(model, Invoice)

invoice = generator(\"Extract invoice data: Acme Corp charged $150.00 USD on 2025-03-15\")
# invoice.vendor == \"Acme Corp\", invoice.amount == 150.0, etc.
", Header 3 ("with-vllm-for-high-throughput-inference", ["unnumbered", "unlisted"], []) [Str "With vLLM for High-Throughput Inference"], CodeBlock ("", ["python"], []) "import outlines
from pydantic import BaseModel

class Entity(BaseModel):
    name: str
    entity_type: str
    confidence: float

model = outlines.models.vllm(\"mistralai/Mistral-7B-v0.1\")
generator = outlines.generate.json(model, Entity)

results = [generator(text) for text in batch_of_texts]
", Header 3 ("with-react-agent-patterns", ["unnumbered", "unlisted"], []) [Str "With ReAct Agent Patterns"], CodeBlock ("", ["python"], []) "from pydantic import BaseModel
from typing import Literal
import outlines

class AgentStep(BaseModel):
    thought: str
    action: Literal[\"search\", \"calculate\", \"respond\"]
    action_input: str

model = outlines.models.openai(\"gpt-4o\")
generator = outlines.generate.json(model, AgentStep)

step = generator(\"The user asked about the weather in Paris. Think step by step.\")
# step.action is guaranteed to be one of the valid actions
", Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Header 3 ("named-entity-recognition", ["unnumbered", "unlisted"], []) [Str "Named Entity Recognition"], CodeBlock ("", ["python"], []) "from pydantic import BaseModel
import outlines

class ExtractedEntities(BaseModel):
    persons: list[str]
    organizations: list[str]
    locations: list[str]

model = outlines.models.openai(\"gpt-4o\")
generator = outlines.generate.json(model, ExtractedEntities)

text = \"Tim Cook announced that Apple will open a new office in Austin, Texas.\"
entities = generator(f\"Extract named entities from: {text}\")
# entities.persons == [\"Tim Cook\"]
# entities.organizations == [\"Apple\"]
# entities.locations == [\"Austin\", \"Texas\"]
", Header 3 ("text-classification-with-confidence", ["unnumbered", "unlisted"], []) [Str "Text Classification with Confidence"], CodeBlock ("", ["python"], []) "from pydantic import BaseModel
from typing import Literal
import outlines

class Classification(BaseModel):
    label: Literal[\"spam\", \"not_spam\"]
    confidence: float
    reasoning: str

model = outlines.models.openai(\"gpt-4o\")
generator = outlines.generate.json(model, Classification)

result = generator(\"Classify this email: 'Congratulations! You won a free iPhone!'\")
# result.label is guaranteed to be \"spam\" or \"not_spam\"
", Header 3 ("structured-qa-with-citations", ["unnumbered", "unlisted"], []) [Str "Structured Q&A with Citations"], CodeBlock ("", ["python"], []) "from pydantic import BaseModel
import outlines

class Citation(BaseModel):
    text: str
    source: str
    page: int

class Answer(BaseModel):
    answer: str
    citations: list[Citation]

model = outlines.models.openai(\"gpt-4o\")
generator = outlines.generate.json(model, Answer)

context = \"According to Smith (2024, p.12), transformers revolutionized NLP...\"
result = generator(f\"Answer with citations based on: {context}\\nQuestion: What revolutionized NLP?\")
# result.citations contains structured Citation objects
", Header 2 ("limitations-and-considerations", ["unnumbered", "unlisted"], []) [Str "Limitations and Considerations"], BulletList [[Plain [Strong [Str "Compilation Overhead"], Str ": The initial compilation of a schema into an FSM adds latency to the first generation call. Complex schemas with deeply nested structures or large enumerations increase this overhead."]], [Plain [Strong [Str "Grammar Support Variability"], Str ": CFG support may vary across backends. Not all providers support grammar-based constrained generation natively."]], [Plain [Strong [Str "Structural vs. Semantic Guarantees"], Str ": The constrained decoding operates at the token level, which means the model may produce semantically incorrect but structurally valid output. The structure is guaranteed; the semantic quality depends on the underlying model."]], [Plain [Strong [Str "Backend Feature Parity"], Str ": Not all backends support every generation mode. Some constrained generation features may be available only with local model backends (Transformers, vLLM) and not with API-based providers."]], [Plain [Strong [Str "Local Model Requirements"], Str ": When using Transformers or vLLM backends, adequate GPU memory and compute resources are required to run the models locally."]]], Header 2 ("changelog-highlights", ["unnumbered", "unlisted"], []) [Str "Changelog Highlights"], Para [Str "Outlines is under active development. The project maintains releases on PyPI and GitHub. Refer to the ", Link ("", [], []) [Str "GitHub releases page"] ("https://github.com/dottxt-ai/outlines/releases", ""), Str " for version history and detailed changelogs."], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " ", Link ("", [], []) [Str "Outlines Documentation"] ("https://dottxt-ai.github.io/outlines/latest/", "")]]], Header 2 ("discovery-signals", ["unnumbered", "unlisted"], []) [Str "Discovery Signals"], BlockQuote [Para [Str "Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search."]], Header 3 ("keywords", ["unnumbered", "unlisted"], []) [Str "Keywords"], BulletList [[Plain [Str "Outlines"]], [Plain [Str "constrained decoding"]], [Plain [Str "token-level constraints"]], [Plain [Str "finite-state machine"]], [Plain [Str "FSM-guided decoding"]], [Plain [Str "JSON Schema generation"]], [Plain [Str "regex-constrained generation"]], [Plain [Str "context-free grammar"]], [Plain [Str "CFG support"]], [Plain [Str "multiple-choice selection"]], [Plain [Str "guaranteed valid output"]], [Plain [Str "schema compilation"]], [Plain [Str "Pydantic generator"]], [Plain [Str "structured generation"]], [Plain [Str "backend-agnostic"]], [Plain [Str "vLLM constrained"]], [Plain [Str "Transformers backend"]], [Plain [Str "Ollama constrained"]], [Plain [Str "one-time compilation"]], [Plain [Str "dottxt-ai"]]], Header 3 ("verb-noun-tasks", ["unnumbered", "unlisted"], []) [Str "Verb-Noun Tasks"], BulletList [[Plain [Str "Constrain LLM output to a JSON Schema at the token level"]], [Plain [Str "Generate text matching a regex pattern (dates, phone numbers, IDs)"]], [Plain [Str "Restrict output to a fixed set of choices for classification"]], [Plain [Str "Generate output conforming to a context-free grammar"]], [Plain [Str "Compile a Pydantic model into an FSM once and reuse across calls"]], [Plain [Str "Generate typed Python objects directly from a Pydantic class"]], [Plain [Str "Run constrained generation locally via Transformers or vLLM"]], [Plain [Str "Build a ReAct agent with guaranteed-valid action enums"]], [Plain [Str "Extract named entities into a typed list of strings"]], [Plain [Str "Produce structured citations alongside generated answers"]]], Header 3 ("user-intent-phrases", ["unnumbered", "unlisted"], []) [Str "User Intent Phrases"], BulletList [[Plain [Str "How do I guarantee an LLM produces valid JSON without retries?"]], [Plain [Str "How can I constrain output to a regex pattern?"]], [Plain [Str "I want generation that's always schema-conformant by construction"]], [Plain [Str "How do I make the model only choose from a fixed list of labels?"]], [Plain [Str "How do I generate output following a formal grammar?"]], [Plain [Str "How can I run constrained decoding on a local Mistral model?"]], [Plain [Str "How do I avoid parse failures on LLM-generated JSON entirely?"]], [Plain [Str "How do I get typed Pydantic instances back without parsing strings?"]], [Plain [Str "How does FSM-guided decoding work?"]], [Plain [Str "Can I use the same schema across OpenAI, vLLM, and Ollama?"]]], Header 3 ("problem-statements", ["unnumbered", "unlisted"], []) [Str "Problem Statements"], BulletList [[Plain [Str "Post-hoc parsing of LLM strings fails on malformed JSON"]], [Plain [Str "Retry-based approaches add latency and cost per validation failure"]], [Plain [Str "Free-form generation cannot guarantee schema conformance"]], [Plain [Str "Schema compilation overhead adds latency on the first call"]], [Plain [Str "Grammar support varies across backends (some API providers don't expose it)"]], [Plain [Str "Constrained generation guarantees structure, not semantic correctness"]], [Plain [Str "Local backends (Transformers, vLLM) require GPU resources"]]], Header 3 ("when-to-pick-this", ["unnumbered", "unlisted"], []) [Str "When to Pick This"], BulletList [[Plain [Str "Pick Outlines when retries are unacceptable and you need guaranteed valid output at the token level"]], [Plain [Str "Pick Outlines when you have local model access (Transformers, vLLM, Ollama) and want maximum throughput with constrained decoding"]], [Plain [Str "Pick Outlines for latency-critical systems where validation-and-retry is too expensive"]], [Plain [Str "Pick Outlines when output must match a regex or context-free grammar, not just a JSON schema"]], [Plain [Str "Pick Instructor instead when you need broad hosted-provider support and don't have token-level model access"]], [Plain [Str "Pick BAML instead when you want a typed DSL plus code generation across multiple languages"]], [Plain [Str "Pick DSPy instead when you want to optimize the prompt itself against a metric"]]], Header 3 ("related-terms-and-aliases", ["unnumbered", "unlisted"], []) [Str "Related Terms and Aliases"], BulletList [[Plain [Str "guided generation"]], [Plain [Str "structured generation library"]], [Plain [Str "FSM-based decoding"]], [Plain [Str "token-masking generation"]], [Plain [Str "grammar-constrained LLM"]], [Plain [Str "regex-constrained LLM"]], [Plain [Str "JSON-Schema-constrained LLM"]], [Plain [Str "dottxt outlines"]], [Plain [Str "constrained sampling"]]]]