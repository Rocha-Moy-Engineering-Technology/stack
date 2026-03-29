# DeepEval

> LLM evaluation framework with 50+ research-backed metrics and Pytest integration

| Field | Value |
|-------|-------|
| Group | Evaluation & Testing |
| Type | SDK |
| Open Source | Yes |
| GitHub | [https://github.com/confident-ai/deepeval](https://github.com/confident-ai/deepeval) |
| Stars | 13756 |
| Documentation | [Official Docs](https://deepeval.com/docs/getting-started) |

## Overview

DeepEval is an open-source evaluation framework for Large Language Models (LLMs), designed to bring the rigor of unit testing to LLM application development. Built by Confident AI, it functions as "Pytest for LLM outputs," enabling developers to write deterministic, repeatable evaluation suites that validate LLM behavior against research-backed metrics. The framework scores every metric on a normalized 0-to-1 scale with configurable pass/fail thresholds, supporting both single-turn and multi-turn conversational evaluations.

DeepEval provides over 50 plug-and-use metrics spanning Retrieval-Augmented Generation (RAG) quality, agentic behavior, conversational coherence, safety, bias, multimodal content, and custom criteria. It supports two evaluation modes: end-to-end (black-box) testing that treats the application as a unified system, and component-level (white-box) testing that uses the `@observe` decorator for tracing and evaluating individual components within a pipeline.

Beyond evaluation, DeepEval includes synthetic dataset generation with evolution techniques, regression testing for comparing model versions side-by-side, and integration with DeepTeam for red-teaming LLM applications against 40+ vulnerability categories using 10+ adversarial attack methods. An optional cloud platform, Confident AI, provides centralized reporting, dataset curation, prompt optimization, and production monitoring.

## Core Concepts

**LLMTestCase** is the fundamental unit of evaluation, representing a single atomic interaction with an LLM application. It requires an `input` field (the user query) and an `actual_output` field (what the LLM produced). Optional fields include `expected_output`, `context` (ground truth facts), `retrieval_context` (what the RAG pipeline actually retrieved), `tools_called`, `expected_tools`, `token_cost`, and `completion_time`. Test cases are evaluated against one or more metrics to produce scores and pass/fail determinations.

**ConversationalTestCase** extends evaluation to multi-turn interactions. It contains a sequence of `Turn` objects, each with a `role` and `content`, representing a full dialogue between a user and an LLM assistant. Conversational metrics such as Knowledge Retention and Role Adherence operate on these multi-turn structures.

**MLLMTestCase** supports multimodal evaluation, enabling test cases that include both text and images via `MLLMImage` objects. Images can be provided as local file paths, remote URLs, or base64-encoded data.

**Goldens** are ideal input-output pairs that serve as reusable templates for evaluation. Unlike test cases, Goldens require only the `input` field for initialization and do not need an `actual_output` until evaluation time. They represent the expected behavior of the system and can be reused across multiple LLM iterations without regeneration.

**EvaluationDataset** is a collection of Goldens (single-turn `Golden` objects or multi-turn `ConversationalGolden` objects). Datasets manage the lifecycle of evaluation data: loading Goldens, generating actual outputs by running them through the LLM application, converting them to test cases, and executing metrics. Datasets can be persisted locally as JSON or CSV, or pushed to the Confident AI cloud platform.

**Metrics** are scoring functions that evaluate test cases on a 0-to-1 scale. Each metric produces a score, a human-readable reason explaining the score, and a pass/fail status based on a configurable threshold (default 0.5). Most metrics use an "LLM-as-a-judge" approach, where a separate LLM evaluates the quality of the output.

**Tracing** uses the `@observe` decorator to instrument LLM application components, creating a hierarchical trace of execution. Traces capture inputs, outputs, and intermediate states at each level, enabling component-level evaluation where individual pipeline stages (retriever, generator, tool caller) are scored independently.

## Architecture

DeepEval is organized around a layered evaluation pipeline:

```
Test Cases / Goldens          Atomic units of LLM interaction (input + output)
        |
Evaluation Dataset             Collection of Goldens managed as a cohesive set
        |
Metrics Engine                 50+ metrics: G-Eval, RAG, Agentic, Safety, Multimodal
        |
LLM Judge                     Configurable judge model (OpenAI, Anthropic, Ollama, custom)
        |
Scoring & Reporting           0-1 scores, reasoning, pass/fail, regression comparison
        |
Confident AI (optional)       Cloud platform for centralized results and monitoring
```

**Evaluation Pipeline**: Goldens are loaded into an `EvaluationDataset`, processed through the target LLM application to generate `actual_output` values, converted into test cases, and then evaluated against selected metrics. Each metric invokes the configured LLM judge to produce a score and reasoning.

**Metric Evaluation Techniques**: Metrics employ several judge-based approaches including G-Eval (criteria-based evaluation with chain-of-thought), Question-Answer Generation (QAG) for factual verification, and Deep Acyclic Graphs (DAG) for structured evaluation flows.

**Tracing Layer**: The `@observe` decorator instruments application code to create hierarchical traces. Each decorated function becomes a span within a trace, and metrics can be attached to individual spans for component-level evaluation rather than end-to-end only.

**Synthetic Data Pipeline**: The `Synthesizer` follows a four-step process -- input generation, filtration (quality scoring on self-containment and clarity), evolution (complexity escalation through 7 techniques), and styling (format customization).

**Red-Teaming Pipeline**: DeepTeam generates baseline adversarial attacks, enhances them using attack methods (prompt injection, jailbreaking, encoding), feeds them to the target LLM, and scores responses using vulnerability-specific metrics in strict binary mode (0 or 1).

## Key Features and Functionality

**G-Eval Custom Metrics**: Define evaluation criteria in natural language and let an LLM judge score outputs accordingly. G-Eval achieves human-like evaluation accuracy by using chain-of-thought reasoning before assigning scores.

```python
from deepeval.metrics import GEval
from deepeval.test_case import LLMTestCaseParams

correctness_metric = GEval(
    name="Correctness",
    criteria="Determine whether the actual output is factually correct based on the expected output.",
    evaluation_params=[
        LLMTestCaseParams.ACTUAL_OUTPUT,
        LLMTestCaseParams.EXPECTED_OUTPUT,
    ],
    threshold=0.5,
)
```

**RAG Evaluation**: Dedicated metrics for both the retriever and generator stages of RAG pipelines. Retriever metrics (Contextual Relevancy, Contextual Precision, Contextual Recall) evaluate whether the right documents were retrieved. Generator metrics (Answer Relevancy, Faithfulness) evaluate whether the LLM produced accurate, grounded responses.

**Agentic Metrics**: Evaluate autonomous agent behavior with Task Completion, Tool Correctness, Argument Correctness, Step Efficiency, Plan Adherence, and Plan Quality metrics. Test cases include `tools_called` and `expected_tools` fields for verifying agent tool usage.

**Conversational Metrics**: Assess multi-turn dialogue quality with Knowledge Retention (does the assistant remember earlier context), Role Adherence (does it stay in character), Conversation Completeness, and Conversation Relevancy.

**Safety and Bias Detection**: Metrics for Bias, Toxicity, PII Leakage, Role Violation, Misuse, and Non-Advice detect harmful or inappropriate outputs.

**Multimodal Evaluation**: Image Coherence, Image Helpfulness, Image Reference, Text-to-Image, and Image-Editing metrics evaluate outputs that include visual content.

**Synthetic Data Generation**: The `Synthesizer` class generates evaluation datasets from documents, pre-prepared contexts, existing Goldens, or from scratch. Seven evolution techniques (Multicontext, Concretizing, Constrained, Comparative, Reasoning, Hypothetical, In-Breadth) progressively increase complexity.

**Red-Teaming with DeepTeam**: Automated adversarial testing against 40+ vulnerabilities across data privacy, responsible AI, security, safety, business risk, and agentic categories. Attack methods include prompt injection, jailbreaking, ROT13 encoding, and multi-turn refinement.

**Regression Testing**: Compare evaluation results across model versions, prompt iterations, or configuration changes with side-by-side scoring comparisons.

**Async Execution**: All metrics support asynchronous execution by default via `async_mode=True`, enabling concurrent evaluation of multiple test cases.

## Use Cases

**RAG Pipeline Validation**: Evaluate retrieval quality and generation faithfulness across a dataset of questions, verifying that the RAG system retrieves relevant context and produces grounded answers.

**Agent Behavior Testing**: Validate that autonomous agents select the correct tools, pass appropriate arguments, and complete tasks efficiently by scoring against agentic metrics.

**Chatbot Quality Assurance**: Assess multi-turn conversational agents for knowledge retention, role adherence, and conversation completeness across simulated user interactions.

**Safety Compliance Testing**: Run red-teaming scans against production LLM applications to identify vulnerabilities to prompt injection, data leakage, bias, toxicity, and other safety risks before deployment.

**Continuous Integration (CI) Evaluation**: Integrate DeepEval into CI/CD pipelines using the Pytest runner to automatically evaluate LLM outputs on every code change, catching regressions before they reach production.

**Model Comparison**: Generate a fixed evaluation dataset and run it against multiple models or prompt configurations to identify the best-performing combination for a specific use case.

**Synthetic Dataset Generation**: Bootstrap evaluation suites for new LLM applications by generating Goldens from domain documents, eliminating the need for manual test case creation.

**Custom Criteria Evaluation**: Define domain-specific evaluation criteria (helpfulness, tone, compliance with business rules) using G-Eval and apply them consistently across test suites.

## API Reference Summary

**Test Cases**: `LLMTestCase(input, actual_output, expected_output, context, retrieval_context, tools_called, expected_tools, token_cost, completion_time)` -- single interaction unit. `ConversationalTestCase` -- multi-turn dialogue with `Turn` objects. `MLLMTestCase` -- multimodal test case with `MLLMImage` objects. `ToolCall(name, description, reasoning, output, input_parameters)` -- agent tool invocation record.

**Datasets**: `EvaluationDataset(goldens)` -- collection of Goldens. `Golden(input, expected_output, context, expected_tools, additional_metadata)` -- single-turn ideal pair. `ConversationalGolden(scenario, expected_outcome, user_description, context)` -- multi-turn ideal pair. Methods: `add_golden()`, `add_test_case()`, `push(alias)`, `pull(alias)`, `save_as(file_type, directory)`, `add_goldens_from_json_file()`, `add_goldens_from_csv_file()`.

**Metrics**: `GEval(name, criteria, evaluation_params, threshold)` -- custom criteria evaluation. `AnswerRelevancyMetric(threshold)` -- RAG answer relevance. `FaithfulnessMetric(threshold)` -- RAG faithfulness to context. `ContextualRelevancyMetric(threshold)` -- retrieval relevance. `ContextualPrecisionMetric(threshold)` -- retrieval precision. `ContextualRecallMetric(threshold)` -- retrieval recall. `BiasMetric(threshold)` -- bias detection. `ToxicityMetric(threshold)` -- toxicity detection. `HallucinationMetric(threshold)` -- hallucination detection. `SummarizationMetric(threshold)` -- summarization quality. `TaskCompletionMetric(threshold)` -- agent task completion. `ToolCorrectnessMetric(threshold)` -- agent tool selection accuracy. All metrics expose `measure(test_case)`, `a_measure(test_case)`, `.score`, `.reason`, and `.status`.

**Evaluation**: `assert_test(test_case, metrics)` -- Pytest-style assertion that fails if any metric is below threshold. `evaluate(test_cases, metrics)` -- batch evaluation returning aggregate results.

**Synthesizer**: `Synthesizer(model, async_mode, max_concurrent, filtration_config, evolution_config)` -- dataset generator. Methods: `generate_goldens_from_docs(document_paths)`, `generate_goldens_from_contexts(contexts)`, `generate_goldens_from_scratch()`, `generate_goldens_from_goldens(goldens)`.

**Red-Teaming**: `red_team(model_callback, vulnerabilities, attacks)` -- functional API. `RedTeamer()` -- stateful class with attack caching. Vulnerabilities: `Bias`, `Toxicity`, `PIILeakage`, `PromptLeakage`, `SQLInjection`, and 35+ more. Attacks: `PromptInjection`, `Jailbreaking`, `ROT13`, and others.

**Tracing**: `@observe()` -- decorator for component-level instrumentation. `update_current_span(test_case)` -- attach test case to the current traced span.

## Configuration and Customization

**Metric Thresholds**: Every metric accepts a `threshold` parameter (default 0.5). A test case passes when its score equals or exceeds the threshold:

```python
metric = AnswerRelevancyMetric(threshold=0.7)
```

**Strict Mode**: When `strict_mode=True`, scores are binarized to 0 or 1 rather than continuous values:

```python
metric = GEval(
    name="Correctness",
    criteria="Is the output factually correct?",
    evaluation_params=[LLMTestCaseParams.ACTUAL_OUTPUT],
    strict_mode=True,
)
```

**Verbose Mode**: Enable detailed debug output during metric evaluation:

```python
metric = AnswerRelevancyMetric(threshold=0.5, verbose_mode=True)
```

**Custom LLM Judge**: Override the default OpenAI judge with any provider:

```python
metric = GEval(
    name="Correctness",
    criteria="Is the output correct?",
    evaluation_params=[LLMTestCaseParams.ACTUAL_OUTPUT],
    model="gpt-4o",
)
```

**Custom LLM Provider**: Implement a custom judge by inheriting from `DeepEvalBaseLLM`:

```python
from deepeval.models import DeepEvalBaseLLM

class CustomLLM(DeepEvalBaseLLM):
    def get_model_name(self):
        return "custom-model"

    def load_model(self):
        return self.model

    def generate(self, prompt: str) -> str:
        # Call your model here
        return response

    async def a_generate(self, prompt: str) -> str:
        # Async implementation
        return response
```

**Custom Evaluation Templates**: Override the prompt template used by a metric:

```python
from deepeval.metrics import AnswerRelevancyMetric
from deepeval.metrics.answer_relevancy import AnswerRelevancyTemplate

class CustomTemplate(AnswerRelevancyTemplate):
    @staticmethod
    def generate_statements(actual_output: str):
        return f"Custom evaluation prompt: {actual_output}"

metric = AnswerRelevancyMetric(evaluation_template=CustomTemplate)
```

**Results Storage**: Configure where evaluation results are saved locally:

```bash
export DEEPEVAL_RESULTS_FOLDER="./my-results"
```

**Synthetic Data Configuration**: Control data generation quality and complexity:

```python
from deepeval.synthesizer import Synthesizer
from deepeval.synthesizer.config import EvolutionConfig, FiltrationConfig

synthesizer = Synthesizer(
    model="gpt-4.1",
    max_concurrent=100,
    evolution_config=EvolutionConfig(num_evolutions=3),
    filtration_config=FiltrationConfig(max_quality_retries=3),
)
```

## Integration Patterns

**Pytest Integration**: DeepEval integrates natively with Pytest, enabling LLM evaluation to run as standard test suites within existing CI/CD pipelines:

```python
import pytest
from deepeval import assert_test
from deepeval.metrics import AnswerRelevancyMetric
from deepeval.test_case import LLMTestCase

@pytest.mark.parametrize("test_case", dataset.test_cases)
def test_llm_output(test_case: LLMTestCase):
    assert_test(test_case, [AnswerRelevancyMetric(threshold=0.7)])
```

Execute with: `deepeval test run test_llm_output.py`

**Provider Flexibility**: DeepEval supports OpenAI, Azure OpenAI, Anthropic, Google Gemini, Ollama, and custom LLM implementations as the judge model. The target application being evaluated can use any provider or framework.

**Component-Level Evaluation**: Use the `@observe` decorator to trace and evaluate individual pipeline components:

```python
from deepeval import observe, update_current_span
from deepeval.metrics import AnswerRelevancyMetric
from deepeval.test_case import LLMTestCase

@observe()
def llm_app(user_input: str):
    context = retrieve_documents(user_input)
    response = generate_answer(user_input, context)

    @observe(metrics=[AnswerRelevancyMetric()])
    def evaluate_generation():
        update_current_span(
            test_case=LLMTestCase(input=user_input, actual_output=response)
        )
    evaluate_generation()
    return response
```

**Dataset-Driven Evaluation**: Load Goldens, generate outputs, and evaluate in a structured pipeline:

```python
from deepeval.dataset import EvaluationDataset, Golden
from deepeval import evaluate
from deepeval.metrics import AnswerRelevancyMetric

dataset = EvaluationDataset(goldens=[
    Golden(input="What is your return policy?"),
    Golden(input="How do I track my order?"),
])

for golden in dataset.goldens:
    test_case = LLMTestCase(
        input=golden.input,
        actual_output=your_llm_app(golden.input),
    )
    dataset.add_test_case(test_case)

evaluate(test_cases=dataset.test_cases, metrics=[AnswerRelevancyMetric()])
```

**Confident AI Cloud Integration**: Push datasets and results to the cloud platform for centralized reporting and collaboration:

```python
dataset.push(alias="production-eval-v2")
```

**Red-Teaming Pipeline**: Integrate adversarial testing into pre-deployment safety checks:

```python
from deepteam import red_team
from deepteam.vulnerabilities import Bias, Toxicity
from deepteam.attacks.single_turn import PromptInjection

async def model_callback(input: str) -> str:
    return your_llm_app(input)

risk_assessment = red_team(
    model_callback=model_callback,
    vulnerabilities=[Bias(types=["race", "gender"]), Toxicity()],
    attacks=[PromptInjection()],
)
risk_assessment.save(to="./red-team-results/")
```

## Examples

**Basic Correctness Evaluation**:

```python
from deepeval import assert_test
from deepeval.metrics import GEval
from deepeval.test_case import LLMTestCase, LLMTestCaseParams

correctness_metric = GEval(
    name="Correctness",
    criteria="Determine whether the actual output is factually correct based on the expected output.",
    evaluation_params=[
        LLMTestCaseParams.ACTUAL_OUTPUT,
        LLMTestCaseParams.EXPECTED_OUTPUT,
    ],
    threshold=0.5,
)

test_case = LLMTestCase(
    input="What if these shoes don't fit?",
    actual_output="We offer a 30-day full refund at no extra cost.",
    expected_output="You're eligible for a 30 day refund at no extra cost.",
)

assert_test(test_case, [correctness_metric])
```

**RAG Pipeline Evaluation**:

```python
from deepeval import evaluate
from deepeval.metrics import (
    AnswerRelevancyMetric,
    FaithfulnessMetric,
    ContextualRelevancyMetric,
)
from deepeval.test_case import LLMTestCase

test_case = LLMTestCase(
    input="What is the refund policy for electronics?",
    actual_output="Electronics can be returned within 14 days for a full refund.",
    context=["Electronics have a 14-day return window. Full refund is issued upon return."],
    retrieval_context=["Electronics returns: 14 days. Refund policy applies to all items."],
)

evaluate(
    test_cases=[test_case],
    metrics=[
        AnswerRelevancyMetric(threshold=0.7),
        FaithfulnessMetric(threshold=0.7),
        ContextualRelevancyMetric(threshold=0.7),
    ],
)
```

**Agent Tool-Calling Evaluation**:

```python
from deepeval import assert_test
from deepeval.metrics import ToolCorrectnessMetric
from deepeval.test_case import LLMTestCase, ToolCall

test_case = LLMTestCase(
    input="What's the weather in San Francisco?",
    actual_output="It's 72F and sunny in San Francisco.",
    tools_called=[ToolCall(name="get_weather", input_parameters={"city": "San Francisco"})],
    expected_tools=[ToolCall(name="get_weather")],
)

assert_test(test_case, [ToolCorrectnessMetric(threshold=0.5)])
```

**Async Batch Evaluation**:

```python
import asyncio
from deepeval.metrics import AnswerRelevancyMetric, FaithfulnessMetric
from deepeval.test_case import LLMTestCase

test_case = LLMTestCase(
    input="Explain quantum computing.",
    actual_output="Quantum computing uses qubits that can exist in superposition.",
    context=["Quantum computers use quantum bits (qubits) that leverage superposition and entanglement."],
)

async def run_evaluation():
    relevancy = AnswerRelevancyMetric(threshold=0.7)
    faithfulness = FaithfulnessMetric(threshold=0.7)
    await asyncio.gather(
        relevancy.a_measure(test_case),
        faithfulness.a_measure(test_case),
    )
    print(f"Relevancy: {relevancy.score} - {relevancy.reason}")
    print(f"Faithfulness: {faithfulness.score} - {faithfulness.reason}")

asyncio.run(run_evaluation())
```

**Synthetic Dataset Generation from Documents**:

```python
from deepeval.synthesizer import Synthesizer
from deepeval.dataset import EvaluationDataset

synthesizer = Synthesizer(model="gpt-4.1")
goldens = synthesizer.generate_goldens_from_docs(
    document_paths=["knowledge_base.pdf", "faq.txt"],
    include_expected_output=True,
)

dataset = EvaluationDataset(goldens=goldens)
dataset.save_as(file_type="json", directory="./eval-datasets")
```

## Limitations and Considerations

**LLM Judge Dependency**: Most metrics require an LLM judge call, which introduces latency and cost per evaluation. Running 50+ metrics across large datasets can become expensive, particularly with high-capability judge models like GPT-4o.

**Judge Model Variability**: Evaluation scores can vary depending on which LLM is used as the judge. Scores produced by GPT-4o may differ from those produced by a local Ollama model, making cross-judge comparisons unreliable.

**Metric Selection Complexity**: With over 50 available metrics, selecting the right subset for a given use case requires domain knowledge. The documentation recommends limiting to 5 metrics maximum (2-3 system-specific, 1-2 custom) to avoid evaluation fatigue and conflicting signals.

**Confident AI Lock-In**: While the core framework is open source, advanced features such as centralized reporting, regression dashboards, prompt optimization, and production monitoring require the Confident AI cloud platform. Teams running entirely self-hosted may not have access to these capabilities.

**Red-Teaming Scope**: DeepTeam (the red-teaming component) was separated into its own package and documentation site. The boundary between DeepEval and DeepTeam can be confusing, and vulnerability coverage depends on the attack methods employed rather than providing exhaustive security guarantees.

**Synthetic Data Quality**: Generated Goldens depend on the quality of source documents and the generating model. Filtration and evolution improve quality, but manual review of generated datasets is still recommended before using them as evaluation benchmarks.

**Rate Limiting**: When evaluating large datasets, the number of LLM judge calls can trigger rate limits from providers. DeepEval includes exponential backoff retry logic (1-second initial delay, 2x base, 5-second cap), but throughput-sensitive pipelines may need additional rate management.

**Threshold Sensitivity**: The default 0.5 threshold is permissive. Teams must calibrate thresholds based on their quality requirements, and scores near the threshold boundary may produce inconsistent pass/fail results across evaluation runs due to LLM judge non-determinism.

## Changelog Highlights

- **50+ metrics library**: Expanded from initial release to over 50 research-backed metrics covering RAG, agentic, conversational, safety, and multimodal evaluation.
- **DeepTeam separation**: Red-teaming capabilities extracted into a dedicated `deepteam` package with its own documentation and CLI, supporting 40+ vulnerabilities and 10+ attack methods.
- **Multimodal support**: Added `MLLMTestCase` and `MLLMImage` for evaluating applications that produce or consume images alongside text.
- **Agentic metrics**: Introduced Task Completion, Tool Correctness, Argument Correctness, Step Efficiency, Plan Adherence, and Plan Quality for evaluating autonomous agent behavior.
- **DAG metrics**: Added Deep Acyclic Graph evaluation as a structured alternative to G-Eval for complex criteria.
- **Arena G-Eval**: Comparative evaluation metric for head-to-head model comparison.
- **Conversation Simulator**: Added `ConversationSimulator` for generating multi-turn test scenarios with configurable user profiles and stopping criteria.
- **Synthesizer evolution**: Seven evolution techniques (Multicontext, Concretizing, Constrained, Comparative, Reasoning, Hypothetical, In-Breadth) for progressive dataset complexity.

## Citations

- [1] DeepEval Documentation - https://deepeval.com/docs/getting-started
- [2] DeepEval GitHub Repository - https://github.com/confident-ai/deepeval
- [3] DeepTeam Red-Teaming Documentation - https://www.trydeepteam.com/docs/red-teaming-introduction
- [4] Confident AI Platform - https://app.confident-ai.com
