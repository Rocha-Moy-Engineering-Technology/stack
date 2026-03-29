[Header 1 ("instructor", [], []) [Str "Instructor"], BlockQuote [Para [Str "Multi-language library for extracting structured data from LLMs"]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.5)), (AlignDefault, (ColWidth 0.5))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Name"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Instructor"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Structured Generation"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "SDK"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Yes"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "instructor-ai/instructor"] ("https://github.com/instructor-ai/instructor", "")]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "12,413"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Docs"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "python.useinstructor.com"] ("https://python.useinstructor.com/", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "Instructor is a Python library for extracting structured, validated data from Large Language Models (LLMs). With over 3 million monthly downloads, 12,500+ GitHub stars, and 100+ contributors, it is one of the most widely adopted tools for structured output extraction. Built on top of Pydantic, Instructor lets developers define response schemas as Python models and have the LLM fill them in directly. When the LLM output fails validation, Instructor automatically retries the request with the validation error context, enabling self-correcting extraction pipelines. The library supports 23+ LLM providers through a unified interface, offers streaming for partial results, and provides full type inference with IDE autocompletion. Instructor is available in Python, TypeScript, Go, Ruby, Elixir, and Rust, though the Python implementation is the most mature and widely used. The project is licensed under the MIT License and authored by Jason Liu."], Para [Str "Instructor positions itself as a focused tool for structured extraction rather than a full agent framework. As the documentation notes: \"Instructor for extraction, PydanticAI for agents.\" When a project requires quality gates, shareable runs, or built-in observability, the Pydantic team recommends PydanticAI as the complementary agent runtime that works alongside Instructor models."], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], Header 3 ("structured-outputs-via-pydantic-models", ["unnumbered", "unlisted"], []) [Str "Structured Outputs via Pydantic Models"], Para [Str "The fundamental idea behind Instructor is that a Pydantic model defines the expected shape of the LLM response. The ", Code ("", [], []) "response_model", Str " parameter serves three purposes: defining the schema and prompts for the language model, validating API responses, and returning Pydantic model instances. Docstrings, types, and field annotations are used to generate the prompt automatically. The library injects the model schema into the LLM request (via function calling, tool use, or JSON mode depending on the provider), parses the raw output, and validates it against the model. The developer receives a fully typed Python object rather than a string."], Header 3 ("automatic-retries-reasks", ["unnumbered", "unlisted"], []) [Str "Automatic Retries (Reasks)"], Para [Str "When the LLM returns output that fails Pydantic validation, Instructor does not simply raise an error. Instead, it feeds the validation error message back to the LLM as context and retries the request. This \"reask\" loop continues up to a configurable maximum number of retries (", Code ("", [], []) "max_retries", Str "), giving the model the opportunity to self-correct. The system defends against two error types: Pydantic validation failures and JSON decoding errors. This is especially useful for enforcing constraints that are difficult to express purely in a prompt (value ranges, string formats, cross-field dependencies). Instructor also integrates with the Tenacity library for more advanced retry strategies including exponential backoff, error-specific retries, and result-based retries."], Header 3 ("client-patching", ["unnumbered", "unlisted"], []) [Str "Client Patching"], Para [Str "Instructor works by wrapping (patching) existing LLM client libraries. Rather than replacing the client, it augments it with structured output capabilities. The patching process intercepts calls to completion methods like ", Code ("", [], []) "create()", Str ", transforms Pydantic models into provider-specific formats, checks outputs against the defined model, and handles retries when validation fails. Developers keep their existing authentication, configuration, and error handling while gaining schema-driven extraction on top. The patched client gains three new parameters: ", Code ("", [], []) "response_model", Str " (defines expected output structure), ", Code ("", [], []) "max_retries", Str " (retry attempts on validation failure), and ", Code ("", [], []) "context", Str " (additional validation context and Jinja template variables)."], Header 3 ("extraction-modes", ["unnumbered", "unlisted"], []) [Str "Extraction Modes"], Para [Str "Instructor supports multiple extraction modes depending on provider capabilities:"], BulletList [[Plain [Strong [Str "TOOLS"], Str " -- Uses the provider's function/tool calling API. The default and recommended mode for OpenAI, Anthropic, Google, and Ollama."]], [Plain [Strong [Str "JSON_SCHEMA"], Str " -- Native schema support when providers offer it. Strict schema enforcement."]], [Plain [Strong [Str "MD_JSON"], Str " -- Extracts JSON from markdown code blocks in the response. Useful for providers without tool calling support."]], [Plain [Strong [Str "PARALLEL_TOOLS"], Str " -- Multiple tool calls in a single response for batch extraction."]], [Plain [Strong [Str "RESPONSES_TOOLS"], Str " -- OpenAI Responses API tools integration."]]], Para [Str "The ", Code ("", [], []) "from_provider()", Str " function automatically selects the optimal mode for each provider, though modes can be overridden manually."], Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Para [Str "Instructor sits as a thin middleware layer between the application and the LLM provider client:"], OrderedList (1, DefaultStyle, DefaultDelim) [[Plain [Strong [Str "Application Layer"], Str " -- Defines Pydantic response models and sends messages through the Instructor-patched client."]], [Plain [Strong [Str "Instructor Layer"], Str " -- Injects the Pydantic model schema into the LLM request, parses the response, runs Pydantic validation, and handles retries on failure."]], [Plain [Strong [Str "Provider Client Layer"], Str " -- The underlying LLM SDK (OpenAI, Anthropic, Google, and others) handles authentication, transport, and raw API communication."]]], Para [Str "The main execution pipeline flows through several stages: caching and templating, retry mechanisms powered by Tenacity, provider communication, and response dispatching. The dispatcher routes responses through mode-specific handlers (streaming, partial, standard) before parsing into Pydantic models. If validation fails, a reask handler prepares error feedback for retry attempts."], Header 3 ("retry-flow", ["unnumbered", "unlisted"], []) [Str "Retry Flow"], CodeBlock ("", [""], []) "Application -> Instructor -> LLM Provider
                  |
                  v
          Parse response
                  |
           Validate with Pydantic
                  |
         [Pass] -> Return typed object
         [Fail] -> Append validation error to messages -> Retry LLM call
", Para [Str "When retry attempts are exhausted, Instructor raises ", Code ("", [], []) "InstructorRetryException", Str " containing the full attempt history, final completion data, and reproduction parameters."], Header 3 ("instrumentation-points", ["unnumbered", "unlisted"], []) [Str "Instrumentation Points"], Para [Str "The framework provides hooks at critical points in the pipeline:"], BulletList [[Plain [Code ("", [], []) "completion:kwargs", Str " -- Before provider invocation"]], [Plain [Code ("", [], []) "completion:response", Str " -- After receiving response"]], [Plain [Code ("", [], []) "completion:error", Str " -- Error before retries"]], [Plain [Code ("", [], []) "parse:error", Str " -- On validation failures"]], [Plain [Code ("", [], []) "completion:last_attempt", Str " -- Before retry exhaustion"]]], Header 2 ("key-features", ["unnumbered", "unlisted"], []) [Str "Key Features"], BulletList [[Plain [Strong [Str "Type Safety and IDE Autocompletion"], Str " -- Response models are standard Pydantic classes, giving full type inference, autocomplete, and static analysis support in editors and type checkers."]], [Plain [Strong [Str "Automatic Retries with Validation Context"], Str " -- Failed validations trigger retries where the error message is included in the next prompt, allowing the LLM to self-correct. Integrates with Tenacity for exponential backoff, error-specific retries, and result-based retries."]], [Plain [Strong [Str "Multi-Provider Support"], Str " -- A single ", Code ("", [], []) "from_provider()", Str " interface supports OpenAI, Anthropic, Google, Ollama, DeepSeek, and 23+ other providers without changing application code."]], [Plain [Strong [Str "Streaming"], Str " -- ", Code ("", [], []) "create_partial()", Str " streams partial results as the LLM generates tokens, enabling progressive UI updates. ", Code ("", [], []) "create_iterable()", Str " streams a sequence of complete objects. Both support async iteration."]], [Plain [Strong [Str "Custom Pydantic Validators"], Str " -- Standard Pydantic field validators and model validators work seamlessly, enabling complex validation logic (regex patterns, cross-field checks, business rules)."]], [Plain [Strong [Str "LLM-Based Validation"], Str " -- The ", Code ("", [], []) "llm_validator", Str " function uses the LLM itself to validate outputs against semantic criteria, generating human-readable error messages for reasking."]], [Plain [Strong [Str "Async/Await"], Str " -- Full async support for non-blocking LLM calls in asynchronous applications via ", Code ("", [], []) "async_client=True", Str "."]], [Plain [Strong [Str "Jinja Templating"], Str " -- Prompt templates can use Jinja syntax for dynamic prompt construction with variables, loops, and conditionals. Templates are rendered in a sandboxed environment for security."]], [Plain [Strong [Str "Hooks System"], Str " -- Lifecycle hooks allow injecting custom logic at various stages of the request/response cycle for logging, metrics, monitoring, or transformation."]], [Plain [Strong [Str "Multimodal Extraction"], Str " -- Unified, provider-agnostic interface for extracting structured data from images, PDFs, and audio files with automatic format handling."]], [Plain [Strong [Str "CLI Tools"], Str " -- Built-in command-line utilities for API usage monitoring (", Code ("", [], []) "instructor usage", Str "), fine-tuning management (", Code ("", [], []) "instructor finetune", Str "), and documentation access (", Code ("", [], []) "instructor docs", Str ")."]], [Plain [Strong [Str "Dynamic Model Creation"], Str " -- Pydantic's ", Code ("", [], []) "create_model()", Str " enables runtime model generation when schemas are determined by database queries, user configurations, or other dynamic sources."]]], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], BulletList [[Plain [Strong [Str "Data Extraction"], Str " -- Pulling structured records (names, dates, amounts, entities) from unstructured text such as emails, documents, or web pages."]], [Plain [Strong [Str "Classification"], Str " -- Categorizing text into predefined enums or labels with guaranteed valid output values."]], [Plain [Strong [Str "Content Generation with Constraints"], Str " -- Generating content that must conform to a specific schema (product descriptions with required fields, quiz questions with exactly four options)."]], [Plain [Strong [Str "Multi-Step Pipelines"], Str " -- Chaining structured outputs where the validated result of one step feeds into the next, with type safety preserved throughout."]], [Plain [Strong [Str "Search and Retrieval Augmented Generation (RAG)"], Str " -- Extracting structured queries or filters from natural language to drive database lookups or search APIs."]], [Plain [Strong [Str "Streaming User Interfaces"], Str " -- Progressively rendering structured data in a UI as the LLM generates it, using partial streaming."]], [Plain [Strong [Str "Document Processing"], Str " -- Extracting structured data from PDFs, images, and audio files using multimodal capabilities."]], [Plain [Strong [Str "Content Moderation"], Str " -- Using ", Code ("", [], []) "llm_validator", Str " to check outputs against semantic criteria and reject objectionable content."]]], Header 2 ("api-reference", ["unnumbered", "unlisted"], []) [Str "API Reference"], Header 3 ("client-creation", ["unnumbered", "unlisted"], []) [Str "Client Creation"], CodeBlock ("", ["python"], []) "import instructor

# Universal provider interface (recommended)
client = instructor.from_provider(\"openai/gpt-4o\")

# Async client
async_client = instructor.from_provider(\"openai/gpt-4o\", async_client=True)

# Provider-specific patching
import openai
client = instructor.from_openai(openai.OpenAI())

import anthropic
client = instructor.from_anthropic(anthropic.Anthropic())

# With mode override
client = instructor.from_provider(\"openai/gpt-4o\", mode=instructor.Mode.JSON)

# With caching
client = instructor.from_provider(\"openai/gpt-4o\", cache=True)
", Para [Str "The ", Code ("", [], []) "from_provider()", Str " function accepts a model string in the format ", Code ("", [], []) "\"provider/model\"", Str " and automatically handles provider-specific configurations. Additional provider-specific constructors include ", Code ("", [], []) "from_openai()", Str ", ", Code ("", [], []) "from_anthropic()", Str ", ", Code ("", [], []) "from_google()", Str ", ", Code ("", [], []) "from_litellm()", Str ", ", Code ("", [], []) "from_ollama()", Str ", and others."], Header 3 ("core-methods", ["unnumbered", "unlisted"], []) [Str "Core Methods"], BulletList [[Plain [Strong [Code ("", [], []) "client.create(response_model, messages, max_retries=3, validation_context=None, context=None, strict=None, hooks=None)"], Str " -- Sends a completion request and returns a validated instance of ", Code ("", [], []) "response_model", Str ". Retries automatically on validation failure up to ", Code ("", [], []) "max_retries", Str "."]], [Plain [Strong [Code ("", [], []) "client.create_with_completion(response_model, messages)"], Str " -- Returns a tuple of ", Code ("", [], []) "(response_model_instance, raw_completion)", Str ", giving access to both the validated object and the raw provider response (token usage, finish reason, metadata)."]], [Plain [Strong [Code ("", [], []) "client.create_partial(response_model, messages)"], Str " -- Returns an iterator that yields progressively more complete instances of ", Code ("", [], []) "response_model", Str " as tokens stream in. All fields become ", Code ("", [], []) "Optional", Str " during streaming. Validators are not applied until the final iteration."]], [Plain [Strong [Code ("", [], []) "client.create_iterable(response_model, messages)"], Str " -- Returns an iterator of complete ", Code ("", [], []) "response_model", Str " instances, useful when the LLM produces a list of structured objects."]]], Header 3 ("hooks-api", ["unnumbered", "unlisted"], []) [Str "Hooks API"], CodeBlock ("", ["python"], []) "# Register hooks
client.on(\"completion:kwargs\", handler_function)
client.on(\"completion:response\", lambda response: print(response))
client.on(\"completion:error\", lambda error: log_error(error))
client.on(\"parse:error\", lambda error: track_validation_failure(error))
client.on(\"completion:last_attempt\", lambda: alert_exhaustion())

# Remove hooks
client.off(\"completion:kwargs\", handler_function)
client.clear(\"completion:kwargs\")  # Clear all handlers for event
client.clear()                      # Clear all hooks

# Per-call hooks
result = client.create(
    response_model=User,
    messages=[...],
    hooks={\"completion:kwargs\": lambda **kw: print(kw)},
)
", Header 3 ("response-model-definition", ["unnumbered", "unlisted"], []) [Str "Response Model Definition"], CodeBlock ("", ["python"], []) "from pydantic import BaseModel, Field, field_validator
from typing import Optional
from enum import Enum

class Priority(str, Enum):
    LOW = \"low\"
    MEDIUM = \"medium\"
    HIGH = \"high\"
    CRITICAL = \"critical\"

class Person(BaseModel):
    \"\"\"Extract person information from text.\"\"\"
    name: str = Field(description=\"Full legal name\")
    age: int = Field(ge=0, le=150, description=\"Age in years\")
    occupation: Optional[str] = Field(None, description=\"Current job title\")

    @field_validator(\"name\")
    @classmethod
    def name_must_not_be_empty(cls, v: str) -> str:
        if not v.strip():
            raise ValueError(\"Name must not be empty\")
        return v.strip()
", Header 3 ("multimodal-input", ["unnumbered", "unlisted"], []) [Str "Multimodal Input"], CodeBlock ("", ["python"], []) "from instructor.multimodal import Image, Audio, PDF

# Image extraction
result = client.create(
    response_model=ImageDescription,
    messages=[{
        \"role\": \"user\",
        \"content\": [\"Describe this image.\", Image.from_url(\"https://example.com/photo.jpg\")],
    }],
)

# PDF extraction
result = client.create(
    response_model=InvoiceData,
    messages=[{
        \"role\": \"user\",
        \"content\": [\"Extract invoice data.\", PDF.from_path(\"invoice.pdf\")],
    }],
)
", Para [Str "Image, Audio, and PDF classes support ", Code ("", [], []) "from_url()", Str ", ", Code ("", [], []) "from_path()", Str ", ", Code ("", [], []) "from_base64()", Str ", ", Code ("", [], []) "from_gs_url()", Str " (Google Cloud Storage), and ", Code ("", [], []) "autodetect()", Str " methods."], Header 2 ("configuration", ["unnumbered", "unlisted"], []) [Str "Configuration"], Header 3 ("retry-configuration", ["unnumbered", "unlisted"], []) [Str "Retry Configuration"], CodeBlock ("", ["python"], []) "# Built-in retry configuration
person = client.create(
    response_model=Person,
    messages=[{\"role\": \"user\", \"content\": \"Extract: John is 28 years old.\"}],
    max_retries=3,
)

# Tenacity integration for advanced retry strategies
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(stop=stop_after_attempt(3), wait=wait_exponential(multiplier=1, min=4, max=10))
def extract_with_backoff(text: str) -> Person:
    return client.create(
        response_model=Person,
        messages=[{\"role\": \"user\", \"content\": text}],
    )
", Para [Str "Recommended retry settings by error type: rate limits (5 attempts, 1-120s delay), validation errors (2-3 attempts, 1-10s delay), network errors (4 attempts, 2-30s delay)."], Header 3 ("provider-selection", ["unnumbered", "unlisted"], []) [Str "Provider Selection"], Para [Str "The ", Code ("", [], []) "from_provider()", Str " method accepts a string in the format ", Code ("", [], []) "\"provider/model\"", Str ":"], CodeBlock ("", ["python"], []) "client = instructor.from_provider(\"openai/gpt-4o\")
client = instructor.from_provider(\"anthropic/claude-sonnet-4-20250514\")
client = instructor.from_provider(\"google/gemini-2.0-flash\")
client = instructor.from_provider(\"ollama/llama3\")
client = instructor.from_provider(\"deepseek/deepseek-chat\")
", Header 3 ("mode-configuration", ["unnumbered", "unlisted"], []) [Str "Mode Configuration"], Para [Str "Override the default extraction mode when needed:"], CodeBlock ("", ["python"], []) "import instructor

# Force JSON mode
client = instructor.from_provider(\"openai/gpt-4o\", mode=instructor.Mode.JSON)

# Use strict JSON schema
client = instructor.from_provider(\"openai/gpt-4o\", mode=instructor.Mode.JSON_SCHEMA)

# Markdown JSON for broader compatibility
client = instructor.from_provider(\"databricks/model\", mode=instructor.Mode.MD_JSON)

# Parallel tool calls
client = instructor.from_provider(\"openai/gpt-4o\", mode=instructor.Mode.PARALLEL_TOOLS)
", Header 3 ("jinja-templating", ["unnumbered", "unlisted"], []) [Str "Jinja Templating"], CodeBlock ("", ["python"], []) "response = client.create(
    messages=[{
        \"role\": \"user\",
        \"content\": \"Extract the information from the following text: {{ data }}\",
    }],
    response_model=User,
    context={\"data\": \"John Doe is thirty years old\"},
)
", Para [Str "Context variables are also accessible within Pydantic field validators through ", Code ("", [], []) "ValidationInfo", Str ", enabling dynamic validation rules based on input context."], Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Header 3 ("basic-extraction", ["unnumbered", "unlisted"], []) [Str "Basic Extraction"], CodeBlock ("", ["python"], []) "import instructor
from pydantic import BaseModel

client = instructor.from_provider(\"openai/gpt-4o\")

class Person(BaseModel):
    name: str
    age: int

person = client.create(
    response_model=Person,
    messages=[{\"role\": \"user\", \"content\": \"Extract: Jason is 25 years old.\"}],
)
# person.name == \"Jason\", person.age == 25
", Header 3 ("streaming-partial-results", ["unnumbered", "unlisted"], []) [Str "Streaming Partial Results"], CodeBlock ("", ["python"], []) "for partial_person in client.create_partial(
    response_model=Person,
    messages=[{\"role\": \"user\", \"content\": \"Extract: Jason is 25 years old.\"}],
):
    print(partial_person)
    # Yields progressively: Person(name=None, age=None) -> Person(name=\"Ja\", age=None) -> ...
", Header 3 ("iterable-extraction", ["unnumbered", "unlisted"], []) [Str "Iterable Extraction"], CodeBlock ("", ["python"], []) "users = client.create_iterable(
    response_model=Person,
    messages=[
        {\"role\": \"user\", \"content\": \"Extract all people: Jason is 25. Sarah is 30.\"}
    ],
)
for user in users:
    print(user)
    # Person(name=\"Jason\", age=25)
    # Person(name=\"Sarah\", age=30)
", Header 3 ("async-usage", ["unnumbered", "unlisted"], []) [Str "Async Usage"], CodeBlock ("", ["python"], []) "import asyncio
import instructor

async_client = instructor.from_provider(\"openai/gpt-4o\", async_client=True)

async def extract():
    person = await async_client.create(
        response_model=Person,
        messages=[{\"role\": \"user\", \"content\": \"Extract: Jason is 25 years old.\"}],
    )
    return person

result = asyncio.run(extract())
", Header 3 ("llm-based-validation", ["unnumbered", "unlisted"], []) [Str "LLM-Based Validation"], CodeBlock ("", ["python"], []) "from pydantic import BaseModel, BeforeValidator
from typing_extensions import Annotated
from instructor import llm_validator

client = instructor.from_provider(\"openai/gpt-4o-mini\")

class QuestionAnswer(BaseModel):
    question: str
    answer: Annotated[
        str,
        BeforeValidator(llm_validator(\"don't say objectionable things\", client=client)),
    ]
", Para [Str "When the answer contains objectionable content, the LLM-based validator generates a human-readable error message that is fed back for reasking."], Header 3 ("context-based-validation", ["unnumbered", "unlisted"], []) [Str "Context-Based Validation"], CodeBlock ("", ["python"], []) "from pydantic import BaseModel, field_validator, ValidationInfo

class CityExtraction(BaseModel):
    city: str

    @field_validator(\"city\")
    @classmethod
    def validate_city(cls, v: str, info: ValidationInfo) -> str:
        allowed = info.context.get(\"allowed_cities\", [])
        if allowed and v not in allowed:
            raise ValueError(f\"City must be one of {allowed}\")
        return v

result = client.create(
    response_model=CityExtraction,
    messages=[{\"role\": \"user\", \"content\": \"Extract the city: I live in Paris.\"}],
    validation_context={\"allowed_cities\": [\"Paris\", \"London\", \"Tokyo\"]},
)
", Header 3 ("with-completion-metadata", ["unnumbered", "unlisted"], []) [Str "With Completion Metadata"], CodeBlock ("", ["python"], []) "person, completion = client.create_with_completion(
    response_model=Person,
    messages=[{\"role\": \"user\", \"content\": \"Extract: Jason is 25 years old.\"}],
)
print(person.name)                    # \"Jason\"
print(completion.usage.total_tokens)  # Access token usage from raw completion
", Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Header 3 ("classification-with-enums", ["unnumbered", "unlisted"], []) [Str "Classification with Enums"], CodeBlock ("", ["python"], []) "from enum import Enum
from pydantic import BaseModel

class Sentiment(str, Enum):
    POSITIVE = \"positive\"
    NEGATIVE = \"negative\"
    NEUTRAL = \"neutral\"

class SentimentResult(BaseModel):
    sentiment: Sentiment
    confidence: float

result = client.create(
    response_model=SentimentResult,
    messages=[{\"role\": \"user\", \"content\": \"Classify: I love this product!\"}],
)
# result.sentiment == Sentiment.POSITIVE
", Header 3 ("nested-models", ["unnumbered", "unlisted"], []) [Str "Nested Models"], CodeBlock ("", ["python"], []) "from pydantic import BaseModel
from typing import List

class Address(BaseModel):
    street: str
    city: str
    country: str

class Company(BaseModel):
    name: str
    address: Address
    employee_count: int
    departments: List[str]

company = client.create(
    response_model=Company,
    messages=[{
        \"role\": \"user\",
        \"content\": \"Extract: Acme Corp at 123 Main St, Springfield, USA with 500 employees in Engineering, Sales, and Marketing.\",
    }],
)
", Header 3 ("hooks-for-logging", ["unnumbered", "unlisted"], []) [Str "Hooks for Logging"], CodeBlock ("", ["python"], []) "import instructor
from pydantic import BaseModel

client = instructor.from_provider(\"openai/gpt-4o-mini\")

client.on(\"completion:kwargs\", lambda **kw: print(\"Called with:\", kw))
client.on(\"completion:error\", lambda e: print(f\"Error: {e}\"))
client.on(\"completion:response\", lambda r: print(f\"Tokens: {r.usage.total_tokens}\"))

class UserInfo(BaseModel):
    name: str
    age: int

user_info = client.create(
    response_model=UserInfo,
    messages=[{\"role\": \"user\", \"content\": \"Extract: John is 20 years old\"}],
)
", Header 3 ("failed-attempt-tracking", ["unnumbered", "unlisted"], []) [Str "Failed Attempt Tracking"], CodeBlock ("", ["python"], []) "from instructor.exceptions import InstructorRetryException

try:
    result = client.create(
        response_model=StrictModel,
        messages=[{\"role\": \"user\", \"content\": \"Extract data...\"}],
        max_retries=3,
    )
except InstructorRetryException as e:
    for attempt in e.failed_attempts:
        print(f\"Attempt {attempt.attempt_number}: {attempt.exception}\")
", Header 2 ("limitations", ["unnumbered", "unlisted"], []) [Str "Limitations"], BulletList [[Plain [Strong [Str "Provider-Dependent Behavior"], Str " -- Extraction quality and reliability vary across LLM providers and models. Smaller models may require more retries or produce lower-quality structured output."]], [Plain [Strong [Str "Retry Cost"], Str " -- Each validation-triggered retry is a full LLM API call, adding latency and token cost. Complex validators on weaker models can lead to retry loops that exhaust the maximum retry count."]], [Plain [Strong [Str "Schema Complexity Ceiling"], Str " -- Deeply nested or very large Pydantic models may exceed the context window or confuse the LLM, leading to incomplete or incorrect extraction."]], [Plain [Strong [Str "No Guaranteed Correctness"], Str " -- Validation ensures structural correctness (types, formats, constraints) but cannot verify factual accuracy of the extracted content. The LLM may hallucinate values that pass validation."]], [Plain [Strong [Str "Streaming Validator Limitation"], Str " -- Partial streaming (", Code ("", [], []) "create_partial", Str ") does not support Pydantic validators during intermediate iterations due to the streaming nature of the response. Validators are only applied on the final complete object."]], [Plain [Strong [Str "Literal Type Streaming"], Str " -- Models using ", Code ("", [], []) "Literal", Str " values in partial streaming must inherit from ", Code ("", [], []) "PartialLiteralMixin", Str " to avoid parsing errors with incomplete values."]], [Plain [Strong [Str "Mode Variability"], Str " -- Different extraction modes may perform differently for the same use case. The documentation recommends testing with actual data and models to find the optimal mode."]]], Header 2 ("changelog", ["unnumbered", "unlisted"], []) [Str "Changelog"], Para [Str "Instructor follows semantic versioning. The current version is v1.14.5. The library has evolved from OpenAI-only function calling support to a multi-provider, multi-language platform. Key milestones include the introduction of ", Code ("", [], []) "from_provider()", Str " for unified provider access, streaming support via ", Code ("", [], []) "create_partial()", Str " and ", Code ("", [], []) "create_iterable()", Str ", expansion to 23+ LLM providers, multimodal extraction capabilities for images, PDFs, and audio, the hooks system for lifecycle instrumentation, Jinja templating for dynamic prompts, and the addition of TypeScript, Go, Ruby, Elixir, and Rust implementations. The project maintains an active release cadence with frequent updates."], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " ", Link ("", [], []) [Str "Instructor Documentation - Home"] ("https://python.useinstructor.com/", "")]], [Plain [Str "[", Str "2", Str "]", Str " ", Link ("", [], []) [Str "Instructor - Patching Concepts"] ("https://python.useinstructor.com/concepts/patching/", "")]], [Plain [Str "[", Str "3", Str "]", Str " ", Link ("", [], []) [Str "Instructor - Retry Mechanism"] ("https://python.useinstructor.com/concepts/retrying/", "")]], [Plain [Str "[", Str "4", Str "]", Str " ", Link ("", [], []) [Str "Instructor - Hooks System"] ("https://python.useinstructor.com/concepts/hooks/", "")]], [Plain [Str "[", Str "5", Str "]", Str " ", Link ("", [], []) [Str "Instructor - Partial Streaming"] ("https://python.useinstructor.com/concepts/partial/", "")]], [Plain [Str "[", Str "6", Str "]", Str " ", Link ("", [], []) [Str "Instructor - Iterable Streaming"] ("https://python.useinstructor.com/concepts/lists/", "")]], [Plain [Str "[", Str "7", Str "]", Str " ", Link ("", [], []) [Str "Instructor - Integrations"] ("https://python.useinstructor.com/integrations/", "")]], [Plain [Str "[", Str "8", Str "]", Str " ", Link ("", [], []) [Str "Instructor - Validation and Reasking"] ("https://python.useinstructor.com/concepts/reask_validation/", "")]], [Plain [Str "[", Str "9", Str "]", Str " ", Link ("", [], []) [Str "Instructor - Jinja Templating"] ("https://python.useinstructor.com/concepts/templating/", "")]], [Plain [Str "[", Str "10", Str "]", Str " ", Link ("", [], []) [Str "Instructor - Mode Comparison"] ("https://python.useinstructor.com/modes-comparison/", "")]], [Plain [Str "[", Str "11", Str "]", Str " ", Link ("", [], []) [Str "Instructor - API Reference"] ("https://python.useinstructor.com/api/", "")]], [Plain [Str "[", Str "12", Str "]", Str " ", Link ("", [], []) [Str "Instructor - Architecture"] ("https://python.useinstructor.com/architecture/", "")]], [Plain [Str "[", Str "13", Str "]", Str " ", Link ("", [], []) [Str "Instructor - CLI Reference"] ("https://python.useinstructor.com/cli/", "")]], [Plain [Str "[", Str "14", Str "]", Str " ", Link ("", [], []) [Str "Instructor - Installation"] ("https://python.useinstructor.com/installation/", "")]], [Plain [Str "[", Str "15", Str "]", Str " ", Link ("", [], []) [Str "Instructor - Pydantic Models"] ("https://python.useinstructor.com/concepts/models/", "")]], [Plain [Str "[", Str "16", Str "]", Str " ", Link ("", [], []) [Str "Instructor - Multimodal Capabilities"] ("https://python.useinstructor.com/concepts/multimodal/", "")]], [Plain [Str "[", Str "17", Str "]", Str " ", Link ("", [], []) [Str "Instructor GitHub Repository"] ("https://github.com/instructor-ai/instructor", "")]]]]