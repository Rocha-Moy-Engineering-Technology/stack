[Header 1 ("guardrails-ai", [], []) [Str "Guardrails AI"], BlockQuote [Para [Str "Python framework for input/output validation guards on LLM applications"]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.5)), (AlignDefault, (ColWidth 0.5))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Guardrails"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "SDK"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Yes"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "https://github.com/guardrails-ai/guardrails"] ("https://github.com/guardrails-ai/guardrails", "")]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "6862"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Documentation"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "Official Docs"] ("https://guardrailsai.com/docs", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "Guardrails AI is a Python framework for building reliable AI applications through input/output validation. It provides two primary capabilities: deploying Input/Output Guards that detect, quantify, and mitigate the presence of specific types of risks in Large Language Model (LLM) interactions, and generating structured data from LLM outputs that conform to predefined schemas."], Para [Str "The framework takes a modular approach to LLM safety. Rather than providing a monolithic filtering system, Guardrails decomposes risk management into composable units called Validators. Each validator targets a specific risk category such as toxic language, personally identifiable information (PII) exposure, hallucinated content, or competitor mentions. Multiple validators combine into Guards, which wrap LLM calls and intercept both inputs and outputs to apply validation rules before data reaches the end user."], Para [Str "Guardrails Hub serves as a centralized marketplace of pre-built validators contributed by the community and the Guardrails team. As of early 2025, the Guardrails Index benchmark compares the performance and latency of 24 guardrails across six common risk categories, providing an empirical basis for validator selection."], Para [Str "The project is licensed under Apache 2.0 and supports Python 3.10 through 3.14. It is available as both an in-application library and a standalone server for centralized validation across multiple services."], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], Para [Strong [Str "Guards"], Str " are the primary interface for Guardrails. A Guard wraps an LLM call and applies one or more validators to the inputs, outputs, or both. Guards can operate in two modes: as a pass-through validator that checks LLM output after generation, or as an active participant that re-asks the LLM when validation fails. Each Guard maintains a call history for debugging and observability."], Para [Strong [Str "Validators"], Str " are the atomic units of risk measurement. Each validator encodes a specific quality criterion and produces a binary outcome: a PassResult when the content meets the criterion (returning the value unchanged) or a FailResult when the content violates the criterion (triggering a configured on-fail action). Validators can be stateless pattern matchers, machine learning classifiers, or LLM-based evaluators depending on the risk category they target."], Para [Strong [Str "On-Fail Actions"], Str " determine what happens when a validator produces a FailResult. Eight actions are available: REASK instructs the LLM to regenerate output with feedback about the failure; FIX programmatically corrects the output when a deterministic fix exists; FILTER removes the failing field from structured output while preserving valid fields; REFRAIN returns None when the output is unsafe for end users; NOOP logs the failure without corrective action; EXCEPTION raises an error immediately; FIX_REASK attempts a deterministic fix first and falls back to re-asking if validation still fails; and CUSTOM executes a user-defined handler function."], Para [Strong [Str "Guardrails Hub"], Str " is the centralized repository of pre-built validators. Hub validators are installed via the CLI (", Code ("", [], []) "guardrails hub install", Str ") or the in-code ", Code ("", [], []) "install()", Str " function. Validators in the Hub cover six primary risk categories: content safety, PII detection, toxic language, jailbreak detection, topic restriction, and output format validation."], Para [Strong [Str "Structured Data Generation"], Str " uses Pydantic models to define the expected schema of LLM output. Guards enforce that LLM responses conform to the schema through either function calling (when the model supports it) or prompt optimization. Field-level validators can be attached to individual fields in the Pydantic model for granular control."], Para [Strong [Str "Validation Metadata"], Str " provides runtime context to validators that need information unavailable at initialization time. This metadata is passed to ", Code ("", [], []) "guard.validate()", Str " or ", Code ("", [], []) "guard()", Str " calls, allowing validators like ", Code ("", [], []) "ExtractedSummarySentencesMatch", Str " to access dynamic information such as file paths or reference documents for comparison operations."], Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Para [Str "Guardrails operates as a middleware layer between application code and LLM providers. The architecture has three primary components:"], Para [Strong [Str "Guard Layer."], Str " The Guard object orchestrates the validation pipeline. When invoked, it sends the request to the LLM, receives the response, and passes it through the configured validators. If any validator fails and the on-fail action requires re-asking, the Guard constructs a corrective prompt and sends it back to the LLM. This loop continues up to a configurable ", Code ("", [], []) "num_reasks", Str " limit."], CodeBlock ("", ["text"], []) "Application Code
       |
       v
   Guard Object
       |
       +---> LLM Provider (OpenAI, Anthropic, Cohere, HuggingFace)
       |          |
       |          v
       +<--- Raw LLM Response
       |
       v
  Validator Pipeline
       |
       +---> PassResult --> Return validated output
       |
       +---> FailResult --> Apply on-fail action
                |
                +---> REASK: Re-prompt LLM with error feedback
                +---> FIX: Apply deterministic correction
                +---> FILTER: Remove failing field
                +---> REFRAIN: Return None
                +---> EXCEPTION: Raise error
", Para [Strong [Str "Validator Pipeline."], Str " Validators execute sequentially on the LLM output. Each validator receives the current value and produces either a PassResult or FailResult. Multiple validators compose into a pipeline where the output of one validator feeds into the next. Validators can operate on the full response text or on individual fields within structured output."], Para [Strong [Str "Guardrails Server."], Str " For production deployments, Guardrails can run as a standalone Flask-based REST API server. The server exposes OpenAI-compatible endpoints, allowing any OpenAI SDK client to route through the Guardrails server for transparent validation. Guards are defined in a Python configuration file and loaded at server startup. The server architecture separates validation from application logic, enabling independent scaling of the validation layer."], CodeBlock ("", ["text"], []) "Client Application
       |
       v
Guardrails Server (Flask/Gunicorn/Uvicorn)
       |
       +---> /guards/{guardName}/openai/v1/chat/completions
       |
       v
  Guard Pipeline --> LLM Provider --> Validators --> Response
", Header 2 ("key-features-and-functionality", ["unnumbered", "unlisted"], []) [Str "Key Features and Functionality"], Para [Strong [Str "Input and Output Validation."], Str " Guards can validate both the input sent to an LLM and the output received. Input guards catch prompt injection attempts, off-topic queries, and policy violations before they reach the model. Output guards catch toxic content, PII leaks, hallucinations, and format violations before the response reaches the user."], Para [Strong [Str "Structured Output Generation."], Str " Guardrails converts free-form LLM text into structured data conforming to Pydantic models. Field-level validators provide granular control over individual attributes. When a field fails validation, the on-fail action applies to that specific field rather than the entire response."], Para [Strong [Str "Re-Ask Loop."], Str " When the REASK on-fail action is configured, Guardrails automatically constructs a corrective prompt that includes the original output and a description of the validation failure. The LLM is asked to regenerate its response to meet the specified criteria. The number of re-ask attempts is configurable via ", Code ("", [], []) "num_reasks", Str "."], Para [Strong [Str "Streaming Support."], Str " Guards can validate streaming LLM responses by processing chunks as they arrive. This allows real-time validation without waiting for the complete response, enabling responsive user experiences while still enforcing safety constraints."], Para [Strong [Str "Async Support."], Str " The ", Code ("", [], []) "AsyncGuard", Str " class provides async/await support for concurrent validation workflows. Async guards allow making concurrent calls to multiple LLMs and processing response chunks as they arrive, providing better performance in I/O-bound applications."], Para [Strong [Str "Call History."], Str " Guards maintain a history of all LLM calls and validation results. Starting with version 0.8.0, history is capped at 10 entries by default, configurable via the ", Code ("", [], []) "history_max_length", Str " parameter. This history is available for debugging, observability, and audit purposes."], Para [Strong [Str "OpenAI-Compatible Server."], Str " The Guardrails Server exposes endpoints compatible with the OpenAI Chat Completions API. Applications using the OpenAI SDK can route through the Guardrails Server by changing their ", Code ("", [], []) "base_url", Str ", gaining validation without code changes to the application layer."], Para [Strong [Str "Hub Ecosystem."], Str " The Guardrails Hub provides a growing collection of pre-built validators that can be installed and composed without writing custom validation logic. Hub validators cover categories including content safety, PII detection, toxic language, jailbreak prevention, format validation (JSON, SQL, regex), and business rule enforcement."], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], Para [Strong [Str "Content Safety Enforcement."], Str " Filter toxic language, hate speech, and inappropriate content from LLM outputs using validators like ", Code ("", [], []) "ToxicLanguage", Str " (powered by the Detoxify multi-label classifier) and custom content policies. On-fail actions control whether unsafe content is removed, replaced, or causes an exception."], Para [Strong [Str "PII Protection."], Str " Detect and redact personally identifiable information such as names, email addresses, phone numbers, and social security numbers from LLM outputs using the ", Code ("", [], []) "DetectPII", Str " validator (powered by Microsoft Presidio). This is critical for applications handling user data subject to privacy regulations."], Para [Strong [Str "Competitor Mention Filtering."], Str " Prevent LLM responses from mentioning competitors using the ", Code ("", [], []) "CompetitorCheck", Str " validator. Configure a list of competitor names and the guard either removes mentions (FIX) or rejects the response entirely (EXCEPTION/REFRAIN)."], Para [Strong [Str "Structured Data Extraction."], Str " Extract structured information from unstructured LLM text. Define a Pydantic model with the desired schema, attach field-level validators, and the guard ensures the LLM output conforms to the schema with all validation rules satisfied."], Para [Strong [Str "Prompt Injection Defense."], Str " Input guards detect and block prompt injection attempts before they reach the LLM. Validators identify common injection patterns, jailbreak attempts, and off-topic inputs that could cause the model to deviate from its intended behavior."], Para [Strong [Str "API Response Validation."], Str " Validate that LLM-generated API responses, SQL queries, or code snippets meet format and safety requirements before execution. Format validators ensure syntactic correctness while content validators prevent injection attacks in generated code."], Para [Strong [Str "Bias and Fairness Monitoring."], Str " Detect bias in LLM outputs across demographic categories using specialized validators. NOOP on-fail actions allow logging bias occurrences without blocking responses, enabling monitoring and analysis of bias patterns over time."], Header 2 ("api-reference-summary", ["unnumbered", "unlisted"], []) [Str "API Reference Summary"], Header 3 ("guard-class", ["unnumbered", "unlisted"], []) [Str "Guard Class"], Para [Str "The primary interface for validation. Core methods:"], Para [Strong [Code ("", [], []) "Guard()"], Str " -- Create a new Guard instance. Accepts optional ", Code ("", [], []) "name", Str " parameter for identification when using Guardrails Server."], Para [Strong [Code ("", [], []) "guard.use(validator, **kwargs)"], Str " -- Add a single validator to the guard with configuration parameters and an ", Code ("", [], []) "on_fail", Str " action."], Para [Strong [Code ("", [], []) "guard.use_many(*validators)"], Str " -- Add multiple validators to the guard simultaneously."], Para [Strong [Code ("", [], []) "guard(model, messages, **kwargs)"], Str " -- Invoke the guard with an LLM call. Mirrors standard LLM SDK call signatures. Returns a ", Code ("", [], []) "GuardResponse", Str " containing the raw output, validated output, and validation status."], Para [Strong [Code ("", [], []) "guard.parse(llm_output, num_reasks=0)"], Str " -- Validate pre-generated LLM output. With ", Code ("", [], []) "num_reasks=0", Str ", operates as a pure post-processor. With higher values, enables re-asking the LLM on failure."], Para [Strong [Code ("", [], []) "guard.validate(value, metadata=None)"], Str " -- Validate a value against the guard's validators without making an LLM call. Useful for validating cached or pre-fetched responses."], Para [Strong [Code ("", [], []) "Guard.for_pydantic(output_class, prompt=None)"], Str " -- Create a guard configured for structured data generation using a Pydantic model."], Header 3 ("guardresponse", ["unnumbered", "unlisted"], []) [Str "GuardResponse"], Para [Str "Returned by guard invocations:"], BulletList [[Plain [Code ("", [], []) "raw_llm_output", Str " -- The unmodified LLM response text"]], [Plain [Code ("", [], []) "validated_output", Str " -- The output after validation and any corrections"]], [Plain [Code ("", [], []) "validation_passed", Str " -- Boolean indicating if all validators passed"]], [Plain [Code ("", [], []) "reask", Str " -- Information about any re-ask attempts"]]], Header 3 ("validation-results", ["unnumbered", "unlisted"], []) [Str "Validation Results"], Para [Strong [Code ("", [], []) "PassResult"], Str " -- Returned by validators when content meets criteria. Contains the validated value, which in most cases is the original value unchanged."], Para [Strong [Code ("", [], []) "FailResult"], Str " -- Returned by validators when content violates criteria. Contains an ", Code ("", [], []) "error_message", Str " describing the failure and an optional ", Code ("", [], []) "fix_value", Str " for deterministic corrections."], Header 3 ("onfailaction-enum", ["unnumbered", "unlisted"], []) [Str "OnFailAction Enum"], CodeBlock ("", ["python"], []) "from guardrails import OnFailAction

OnFailAction.REASK       # Re-ask LLM with failure feedback
OnFailAction.FIX         # Apply deterministic fix_value
OnFailAction.FILTER      # Remove failing field from structured output
OnFailAction.REFRAIN     # Return None for unsafe content
OnFailAction.NOOP        # Log failure, return original value
OnFailAction.EXCEPTION   # Raise ValidationError
OnFailAction.FIX_REASK   # Try FIX first, then REASK if still failing
OnFailAction.CUSTOM      # Execute custom handler function
", Header 2 ("configuration-and-customization", ["unnumbered", "unlisted"], []) [Str "Configuration and Customization"], Header 3 ("guard-configuration", ["unnumbered", "unlisted"], []) [Str "Guard Configuration"], Para [Str "Guards accept configuration at multiple levels:"], CodeBlock ("", ["python"], []) "from guardrails import Guard, OnFailAction
from guardrails.hub import ToxicLanguage, DetectPII, CompetitorCheck

# Basic guard with a single validator
guard = Guard(name=\"content-safety\").use(
    ToxicLanguage,
    threshold=0.5,
    validation_method=\"sentence\",
    on_fail=OnFailAction.REFRAIN,
)

# Guard with multiple validators
guard = Guard(name=\"production-guard\").use_many(
    ToxicLanguage(threshold=0.5, on_fail=OnFailAction.REFRAIN),
    DetectPII(on_fail=OnFailAction.FIX),
    CompetitorCheck(
        competitors=[\"Apple\", \"Microsoft\", \"Google\"],
        on_fail=OnFailAction.FIX,
    ),
)
", Header 3 ("structured-output-with-pydantic", ["unnumbered", "unlisted"], []) [Str "Structured Output with Pydantic"], CodeBlock ("", ["python"], []) "from pydantic import BaseModel, Field
from guardrails import Guard

class UserProfile(BaseModel):
    name: str = Field(description=\"The user's full name\")
    email: str = Field(description=\"The user's email address\")
    age: int = Field(description=\"The user's age in years\", ge=0, le=150)

guard = Guard.for_pydantic(output_class=UserProfile)
", Header 3 ("custom-validators", ["unnumbered", "unlisted"], []) [Str "Custom Validators"], Para [Str "Create validators using the class-based approach:"], CodeBlock ("", ["python"], []) "from guardrails.validators import Validator, register_validator, PassResult, FailResult

@register_validator(name=\"custom/word_count\", data_type=\"string\")
class WordCount(Validator):
    def __init__(self, min_words: int, max_words: int, on_fail=None, **kwargs):
        super().__init__(on_fail=on_fail, min_words=min_words, max_words=max_words, **kwargs)
        self.min_words = min_words
        self.max_words = max_words

    def _validate(self, value, metadata=None) -> PassResult | FailResult:
        word_count = len(value.split())
        if self.min_words <= word_count <= self.max_words:
            return PassResult()
        return FailResult(
            error_message=f\"Expected {self.min_words}-{self.max_words} words, got {word_count}.\",
            fix_value=\" \".join(value.split()[:self.max_words]),
        )
", Header 3 ("server-configuration", ["unnumbered", "unlisted"], []) [Str "Server Configuration"], Para [Str "Define guards in a Python config file for the Guardrails Server:"], CodeBlock ("", ["python"], []) "# config.py
from guardrails import Guard, OnFailAction
from guardrails.hub import ToxicLanguage, DetectPII

content_guard = Guard(name=\"content-guard\").use_many(
    ToxicLanguage(threshold=0.5, on_fail=OnFailAction.REFRAIN),
    DetectPII(on_fail=OnFailAction.FIX),
)
", Para [Str "Start the server:"], CodeBlock ("", ["bash"], []) "guardrails start --config=./config.py
", Header 3 ("environment-variables", ["unnumbered", "unlisted"], []) [Str "Environment Variables"], BulletList [[Plain [Code ("", [], []) "GUARDRAILS_BASE_URL", Str " -- Base URL for the Guardrails Server (default: ", Code ("", [], []) "http://localhost:8000", Str ")"]], [Plain [Code ("", [], []) "GUARDRAILS_API_KEY", Str " -- API key for authenticating with the Guardrails Server"]], [Plain [Code ("", [], []) "OPENAI_API_KEY", Str " -- API key for the OpenAI provider when using OpenAI models"]], [Plain [Code ("", [], []) "ANTHROPIC_API_KEY", Str " -- API key for the Anthropic provider when using Claude models"]]], Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Header 3 ("direct-in-application-usage", ["unnumbered", "unlisted"], []) [Str "Direct In-Application Usage"], Para [Str "The most common pattern wraps LLM calls directly in application code:"], CodeBlock ("", ["python"], []) "from guardrails import Guard, OnFailAction
from guardrails.hub import ToxicLanguage

guard = Guard().use(
    ToxicLanguage,
    threshold=0.5,
    on_fail=OnFailAction.EXCEPTION,
)

result = guard(
    model=\"gpt-4o\",
    messages=[{\"role\": \"user\", \"content\": \"Summarize this article.\"}],
)

print(result.validated_output)
", Header 3 ("openai-sdk-proxy", ["unnumbered", "unlisted"], []) [Str "OpenAI SDK Proxy"], Para [Str "Route existing OpenAI SDK calls through the Guardrails Server without changing application code:"], CodeBlock ("", ["python"], []) "from openai import OpenAI

# Point the OpenAI client at the Guardrails Server
client = OpenAI(
    base_url=\"http://localhost:8000/guards/content-guard/openai/v1/\",
    api_key=\"your-api-key\",
)

response = client.chat.completions.create(
    model=\"gpt-4o\",
    messages=[{\"role\": \"user\", \"content\": \"Hello, world!\"}],
)
", Header 3 ("litellm-integration", ["unnumbered", "unlisted"], []) [Str "LiteLLM Integration"], Para [Str "Guardrails integrates with LiteLLM for multi-provider routing with validation:"], CodeBlock ("", ["python"], []) "import litellm
from guardrails import Guard, OnFailAction
from guardrails.hub import ToxicLanguage

guard = Guard().use(
    ToxicLanguage,
    threshold=0.5,
    on_fail=OnFailAction.REFRAIN,
)

# Use LiteLLM model strings with Guardrails
result = guard(
    model=\"anthropic/claude-3-5-sonnet-latest\",
    messages=[{\"role\": \"user\", \"content\": \"Explain quantum computing.\"}],
)
", Header 3 ("production-deployment-with-gunicorn", ["unnumbered", "unlisted"], []) [Str "Production Deployment with Gunicorn"], Para [Str "Deploy the Guardrails Server behind a production Web Server Gateway Interface (WSGI) server:"], CodeBlock ("", ["bash"], []) "gunicorn \\
    --bind 0.0.0.0:8000 \\
    --timeout=90 \\
    --workers=4 \\
    'guardrails_api.app:create_app(None, \"config.py\")'
", Para [Str "Worker count recommendation: ", Code ("", [], []) "(2 x num_cores) + 1", Str " as a baseline. Adjust based on whether validators are CPU-bound (static validators) or I/O-bound (LLM-based validators)."], Header 3 ("docker-deployment", ["unnumbered", "unlisted"], []) [Str "Docker Deployment"], CodeBlock ("", ["dockerfile"], []) "FROM python:3.12-slim

RUN pip install guardrails-ai
RUN guardrails configure

COPY config.py /app/config.py
WORKDIR /app

CMD [\"guardrails\", \"start\", \"--config=./config.py\"]
", Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Header 3 ("basic-output-validation", ["unnumbered", "unlisted"], []) [Str "Basic Output Validation"], CodeBlock ("", ["python"], []) "from guardrails import Guard, OnFailAction
from guardrails.hub import RegexMatch

# Validate phone number format
guard = Guard().use(
    RegexMatch,
    regex=r\"\\(?\\d{3}\\)?-? *\\d{3}-? *-?\\d{4}\",
    on_fail=OnFailAction.EXCEPTION,
)

# This passes validation
guard.validate(\"123-456-7890\")

# This raises an exception
try:
    guard.validate(\"not a phone number\")
except Exception as e:
    print(f\"Validation failed: {e}\")
", Header 3 ("multi-validator-guard", ["unnumbered", "unlisted"], []) [Str "Multi-Validator Guard"], CodeBlock ("", ["python"], []) "from guardrails import Guard, OnFailAction
from guardrails.hub import CompetitorCheck, ToxicLanguage

guard = Guard().use_many(
    CompetitorCheck(
        competitors=[\"Apple\", \"Microsoft\", \"Google\"],
        on_fail=OnFailAction.FIX,
    ),
    ToxicLanguage(
        threshold=0.5,
        validation_method=\"sentence\",
        on_fail=OnFailAction.REFRAIN,
    ),
)

result = guard(
    model=\"gpt-4o\",
    messages=[{
        \"role\": \"user\",
        \"content\": \"Compare our product to competitors in the market.\",
    }],
)

if result.validation_passed:
    print(result.validated_output)
else:
    print(\"Response was filtered for safety.\")
", Header 3 ("structured-data-extraction-with-pydantic", ["unnumbered", "unlisted"], []) [Str "Structured Data Extraction with Pydantic"], CodeBlock ("", ["python"], []) "from pydantic import BaseModel, Field
from guardrails import Guard

class Pet(BaseModel):
    name: str = Field(description=\"The pet's name\")
    species: str = Field(description=\"The pet's species\")
    age: int = Field(description=\"The pet's age in years\")
    favorite_toy: str = Field(description=\"The pet's favorite toy\")

guard = Guard.for_pydantic(
    output_class=Pet,
    prompt=\"Tell me about a golden retriever named Buddy.\",
)

result = guard(
    model=\"gpt-4o\",
    messages=[{
        \"role\": \"user\",
        \"content\": \"Tell me about a golden retriever named Buddy.\",
    }],
)

pet = result.validated_output
print(f\"{pet['name']} is a {pet['species']}, age {pet['age']}\")
", Header 3 ("on-fail-action-comparison", ["unnumbered", "unlisted"], []) [Str "On-Fail Action Comparison"], Para [Str "The following demonstrates how different on-fail actions handle the same toxic input:"], CodeBlock ("", ["python"], []) "from guardrails import Guard, OnFailAction
from guardrails.hub import ToxicLanguage

toxic_text = \"damn you!\"

# FIX: Removes toxic portion, returns \"you!\"
guard_fix = Guard().use(ToxicLanguage, threshold=0.5, on_fail=OnFailAction.FIX)
result = guard_fix.validate(toxic_text)
print(f\"FIX: {result.validated_output}\")

# REFRAIN: Returns None
guard_refrain = Guard().use(ToxicLanguage, threshold=0.5, on_fail=OnFailAction.REFRAIN)
result = guard_refrain.validate(toxic_text)
print(f\"REFRAIN: {result.validated_output}\")

# NOOP: Returns original text unchanged, logs failure
guard_noop = Guard().use(ToxicLanguage, threshold=0.5, on_fail=OnFailAction.NOOP)
result = guard_noop.validate(toxic_text)
print(f\"NOOP: {result.validated_output}\")

# EXCEPTION: Raises an error
guard_exc = Guard().use(ToxicLanguage, threshold=0.5, on_fail=OnFailAction.EXCEPTION)
try:
    guard_exc.validate(toxic_text)
except Exception as e:
    print(f\"EXCEPTION: {e}\")
", Header 3 ("custom-on-fail-handler", ["unnumbered", "unlisted"], []) [Str "Custom On-Fail Handler"], CodeBlock ("", ["python"], []) "from guardrails import Guard, OnFailAction
from guardrails.hub import ToxicLanguage

def custom_handler(value, fail_result):
    \"\"\"Log the failure and return a sanitized placeholder.\"\"\"
    print(f\"Validation failed: {fail_result.error_message}\")
    return \"[Content moderated]\"

guard = Guard().use(
    ToxicLanguage,
    threshold=0.5,
    on_fail=OnFailAction.CUSTOM,
    on_fail_handler=custom_handler,
)
", Header 2 ("limitations-and-considerations", ["unnumbered", "unlisted"], []) [Str "Limitations and Considerations"], Para [Strong [Str "Validator Latency."], Str " Machine learning-based validators (toxic language, PII detection) add measurable latency to each LLM call. Static validators (regex, format checks) are fast, but ML-based validators may take tens to hundreds of milliseconds per invocation. The Guardrails Index benchmark provides empirical latency data for planning purposes."], Para [Strong [Str "Re-Ask Cost."], Str " The REASK on-fail action triggers additional LLM calls, which increases both latency and token cost. Each re-ask is a full LLM invocation with the original prompt plus corrective feedback. Setting ", Code ("", [], []) "num_reasks", Str " too high can lead to significant cost amplification."], Para [Strong [Str "ML Validator Memory Footprint."], Str " Validators powered by machine learning models (such as ToxicLanguage using Detoxify) require loading model weights into memory. In server deployments, this memory cost multiplies with the number of worker processes. Multithreading via ", Code ("", [], []) "--threads", Str " reduces memory overhead compared to multiprocessing but may introduce race conditions in history manipulation."], Para [Strong [Str "Python Only for Core Framework."], Str " While a JavaScript client exists, the core validation framework and custom validator development require Python. Applications in other languages must interact through the Guardrails Server REST API."], Para [Strong [Str "Hub Dependency."], Str " Some Hub validators require network access to download model artifacts or contact external APIs. Deployments in air-gapped environments may need to pre-download all required validator models and dependencies."], Para [Strong [Str "False Positives and Negatives."], Str " ML-based validators are probabilistic. Threshold tuning is required per use case to balance false positive rates (blocking legitimate content) against false negative rates (allowing violating content through). The ", Code ("", [], []) "threshold", Str " parameter on validators like ", Code ("", [], []) "ToxicLanguage", Str " controls this trade-off."], Para [Strong [Str "Structured Output Reliability."], Str " Structured data generation depends on the LLM's ability to produce output conforming to the Pydantic schema. Smaller or less capable models may require more re-ask attempts. Function calling support in the underlying model significantly improves structured output reliability."], Para [Strong [Str "Server Concurrency."], Str " The Flask-based Guardrails Server does not inherently support high-concurrency workloads. Production deployments should use Gunicorn or Uvicorn with appropriate worker configuration. Guard history manipulation is not thread-safe, requiring careful tuning of worker threads versus processes."], Header 2 ("changelog-highlights", ["unnumbered", "unlisted"], []) [Str "Changelog Highlights"], BulletList [[Plain [Strong [Str "v0.9.0"], Str " (February 2026) -- Major release requiring migration from v0.8.x. Migration guide available at guardrailsai.com."]], [Plain [Strong [Str "v0.8.0"], Str " (February 2026) -- Introduced ", Code ("", [], []) "history_max_length", Str " parameter capping Guard history to 10 entries by default. Limits memory consumption in long-running applications."]], [Plain [Strong [Str "v0.8.1"], Str " (February 2026) -- Fixed custom validators failing without Hub API key. Resolved temperature handling issues. Removed previously deprecated methods (breaking change)."]], [Plain [Strong [Str "v0.7.0"], Str " (November 2025) -- Added LangChain 1.x support for framework interoperability."]], [Plain [Strong [Str "v0.7.3"], Str " (February 2026) -- Added OpenAI 2.x SDK support."]], [Plain [Strong [Str "v0.6.8"], Str " (November 2025) -- Added Python 3.13 support. Upgraded Click and Typer dependency versions."]], [Plain [Strong [Str "v0.6.7"], Str " (September 2025) -- Replaced deprecated ", Code ("", [], []) "pkg_resources", Str " with modern packaging alternatives."]], [Plain [Strong [Str "v0.5.0"], Str " (July 2024) -- Major milestone release establishing the Guard/Validator/Hub architecture."]], [Plain [Strong [Str "v0.1.0"], Str " (March 2023) -- Initial public release."]]], Para [Str "The project has been under active development since January 2023, with approximately 72 contributors and consistent release cadence averaging multiple minor releases per month."], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Link ("", [], []) [Str "Guardrails AI GitHub Repository"] ("https://github.com/guardrails-ai/guardrails", ""), Str " -- Source code, issues, and releases under Apache 2.0 license."]], [Plain [Link ("", [], []) [Str "Guardrails AI Official Documentation"] ("https://guardrailsai.com/docs", ""), Str " -- Concepts, API reference, how-to guides, and deployment documentation."]], [Plain [Link ("", [], []) [Str "Guardrails Hub"] ("https://guardrailsai.com/hub", ""), Str " -- Centralized repository of pre-built validators with documentation and installation instructions."]], [Plain [Link ("", [], []) [Str "Guardrails Index"] ("https://guardrailsai.com/", ""), Str " -- Benchmark comparing performance and latency of 24 guardrails across six common risk categories (launched February 2025)."]], [Plain [Link ("", [], []) [Str "guardrails-ai on PyPI"] ("https://pypi.org/project/guardrails-ai/", ""), Str " -- Package distribution with version history and dependency information."]], [Plain [Link ("", [], []) [Str "Guardrails AI Validator Template"] ("https://github.com/guardrails-ai/validator-template", ""), Str " -- Template repository for creating and submitting custom validators to the Hub."]]], Header 2 ("discovery-signals", ["unnumbered", "unlisted"], []) [Str "Discovery Signals"], BlockQuote [Para [Str "Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search."]], Header 3 ("keywords", ["unnumbered", "unlisted"], []) [Str "Keywords"], BulletList [[Plain [Str "guardrails ai"]], [Plain [Str "validator library"]], [Plain [Str "input output validation"]], [Plain [Str "guards"]], [Plain [Str "pydantic"]], [Plain [Str "on-fail action"]], [Plain [Str "reask"]], [Plain [Str "structured output"]], [Plain [Str "PII detection"]], [Plain [Str "toxic language"]], [Plain [Str "hallucination"]], [Plain [Str "competitor check"]], [Plain [Str "guardrails hub"]], [Plain [Str "re-ask loop"]], [Plain [Str "streaming validation"]], [Plain [Str "OpenAI-compatible server"]], [Plain [Str "Detoxify"]], [Plain [Str "Microsoft Presidio"]], [Plain [Str "LiteLLM integration"]], [Plain [Str "Apache 2.0"]], [Plain [Str "LLM middleware"]], [Plain [Str "jailbreak prevention"]], [Plain [Str "regex validator"]], [Plain [Str "async guard"]], [Plain [Str "FailResult"]]], Header 3 ("verb-noun-tasks", ["unnumbered", "unlisted"], []) [Str "Verb-Noun Tasks"], BulletList [[Plain [Str "Wrap an LLM call with input and output validators"]], [Plain [Str "Block toxic language in LLM responses"]], [Plain [Str "Mask PII in LLM outputs with Presidio"]], [Plain [Str "Enforce a Pydantic schema on LLM output"]], [Plain [Str "Re-ask the LLM when validation fails"]], [Plain [Str "Filter competitor mentions from generated text"]], [Plain [Str "Validate streaming LLM chunks in real time"]], [Plain [Str "Deploy a centralized guardrails server behind Gunicorn"]], [Plain [Str "Proxy OpenAI SDK calls through the Guardrails Server"]], [Plain [Str "Write a custom validator with register_validator"]], [Plain [Str "Install validators from the Guardrails Hub"]], [Plain [Str "Combine multiple validators into a single guard"]], [Plain [Str "Validate generated SQL or JSON before execution"]], [Plain [Str "Log failures with a NOOP on-fail action"]]], Header 3 ("user-intent-phrases", ["unnumbered", "unlisted"], []) [Str "User Intent Phrases"], BulletList [[Plain [Str "How do I add output validation to my OpenAI API calls in Python?"]], [Plain [Str "I need to redact emails and phone numbers from LLM responses."]], [Plain [Str "My LLM keeps mentioning competitors and I want to filter those out."]], [Plain [Str "How can I get the LLM to retry when its output fails schema validation?"]], [Plain [Str "Is there a drop-in OpenAI-compatible proxy that enforces content rules?"]], [Plain [Str "How do I extract structured data from an LLM with field-level validators?"]], [Plain [Str "Show me how to block toxic content using Detoxify in Python."]], [Plain [Str "I want a re-ask loop that fixes invalid JSON automatically."]], [Plain [Str "Can I run guardrails on streaming responses?"]], [Plain [Str "How do I create my own validator and register it with Guardrails?"]]], Header 3 ("problem-statements", ["unnumbered", "unlisted"], []) [Str "Problem Statements"], BulletList [[Plain [Str "LLMs return free-form text instead of conforming JSON schemas."]], [Plain [Str "Toxic or unsafe content leaks into user-facing responses."]], [Plain [Str "PII appears in LLM outputs and violates privacy compliance."]], [Plain [Str "Generated text mentions competitors against brand policy."]], [Plain [Str "No deterministic way to retry the model when validation fails."]], [Plain [Str "Validation logic is duplicated across services with no shared layer."]], [Plain [Str "Probabilistic ML validators produce false positives at default thresholds."]]], Header 3 ("when-to-pick-this", ["unnumbered", "unlisted"], []) [Str "When to Pick This"], BulletList [[Plain [Str "Pick this when you want a Python-first validator library with a re-ask loop and a hub of pre-built validators."]], [Plain [Str "Pick this over NeMo Guardrails when output structure and Pydantic schema enforcement matter more than Colang dialogue flows."]], [Plain [Str "Pick this over Lakera when you need open-source, self-hosted validation rather than a commercial API."]], [Plain [Str "Pick this over OpenAI Moderation when you need PII redaction, competitor filtering, and structured-output enforcement beyond harm-category classification."]], [Plain [Str "Pick this when you want to expose validation as an OpenAI-compatible HTTP server and route existing SDK clients through it."]], [Plain [Str "Pick this when on-fail actions (FIX, REFRAIN, FILTER, REASK) and per-field validation are central requirements."]]], Header 3 ("related-terms-and-aliases", ["unnumbered", "unlisted"], []) [Str "Related Terms and Aliases"], BulletList [[Plain [Str "Guardrails"]], [Plain [Str "guardrails-ai"]], [Plain [Str "LLM output validation"]], [Plain [Str "LLM input validation"]], [Plain [Str "Pydantic guardrails"]], [Plain [Str "structured output enforcement"]], [Plain [Str "re-ask pattern"]], [Plain [Str "content safety library"]], [Plain [Str "LLM middleware"]], [Plain [Str "prompt injection defense"]], [Plain [Str "jailbreak detection"]], [Plain [Str "hallucination filter"]]]]