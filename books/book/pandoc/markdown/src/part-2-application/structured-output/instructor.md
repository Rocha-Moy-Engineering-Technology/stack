[Header 1 ("instructor", [], []) [Str "Instructor"], BlockQuote [Para [Str "Multi-language library for extracting structured, type-safe data from Large Language Models (LLMs) using Pydantic validation and automatic retries."]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.18421052631578946)), (AlignDefault, (ColWidth 0.8157894736842105))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Name"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Instructor"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Structured Output & Prompt Engineering"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "SDK"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Yes"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "License"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "MIT"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "instructor-ai/instructor"] ("https://github.com/instructor-ai/instructor", "")]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "~", Str "12.4k"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Docs"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "python.useinstructor.com"] ("https://python.useinstructor.com/", "")]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Downloads"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "3M+ monthly"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Contributors"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "100+"]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "Instructor is a library that patches LLM API clients to return structured, validated data instead of raw text. Built on top of Pydantic, it lets developers define response schemas as Python models and have the LLM fill them in directly. When the LLM output fails validation, Instructor automatically retries the request with the validation error context, enabling self-correcting extraction pipelines. The library supports 15+ LLM providers through a unified interface, offers streaming for partial results, and provides full type inference with IDE autocompletion. With over 3 million monthly downloads and 100+ contributors, Instructor has become one of the most widely adopted tools for structured output extraction."], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], Header 3 ("structured-outputs-via-pydantic-models", ["unnumbered", "unlisted"], []) [Str "Structured Outputs via Pydantic Models"], Para [Str "The fundamental idea behind Instructor is that a Pydantic model defines the expected shape of the LLM response. The library injects the model schema into the LLM request (via function calling, tool use, or JSON mode depending on the provider), parses the raw output, and validates it against the model. The developer receives a fully typed Python object rather than a string."], Header 3 ("automatic-retries-reasks", ["unnumbered", "unlisted"], []) [Str "Automatic Retries (Reasks)"], Para [Str "When the LLM returns output that fails Pydantic validation, Instructor does not simply raise an error. Instead, it feeds the validation error message back to the LLM as context and retries the request. This \"reask\" loop continues up to a configurable maximum number of retries, giving the model the opportunity to self-correct. This is especially useful for enforcing constraints that are difficult to express purely in a prompt (for example, value ranges, string formats, or cross-field dependencies)."], Header 3 ("client-patching", ["unnumbered", "unlisted"], []) [Str "Client Patching"], Para [Str "Instructor works by wrapping (patching) existing LLM client libraries. Rather than replacing the client, it augments it with structured output capabilities. This means developers keep their existing authentication, configuration, and error handling while gaining schema-driven extraction on top."], Header 2 ("installation", ["unnumbered", "unlisted"], []) [Str "Installation"], CodeBlock ("", ["bash"], []) "pip install instructor
", Para [Str "Provider-specific extras may be required depending on the target LLM backend (for example, ", Code ("", [], []) "pip install openai", Str " for OpenAI, ", Code ("", [], []) "pip install anthropic", Str " for Anthropic)."], Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Para [Str "Instructor sits as a thin middleware layer between the application and the LLM provider client:"], OrderedList (1, DefaultStyle, DefaultDelim) [[Plain [Strong [Str "Application Layer"], Str " -- Defines Pydantic response models and sends messages through the Instructor-patched client."]], [Plain [Strong [Str "Instructor Layer"], Str " -- Injects the Pydantic model schema into the LLM request, parses the response, runs Pydantic validation, and handles retries on failure."]], [Plain [Strong [Str "Provider Client Layer"], Str " -- The underlying LLM SDK (OpenAI, Anthropic, Google, and others) handles authentication, transport, and raw API communication."]]], Para [Str "The patching mechanism wraps the provider client's completion method so that all existing client configuration (API keys, base URLs, timeouts) is preserved. Instructor intercepts only the response parsing step."], Header 3 ("retry-flow", ["unnumbered", "unlisted"], []) [Str "Retry Flow"], CodeBlock ("", [""], []) "Application -> Instructor -> LLM Provider
                  |
                  v
          Parse response
                  |
           Validate with Pydantic
                  |
         [Pass] -> Return typed object
         [Fail] -> Append validation error to messages -> Retry LLM call
", Header 2 ("key-features", ["unnumbered", "unlisted"], []) [Str "Key Features"], BulletList [[Plain [Strong [Str "Type Safety and IDE Autocompletion"], Str " -- Response models are standard Pydantic classes, giving full type inference, autocomplete, and static analysis support in editors and type checkers."]], [Plain [Strong [Str "Automatic Retries with Validation Context"], Str " -- Failed validations trigger retries where the error message is included in the next prompt, allowing the LLM to self-correct."]], [Plain [Strong [Str "Multi-Provider Support"], Str " -- A single ", Code ("", [], []) "from_provider()", Str " interface supports OpenAI, Anthropic, Google, Ollama, DeepSeek, and 15+ other providers without changing application code."]], [Plain [Strong [Str "Streaming"], Str " -- ", Code ("", [], []) "create_partial()", Str " streams partial results as the LLM generates tokens, enabling progressive UI updates. ", Code ("", [], []) "create_iterable()", Str " streams a sequence of complete objects."]], [Plain [Strong [Str "Custom Pydantic Validators"], Str " -- Standard Pydantic field validators and model validators work seamlessly, enabling arbitrarily complex validation logic (regex patterns, cross-field checks, business rules)."]], [Plain [Strong [Str "Async/Await"], Str " -- Full async support for non-blocking LLM calls in asynchronous applications."]], [Plain [Strong [Str "Jinja Templating"], Str " -- Prompt templates can use Jinja syntax for dynamic prompt construction with variables and control flow."]], [Plain [Strong [Str "Hooks"], Str " -- Lifecycle hooks allow injecting custom logic at various stages of the request/response cycle (for example, logging, metrics, or transformation)."]]], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], BulletList [[Plain [Strong [Str "Data Extraction"], Str " -- Pulling structured records (names, dates, amounts, entities) from unstructured text such as emails, documents, or web pages."]], [Plain [Strong [Str "Classification"], Str " -- Categorizing text into predefined enums or labels with guaranteed valid output values."]], [Plain [Strong [Str "Content Generation with Constraints"], Str " -- Generating content that must conform to a specific schema (for example, product descriptions with required fields, quiz questions with exactly four options)."]], [Plain [Strong [Str "Multi-Step Pipelines"], Str " -- Chaining structured outputs where the validated result of one step feeds into the next, with type safety preserved throughout."]], [Plain [Strong [Str "Search and Retrieval Augmented Generation (RAG)"], Str " -- Extracting structured queries or filters from natural language to drive database lookups or search APIs."]], [Plain [Strong [Str "Streaming User Interfaces"], Str " -- Progressively rendering structured data in a UI as the LLM generates it, using partial streaming."]]], Header 2 ("api-reference", ["unnumbered", "unlisted"], []) [Str "API Reference"], Header 3 ("client-creation", ["unnumbered", "unlisted"], []) [Str "Client Creation"], CodeBlock ("", ["python"], []) "import instructor

# Universal provider interface
client = instructor.from_provider(\"openai/gpt-5-nano\")

# Provider-specific patching
import openai
client = instructor.from_openai(openai.OpenAI())

import anthropic
client = instructor.from_anthropic(anthropic.Anthropic())
", Header 3 ("core-methods", ["unnumbered", "unlisted"], []) [Str "Core Methods"], BulletList [[Plain [Strong [Code ("", [], []) "client.create(response_model, messages, max_retries=...)"], Str " -- Sends a completion request and returns a validated instance of ", Code ("", [], []) "response_model", Str ". Retries automatically on validation failure up to ", Code ("", [], []) "max_retries", Str "."]], [Plain [Strong [Code ("", [], []) "client.create_with_completion(response_model, messages)"], Str " -- Returns a tuple of ", Code ("", [], []) "(response_model_instance, raw_completion)", Str ", giving access to both the validated object and the raw provider response (token usage, finish reason, and similar metadata)."]], [Plain [Strong [Code ("", [], []) "client.create_partial(response_model, messages)"], Str " -- Returns an iterator that yields progressively more complete instances of ", Code ("", [], []) "response_model", Str " as tokens stream in. Fields populate incrementally."]], [Plain [Strong [Code ("", [], []) "client.create_iterable(response_model, messages)"], Str " -- Returns an iterator of complete ", Code ("", [], []) "response_model", Str " instances, useful when the LLM is expected to produce a list of structured objects."]]], Header 3 ("response-model-definition", ["unnumbered", "unlisted"], []) [Str "Response Model Definition"], CodeBlock ("", ["python"], []) "from pydantic import BaseModel, Field, field_validator

class Person(BaseModel):
    name: str = Field(description=\"Full legal name\")
    age: int = Field(ge=0, le=150, description=\"Age in years\")

    @field_validator(\"name\")
    @classmethod
    def name_must_not_be_empty(cls, v: str) -> str:
        if not v.strip():
            raise ValueError(\"Name must not be empty\")
        return v.strip()
", Header 2 ("configuration", ["unnumbered", "unlisted"], []) [Str "Configuration"], Header 3 ("retry-configuration", ["unnumbered", "unlisted"], []) [Str "Retry Configuration"], CodeBlock ("", ["python"], []) "person = client.create(
    response_model=Person,
    messages=[{\"role\": \"user\", \"content\": \"Extract: John is 28 years old.\"}],
    max_retries=3,  # Maximum number of validation retry attempts
)
", Header 3 ("provider-selection", ["unnumbered", "unlisted"], []) [Str "Provider Selection"], Para [Str "The ", Code ("", [], []) "from_provider()", Str " method accepts a string in the format ", Code ("", [], []) "\"provider/model\"", Str ":"], CodeBlock ("", ["python"], []) "client = instructor.from_provider(\"openai/gpt-5-nano\")
client = instructor.from_provider(\"anthropic/claude-sonnet-4-20250514\")
client = instructor.from_provider(\"google/gemini-2.0-flash\")
", Header 3 ("mode-configuration", ["unnumbered", "unlisted"], []) [Str "Mode Configuration"], Para [Str "Instructor supports multiple extraction modes depending on provider capabilities:"], BulletList [[Plain [Strong [Str "Function Calling / Tool Use"], Str " -- The default for providers that support it. The schema is passed as a function or tool definition."]], [Plain [Strong [Str "JSON Mode"], Str " -- Forces the LLM to output valid JSON, which is then parsed against the Pydantic model."]], [Plain [Strong [Str "Markdown JSON"], Str " -- Extracts JSON from markdown code blocks in the response."]]], Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Header 3 ("basic-extraction", ["unnumbered", "unlisted"], []) [Str "Basic Extraction"], CodeBlock ("", ["python"], []) "import instructor
from pydantic import BaseModel

client = instructor.from_provider(\"openai/gpt-5-nano\")

class Person(BaseModel):
    name: str
    age: int

person = client.create(
    response_model=Person,
    messages=[{\"role\": \"user\", \"content\": \"Extract: Jason is 25 years old.\"}],
)
# person.name == \"Jason\"
# person.age == 25
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

async_client = instructor.from_provider(\"openai/gpt-5-nano\", async_=True)

async def extract():
    person = await async_client.create(
        response_model=Person,
        messages=[{\"role\": \"user\", \"content\": \"Extract: Jason is 25 years old.\"}],
    )
    return person

result = asyncio.run(extract())
", Header 3 ("custom-validators-for-business-logic", ["unnumbered", "unlisted"], []) [Str "Custom Validators for Business Logic"], CodeBlock ("", ["python"], []) "from pydantic import BaseModel, field_validator

class UserProfile(BaseModel):
    username: str
    email: str
    age: int

    @field_validator(\"email\")
    @classmethod
    def validate_email(cls, v: str) -> str:
        if \"@\" not in v:
            raise ValueError(\"Invalid email format\")
        return v

    @field_validator(\"age\")
    @classmethod
    def validate_age(cls, v: int) -> int:
        if v < 18:
            raise ValueError(\"User must be at least 18 years old\")
        return v
", Para [Str "When the LLM produces an email without ", Code ("", [], []) "@", Str " or an age below 18, Instructor feeds the validation error back to the model and retries, guiding it toward a valid response."], Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Header 3 ("classification-with-enums", ["unnumbered", "unlisted"], []) [Str "Classification with Enums"], CodeBlock ("", ["python"], []) "from enum import Enum
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

class Address(BaseModel):
    street: str
    city: str
    country: str

class Company(BaseModel):
    name: str
    address: Address
    employee_count: int

company = client.create(
    response_model=Company,
    messages=[
        {
            \"role\": \"user\",
            \"content\": \"Extract company info: Acme Corp is based at 123 Main St, Springfield, USA with 500 employees.\",
        }
    ],
)
", Header 3 ("with-completion-metadata", ["unnumbered", "unlisted"], []) [Str "With Completion Metadata"], CodeBlock ("", ["python"], []) "person, completion = client.create_with_completion(
    response_model=Person,
    messages=[{\"role\": \"user\", \"content\": \"Extract: Jason is 25 years old.\"}],
)
print(person.name)  # \"Jason\"
print(completion.usage.total_tokens)  # Access token usage from raw completion
", Header 2 ("limitations", ["unnumbered", "unlisted"], []) [Str "Limitations"], BulletList [[Plain [Strong [Str "Provider-Dependent Behavior"], Str " -- Extraction quality and reliability vary across LLM providers and models. Smaller models may require more retries or produce lower-quality structured output."]], [Plain [Strong [Str "Retry Cost"], Str " -- Each validation-triggered retry is a full LLM API call, adding latency and token cost. Complex validators on weaker models can lead to retry loops that exhaust the maximum retry count."]], [Plain [Strong [Str "Schema Complexity Ceiling"], Str " -- Deeply nested or very large Pydantic models may exceed the context window or confuse the LLM, leading to incomplete or incorrect extraction."]], [Plain [Strong [Str "No Guaranteed Correctness"], Str " -- Validation ensures structural correctness (types, formats, constraints) but cannot verify factual accuracy of the extracted content. The LLM may hallucinate values that pass validation."]], [Plain [Strong [Str "Streaming Limitations"], Str " -- Partial streaming yields incomplete objects during generation. Application code must handle ", Code ("", [], []) "None", Str " fields and incomplete state gracefully."]]], Header 2 ("changelog", ["unnumbered", "unlisted"], []) [Str "Changelog"], Para [Str "Instructor follows semantic versioning. The library has evolved from OpenAI-only function calling support to a multi-provider, multi-language platform. Key milestones include the introduction of ", Code ("", [], []) "from_provider()", Str " for unified provider access, streaming support via ", Code ("", [], []) "create_partial()", Str " and ", Code ("", [], []) "create_iterable()", Str ", and expansion to 15+ LLM providers. The project maintains an active release cadence with frequent updates."], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " ", Link ("", [], []) [Str "Instructor Documentation"] ("https://python.useinstructor.com/", "")]]]]