# DSPy

> Stanford framework for programming LMs declaratively with automatic prompt optimization

| Field | Value |
|-------|-------|
| Name | DSPy |
| Group | Structured Generation |
| Type | SDK |
| Open Source | Yes |
| GitHub | [stanfordnlp/dspy](https://github.com/stanfordnlp/dspy) |
| Stars | 32,331 |
| Docs | [dspy.ai](https://dspy.ai/) |
| License | MIT |

## Overview

DSPy replaces hand-written prompts and brittle string manipulation with a programming model for language models. Rather than crafting prompt templates, developers define typed signatures that specify input and output fields, compose them into modules, and let optimizers automatically tune the prompts and weights for a given metric. The framework treats LM calls as declarative operations, compiling high-level AI programs into efficient prompts or fine-tuning configurations. DSPy supports numerous LM providers through LiteLLM integration and delivers measurable accuracy improvements through its optimization pipeline.

The core philosophy is "programming, not prompting." DSPy separates concerns that conventional prompting couples together: signature definitions (what), adapter formatting (how it is serialized), module logic (inference strategy), and optimization (automatic tuning). This separation enables language model swapping without logic changes, module substitution (for example, replacing ChainOfThought with ProgramOfThought), prompt optimization without architecture modification, and fine-tuning capabilities across programs.

## Core Concepts

### Signatures

Signatures are typed declarations of input and output fields for a language model call. They replace free-form prompt strings with structured contracts. DSPy supports two forms of signatures.

**Inline signatures** use string notation with optional type specifications:

```python
"question -> answer"                                    # basic string types
"sentence -> sentiment: bool"                           # typed output
"context: list[str], question: str -> answer: str"      # multiple typed fields
"question, choices: list[str] -> reasoning: str, selection: int"
```

**Class-based signatures** use Python class definitions with docstrings and field descriptors for complex tasks:

```python
import dspy

class Assess(dspy.Signature):
    """Assess the quality of a tweet along the specified dimension."""
    assessed_text: str = dspy.InputField()
    assessment_question: str = dspy.InputField()
    assessment_answer: float = dspy.OutputField()
```

Signatures define what the LM should do without prescribing how. The framework handles prompt formatting, parsing, and retry logic. Field names carry semantic meaning and are used by adapters to construct prompts. Supported types include basic Python types (str, int, bool, float), typing module constructs (list, dict, Optional, Union, Literal), custom Pydantic BaseModel classes, and special DSPy types (dspy.Image, dspy.History).

InputField and OutputField accept a `desc` parameter that provides additional context to the language model about what the field represents.

### Modules

Modules are composable building blocks that wrap signatures with specific inference strategies. Each module abstracts a prompting technique while maintaining generalizability across any signature. Modules contain learnable parameters (instructions, demonstrations) and can be composed into larger programs. The design draws inspiration from PyTorch's neural network architecture.

Built-in modules:

- **Predict** -- Direct signature invocation. The foundational module that handles basic prediction, managing instruction storage, demonstrations, and language model weight updates.
- **ChainOfThought** -- Instructs the LM to think step-by-step before committing to the signature's response. Injects a reasoning field before output fields, improving output quality on complex tasks.
- **ReAct** -- Reasoning and Acting agent module. Interleaves reasoning with tool use actions in an iterative loop, automatically selecting and calling tools until the task is complete. Accepts a `max_iters` parameter (default 20).
- **CodeAct** -- Generates and executes Python code snippets within a sandboxed interpreter, combining code generation with tool execution. Inherits from both ReAct and ProgramOfThought.
- **ProgramOfThought** -- Directs the LM to generate executable code where execution results determine the final response.
- **MultiChainComparison** -- Compares multiple ChainOfThought outputs to produce refined predictions.
- **BestOfN** -- Generates N candidates and selects the best one according to a metric.
- **Refine** -- Iteratively improves an output by reflecting on it and revising.
- **Parallel** -- Runs multiple modules concurrently.
- **RLM** -- Recursive language model for handling contexts too large for standard prompts.
- **majority** -- A voting function returning the most popular response from multiple predictions.

Custom modules inherit from `dspy.Module` and compose other modules in their `forward` method:

```python
class MultiHopSearch(dspy.Module):
    def __init__(self, num_docs=10, num_hops=4):
        self.generate_query = dspy.ChainOfThought("claim, notes -> query")
        self.append_notes = dspy.ChainOfThought("claim, notes, context -> new_notes")

    def forward(self, claim: str) -> list[str]:
        notes = "No notes yet."
        for hop in range(self.num_hops):
            query = self.generate_query(claim=claim, notes=notes).query
            context = search(query, k=self.num_docs)
            notes = self.append_notes(claim=claim, notes=notes, context=context).new_notes
        return notes
```

### Adapters

Adapters serve as the connection layer between dspy.Predict and language models. They translate DSPy signatures into system messages, format input data, parse LM responses into dspy.Prediction instances, manage conversation history, and convert DSPy types (Tools, Images) into prompt messages.

Built-in adapters:

- **ChatAdapter** (default) -- Uses `[[ ## field_name ## ]]` markers to delineate fields. Universally compatible across all language models. Includes fallback protection that automatically retries with JSONAdapter if parsing fails. More verbose in output tokens.
- **JSONAdapter** -- Outputs structured as pure JSON objects leveraging native model capabilities via the `response_format` parameter. Lower latency and minimal boilerplate, but incompatible with models lacking native structured output support.

Configure adapters globally or per-context:

```python
dspy.configure(lm=dspy.LM("openai/gpt-4o-mini"), adapter=dspy.ChatAdapter())

with dspy.context(adapter=dspy.JSONAdapter()):
    result = program(question="...")
```

### Optimizers

Optimizers automatically improve program performance against a defined metric by tuning prompts, few-shot examples, or model weights. DSPy recommends allocating 20% of data for training and 80% for validation with prompt-based optimizers, as they tend to overfit on small training sets.

Available optimizers:

- **BootstrapFewShot** -- Generates few-shot demonstrations by running the program on training examples and keeping successful traces.
- **BootstrapRS** (alias for BootstrapFewShotWithRandomSearch) -- Random search over bootstrapped few-shot demonstrations with multiple candidate programs.
- **BootstrapFinetune** -- Uses bootstrapped demonstrations to fine-tune model weights rather than optimize prompts.
- **MIPROv2** -- Multi-prompt Instruction Proposal Optimizer. Jointly optimizes instructions and few-shot examples using Bayesian Optimization across three stages: bootstrap demonstrations, propose instruction candidates, then search for optimal combinations. Supports `auto` modes: light, medium, heavy.
- **COPRO** -- Coordinate-based prompt optimization.
- **GEPA** -- Genetic-Pareto reflective optimizer that adaptively evolves textual components of arbitrary systems. Maintains a Pareto frontier of candidates, uses LLM-driven reflection on execution traces to propose targeted improvements, and accepts both scalar scores and textual feedback. Supports integration with Weights & Biases and MLflow.
- **SIMBA** -- Optimization strategy for complex programs.
- **BetterTogether** -- Joint optimization of prompts and weights together.
- **InferRules** -- Infers optimization rules from successful traces.
- **LabeledFewShot** -- Uses labeled examples directly as demonstrations.
- **KNN/KNNFewShot** -- k-nearest neighbor selection of demonstrations at inference time.
- **Ensemble** -- Combines multiple optimized programs.

### Examples and Datasets

The `dspy.Example` class is a flexible data container for training and evaluation data. It supports dictionary-like access, input/output field separation, and serialization:

```python
example = dspy.Example(question="What is DSPy?", answer="A framework for LM programming")
example = example.with_inputs("question")

inputs = example.inputs()    # only question
labels = example.labels()    # only answer
```

## Installation

```bash
pip install -U dspy
```

For specific provider support:

```bash
pip install -U dspy[anthropic]    # Anthropic Claude
pip install -U dspy[google]       # Google Gemini
pip install -U "dspy[mcp]"        # Model Context Protocol support
```

Requires Python 3.9 or higher.

### Quick Start

```python
import dspy

lm = dspy.LM("openai/gpt-4o-mini")
dspy.configure(lm=lm)

qa = dspy.ChainOfThought("question -> answer")
result = qa(question="What is the tallest mountain in the world?")
print(result.answer)
```

## Architecture

DSPy follows a Define, Evaluate, Compile, Deploy workflow:

1. **Define** -- Write signatures and compose modules into a program. Identify system inputs and desired outputs. Start simple with a single module, then add complexity incrementally.
2. **Evaluate** -- Measure program accuracy on a development dataset (20-200+ examples) using a metric function. Metrics range from simple accuracy to complex DSPy programs that verify multiple output properties.
3. **Compile** -- Run an optimizer that searches for better prompts, demonstrations, or weights. The optimizer systematically explores the space of possible prompts and demonstrations, guided by the evaluation metric.
4. **Deploy** -- Use the compiled program with the tuned configuration in production via FastAPI or MLflow.

```
Signature --> Module --> Program --> Optimizer --> Compiled Program
    ^                                   ^
  Types                              Metric
  Fields                             Dataset
```

The compilation step differentiates DSPy from standard prompt engineering. Instead of manually iterating on prompt text, the optimizer systematically explores the search space guided by the evaluation metric.

## Key Features

- **Declarative LM programming** -- Define what the model should compute via typed signatures rather than how via prompt strings.
- **Automatic prompt optimization** -- Optimizers tune prompts and few-shot examples to maximize a user-defined metric. MIPROv2 jointly optimizes instructions and demonstrations using Bayesian Optimization. GEPA uses genetic evolution with LLM-driven reflection.
- **Composable modules** -- Chain, nest, and reuse modules like standard software components using Python control flow.
- **Provider agnostic** -- Supports numerous LM providers through LiteLLM integration: OpenAI, Anthropic, Google Gemini, Vertex AI, Databricks, SGLang, Ollama, Azure, AWS SageMaker, Together AI, Anyscale, and more.
- **Typed input/output** -- Signatures enforce structured contracts with support for str, int, bool, float, list, dict, Literal, Pydantic models, dspy.Image, and dspy.History.
- **Assertion and constraint system** -- `dspy.Assert` raises a hard failure when a constraint is not met. `dspy.Suggest` provides a soft signal the optimizer can use during compilation to improve outputs.
- **Automatic few-shot bootstrapping** -- Generates high-quality demonstrations from training data without manual curation.
- **Tool integration** -- ReAct and CodeAct modules support external tool use. Native function calling via adapters. MCP (Model Context Protocol) integration for standardized tool discovery via `dspy.Tool.from_mcp_tool()`.
- **Async and streaming** -- Native `acall()` async execution on most modules. `dspy.streamify()` for real-time token streaming and intermediate status updates. `dspy.asyncify()` for running sync programs in thread pools.
- **Thread-safe configuration** -- `dspy.configure()` and `dspy.context()` are thread-safe. Track usage statistics with `dspy.configure(track_usage=True)`.
- **Caching** -- LM calls are cached by default. Bypass with `cache=False` or `rollout_id` parameter.
- **Reproducible optimization** -- Compilation produces deterministic, serializable configurations. Save/load programs as JSON or pickle.
- **Responses API** -- Support for models with enhanced reasoning via `model_type="responses"`.

## Use Cases

- **Multi-hop question answering** -- Compose retrieval and reasoning modules to answer questions requiring multiple evidence steps. Reported improvements from 24% to 51% on HotPotQA with ReAct optimization.
- **Classification pipelines** -- Build typed classifiers with automatic few-shot optimization. Reported improvements from 66% to 87% accuracy.
- **Agentic workflows** -- Use ReAct modules for tool-augmented reasoning with automatic tool selection and error recovery. CodeAct for code-generation-based agents.
- **Information extraction** -- Define output signatures with structured fields for entity and relation extraction. Supports Literal type constraints for categorical outputs.
- **RAG systems** -- Combine retrieval modules with generation modules, optimizing the full pipeline end-to-end including retrieval quality.
- **Data labeling and assessment** -- Use typed output fields (including floats and enums) for structured scoring tasks.
- **Customer service agents** -- Build tool-using agents with MCP integration for database access, booking systems, and ticket management.
- **Image and audio processing** -- Multi-modal support via dspy.Image and dspy.Audio types in signatures.
- **Privacy-conscious delegation** -- PAPILLON pattern for delegating tasks while preserving privacy constraints.

## API Reference

### Language Model Configuration

```python
import dspy

# Configure default LM with parameters
lm = dspy.LM("openai/gpt-4o-mini", temperature=0.7, max_tokens=3000, cache=True)
dspy.configure(lm=lm)

# Anthropic
lm = dspy.LM("anthropic/claude-sonnet-4-20250514")

# Google Gemini
lm = dspy.LM("gemini/gemini-2.0-flash", api_key="GEMINI_API_KEY")

# Local via Ollama
lm = dspy.LM("ollama_chat/llama3.2", api_base="http://localhost:11434")

# Responses API for enhanced reasoning
lm = dspy.LM("openai/gpt-5-mini", model_type="responses", temperature=1.0)

# Direct LM calls
lm("Say this is a test!", temperature=0.7)
lm(messages=[{"role": "user", "content": "Say this is a test!"}])

# Access history and metadata
len(lm.history)
lm.history[-1]  # prompt, messages, kwargs, response, outputs, usage, cost, timestamp
```

### Signatures

```python
# Inline notation
"question -> answer"
"context, question -> answer"
"question -> answer: float"
"sentence -> sentiment: bool"

# Class-based with field descriptors
class Summarize(dspy.Signature):
    """Summarize the document in one sentence."""
    document: str = dspy.InputField(desc="The document to summarize")
    summary: str = dspy.OutputField(desc="A one-sentence summary")

# With Literal type constraints
class ClassifyEmotion(dspy.Signature):
    text: str = dspy.InputField()
    emotion: Literal["joy", "sadness", "anger", "fear"] = dspy.OutputField()

# With Pydantic models
class ExtractedEntity(BaseModel):
    name: str
    entity_type: str

class ExtractEntities(dspy.Signature):
    text: str = dspy.InputField()
    entities: list[ExtractedEntity] = dspy.OutputField()
```

### Modules

```python
# Basic prediction
predict = dspy.Predict(Summarize)
result = predict(document="...")

# Chain of thought
cot = dspy.ChainOfThought("question -> answer")
result = cot(question="What is the capital of France?")
print(result.reasoning)  # intermediate reasoning
print(result.answer)     # final answer

# ReAct with tools
def get_weather(city: str) -> str:
    """Get weather for a city."""
    return f"Sunny in {city}"

react = dspy.ReAct("question -> answer", tools=[get_weather], max_iters=5)
result = react(question="What is the weather in Tokyo?")

# CodeAct with sandboxed execution
act = dspy.CodeAct("n -> factorial_result", tools=[factorial], max_iters=5)
result = act(n=5)

# Custom module
class RAGModule(dspy.Module):
    def __init__(self, num_passages=3):
        self.retrieve = dspy.Retrieve(k=num_passages)
        self.generate = dspy.ChainOfThought("context, question -> answer")

    def forward(self, question):
        context = self.retrieve(question).passages
        return self.generate(context=context, question=question)
```

### Evaluation

```python
def answer_exact_match(example, prediction, trace=None):
    return example.answer.lower() == prediction.answer.lower()

evaluate = dspy.Evaluate(
    devset=dev_examples,
    metric=answer_exact_match,
    num_threads=4,
    display_progress=True
)
score = evaluate(program)
```

Built-in metrics: `answer_exact_match`, `answer_passage_match`, `SemanticF1`, `CompleteAndGrounded`.

### Optimization

```python
# MIPROv2 for instruction and demonstration optimization
optimizer = dspy.MIPROv2(
    metric=answer_exact_match,
    auto="medium",              # light, medium, or heavy
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
    reflection_lm=dspy.LM("openai/gpt-4o"),
    auto="medium",
    log_dir="./gepa_logs"
)
compiled_program = optimizer.compile(program, trainset=train_examples, valset=val_examples)
```

### Tools and MCP

```python
# Define tools
def search_wikipedia(query: str) -> str:
    """Search Wikipedia for information."""
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
result = await react.acall(user_request="...")
```

### Saving and Loading

```python
# State-only saving (JSON, recommended)
compiled_program.save("optimized_program.json", save_program=False)
program = RAGModule()
program.load("optimized_program.json")

# Whole program saving (dspy >= 2.6.0)
compiled_program.save("./dspy_program/", save_program=True)
loaded = dspy.load("./dspy_program/")

# With custom module serialization
program.save("./path/", save_program=True, modules_to_serialize=[custom_module])
```

## Configuration

### Global Settings

```python
dspy.configure(
    lm=dspy.LM("openai/gpt-4o-mini"),
    rm=dspy.ColBERTv2(url="http://localhost:8893"),  # retrieval model
    adapter=dspy.ChatAdapter(),
    trace=[],
    track_usage=True
)
```

### Per-Call Overrides

```python
with dspy.context(lm=dspy.LM("anthropic/claude-sonnet-4-20250514")):
    result = program(question="...")

# Module-level configuration
predict = dspy.Predict("question -> answer", temperature=1.0)
predict(question="...", config={"rollout_id": 5, "temperature": 0.5})
```

### Assertions and Constraints

```python
class FactCheckedAnswer(dspy.Module):
    def __init__(self):
        self.generate = dspy.ChainOfThought("question -> answer")

    def forward(self, question):
        result = self.generate(question=question)
        dspy.Assert(
            len(result.answer) > 10,
            "Answer must be substantive (more than 10 characters)"
        )
        dspy.Suggest(
            "citation" in result.answer.lower(),
            "Answer should include citations"
        )
        return result
```

### Caching

LM calls are cached by default. Control caching behavior:

```python
lm = dspy.LM("openai/gpt-4o-mini", cache=False)           # disable entirely
predict(question="...", config={"rollout_id": 5})           # bypass specific cache entry
dspy.configure_cache(enable=False)                          # global disable
```

## Integration Patterns

### With Retrieval Systems

```python
import dspy

colbert = dspy.ColBERTv2(url="http://localhost:8893")
dspy.configure(rm=colbert)

class SearchAndAnswer(dspy.Module):
    def __init__(self):
        self.retrieve = dspy.Retrieve(k=5)
        self.answer = dspy.ChainOfThought("context, question -> answer")

    def forward(self, question):
        passages = self.retrieve(question).passages
        return self.answer(context=passages, question=question)
```

### With Custom Tools via ReAct

```python
def search_wikipedia(query: str) -> str:
    """Search Wikipedia for information."""
    return result

def calculate(expression: str) -> float:
    """Evaluate a math expression."""
    return eval(expression)

react = dspy.ReAct(
    "question -> answer",
    tools=[search_wikipedia, calculate],
    max_iters=10
)
result = react(question="What is the population of France times 2?")
```

### Pipeline Composition

```python
class MultiStepPipeline(dspy.Module):
    def __init__(self):
        self.extract = dspy.Predict("document -> entities: list[str]")
        self.classify = dspy.ChainOfThought("entity, context -> category")
        self.summarize = dspy.Predict("entities, categories -> summary")

    def forward(self, document):
        entities = self.extract(document=document).entities
        categories = [
            self.classify(entity=e, context=document).category
            for e in entities
        ]
        return self.summarize(entities=entities, categories=categories)
```

### FastAPI Deployment

```python
from fastapi import FastAPI
import dspy

app = FastAPI()
program = dspy.ChainOfThought("question -> answer")
program.load("optimized.json")

async_program = dspy.asyncify(program)

@app.post("/predict")
async def predict(question: str):
    result = await async_program(question=question)
    return {"answer": result.answer}
```

### Streaming

```python
import dspy

predict = dspy.Predict("question -> answer")
stream_predict = dspy.streamify(
    predict,
    stream_listeners=[dspy.streaming.StreamListener(signature_field_name="answer")]
)

# Async streaming
async for chunk in stream_predict(question="Why?"):
    print(chunk)

# Synchronous streaming
stream_predict = dspy.streamify(predict, stream_listeners=[...], async_streaming=False)
for chunk in stream_predict(question="Why?"):
    print(chunk)
```

### MLflow Deployment

```python
import mlflow
import dspy

class MyProgram(dspy.Module):
    def forward(self, question):
        cot = dspy.ChainOfThought("question -> answer")
        return cot(question=question)

with mlflow.start_run():
    mlflow.dspy.log_model(MyProgram(), "model", task="llm/v1/chat")
# Serve: mlflow models serve -m runs:/{run_id}/model -p 6000
```

## Examples

### Optimized Classification

```python
import dspy

class ClassifyIntent(dspy.Signature):
    """Classify the user message into an intent category."""
    message: str = dspy.InputField()
    intent: str = dspy.OutputField(
        desc="One of: greeting, question, complaint, feedback"
    )

classifier = dspy.Predict(ClassifyIntent)

def intent_match(example, prediction, trace=None):
    return example.intent == prediction.intent

optimizer = dspy.MIPROv2(metric=intent_match, auto="light")
optimized = optimizer.compile(classifier, trainset=train_data)

result = optimized(message="I'm having trouble with my order")
print(result.intent)
```

### Multi-Hop RAG with Optimization

```python
import dspy

class MultiHopRAG(dspy.Module):
    def __init__(self, passages_per_hop=3, num_hops=2):
        self.retrieve = [dspy.Retrieve(k=passages_per_hop) for _ in range(num_hops)]
        self.generate_query = dspy.ChainOfThought("context, question -> search_query")
        self.generate_answer = dspy.ChainOfThought("context, question -> answer")

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

optimizer = dspy.MIPROv2(metric=answer_f1, auto="medium")
compiled_rag = optimizer.compile(MultiHopRAG(), trainset=train_examples)
```

### ReAct Agent with MCP Tools

```python
import dspy
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

class AirlineAgent(dspy.Signature):
    """You are an airline customer service agent with access to tools."""
    user_request: str = dspy.InputField()
    process_result: str = dspy.OutputField(
        desc="Summary with confirmation numbers or relevant info"
    )

async def build_agent():
    server_params = StdioServerParameters(command="python", args=["mcp_server.py"])
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(read, write) as session:
            await session.initialize()
            tools = await session.list_tools()
            dspy_tools = [dspy.Tool.from_mcp_tool(session, t) for t in tools.tools]

            dspy.configure(lm=dspy.LM("openai/gpt-4o-mini"))
            react = dspy.ReAct(AirlineAgent, tools=dspy_tools)
            result = await react.acall(user_request="Book a flight from SFO to JFK")
            return result
```

### Structured Assessment

```python
import dspy

class TweetAssessment(dspy.Signature):
    """Assess tweet quality on a numeric scale."""
    tweet: str = dspy.InputField()
    dimension: str = dspy.InputField(desc="The quality dimension to assess")
    score: float = dspy.OutputField(desc="Quality score from 0.0 to 1.0")

assessor = dspy.ChainOfThought(TweetAssessment)
result = assessor(
    tweet="DSPy lets you program LMs declaratively!",
    dimension="informativeness"
)
print(f"Score: {result.score}")
```

## Limitations

- **Optimization cost** -- Compiling programs requires LM calls over the training set, incurring API costs and wall-clock time. MIPROv2 with auto="medium" runs many trials of Bayesian Optimization.
- **Dataset requirement** -- Optimizers need labeled examples to tune against. Cold-start scenarios with no evaluation data cannot leverage compilation. Recommended minimum of 20-200 input examples for evaluation.
- **Debugging complexity** -- Compiled programs with optimized prompts and demonstrations can be harder to inspect and debug than hand-written prompts. Use `dspy.inspect_history()` and MLflow tracing for observability.
- **Provider-specific behavior** -- While DSPy abstracts across providers, underlying model differences can cause optimized programs to transfer poorly between LMs. Native function calling support varies by provider.
- **Learning curve** -- The programming model (signatures, modules, optimizers, adapters) introduces concepts that differ from conventional prompt engineering workflows.
- **Non-determinism** -- LM outputs are inherently stochastic. Optimization results may vary across runs depending on training data sampling and random seeds.
- **CodeAct limitations** -- Only accepts pure functions (not callable objects). Tools cannot depend on external packages. All function dependencies must be explicitly passed.
- **Backward compatibility** -- Saving and loading programs across different DSPy versions is not yet supported. Use identical versions until 3.0.0 introduces guaranteed major-version compatibility.
- **Async complexity** -- While DSPy supports native async, mixing sync and async tools requires explicit context management (`allow_tool_async_sync_conversion`). Async involves more complex error handling and debugging.

## Changelog

- DSPy 2.0 introduced the current signature and module system, replacing the earlier template-based approach
- DSPy 2.6.0 added whole-program saving with `save_program=True` and streaming support via `dspy.streamify()`
- Adapter system added for structured output mapping across providers (ChatAdapter as default, JSONAdapter for native structured output)
- MIPROv2 optimizer added for joint instruction and demonstration optimization using Bayesian Optimization
- GEPA optimizer introduced for reflective prompt evolution using genetic-Pareto strategies (Agrawal et al., 2025)
- BetterTogether optimizer added for joint prompt and weight tuning
- CodeAct module added combining code generation with sandboxed execution
- BestOfN and Refine modules added for output quality improvement
- MCP (Model Context Protocol) integration added via `dspy.Tool.from_mcp_tool()` for standardized tool discovery
- Native function calling support added to ChatAdapter and JSONAdapter
- Responses API support added for models with enhanced reasoning capabilities
- MLflow integration for production deployment, tracing, and experiment tracking
- FastAPI deployment guide with async support and streaming endpoints
- Assertion and suggestion system introduced for runtime constraint enforcement

## Citations

- [1] [DSPy Documentation - Home](https://dspy.ai/)
- [2] [DSPy Programming Overview](https://dspy.ai/learn/programming/overview/)
- [3] [DSPy Language Models](https://dspy.ai/learn/programming/language_models/)
- [4] [DSPy Signatures](https://dspy.ai/learn/programming/signatures/)
- [5] [DSPy Modules](https://dspy.ai/learn/programming/modules/)
- [6] [DSPy Adapters](https://dspy.ai/learn/programming/adapters/)
- [7] [DSPy Tools](https://dspy.ai/learn/programming/tools/)
- [8] [DSPy Evaluation Overview](https://dspy.ai/learn/evaluation/overview/)
- [9] [DSPy Optimization Overview](https://dspy.ai/learn/optimization/overview/)
- [10] [DSPy API Reference](https://dspy.ai/api/)
- [11] [DSPy Tutorials](https://dspy.ai/tutorials/)
- [12] [DSPy ReAct API](https://dspy.ai/api/modules/ReAct/)
- [13] [DSPy CodeAct API](https://dspy.ai/api/modules/CodeAct/)
- [14] [DSPy MIPROv2 API](https://dspy.ai/api/optimizers/MIPROv2/)
- [15] [DSPy GEPA Overview](https://dspy.ai/api/optimizers/GEPA/overview/)
- [16] [DSPy Production](https://dspy.ai/production/)
- [17] [DSPy Saving and Loading](https://dspy.ai/tutorials/saving/)
- [18] [DSPy Streaming](https://dspy.ai/tutorials/streaming/)
- [19] [DSPy Deployment](https://dspy.ai/tutorials/deployment/)
- [20] [DSPy MCP Integration](https://dspy.ai/tutorials/mcp/)
- [21] [DSPy Async](https://dspy.ai/tutorials/async/)
- [22] [DSPy Example API](https://dspy.ai/api/primitives/Example/)
