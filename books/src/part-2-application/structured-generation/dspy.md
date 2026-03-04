# DSPy

> Stanford NLP declarative framework for programming language models. "Iterate fast on structured code, rather than brittle strings." Compiles AI programs into optimized prompts and weights.

| Field | Value |
|-------|-------|
| Name | DSPy |
| Group | Structured Output & Prompt Engineering |
| Type | SDK |
| Open Source | Yes |
| GitHub | [stanfordnlp/dspy](https://github.com/stanfordnlp/dspy) |
| Stars | 32,331 |
| Docs | [dspy.ai](https://dspy.ai/) |
| License | MIT |
| Contributors | 250+ |

## Overview

DSPy replaces hand-written prompts and brittle string manipulation with a programming model for language models. Rather than crafting prompt templates, developers define typed signatures that specify input and output fields, compose them into modules, and let optimizers automatically tune the prompts and weights for a given metric. The framework treats LM calls as declarative operations, compiling high-level AI programs into efficient prompts or fine-tuning configurations. DSPy supports 40+ LM providers and delivers measurable accuracy improvements through its optimization pipeline, typically costing around $2 USD and taking approximately 20 minutes to run.

## Core Concepts

### Signatures

Signatures are typed declarations of input and output fields for a language model call. They replace free-form prompt strings with structured contracts.

```python
import dspy

# Inline signature: question -> answer
predict = dspy.Predict("question -> answer")

# Class-based signature with typed fields
class Assess(dspy.Signature):
    """Assess the quality of a tweet along the specified dimension."""
    assessed_text: str = dspy.InputField()
    assessment_question: str = dspy.InputField()
    assessment_answer: float = dspy.OutputField()
```

Signatures define what the LM should do without prescribing how it should do it. The framework handles prompt formatting, parsing, and retry logic.

### Modules

Modules are composable building blocks that wrap signatures with specific inference strategies.

- **Predict** - Direct signature invocation, the simplest module
- **ChainOfThought** - Adds intermediate reasoning steps before producing the final output
- **ReAct** - Interleaves reasoning with tool use actions in a loop
- **ProgramOfThought** - Generates and executes code to arrive at answers
- **MultiChainComparison** - Runs multiple chains and selects the best output
- **RAG** - Retrieval-augmented generation combining a retriever with a generator

Custom modules inherit from `dspy.Module` and compose other modules in their `forward` method.

### Optimizers

Optimizers automatically improve program performance against a defined metric by tuning prompts, few-shot examples, or model weights.

- **BootstrapRS** - Random search over bootstrapped few-shot demonstrations
- **MIPROv2** - Multi-prompt instruction proposal with Bayesian optimization
- **GEPA** - Genetic evolutionary prompt algorithm for instruction tuning
- **BetterTogether** - Joint optimization of prompts and weights together

### Adapters

Adapters translate signatures into the specific prompt format expected by a given LM provider. They handle serialization, field mapping, and structured output parsing transparently.

## Installation and Setup

```bash
pip install -U dspy
```

For specific provider support:

```bash
# With Anthropic
pip install -U dspy[anthropic]

# With Google
pip install -U dspy[google]
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

1. **Define** - Write signatures and compose modules into a program
2. **Evaluate** - Measure program accuracy on a development dataset using a metric function
3. **Compile** - Run an optimizer that searches for better prompts, demonstrations, or weights
4. **Deploy** - Use the compiled program with the tuned configuration in production

```
Signature --> Module --> Program --> Optimizer --> Compiled Program
    ^                                   ^
  Types                              Metric
  Fields                             Dataset
```

The compilation step differentiates DSPy from standard prompt engineering. Instead of manually iterating on prompt text, the optimizer systematically explores the space of possible prompts and demonstrations, guided by the evaluation metric.

## Key Features and Functionality

- **Declarative LM programming** - Define what the model should compute via typed signatures rather than how via prompt strings
- **Automatic prompt optimization** - Optimizers tune prompts and few-shot examples to maximize a user-defined metric
- **Composable modules** - Chain, nest, and reuse modules like standard software components
- **Provider agnostic** - Supports 40+ LM providers through a unified interface
- **Typed input/output** - Signatures enforce structured contracts between program components
- **Reproducible optimization** - Compilation produces deterministic, serializable configurations
- **Built-in evaluation** - Integrated metric evaluation over datasets for systematic benchmarking
- **Assertion and constraint system** - `dspy.Assert` and `dspy.Suggest` enforce runtime constraints on LM outputs
- **Automatic few-shot bootstrapping** - Generates high-quality demonstrations from training data without manual curation

## Use Cases

- **Multi-hop question answering** - Compose retrieval and reasoning modules to answer questions requiring multiple evidence steps
- **Classification pipelines** - Build typed classifiers with automatic few-shot optimization (reported improvements from 66% to 87%)
- **Agentic workflows** - Use ReAct modules for tool-augmented reasoning (reported improvements from 24% to 51% on HotPotQA)
- **Information extraction** - Define output signatures with structured fields for entity and relation extraction
- **RAG systems** - Combine retrieval modules with generation modules, optimizing the full pipeline end-to-end
- **Data labeling and assessment** - Use typed output fields (including floats and enums) for structured scoring tasks

## API Reference Summary

### Language Model Configuration

```python
import dspy

# Configure the default LM
lm = dspy.LM("openai/gpt-4o-mini", temperature=0.7)
dspy.configure(lm=lm)

# Use Anthropic
lm = dspy.LM("anthropic/claude-sonnet-4-20250514")
dspy.configure(lm=lm)
```

### Signatures

```python
# Inline notation
"question -> answer"
"context, question -> answer"
"question -> answer: float"

# Class-based
class Summarize(dspy.Signature):
    """Summarize the document in one sentence."""
    document: str = dspy.InputField(desc="The document to summarize")
    summary: str = dspy.OutputField(desc="A one-sentence summary")
```

### Modules

```python
# Basic prediction
predict = dspy.Predict(Summarize)
result = predict(document="...")

# Chain of thought
cot = dspy.ChainOfThought("question -> answer")
result = cot(question="What is the capital of France?")

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

### Optimization

```python
# Bootstrap few-shot examples
optimizer = dspy.BootstrapRS(
    metric=answer_exact_match,
    max_bootstrapped_demos=4,
    num_candidate_programs=10
)
compiled_program = optimizer.compile(program, trainset=train_examples)

# MIPROv2 for instruction optimization
optimizer = dspy.MIPROv2(
    metric=answer_exact_match,
    auto="medium"
)
compiled_program = optimizer.compile(program, trainset=train_examples)
```

### Saving and Loading

```python
compiled_program.save("optimized_program.json")

program = RAGModule()
program.load("optimized_program.json")
```

## Configuration and Customization

### Global Settings

```python
dspy.configure(
    lm=dspy.LM("openai/gpt-4o-mini"),
    rm=dspy.ColBERTv2(url="http://localhost:8893"),  # retrieval model
    trace=[],  # enable tracing
)
```

### Per-Call Overrides

```python
with dspy.context(lm=dspy.LM("anthropic/claude-sonnet-4-20250514")):
    result = program(question="...")
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
        return result
```

`dspy.Assert` raises a hard failure when the constraint is not met, while `dspy.Suggest` provides a soft signal that the optimizer can use during compilation to improve outputs.

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

### With Custom Tools

```python
def search_wikipedia(query: str) -> str:
    """Search Wikipedia for information."""
    # implementation
    return result

react = dspy.ReAct(
    "question -> answer",
    tools=[search_wikipedia]
)
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

optimizer = dspy.BootstrapRS(metric=intent_match, max_bootstrapped_demos=4)
optimized = optimizer.compile(classifier, trainset=train_data)

result = optimized(message="I'm having trouble with my order")
print(result.intent)
```

### Multi-Hop RAG with Optimization

```python
import dspy

class MultiHopRAG(dspy.Module):
    def __init__(self, passages_per_hop=3, num_hops=2):
        self.retrieve = [
            dspy.Retrieve(k=passages_per_hop)
            for _ in range(num_hops)
        ]
        self.generate_query = dspy.ChainOfThought(
            "context, question -> search_query"
        )
        self.generate_answer = dspy.ChainOfThought(
            "context, question -> answer"
        )

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

### Structured Assessment

```python
import dspy

class TweetAssessment(dspy.Signature):
    """Assess tweet quality on a numeric scale."""
    tweet: str = dspy.InputField()
    dimension: str = dspy.InputField(
        desc="The quality dimension to assess"
    )
    score: float = dspy.OutputField(
        desc="Quality score from 0.0 to 1.0"
    )

assessor = dspy.ChainOfThought(TweetAssessment)
result = assessor(
    tweet="DSPy lets you program LMs declaratively!",
    dimension="informativeness"
)
print(f"Score: {result.score}")
```

## Limitations and Considerations

- **Optimization cost** - Compiling programs requires LM calls over the training set, which incurs API costs and wall-clock time (typically ~$2 and ~20 minutes for medium-sized programs)
- **Dataset requirement** - Optimizers need labeled examples to tune against; cold-start scenarios with no evaluation data cannot leverage compilation
- **Debugging complexity** - Compiled programs with optimized prompts and demonstrations can be harder to inspect and debug than hand-written prompts
- **Provider-specific behavior** - While DSPy abstracts across providers, underlying model differences can cause optimized programs to transfer poorly between LMs
- **Learning curve** - The programming model (signatures, modules, optimizers) introduces concepts that differ from conventional prompt engineering workflows
- **Non-determinism** - LM outputs are inherently stochastic; optimization results may vary across runs depending on the training data sampling

## Changelog Highlights

- DSPy 2.0 introduced the current signature and module system, replacing the earlier template-based approach
- Adapter system added for structured output mapping across providers
- MIPROv2 and BetterTogether optimizers added for more sophisticated prompt and weight tuning
- Support expanded to 40+ LM providers
- Assertion and suggestion system introduced for runtime constraint enforcement

## Citations

- [1] [DSPy Documentation](https://dspy.ai/)
