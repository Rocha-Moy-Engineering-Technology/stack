[Header 1 ("dspy", [], []) [Str "DSPy"], BlockQuote [Para [Str "Stanford framework for programming LMs declaratively with automatic prompt optimization"]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, ColWidthDefault), (AlignDefault, ColWidthDefault)] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Name"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "DSPy"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Structured Generation"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "SDK"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Yes"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "stanfordnlp/dspy"] ("https://github.com/stanfordnlp/dspy", "")]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "34420"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Docs"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "dspy.ai"] ("https://dspy.ai/", "")]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "License"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "MIT"]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "DSPy replaces hand-written prompts and brittle string manipulation with a programming model for language models. Rather than crafting prompt templates, developers define typed signatures that specify input and output fields, compose them into modules, and let optimizers automatically tune the prompts and weights for a given metric. The framework treats LM calls as declarative operations, compiling high-level AI programs into efficient prompts or fine-tuning configurations. DSPy supports numerous LM providers through LiteLLM integration and delivers measurable accuracy improvements through its optimization pipeline."], Para [Str "The core philosophy is \"programming, not prompting.\" DSPy separates concerns that conventional prompting couples together: signature definitions (what), adapter formatting (how it is serialized), module logic (inference strategy), and optimization (automatic tuning). This separation enables language model swapping without logic changes, module substitution (for example, replacing ChainOfThought with ProgramOfThought), prompt optimization without architecture modification, and fine-tuning capabilities across programs."], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], Header 3 ("signatures", ["unnumbered", "unlisted"], []) [Str "Signatures"], Para [Str "Signatures are typed declarations of input and output fields for a language model call. They replace free-form prompt strings with structured contracts. DSPy supports two forms of signatures."], Para [Strong [Str "Inline signatures"], Str " use string notation with optional type specifications:"], CodeBlock ("", ["python"], []) "\"question -> answer\"                                    # basic string types
\"sentence -> sentiment: bool\"                           # typed output
\"context: list[str], question: str -> answer: str\"      # multiple typed fields
\"question, choices: list[str] -> reasoning: str, selection: int\"
", Para [Strong [Str "Class-based signatures"], Str " use Python class definitions with docstrings and field descriptors for complex tasks:"], CodeBlock ("", ["python"], []) "import dspy

class Assess(dspy.Signature):
    \"\"\"Assess the quality of a tweet along the specified dimension.\"\"\"
    assessed_text: str = dspy.InputField()
    assessment_question: str = dspy.InputField()
    assessment_answer: float = dspy.OutputField()
", Para [Str "Signatures define what the LM should do without prescribing how. The framework handles prompt formatting, parsing, and retry logic. Field names carry semantic meaning and are used by adapters to construct prompts. Supported types include basic Python types (str, int, bool, float), typing module constructs (list, dict, Optional, Union, Literal), custom Pydantic BaseModel classes, and special DSPy types (dspy.Image, dspy.History)."], Para [Str "InputField and OutputField accept a ", Code ("", [], []) "desc", Str " parameter that provides additional context to the language model about what the field represents."], Header 3 ("modules", ["unnumbered", "unlisted"], []) [Str "Modules"], Para [Str "Modules are composable building blocks that wrap signatures with specific inference strategies. Each module abstracts a prompting technique while maintaining generalizability across any signature. Modules contain learnable parameters (instructions, demonstrations) and can be composed into larger programs. The design draws inspiration from PyTorch's neural network architecture."], Para [Str "Built-in modules:"], BulletList [[Plain [Strong [Str "Predict"], Str " -- Direct signature invocation. The foundational module that handles basic prediction, managing instruction storage, demonstrations, and language model weight updates."]], [Plain [Strong [Str "ChainOfThought"], Str " -- Instructs the LM to think step-by-step before committing to the signature's response. Injects a reasoning field before output fields, improving output quality on complex tasks."]], [Plain [Strong [Str "ReAct"], Str " -- Reasoning and Acting agent module. Interleaves reasoning with tool use actions in an iterative loop, automatically selecting and calling tools until the task is complete. Accepts a ", Code ("", [], []) "max_iters", Str " parameter (default 20)."]], [Plain [Strong [Str "CodeAct"], Str " -- Generates and executes Python code snippets within a sandboxed interpreter, combining code generation with tool execution. Inherits from both ReAct and ProgramOfThought."]], [Plain [Strong [Str "ProgramOfThought"], Str " -- Directs the LM to generate executable code where execution results determine the final response."]], [Plain [Strong [Str "MultiChainComparison"], Str " -- Compares multiple ChainOfThought outputs to produce refined predictions."]], [Plain [Strong [Str "BestOfN"], Str " -- Generates N candidates and selects the best one according to a metric."]], [Plain [Strong [Str "Refine"], Str " -- Iteratively improves an output by reflecting on it and revising."]], [Plain [Strong [Str "Parallel"], Str " -- Runs multiple modules concurrently."]], [Plain [Strong [Str "RLM"], Str " -- Recursive language model for handling contexts too large for standard prompts."]], [Plain [Strong [Str "majority"], Str " -- A voting function returning the most popular response from multiple predictions."]]], Para [Str "Custom modules inherit from ", Code ("", [], []) "dspy.Module", Str " and compose other modules in their ", Code ("", [], []) "forward", Str " method:"], CodeBlock ("", ["python"], []) "class MultiHopSearch(dspy.Module):
    def __init__(self, num_docs=10, num_hops=4):
        self.generate_query = dspy.ChainOfThought(\"claim, notes -> query\")
        self.append_notes = dspy.ChainOfThought(\"claim, notes, context -> new_notes\")

    def forward(self, claim: str) -> list[str]:
        notes = \"No notes yet.\"
        for hop in range(self.num_hops):
            query = self.generate_query(claim=claim, notes=notes).query
            context = search(query, k=self.num_docs)
            notes = self.append_notes(claim=claim, notes=notes, context=context).new_notes
        return notes
", Header 3 ("adapters", ["unnumbered", "unlisted"], []) [Str "Adapters"], Para [Str "Adapters serve as the connection layer between dspy.Predict and language models. They translate DSPy signatures into system messages, format input data, parse LM responses into dspy.Prediction instances, manage conversation history, and convert DSPy types (Tools, Images) into prompt messages."], Para [Str "Built-in adapters:"], BulletList [[Plain [Strong [Str "ChatAdapter"], Str " (default) -- Uses ", Code ("", [], []) "[[ ## field_name ## ]]", Str " markers to delineate fields. Universally compatible across all language models. Includes fallback protection that automatically retries with JSONAdapter if parsing fails. More verbose in output tokens."]], [Plain [Strong [Str "JSONAdapter"], Str " -- Outputs structured as pure JSON objects leveraging native model capabilities via the ", Code ("", [], []) "response_format", Str " parameter. Lower latency and minimal boilerplate, but incompatible with models lacking native structured output support."]]], Para [Str "Configure adapters globally or per-context:"], CodeBlock ("", ["python"], []) "dspy.configure(lm=dspy.LM(\"openai/gpt-4o-mini\"), adapter=dspy.ChatAdapter())

with dspy.context(adapter=dspy.JSONAdapter()):
    result = program(question=\"...\")
", Header 3 ("optimizers", ["unnumbered", "unlisted"], []) [Str "Optimizers"], Para [Str "Optimizers automatically improve program performance against a defined metric by tuning prompts, few-shot examples, or model weights. DSPy recommends allocating 20% of data for training and 80% for validation with prompt-based optimizers, as they tend to overfit on small training sets."], Para [Str "Available optimizers:"], BulletList [[Plain [Strong [Str "BootstrapFewShot"], Str " -- Generates few-shot demonstrations by running the program on training examples and keeping successful traces."]], [Plain [Strong [Str "BootstrapRS"], Str " (alias for BootstrapFewShotWithRandomSearch) -- Random search over bootstrapped few-shot demonstrations with multiple candidate programs."]], [Plain [Strong [Str "BootstrapFinetune"], Str " -- Uses bootstrapped demonstrations to fine-tune model weights rather than optimize prompts."]], [Plain [Strong [Str "MIPROv2"], Str " -- Multi-prompt Instruction Proposal Optimizer. Jointly optimizes instructions and few-shot examples using Bayesian Optimization across three stages: bootstrap demonstrations, propose instruction candidates, then search for optimal combinations. Supports ", Code ("", [], []) "auto", Str " modes: light, medium, heavy."]], [Plain [Strong [Str "COPRO"], Str " -- Coordinate-based prompt optimization."]], [Plain [Strong [Str "GEPA"], Str " -- Genetic-Pareto reflective optimizer that adaptively evolves textual components of arbitrary systems. Maintains a Pareto frontier of candidates, uses LLM-driven reflection on execution traces to propose targeted improvements, and accepts both scalar scores and textual feedback. Supports integration with Weights & Biases and MLflow."]], [Plain [Strong [Str "SIMBA"], Str " -- Optimization strategy for complex programs."]], [Plain [Strong [Str "BetterTogether"], Str " -- Joint optimization of prompts and weights together."]], [Plain [Strong [Str "InferRules"], Str " -- Infers optimization rules from successful traces."]], [Plain [Strong [Str "LabeledFewShot"], Str " -- Uses labeled examples directly as demonstrations."]], [Plain [Strong [Str "KNN/KNNFewShot"], Str " -- k-nearest neighbor selection of demonstrations at inference time."]], [Plain [Strong [Str "Ensemble"], Str " -- Combines multiple optimized programs."]]], Header 3 ("examples-and-datasets", ["unnumbered", "unlisted"], []) [Str "Examples and Datasets"], Para [Str "The ", Code ("", [], []) "dspy.Example", Str " class is a flexible data container for training and evaluation data. It supports dictionary-like access, input/output field separation, and serialization:"], CodeBlock ("", ["python"], []) "example = dspy.Example(question=\"What is DSPy?\", answer=\"A framework for LM programming\")
example = example.with_inputs(\"question\")

inputs = example.inputs()    # only question
labels = example.labels()    # only answer
", Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Para [Str "DSPy follows a Define, Evaluate, Compile, Deploy workflow:"], OrderedList (1, DefaultStyle, DefaultDelim) [[Plain [Strong [Str "Define"], Str " -- Write signatures and compose modules into a program. Identify system inputs and desired outputs. Start simple with a single module, then add complexity incrementally."]], [Plain [Strong [Str "Evaluate"], Str " -- Measure program accuracy on a development dataset (20-200+ examples) using a metric function. Metrics range from simple accuracy to complex DSPy programs that verify multiple output properties."]], [Plain [Strong [Str "Compile"], Str " -- Run an optimizer that searches for better prompts, demonstrations, or weights. The optimizer systematically explores the space of possible prompts and demonstrations, guided by the evaluation metric."]], [Plain [Strong [Str "Deploy"], Str " -- Use the compiled program with the tuned configuration in production via FastAPI or MLflow."]]], CodeBlock ("", [""], []) "Signature --> Module --> Program --> Optimizer --> Compiled Program
    ^                                   ^
  Types                              Metric
  Fields                             Dataset
", Para [Str "The compilation step differentiates DSPy from standard prompt engineering. Instead of manually iterating on prompt text, the optimizer systematically explores the search space guided by the evaluation metric."], Header 2 ("key-features", ["unnumbered", "unlisted"], []) [Str "Key Features"], BulletList [[Plain [Strong [Str "Declarative LM programming"], Str " -- Define what the model should compute via typed signatures rather than how via prompt strings."]], [Plain [Strong [Str "Automatic prompt optimization"], Str " -- Optimizers tune prompts and few-shot examples to maximize a user-defined metric. MIPROv2 jointly optimizes instructions and demonstrations using Bayesian Optimization. GEPA uses genetic evolution with LLM-driven reflection."]], [Plain [Strong [Str "Composable modules"], Str " -- Chain, nest, and reuse modules like standard software components using Python control flow."]], [Plain [Strong [Str "Provider agnostic"], Str " -- Supports numerous LM providers through LiteLLM integration: OpenAI, Anthropic, Google Gemini, Vertex AI, Databricks, SGLang, Ollama, Azure, AWS SageMaker, Together AI, Anyscale, and more."]], [Plain [Strong [Str "Typed input/output"], Str " -- Signatures enforce structured contracts with support for str, int, bool, float, list, dict, Literal, Pydantic models, dspy.Image, and dspy.History."]], [Plain [Strong [Str "Assertion and constraint system"], Str " -- ", Code ("", [], []) "dspy.Assert", Str " raises a hard failure when a constraint is not met. ", Code ("", [], []) "dspy.Suggest", Str " provides a soft signal the optimizer can use during compilation to improve outputs."]], [Plain [Strong [Str "Automatic few-shot bootstrapping"], Str " -- Generates high-quality demonstrations from training data without manual curation."]], [Plain [Strong [Str "Tool integration"], Str " -- ReAct and CodeAct modules support external tool use. Native function calling via adapters. MCP (Model Context Protocol) integration for standardized tool discovery via ", Code ("", [], []) "dspy.Tool.from_mcp_tool()", Str "."]], [Plain [Strong [Str "Async and streaming"], Str " -- Native ", Code ("", [], []) "acall()", Str " async execution on most modules. ", Code ("", [], []) "dspy.streamify()", Str " for real-time token streaming and intermediate status updates. ", Code ("", [], []) "dspy.asyncify()", Str " for running sync programs in thread pools."]], [Plain [Strong [Str "Thread-safe configuration"], Str " -- ", Code ("", [], []) "dspy.configure()", Str " and ", Code ("", [], []) "dspy.context()", Str " are thread-safe. Track usage statistics with ", Code ("", [], []) "dspy.configure(track_usage=True)", Str "."]], [Plain [Strong [Str "Caching"], Str " -- LM calls are cached by default. Bypass with ", Code ("", [], []) "cache=False", Str " or ", Code ("", [], []) "rollout_id", Str " parameter."]], [Plain [Strong [Str "Reproducible optimization"], Str " -- Compilation produces deterministic, serializable configurations. Save/load programs as JSON or pickle."]], [Plain [Strong [Str "Responses API"], Str " -- Support for models with enhanced reasoning via ", Code ("", [], []) "model_type=\"responses\"", Str "."]]], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], BulletList [[Plain [Strong [Str "Multi-hop question answering"], Str " -- Compose retrieval and reasoning modules to answer questions requiring multiple evidence steps. Reported improvements from 24% to 51% on HotPotQA with ReAct optimization."]], [Plain [Strong [Str "Classification pipelines"], Str " -- Build typed classifiers with automatic few-shot optimization. Reported improvements from 66% to 87% accuracy."]], [Plain [Strong [Str "Agentic workflows"], Str " -- Use ReAct modules for tool-augmented reasoning with automatic tool selection and error recovery. CodeAct for code-generation-based agents."]], [Plain [Strong [Str "Information extraction"], Str " -- Define output signatures with structured fields for entity and relation extraction. Supports Literal type constraints for categorical outputs."]], [Plain [Strong [Str "RAG systems"], Str " -- Combine retrieval modules with generation modules, optimizing the full pipeline end-to-end including retrieval quality."]], [Plain [Strong [Str "Data labeling and assessment"], Str " -- Use typed output fields (including floats and enums) for structured scoring tasks."]], [Plain [Strong [Str "Customer service agents"], Str " -- Build tool-using agents with MCP integration for database access, booking systems, and ticket management."]], [Plain [Strong [Str "Image and audio processing"], Str " -- Multi-modal support via dspy.Image and dspy.Audio types in signatures."]], [Plain [Strong [Str "Privacy-conscious delegation"], Str " -- PAPILLON pattern for delegating tasks while preserving privacy constraints."]]], Header 2 ("api-reference", ["unnumbered", "unlisted"], []) [Str "API Reference"], Header 3 ("language-model-configuration", ["unnumbered", "unlisted"], []) [Str "Language Model Configuration"], CodeBlock ("", ["python"], []) "import dspy

# Configure default LM with parameters
lm = dspy.LM(\"openai/gpt-4o-mini\", temperature=0.7, max_tokens=3000, cache=True)
dspy.configure(lm=lm)

# Anthropic
lm = dspy.LM(\"anthropic/claude-sonnet-4-20250514\")

# Google Gemini
lm = dspy.LM(\"gemini/gemini-2.0-flash\", api_key=\"GEMINI_API_KEY\")

# Local via Ollama
lm = dspy.LM(\"ollama_chat/llama3.2\", api_base=\"http://localhost:11434\")

# Responses API for enhanced reasoning
lm = dspy.LM(\"openai/gpt-5-mini\", model_type=\"responses\", temperature=1.0)

# Direct LM calls
lm(\"Say this is a test!\", temperature=0.7)
lm(messages=[{\"role\": \"user\", \"content\": \"Say this is a test!\"}])

# Access history and metadata
len(lm.history)
lm.history[-1]  # prompt, messages, kwargs, response, outputs, usage, cost, timestamp
", Header 3 ("signatures-1", ["unnumbered", "unlisted"], []) [Str "Signatures"], CodeBlock ("", ["python"], []) "# Inline notation
\"question -> answer\"
\"context, question -> answer\"
\"question -> answer: float\"
\"sentence -> sentiment: bool\"

# Class-based with field descriptors
class Summarize(dspy.Signature):
    \"\"\"Summarize the document in one sentence.\"\"\"
    document: str = dspy.InputField(desc=\"The document to summarize\")
    summary: str = dspy.OutputField(desc=\"A one-sentence summary\")

# With Literal type constraints
class ClassifyEmotion(dspy.Signature):
    text: str = dspy.InputField()
    emotion: Literal[\"joy\", \"sadness\", \"anger\", \"fear\"] = dspy.OutputField()

# With Pydantic models
class ExtractedEntity(BaseModel):
    name: str
    entity_type: str

class ExtractEntities(dspy.Signature):
    text: str = dspy.InputField()
    entities: list[ExtractedEntity] = dspy.OutputField()
", Header 3 ("modules-1", ["unnumbered", "unlisted"], []) [Str "Modules"], CodeBlock ("", ["python"], []) "# Basic prediction
predict = dspy.Predict(Summarize)
result = predict(document=\"...\")

# Chain of thought
cot = dspy.ChainOfThought(\"question -> answer\")
result = cot(question=\"What is the capital of France?\")
print(result.reasoning)  # intermediate reasoning
print(result.answer)     # final answer

# ReAct with tools
def get_weather(city: str) -> str:
    \"\"\"Get weather for a city.\"\"\"
    return f\"Sunny in {city}\"

react = dspy.ReAct(\"question -> answer\", tools=[get_weather], max_iters=5)
result = react(question=\"What is the weather in Tokyo?\")

# CodeAct with sandboxed execution
act = dspy.CodeAct(\"n -> factorial_result\", tools=[factorial], max_iters=5)
result = act(n=5)

# Custom module
class RAGModule(dspy.Module):
    def __init__(self, num_passages=3):
        self.retrieve = dspy.Retrieve(k=num_passages)
        self.generate = dspy.ChainOfThought(\"context, question -> answer\")

    def forward(self, question):
        context = self.retrieve(question).passages
        return self.generate(context=context, question=question)
", Header 3 ("evaluation", ["unnumbered", "unlisted"], []) [Str "Evaluation"], CodeBlock ("", ["python"], []) "def answer_exact_match(example, prediction, trace=None):
    return example.answer.lower() == prediction.answer.lower()

evaluate = dspy.Evaluate(
    devset=dev_examples,
    metric=answer_exact_match,
    num_threads=4,
    display_progress=True
)
score = evaluate(program)
", Para [Str "Built-in metrics: ", Code ("", [], []) "answer_exact_match", Str ", ", Code ("", [], []) "answer_passage_match", Str ", ", Code ("", [], []) "SemanticF1", Str ", ", Code ("", [], []) "CompleteAndGrounded", Str "."], Header 3 ("optimization", ["unnumbered", "unlisted"], []) [Str "Optimization"], CodeBlock ("", ["python"], []) "# MIPROv2 for instruction and demonstration optimization
optimizer = dspy.MIPROv2(
    metric=answer_exact_match,
    auto=\"medium\",              # light, medium, or heavy
    max_bootstrapped_demos=4,
    max_labeled_demos=4,
    verbose=True
)
compiled_program = optimizer.compile(
    program,
    trainset=train_examples,
    valset=val_examples
)

# BootstrapRS for few-shot demonstration search
optimizer = dspy.BootstrapRS(
    metric=answer_exact_match,
    max_bootstrapped_demos=4,
    num_candidate_programs=10
)
compiled_program = optimizer.compile(program, trainset=train_examples)

# GEPA for reflective prompt evolution
optimizer = dspy.teleprompt.GEPA(
    metric=metric_with_feedback,
    reflection_lm=dspy.LM(\"openai/gpt-4o\"),
    auto=\"medium\",
    log_dir=\"./gepa_logs\"
)
compiled_program = optimizer.compile(program, trainset=train_examples, valset=val_examples)
", Header 3 ("tools-and-mcp", ["unnumbered", "unlisted"], []) [Str "Tools and MCP"], CodeBlock ("", ["python"], []) "# Define tools
def search_wikipedia(query: str) -> str:
    \"\"\"Search Wikipedia for information.\"\"\"
    return result

tool = dspy.Tool(search_wikipedia)
print(tool.name)    # function name
print(tool.desc)    # docstring
print(tool.args)    # parameter schema

# Manual tool handling with ToolCalls
class ToolSignature(dspy.Signature):
    question: str = dspy.InputField()
    tools: list[dspy.Tool] = dspy.InputField()
    outputs: dspy.ToolCalls = dspy.OutputField()

# Native function calling via adapter
chat_adapter = dspy.ChatAdapter(use_native_function_calling=True)
dspy.configure(adapter=chat_adapter)

# MCP integration
from mcp import ClientSession, StdioServerParameters
dspy_tools = [dspy.Tool.from_mcp_tool(session, tool) for tool in mcp_tools]
react = dspy.ReAct(signature, tools=dspy_tools)
result = await react.acall(user_request=\"...\")
", Header 3 ("saving-and-loading", ["unnumbered", "unlisted"], []) [Str "Saving and Loading"], CodeBlock ("", ["python"], []) "# State-only saving (JSON, recommended)
compiled_program.save(\"optimized_program.json\", save_program=False)
program = RAGModule()
program.load(\"optimized_program.json\")

# Whole program saving (dspy >= 2.6.0)
compiled_program.save(\"./dspy_program/\", save_program=True)
loaded = dspy.load(\"./dspy_program/\")

# With custom module serialization
program.save(\"./path/\", save_program=True, modules_to_serialize=[custom_module])
", Header 2 ("configuration", ["unnumbered", "unlisted"], []) [Str "Configuration"], Header 3 ("global-settings", ["unnumbered", "unlisted"], []) [Str "Global Settings"], CodeBlock ("", ["python"], []) "dspy.configure(
    lm=dspy.LM(\"openai/gpt-4o-mini\"),
    rm=dspy.ColBERTv2(url=\"http://localhost:8893\"),  # retrieval model
    adapter=dspy.ChatAdapter(),
    trace=[],
    track_usage=True
)
", Header 3 ("per-call-overrides", ["unnumbered", "unlisted"], []) [Str "Per-Call Overrides"], CodeBlock ("", ["python"], []) "with dspy.context(lm=dspy.LM(\"anthropic/claude-sonnet-4-20250514\")):
    result = program(question=\"...\")

# Module-level configuration
predict = dspy.Predict(\"question -> answer\", temperature=1.0)
predict(question=\"...\", config={\"rollout_id\": 5, \"temperature\": 0.5})
", Header 3 ("assertions-and-constraints", ["unnumbered", "unlisted"], []) [Str "Assertions and Constraints"], CodeBlock ("", ["python"], []) "class FactCheckedAnswer(dspy.Module):
    def __init__(self):
        self.generate = dspy.ChainOfThought(\"question -> answer\")

    def forward(self, question):
        result = self.generate(question=question)
        dspy.Assert(
            len(result.answer) > 10,
            \"Answer must be substantive (more than 10 characters)\"
        )
        dspy.Suggest(
            \"citation\" in result.answer.lower(),
            \"Answer should include citations\"
        )
        return result
", Header 3 ("caching", ["unnumbered", "unlisted"], []) [Str "Caching"], Para [Str "LM calls are cached by default. Control caching behavior:"], CodeBlock ("", ["python"], []) "lm = dspy.LM(\"openai/gpt-4o-mini\", cache=False)           # disable entirely
predict(question=\"...\", config={\"rollout_id\": 5})           # bypass specific cache entry
dspy.configure_cache(enable=False)                          # global disable
", Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Header 3 ("with-retrieval-systems", ["unnumbered", "unlisted"], []) [Str "With Retrieval Systems"], CodeBlock ("", ["python"], []) "import dspy

colbert = dspy.ColBERTv2(url=\"http://localhost:8893\")
dspy.configure(rm=colbert)

class SearchAndAnswer(dspy.Module):
    def __init__(self):
        self.retrieve = dspy.Retrieve(k=5)
        self.answer = dspy.ChainOfThought(\"context, question -> answer\")

    def forward(self, question):
        passages = self.retrieve(question).passages
        return self.answer(context=passages, question=question)
", Header 3 ("with-custom-tools-via-react", ["unnumbered", "unlisted"], []) [Str "With Custom Tools via ReAct"], CodeBlock ("", ["python"], []) "def search_wikipedia(query: str) -> str:
    \"\"\"Search Wikipedia for information.\"\"\"
    return result

def calculate(expression: str) -> float:
    \"\"\"Evaluate a math expression.\"\"\"
    return eval(expression)

react = dspy.ReAct(
    \"question -> answer\",
    tools=[search_wikipedia, calculate],
    max_iters=10
)
result = react(question=\"What is the population of France times 2?\")
", Header 3 ("pipeline-composition", ["unnumbered", "unlisted"], []) [Str "Pipeline Composition"], CodeBlock ("", ["python"], []) "class MultiStepPipeline(dspy.Module):
    def __init__(self):
        self.extract = dspy.Predict(\"document -> entities: list[str]\")
        self.classify = dspy.ChainOfThought(\"entity, context -> category\")
        self.summarize = dspy.Predict(\"entities, categories -> summary\")

    def forward(self, document):
        entities = self.extract(document=document).entities
        categories = [
            self.classify(entity=e, context=document).category
            for e in entities
        ]
        return self.summarize(entities=entities, categories=categories)
", Header 3 ("fastapi-deployment", ["unnumbered", "unlisted"], []) [Str "FastAPI Deployment"], CodeBlock ("", ["python"], []) "from fastapi import FastAPI
import dspy

app = FastAPI()
program = dspy.ChainOfThought(\"question -> answer\")
program.load(\"optimized.json\")

async_program = dspy.asyncify(program)

@app.post(\"/predict\")
async def predict(question: str):
    result = await async_program(question=question)
    return {\"answer\": result.answer}
", Header 3 ("streaming", ["unnumbered", "unlisted"], []) [Str "Streaming"], CodeBlock ("", ["python"], []) "import dspy

predict = dspy.Predict(\"question -> answer\")
stream_predict = dspy.streamify(
    predict,
    stream_listeners=[dspy.streaming.StreamListener(signature_field_name=\"answer\")]
)

# Async streaming
async for chunk in stream_predict(question=\"Why?\"):
    print(chunk)

# Synchronous streaming
stream_predict = dspy.streamify(predict, stream_listeners=[...], async_streaming=False)
for chunk in stream_predict(question=\"Why?\"):
    print(chunk)
", Header 3 ("mlflow-deployment", ["unnumbered", "unlisted"], []) [Str "MLflow Deployment"], CodeBlock ("", ["python"], []) "import mlflow
import dspy

class MyProgram(dspy.Module):
    def forward(self, question):
        cot = dspy.ChainOfThought(\"question -> answer\")
        return cot(question=question)

with mlflow.start_run():
    mlflow.dspy.log_model(MyProgram(), \"model\", task=\"llm/v1/chat\")
# Serve: mlflow models serve -m runs:/{run_id}/model -p 6000
", Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Header 3 ("optimized-classification", ["unnumbered", "unlisted"], []) [Str "Optimized Classification"], CodeBlock ("", ["python"], []) "import dspy

class ClassifyIntent(dspy.Signature):
    \"\"\"Classify the user message into an intent category.\"\"\"
    message: str = dspy.InputField()
    intent: str = dspy.OutputField(
        desc=\"One of: greeting, question, complaint, feedback\"
    )

classifier = dspy.Predict(ClassifyIntent)

def intent_match(example, prediction, trace=None):
    return example.intent == prediction.intent

optimizer = dspy.MIPROv2(metric=intent_match, auto=\"light\")
optimized = optimizer.compile(classifier, trainset=train_data)

result = optimized(message=\"I'm having trouble with my order\")
print(result.intent)
", Header 3 ("multi-hop-rag-with-optimization", ["unnumbered", "unlisted"], []) [Str "Multi-Hop RAG with Optimization"], CodeBlock ("", ["python"], []) "import dspy

class MultiHopRAG(dspy.Module):
    def __init__(self, passages_per_hop=3, num_hops=2):
        self.retrieve = [dspy.Retrieve(k=passages_per_hop) for _ in range(num_hops)]
        self.generate_query = dspy.ChainOfThought(\"context, question -> search_query\")
        self.generate_answer = dspy.ChainOfThought(\"context, question -> answer\")

    def forward(self, question):
        context = []
        for hop in range(len(self.retrieve)):
            if hop == 0:
                passages = self.retrieve[hop](question).passages
            else:
                query = self.generate_query(
                    context=context, question=question
                ).search_query
                passages = self.retrieve[hop](query).passages
            context = deduplicate(context + passages)
        return self.generate_answer(context=context, question=question)

optimizer = dspy.MIPROv2(metric=answer_f1, auto=\"medium\")
compiled_rag = optimizer.compile(MultiHopRAG(), trainset=train_examples)
", Header 3 ("react-agent-with-mcp-tools", ["unnumbered", "unlisted"], []) [Str "ReAct Agent with MCP Tools"], CodeBlock ("", ["python"], []) "import dspy
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

class AirlineAgent(dspy.Signature):
    \"\"\"You are an airline customer service agent with access to tools.\"\"\"
    user_request: str = dspy.InputField()
    process_result: str = dspy.OutputField(
        desc=\"Summary with confirmation numbers or relevant info\"
    )

async def build_agent():
    server_params = StdioServerParameters(command=\"python\", args=[\"mcp_server.py\"])
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(read, write) as session:
            await session.initialize()
            tools = await session.list_tools()
            dspy_tools = [dspy.Tool.from_mcp_tool(session, t) for t in tools.tools]

            dspy.configure(lm=dspy.LM(\"openai/gpt-4o-mini\"))
            react = dspy.ReAct(AirlineAgent, tools=dspy_tools)
            result = await react.acall(user_request=\"Book a flight from SFO to JFK\")
            return result
", Header 3 ("structured-assessment", ["unnumbered", "unlisted"], []) [Str "Structured Assessment"], CodeBlock ("", ["python"], []) "import dspy

class TweetAssessment(dspy.Signature):
    \"\"\"Assess tweet quality on a numeric scale.\"\"\"
    tweet: str = dspy.InputField()
    dimension: str = dspy.InputField(desc=\"The quality dimension to assess\")
    score: float = dspy.OutputField(desc=\"Quality score from 0.0 to 1.0\")

assessor = dspy.ChainOfThought(TweetAssessment)
result = assessor(
    tweet=\"DSPy lets you program LMs declaratively!\",
    dimension=\"informativeness\"
)
print(f\"Score: {result.score}\")
", Header 2 ("limitations", ["unnumbered", "unlisted"], []) [Str "Limitations"], BulletList [[Plain [Strong [Str "Optimization cost"], Str " -- Compiling programs requires LM calls over the training set, incurring API costs and wall-clock time. MIPROv2 with auto=\"medium\" runs many trials of Bayesian Optimization."]], [Plain [Strong [Str "Dataset requirement"], Str " -- Optimizers need labeled examples to tune against. Cold-start scenarios with no evaluation data cannot leverage compilation. Recommended minimum of 20-200 input examples for evaluation."]], [Plain [Strong [Str "Debugging complexity"], Str " -- Compiled programs with optimized prompts and demonstrations can be harder to inspect and debug than hand-written prompts. Use ", Code ("", [], []) "dspy.inspect_history()", Str " and MLflow tracing for observability."]], [Plain [Strong [Str "Provider-specific behavior"], Str " -- While DSPy abstracts across providers, underlying model differences can cause optimized programs to transfer poorly between LMs. Native function calling support varies by provider."]], [Plain [Strong [Str "Learning curve"], Str " -- The programming model (signatures, modules, optimizers, adapters) introduces concepts that differ from conventional prompt engineering workflows."]], [Plain [Strong [Str "Non-determinism"], Str " -- LM outputs are inherently stochastic. Optimization results may vary across runs depending on training data sampling and random seeds."]], [Plain [Strong [Str "CodeAct limitations"], Str " -- Only accepts pure functions (not callable objects). Tools cannot depend on external packages. All function dependencies must be explicitly passed."]], [Plain [Strong [Str "Backward compatibility"], Str " -- Saving and loading programs across different DSPy versions is not yet supported. Use identical versions until 3.0.0 introduces guaranteed major-version compatibility."]], [Plain [Strong [Str "Async complexity"], Str " -- While DSPy supports native async, mixing sync and async tools requires explicit context management (", Code ("", [], []) "allow_tool_async_sync_conversion", Str "). Async involves more complex error handling and debugging."]]], Header 2 ("changelog", ["unnumbered", "unlisted"], []) [Str "Changelog"], BulletList [[Plain [Str "DSPy 2.0 introduced the current signature and module system, replacing the earlier template-based approach"]], [Plain [Str "DSPy 2.6.0 added whole-program saving with ", Code ("", [], []) "save_program=True", Str " and streaming support via ", Code ("", [], []) "dspy.streamify()"]], [Plain [Str "Adapter system added for structured output mapping across providers (ChatAdapter as default, JSONAdapter for native structured output)"]], [Plain [Str "MIPROv2 optimizer added for joint instruction and demonstration optimization using Bayesian Optimization"]], [Plain [Str "GEPA optimizer introduced for reflective prompt evolution using genetic-Pareto strategies (Agrawal et al., 2025)"]], [Plain [Str "BetterTogether optimizer added for joint prompt and weight tuning"]], [Plain [Str "CodeAct module added combining code generation with sandboxed execution"]], [Plain [Str "BestOfN and Refine modules added for output quality improvement"]], [Plain [Str "MCP (Model Context Protocol) integration added via ", Code ("", [], []) "dspy.Tool.from_mcp_tool()", Str " for standardized tool discovery"]], [Plain [Str "Native function calling support added to ChatAdapter and JSONAdapter"]], [Plain [Str "Responses API support added for models with enhanced reasoning capabilities"]], [Plain [Str "MLflow integration for production deployment, tracing, and experiment tracking"]], [Plain [Str "FastAPI deployment guide with async support and streaming endpoints"]], [Plain [Str "Assertion and suggestion system introduced for runtime constraint enforcement"]]], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " ", Link ("", [], []) [Str "DSPy Documentation - Home"] ("https://dspy.ai/", "")]], [Plain [Str "[", Str "2", Str "]", Str " ", Link ("", [], []) [Str "DSPy Programming Overview"] ("https://dspy.ai/learn/programming/overview/", "")]], [Plain [Str "[", Str "3", Str "]", Str " ", Link ("", [], []) [Str "DSPy Language Models"] ("https://dspy.ai/learn/programming/language_models/", "")]], [Plain [Str "[", Str "4", Str "]", Str " ", Link ("", [], []) [Str "DSPy Signatures"] ("https://dspy.ai/learn/programming/signatures/", "")]], [Plain [Str "[", Str "5", Str "]", Str " ", Link ("", [], []) [Str "DSPy Modules"] ("https://dspy.ai/learn/programming/modules/", "")]], [Plain [Str "[", Str "6", Str "]", Str " ", Link ("", [], []) [Str "DSPy Adapters"] ("https://dspy.ai/learn/programming/adapters/", "")]], [Plain [Str "[", Str "7", Str "]", Str " ", Link ("", [], []) [Str "DSPy Tools"] ("https://dspy.ai/learn/programming/tools/", "")]], [Plain [Str "[", Str "8", Str "]", Str " ", Link ("", [], []) [Str "DSPy Evaluation Overview"] ("https://dspy.ai/learn/evaluation/overview/", "")]], [Plain [Str "[", Str "9", Str "]", Str " ", Link ("", [], []) [Str "DSPy Optimization Overview"] ("https://dspy.ai/learn/optimization/overview/", "")]], [Plain [Str "[", Str "10", Str "]", Str " ", Link ("", [], []) [Str "DSPy API Reference"] ("https://dspy.ai/api/", "")]], [Plain [Str "[", Str "11", Str "]", Str " ", Link ("", [], []) [Str "DSPy Tutorials"] ("https://dspy.ai/tutorials/", "")]], [Plain [Str "[", Str "12", Str "]", Str " ", Link ("", [], []) [Str "DSPy ReAct API"] ("https://dspy.ai/api/modules/ReAct/", "")]], [Plain [Str "[", Str "13", Str "]", Str " ", Link ("", [], []) [Str "DSPy CodeAct API"] ("https://dspy.ai/api/modules/CodeAct/", "")]], [Plain [Str "[", Str "14", Str "]", Str " ", Link ("", [], []) [Str "DSPy MIPROv2 API"] ("https://dspy.ai/api/optimizers/MIPROv2/", "")]], [Plain [Str "[", Str "15", Str "]", Str " ", Link ("", [], []) [Str "DSPy GEPA Overview"] ("https://dspy.ai/api/optimizers/GEPA/overview/", "")]], [Plain [Str "[", Str "16", Str "]", Str " ", Link ("", [], []) [Str "DSPy Production"] ("https://dspy.ai/production/", "")]], [Plain [Str "[", Str "17", Str "]", Str " ", Link ("", [], []) [Str "DSPy Saving and Loading"] ("https://dspy.ai/tutorials/saving/", "")]], [Plain [Str "[", Str "18", Str "]", Str " ", Link ("", [], []) [Str "DSPy Streaming"] ("https://dspy.ai/tutorials/streaming/", "")]], [Plain [Str "[", Str "19", Str "]", Str " ", Link ("", [], []) [Str "DSPy Deployment"] ("https://dspy.ai/tutorials/deployment/", "")]], [Plain [Str "[", Str "20", Str "]", Str " ", Link ("", [], []) [Str "DSPy MCP Integration"] ("https://dspy.ai/tutorials/mcp/", "")]], [Plain [Str "[", Str "21", Str "]", Str " ", Link ("", [], []) [Str "DSPy Async"] ("https://dspy.ai/tutorials/async/", "")]], [Plain [Str "[", Str "22", Str "]", Str " ", Link ("", [], []) [Str "DSPy Example API"] ("https://dspy.ai/api/primitives/Example/", "")]]], Header 2 ("discovery-signals", ["unnumbered", "unlisted"], []) [Str "Discovery Signals"], BlockQuote [Para [Str "Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search."]], Header 3 ("keywords", ["unnumbered", "unlisted"], []) [Str "Keywords"], BulletList [[Plain [Str "DSPy"]], [Plain [Str "Stanford NLP"]], [Plain [Str "programming not prompting"]], [Plain [Str "declarative LM programming"]], [Plain [Str "signatures"]], [Plain [Str "modules"]], [Plain [Str "Predict"]], [Plain [Str "ChainOfThought"]], [Plain [Str "ReAct"]], [Plain [Str "CodeAct"]], [Plain [Str "ProgramOfThought"]], [Plain [Str "BestOfN"]], [Plain [Str "Refine"]], [Plain [Str "adapters"]], [Plain [Str "ChatAdapter"]], [Plain [Str "JSONAdapter"]], [Plain [Str "optimizers"]], [Plain [Str "MIPROv2"]], [Plain [Str "GEPA"]], [Plain [Str "BootstrapFewShot"]], [Plain [Str "BetterTogether"]], [Plain [Str "typed input/output"]], [Plain [Str "LiteLLM"]], [Plain [Str "compile programs"]], [Plain [Str "dspy.Assert"]], [Plain [Str "dspy.Suggest"]], [Plain [Str "few-shot bootstrapping"]], [Plain [Str "prompt optimization"]]], Header 3 ("verb-noun-tasks", ["unnumbered", "unlisted"], []) [Str "Verb-Noun Tasks"], BulletList [[Plain [Str "Declare a typed signature like ", Code ("", [], []) "\"question -> answer\"", Str " instead of writing prompts"]], [Plain [Str "Compose modules (Predict, ChainOfThought, ReAct) into an LM program"]], [Plain [Str "Compile a program with MIPROv2 to optimize instructions and few-shot examples"]], [Plain [Str "Run GEPA reflective prompt evolution against a metric and Pareto frontier"]], [Plain [Str "Swap LMs without changing program logic via ", Code ("", [], []) "dspy.configure(lm=...)"]], [Plain [Str "Build a ReAct agent that calls tools or MCP servers"]], [Plain [Str "Generate executable Python via CodeAct in a sandboxed interpreter"]], [Plain [Str "Stream tokens or intermediate fields via ", Code ("", [], []) "dspy.streamify"]], [Plain [Str "Evaluate a program on a dev set with a metric function"]], [Plain [Str "Save and load compiled programs as JSON or full programs"]], [Plain [Str "Build a multi-hop RAG pipeline with end-to-end optimization"]], [Plain [Str "Enforce runtime constraints with ", Code ("", [], []) "dspy.Assert", Str " and ", Code ("", [], []) "dspy.Suggest"]]], Header 3 ("user-intent-phrases", ["unnumbered", "unlisted"], []) [Str "User Intent Phrases"], BulletList [[Plain [Str "How do I stop hand-writing prompts and let optimizers tune them?"]], [Plain [Str "How can I declare what the model should compute, not how?"]], [Plain [Str "How do I automatically find the best prompt and few-shot examples?"]], [Plain [Str "How do I build multi-hop question answering with retrieval?"]], [Plain [Str "How can I swap from GPT-4o to Claude without rewriting prompts?"]], [Plain [Str "How do I get measurable accuracy improvements through compilation?"]], [Plain [Str "How do I build a ReAct agent that uses MCP tools?"]], [Plain [Str "How do I optimize a full RAG pipeline end-to-end?"]], [Plain [Str "How can I treat LLM programs like PyTorch modules?"]], [Plain [Str "How do I deploy a compiled DSPy program behind FastAPI or MLflow?"]]], Header 3 ("problem-statements", ["unnumbered", "unlisted"], []) [Str "Problem Statements"], BulletList [[Plain [Str "Manual prompt iteration is slow, brittle, and untransferable across models"]], [Plain [Str "Compilation requires many LM calls over the training set, incurring cost and time"]], [Plain [Str "Optimizers need 20-200+ labeled examples; cold start has no data"]], [Plain [Str "Compiled programs are harder to inspect and debug than hand-written prompts"]], [Plain [Str "Optimized prompts may transfer poorly between LMs"]], [Plain [Str "Backward compatibility across DSPy versions is not yet guaranteed"]], [Plain [Str "Async mixing and CodeAct tool restrictions add complexity"]]], Header 3 ("when-to-pick-this", ["unnumbered", "unlisted"], []) [Str "When to Pick This"], BulletList [[Plain [Str "Pick DSPy when you want programmatic optimization of prompts and few-shot examples against a measurable metric"]], [Plain [Str "Pick DSPy when you have a labeled evaluation set (20-200+ examples) and care about systematic accuracy gains"]], [Plain [Str "Pick DSPy when you want to compose LM calls like PyTorch modules and swap inference strategies (CoT vs PoT vs ReAct)"]], [Plain [Str "Pick DSPy when you need provider-agnostic abstractions via LiteLLM across many LMs"]], [Plain [Str "Pick Instructor instead when you just want Pydantic validation-and-retry without an optimizer or compilation step"]], [Plain [Str "Pick Outlines instead when you need token-level constrained decoding for guaranteed-valid output"]], [Plain [Str "Pick BAML instead when you want a typed DSL with cross-language code generation rather than Python-only programmatic optimization"]]], Header 3 ("related-terms-and-aliases", ["unnumbered", "unlisted"], []) [Str "Related Terms and Aliases"], BulletList [[Plain [Str "Stanford DSPy"]], [Plain [Str "declarative prompting"]], [Plain [Str "prompt compiler"]], [Plain [Str "LM programming framework"]], [Plain [Str "programmatic prompt optimization"]], [Plain [Str "signature-and-module framework"]], [Plain [Str "Demonstrate-Search-Predict"]], [Plain [Str "automated prompt engineering"]], [Plain [Str "few-shot bootstrapping framework"]], [Plain [Str "ReAct framework"]]]]