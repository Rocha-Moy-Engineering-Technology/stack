[Header 1 ("deepeval", [], []) [Str "DeepEval"], BlockQuote [Para [Str "LLM evaluation framework with 50+ research-backed metrics and Pytest integration"]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.5)), (AlignDefault, (ColWidth 0.5))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Evaluation & Testing"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "SDK"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Yes"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "https://github.com/confident-ai/deepeval"] ("https://github.com/confident-ai/deepeval", "")]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "13756"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Documentation"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "Official Docs"] ("https://deepeval.com/docs/getting-started", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "DeepEval is an open-source evaluation framework for Large Language Models (LLMs), designed to bring the rigor of unit testing to LLM application development. Built by Confident AI, it functions as \"Pytest for LLM outputs,\" enabling developers to write deterministic, repeatable evaluation suites that validate LLM behavior against research-backed metrics. The framework scores every metric on a normalized 0-to-1 scale with configurable pass/fail thresholds, supporting both single-turn and multi-turn conversational evaluations."], Para [Str "DeepEval provides over 50 plug-and-use metrics spanning Retrieval-Augmented Generation (RAG) quality, agentic behavior, conversational coherence, safety, bias, multimodal content, and custom criteria. It supports two evaluation modes: end-to-end (black-box) testing that treats the application as a unified system, and component-level (white-box) testing that uses the ", Code ("", [], []) "@observe", Str " decorator for tracing and evaluating individual components within a pipeline."], Para [Str "Beyond evaluation, DeepEval includes synthetic dataset generation with evolution techniques, regression testing for comparing model versions side-by-side, and integration with DeepTeam for red-teaming LLM applications against 40+ vulnerability categories using 10+ adversarial attack methods. An optional cloud platform, Confident AI, provides centralized reporting, dataset curation, prompt optimization, and production monitoring."], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], Para [Strong [Str "LLMTestCase"], Str " is the fundamental unit of evaluation, representing a single atomic interaction with an LLM application. It requires an ", Code ("", [], []) "input", Str " field (the user query) and an ", Code ("", [], []) "actual_output", Str " field (what the LLM produced). Optional fields include ", Code ("", [], []) "expected_output", Str ", ", Code ("", [], []) "context", Str " (ground truth facts), ", Code ("", [], []) "retrieval_context", Str " (what the RAG pipeline actually retrieved), ", Code ("", [], []) "tools_called", Str ", ", Code ("", [], []) "expected_tools", Str ", ", Code ("", [], []) "token_cost", Str ", and ", Code ("", [], []) "completion_time", Str ". Test cases are evaluated against one or more metrics to produce scores and pass/fail determinations."], Para [Strong [Str "ConversationalTestCase"], Str " extends evaluation to multi-turn interactions. It contains a sequence of ", Code ("", [], []) "Turn", Str " objects, each with a ", Code ("", [], []) "role", Str " and ", Code ("", [], []) "content", Str ", representing a full dialogue between a user and an LLM assistant. Conversational metrics such as Knowledge Retention and Role Adherence operate on these multi-turn structures."], Para [Strong [Str "MLLMTestCase"], Str " supports multimodal evaluation, enabling test cases that include both text and images via ", Code ("", [], []) "MLLMImage", Str " objects. Images can be provided as local file paths, remote URLs, or base64-encoded data."], Para [Strong [Str "Goldens"], Str " are ideal input-output pairs that serve as reusable templates for evaluation. Unlike test cases, Goldens require only the ", Code ("", [], []) "input", Str " field for initialization and do not need an ", Code ("", [], []) "actual_output", Str " until evaluation time. They represent the expected behavior of the system and can be reused across multiple LLM iterations without regeneration."], Para [Strong [Str "EvaluationDataset"], Str " is a collection of Goldens (single-turn ", Code ("", [], []) "Golden", Str " objects or multi-turn ", Code ("", [], []) "ConversationalGolden", Str " objects). Datasets manage the lifecycle of evaluation data: loading Goldens, generating actual outputs by running them through the LLM application, converting them to test cases, and executing metrics. Datasets can be persisted locally as JSON or CSV, or pushed to the Confident AI cloud platform."], Para [Strong [Str "Metrics"], Str " are scoring functions that evaluate test cases on a 0-to-1 scale. Each metric produces a score, a human-readable reason explaining the score, and a pass/fail status based on a configurable threshold (default 0.5). Most metrics use an \"LLM-as-a-judge\" approach, where a separate LLM evaluates the quality of the output."], Para [Strong [Str "Tracing"], Str " uses the ", Code ("", [], []) "@observe", Str " decorator to instrument LLM application components, creating a hierarchical trace of execution. Traces capture inputs, outputs, and intermediate states at each level, enabling component-level evaluation where individual pipeline stages (retriever, generator, tool caller) are scored independently."], Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Para [Str "DeepEval is organized around a layered evaluation pipeline:"], CodeBlock ("", [""], []) "Test Cases / Goldens          Atomic units of LLM interaction (input + output)
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
", Para [Strong [Str "Evaluation Pipeline"], Str ": Goldens are loaded into an ", Code ("", [], []) "EvaluationDataset", Str ", processed through the target LLM application to generate ", Code ("", [], []) "actual_output", Str " values, converted into test cases, and then evaluated against selected metrics. Each metric invokes the configured LLM judge to produce a score and reasoning."], Para [Strong [Str "Metric Evaluation Techniques"], Str ": Metrics employ several judge-based approaches including G-Eval (criteria-based evaluation with chain-of-thought), Question-Answer Generation (QAG) for factual verification, and Deep Acyclic Graphs (DAG) for structured evaluation flows."], Para [Strong [Str "Tracing Layer"], Str ": The ", Code ("", [], []) "@observe", Str " decorator instruments application code to create hierarchical traces. Each decorated function becomes a span within a trace, and metrics can be attached to individual spans for component-level evaluation rather than end-to-end only."], Para [Strong [Str "Synthetic Data Pipeline"], Str ": The ", Code ("", [], []) "Synthesizer", Str " follows a four-step process -- input generation, filtration (quality scoring on self-containment and clarity), evolution (complexity escalation through 7 techniques), and styling (format customization)."], Para [Strong [Str "Red-Teaming Pipeline"], Str ": DeepTeam generates baseline adversarial attacks, enhances them using attack methods (prompt injection, jailbreaking, encoding), feeds them to the target LLM, and scores responses using vulnerability-specific metrics in strict binary mode (0 or 1)."], Header 2 ("key-features-and-functionality", ["unnumbered", "unlisted"], []) [Str "Key Features and Functionality"], Para [Strong [Str "G-Eval Custom Metrics"], Str ": Define evaluation criteria in natural language and let an LLM judge score outputs accordingly. G-Eval achieves human-like evaluation accuracy by using chain-of-thought reasoning before assigning scores."], CodeBlock ("", ["python"], []) "from deepeval.metrics import GEval
from deepeval.test_case import LLMTestCaseParams

correctness_metric = GEval(
    name=\"Correctness\",
    criteria=\"Determine whether the actual output is factually correct based on the expected output.\",
    evaluation_params=[
        LLMTestCaseParams.ACTUAL_OUTPUT,
        LLMTestCaseParams.EXPECTED_OUTPUT,
    ],
    threshold=0.5,
)
", Para [Strong [Str "RAG Evaluation"], Str ": Dedicated metrics for both the retriever and generator stages of RAG pipelines. Retriever metrics (Contextual Relevancy, Contextual Precision, Contextual Recall) evaluate whether the right documents were retrieved. Generator metrics (Answer Relevancy, Faithfulness) evaluate whether the LLM produced accurate, grounded responses."], Para [Strong [Str "Agentic Metrics"], Str ": Evaluate autonomous agent behavior with Task Completion, Tool Correctness, Argument Correctness, Step Efficiency, Plan Adherence, and Plan Quality metrics. Test cases include ", Code ("", [], []) "tools_called", Str " and ", Code ("", [], []) "expected_tools", Str " fields for verifying agent tool usage."], Para [Strong [Str "Conversational Metrics"], Str ": Assess multi-turn dialogue quality with Knowledge Retention (does the assistant remember earlier context), Role Adherence (does it stay in character), Conversation Completeness, and Conversation Relevancy."], Para [Strong [Str "Safety and Bias Detection"], Str ": Metrics for Bias, Toxicity, PII Leakage, Role Violation, Misuse, and Non-Advice detect harmful or inappropriate outputs."], Para [Strong [Str "Multimodal Evaluation"], Str ": Image Coherence, Image Helpfulness, Image Reference, Text-to-Image, and Image-Editing metrics evaluate outputs that include visual content."], Para [Strong [Str "Synthetic Data Generation"], Str ": The ", Code ("", [], []) "Synthesizer", Str " class generates evaluation datasets from documents, pre-prepared contexts, existing Goldens, or from scratch. Seven evolution techniques (Multicontext, Concretizing, Constrained, Comparative, Reasoning, Hypothetical, In-Breadth) progressively increase complexity."], Para [Strong [Str "Red-Teaming with DeepTeam"], Str ": Automated adversarial testing against 40+ vulnerabilities across data privacy, responsible AI, security, safety, business risk, and agentic categories. Attack methods include prompt injection, jailbreaking, ROT13 encoding, and multi-turn refinement."], Para [Strong [Str "Regression Testing"], Str ": Compare evaluation results across model versions, prompt iterations, or configuration changes with side-by-side scoring comparisons."], Para [Strong [Str "Async Execution"], Str ": All metrics support asynchronous execution by default via ", Code ("", [], []) "async_mode=True", Str ", enabling concurrent evaluation of multiple test cases."], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], Para [Strong [Str "RAG Pipeline Validation"], Str ": Evaluate retrieval quality and generation faithfulness across a dataset of questions, verifying that the RAG system retrieves relevant context and produces grounded answers."], Para [Strong [Str "Agent Behavior Testing"], Str ": Validate that autonomous agents select the correct tools, pass appropriate arguments, and complete tasks efficiently by scoring against agentic metrics."], Para [Strong [Str "Chatbot Quality Assurance"], Str ": Assess multi-turn conversational agents for knowledge retention, role adherence, and conversation completeness across simulated user interactions."], Para [Strong [Str "Safety Compliance Testing"], Str ": Run red-teaming scans against production LLM applications to identify vulnerabilities to prompt injection, data leakage, bias, toxicity, and other safety risks before deployment."], Para [Strong [Str "Continuous Integration (CI) Evaluation"], Str ": Integrate DeepEval into CI/CD pipelines using the Pytest runner to automatically evaluate LLM outputs on every code change, catching regressions before they reach production."], Para [Strong [Str "Model Comparison"], Str ": Generate a fixed evaluation dataset and run it against multiple models or prompt configurations to identify the best-performing combination for a specific use case."], Para [Strong [Str "Synthetic Dataset Generation"], Str ": Bootstrap evaluation suites for new LLM applications by generating Goldens from domain documents, eliminating the need for manual test case creation."], Para [Strong [Str "Custom Criteria Evaluation"], Str ": Define domain-specific evaluation criteria (helpfulness, tone, compliance with business rules) using G-Eval and apply them consistently across test suites."], Header 2 ("api-reference-summary", ["unnumbered", "unlisted"], []) [Str "API Reference Summary"], Para [Strong [Str "Test Cases"], Str ": ", Code ("", [], []) "LLMTestCase(input, actual_output, expected_output, context, retrieval_context, tools_called, expected_tools, token_cost, completion_time)", Str " -- single interaction unit. ", Code ("", [], []) "ConversationalTestCase", Str " -- multi-turn dialogue with ", Code ("", [], []) "Turn", Str " objects. ", Code ("", [], []) "MLLMTestCase", Str " -- multimodal test case with ", Code ("", [], []) "MLLMImage", Str " objects. ", Code ("", [], []) "ToolCall(name, description, reasoning, output, input_parameters)", Str " -- agent tool invocation record."], Para [Strong [Str "Datasets"], Str ": ", Code ("", [], []) "EvaluationDataset(goldens)", Str " -- collection of Goldens. ", Code ("", [], []) "Golden(input, expected_output, context, expected_tools, additional_metadata)", Str " -- single-turn ideal pair. ", Code ("", [], []) "ConversationalGolden(scenario, expected_outcome, user_description, context)", Str " -- multi-turn ideal pair. Methods: ", Code ("", [], []) "add_golden()", Str ", ", Code ("", [], []) "add_test_case()", Str ", ", Code ("", [], []) "push(alias)", Str ", ", Code ("", [], []) "pull(alias)", Str ", ", Code ("", [], []) "save_as(file_type, directory)", Str ", ", Code ("", [], []) "add_goldens_from_json_file()", Str ", ", Code ("", [], []) "add_goldens_from_csv_file()", Str "."], Para [Strong [Str "Metrics"], Str ": ", Code ("", [], []) "GEval(name, criteria, evaluation_params, threshold)", Str " -- custom criteria evaluation. ", Code ("", [], []) "AnswerRelevancyMetric(threshold)", Str " -- RAG answer relevance. ", Code ("", [], []) "FaithfulnessMetric(threshold)", Str " -- RAG faithfulness to context. ", Code ("", [], []) "ContextualRelevancyMetric(threshold)", Str " -- retrieval relevance. ", Code ("", [], []) "ContextualPrecisionMetric(threshold)", Str " -- retrieval precision. ", Code ("", [], []) "ContextualRecallMetric(threshold)", Str " -- retrieval recall. ", Code ("", [], []) "BiasMetric(threshold)", Str " -- bias detection. ", Code ("", [], []) "ToxicityMetric(threshold)", Str " -- toxicity detection. ", Code ("", [], []) "HallucinationMetric(threshold)", Str " -- hallucination detection. ", Code ("", [], []) "SummarizationMetric(threshold)", Str " -- summarization quality. ", Code ("", [], []) "TaskCompletionMetric(threshold)", Str " -- agent task completion. ", Code ("", [], []) "ToolCorrectnessMetric(threshold)", Str " -- agent tool selection accuracy. All metrics expose ", Code ("", [], []) "measure(test_case)", Str ", ", Code ("", [], []) "a_measure(test_case)", Str ", ", Code ("", [], []) ".score", Str ", ", Code ("", [], []) ".reason", Str ", and ", Code ("", [], []) ".status", Str "."], Para [Strong [Str "Evaluation"], Str ": ", Code ("", [], []) "assert_test(test_case, metrics)", Str " -- Pytest-style assertion that fails if any metric is below threshold. ", Code ("", [], []) "evaluate(test_cases, metrics)", Str " -- batch evaluation returning aggregate results."], Para [Strong [Str "Synthesizer"], Str ": ", Code ("", [], []) "Synthesizer(model, async_mode, max_concurrent, filtration_config, evolution_config)", Str " -- dataset generator. Methods: ", Code ("", [], []) "generate_goldens_from_docs(document_paths)", Str ", ", Code ("", [], []) "generate_goldens_from_contexts(contexts)", Str ", ", Code ("", [], []) "generate_goldens_from_scratch()", Str ", ", Code ("", [], []) "generate_goldens_from_goldens(goldens)", Str "."], Para [Strong [Str "Red-Teaming"], Str ": ", Code ("", [], []) "red_team(model_callback, vulnerabilities, attacks)", Str " -- functional API. ", Code ("", [], []) "RedTeamer()", Str " -- stateful class with attack caching. Vulnerabilities: ", Code ("", [], []) "Bias", Str ", ", Code ("", [], []) "Toxicity", Str ", ", Code ("", [], []) "PIILeakage", Str ", ", Code ("", [], []) "PromptLeakage", Str ", ", Code ("", [], []) "SQLInjection", Str ", and 35+ more. Attacks: ", Code ("", [], []) "PromptInjection", Str ", ", Code ("", [], []) "Jailbreaking", Str ", ", Code ("", [], []) "ROT13", Str ", and others."], Para [Strong [Str "Tracing"], Str ": ", Code ("", [], []) "@observe()", Str " -- decorator for component-level instrumentation. ", Code ("", [], []) "update_current_span(test_case)", Str " -- attach test case to the current traced span."], Header 2 ("configuration-and-customization", ["unnumbered", "unlisted"], []) [Str "Configuration and Customization"], Para [Strong [Str "Metric Thresholds"], Str ": Every metric accepts a ", Code ("", [], []) "threshold", Str " parameter (default 0.5). A test case passes when its score equals or exceeds the threshold:"], CodeBlock ("", ["python"], []) "metric = AnswerRelevancyMetric(threshold=0.7)
", Para [Strong [Str "Strict Mode"], Str ": When ", Code ("", [], []) "strict_mode=True", Str ", scores are binarized to 0 or 1 rather than continuous values:"], CodeBlock ("", ["python"], []) "metric = GEval(
    name=\"Correctness\",
    criteria=\"Is the output factually correct?\",
    evaluation_params=[LLMTestCaseParams.ACTUAL_OUTPUT],
    strict_mode=True,
)
", Para [Strong [Str "Verbose Mode"], Str ": Enable detailed debug output during metric evaluation:"], CodeBlock ("", ["python"], []) "metric = AnswerRelevancyMetric(threshold=0.5, verbose_mode=True)
", Para [Strong [Str "Custom LLM Judge"], Str ": Override the default OpenAI judge with any provider:"], CodeBlock ("", ["python"], []) "metric = GEval(
    name=\"Correctness\",
    criteria=\"Is the output correct?\",
    evaluation_params=[LLMTestCaseParams.ACTUAL_OUTPUT],
    model=\"gpt-4o\",
)
", Para [Strong [Str "Custom LLM Provider"], Str ": Implement a custom judge by inheriting from ", Code ("", [], []) "DeepEvalBaseLLM", Str ":"], CodeBlock ("", ["python"], []) "from deepeval.models import DeepEvalBaseLLM

class CustomLLM(DeepEvalBaseLLM):
    def get_model_name(self):
        return \"custom-model\"

    def load_model(self):
        return self.model

    def generate(self, prompt: str) -> str:
        # Call your model here
        return response

    async def a_generate(self, prompt: str) -> str:
        # Async implementation
        return response
", Para [Strong [Str "Custom Evaluation Templates"], Str ": Override the prompt template used by a metric:"], CodeBlock ("", ["python"], []) "from deepeval.metrics import AnswerRelevancyMetric
from deepeval.metrics.answer_relevancy import AnswerRelevancyTemplate

class CustomTemplate(AnswerRelevancyTemplate):
    @staticmethod
    def generate_statements(actual_output: str):
        return f\"Custom evaluation prompt: {actual_output}\"

metric = AnswerRelevancyMetric(evaluation_template=CustomTemplate)
", Para [Strong [Str "Results Storage"], Str ": Configure where evaluation results are saved locally:"], CodeBlock ("", ["bash"], []) "export DEEPEVAL_RESULTS_FOLDER=\"./my-results\"
", Para [Strong [Str "Synthetic Data Configuration"], Str ": Control data generation quality and complexity:"], CodeBlock ("", ["python"], []) "from deepeval.synthesizer import Synthesizer
from deepeval.synthesizer.config import EvolutionConfig, FiltrationConfig

synthesizer = Synthesizer(
    model=\"gpt-4.1\",
    max_concurrent=100,
    evolution_config=EvolutionConfig(num_evolutions=3),
    filtration_config=FiltrationConfig(max_quality_retries=3),
)
", Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Para [Strong [Str "Pytest Integration"], Str ": DeepEval integrates natively with Pytest, enabling LLM evaluation to run as standard test suites within existing CI/CD pipelines:"], CodeBlock ("", ["python"], []) "import pytest
from deepeval import assert_test
from deepeval.metrics import AnswerRelevancyMetric
from deepeval.test_case import LLMTestCase

@pytest.mark.parametrize(\"test_case\", dataset.test_cases)
def test_llm_output(test_case: LLMTestCase):
    assert_test(test_case, [AnswerRelevancyMetric(threshold=0.7)])
", Para [Str "Execute with: ", Code ("", [], []) "deepeval test run test_llm_output.py"], Para [Strong [Str "Provider Flexibility"], Str ": DeepEval supports OpenAI, Azure OpenAI, Anthropic, Google Gemini, Ollama, and custom LLM implementations as the judge model. The target application being evaluated can use any provider or framework."], Para [Strong [Str "Component-Level Evaluation"], Str ": Use the ", Code ("", [], []) "@observe", Str " decorator to trace and evaluate individual pipeline components:"], CodeBlock ("", ["python"], []) "from deepeval import observe, update_current_span
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
", Para [Strong [Str "Dataset-Driven Evaluation"], Str ": Load Goldens, generate outputs, and evaluate in a structured pipeline:"], CodeBlock ("", ["python"], []) "from deepeval.dataset import EvaluationDataset, Golden
from deepeval import evaluate
from deepeval.metrics import AnswerRelevancyMetric

dataset = EvaluationDataset(goldens=[
    Golden(input=\"What is your return policy?\"),
    Golden(input=\"How do I track my order?\"),
])

for golden in dataset.goldens:
    test_case = LLMTestCase(
        input=golden.input,
        actual_output=your_llm_app(golden.input),
    )
    dataset.add_test_case(test_case)

evaluate(test_cases=dataset.test_cases, metrics=[AnswerRelevancyMetric()])
", Para [Strong [Str "Confident AI Cloud Integration"], Str ": Push datasets and results to the cloud platform for centralized reporting and collaboration:"], CodeBlock ("", ["python"], []) "dataset.push(alias=\"production-eval-v2\")
", Para [Strong [Str "Red-Teaming Pipeline"], Str ": Integrate adversarial testing into pre-deployment safety checks:"], CodeBlock ("", ["python"], []) "from deepteam import red_team
from deepteam.vulnerabilities import Bias, Toxicity
from deepteam.attacks.single_turn import PromptInjection

async def model_callback(input: str) -> str:
    return your_llm_app(input)

risk_assessment = red_team(
    model_callback=model_callback,
    vulnerabilities=[Bias(types=[\"race\", \"gender\"]), Toxicity()],
    attacks=[PromptInjection()],
)
risk_assessment.save(to=\"./red-team-results/\")
", Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Para [Strong [Str "Basic Correctness Evaluation"], Str ":"], CodeBlock ("", ["python"], []) "from deepeval import assert_test
from deepeval.metrics import GEval
from deepeval.test_case import LLMTestCase, LLMTestCaseParams

correctness_metric = GEval(
    name=\"Correctness\",
    criteria=\"Determine whether the actual output is factually correct based on the expected output.\",
    evaluation_params=[
        LLMTestCaseParams.ACTUAL_OUTPUT,
        LLMTestCaseParams.EXPECTED_OUTPUT,
    ],
    threshold=0.5,
)

test_case = LLMTestCase(
    input=\"What if these shoes don't fit?\",
    actual_output=\"We offer a 30-day full refund at no extra cost.\",
    expected_output=\"You're eligible for a 30 day refund at no extra cost.\",
)

assert_test(test_case, [correctness_metric])
", Para [Strong [Str "RAG Pipeline Evaluation"], Str ":"], CodeBlock ("", ["python"], []) "from deepeval import evaluate
from deepeval.metrics import (
    AnswerRelevancyMetric,
    FaithfulnessMetric,
    ContextualRelevancyMetric,
)
from deepeval.test_case import LLMTestCase

test_case = LLMTestCase(
    input=\"What is the refund policy for electronics?\",
    actual_output=\"Electronics can be returned within 14 days for a full refund.\",
    context=[\"Electronics have a 14-day return window. Full refund is issued upon return.\"],
    retrieval_context=[\"Electronics returns: 14 days. Refund policy applies to all items.\"],
)

evaluate(
    test_cases=[test_case],
    metrics=[
        AnswerRelevancyMetric(threshold=0.7),
        FaithfulnessMetric(threshold=0.7),
        ContextualRelevancyMetric(threshold=0.7),
    ],
)
", Para [Strong [Str "Agent Tool-Calling Evaluation"], Str ":"], CodeBlock ("", ["python"], []) "from deepeval import assert_test
from deepeval.metrics import ToolCorrectnessMetric
from deepeval.test_case import LLMTestCase, ToolCall

test_case = LLMTestCase(
    input=\"What's the weather in San Francisco?\",
    actual_output=\"It's 72F and sunny in San Francisco.\",
    tools_called=[ToolCall(name=\"get_weather\", input_parameters={\"city\": \"San Francisco\"})],
    expected_tools=[ToolCall(name=\"get_weather\")],
)

assert_test(test_case, [ToolCorrectnessMetric(threshold=0.5)])
", Para [Strong [Str "Async Batch Evaluation"], Str ":"], CodeBlock ("", ["python"], []) "import asyncio
from deepeval.metrics import AnswerRelevancyMetric, FaithfulnessMetric
from deepeval.test_case import LLMTestCase

test_case = LLMTestCase(
    input=\"Explain quantum computing.\",
    actual_output=\"Quantum computing uses qubits that can exist in superposition.\",
    context=[\"Quantum computers use quantum bits (qubits) that leverage superposition and entanglement.\"],
)

async def run_evaluation():
    relevancy = AnswerRelevancyMetric(threshold=0.7)
    faithfulness = FaithfulnessMetric(threshold=0.7)
    await asyncio.gather(
        relevancy.a_measure(test_case),
        faithfulness.a_measure(test_case),
    )
    print(f\"Relevancy: {relevancy.score} - {relevancy.reason}\")
    print(f\"Faithfulness: {faithfulness.score} - {faithfulness.reason}\")

asyncio.run(run_evaluation())
", Para [Strong [Str "Synthetic Dataset Generation from Documents"], Str ":"], CodeBlock ("", ["python"], []) "from deepeval.synthesizer import Synthesizer
from deepeval.dataset import EvaluationDataset

synthesizer = Synthesizer(model=\"gpt-4.1\")
goldens = synthesizer.generate_goldens_from_docs(
    document_paths=[\"knowledge_base.pdf\", \"faq.txt\"],
    include_expected_output=True,
)

dataset = EvaluationDataset(goldens=goldens)
dataset.save_as(file_type=\"json\", directory=\"./eval-datasets\")
", Header 2 ("limitations-and-considerations", ["unnumbered", "unlisted"], []) [Str "Limitations and Considerations"], Para [Strong [Str "LLM Judge Dependency"], Str ": Most metrics require an LLM judge call, which introduces latency and cost per evaluation. Running 50+ metrics across large datasets can become expensive, particularly with high-capability judge models like GPT-4o."], Para [Strong [Str "Judge Model Variability"], Str ": Evaluation scores can vary depending on which LLM is used as the judge. Scores produced by GPT-4o may differ from those produced by a local Ollama model, making cross-judge comparisons unreliable."], Para [Strong [Str "Metric Selection Complexity"], Str ": With over 50 available metrics, selecting the right subset for a given use case requires domain knowledge. The documentation recommends limiting to 5 metrics maximum (2-3 system-specific, 1-2 custom) to avoid evaluation fatigue and conflicting signals."], Para [Strong [Str "Confident AI Lock-In"], Str ": While the core framework is open source, advanced features such as centralized reporting, regression dashboards, prompt optimization, and production monitoring require the Confident AI cloud platform. Teams running entirely self-hosted may not have access to these capabilities."], Para [Strong [Str "Red-Teaming Scope"], Str ": DeepTeam (the red-teaming component) was separated into its own package and documentation site. The boundary between DeepEval and DeepTeam can be confusing, and vulnerability coverage depends on the attack methods employed rather than providing exhaustive security guarantees."], Para [Strong [Str "Synthetic Data Quality"], Str ": Generated Goldens depend on the quality of source documents and the generating model. Filtration and evolution improve quality, but manual review of generated datasets is still recommended before using them as evaluation benchmarks."], Para [Strong [Str "Rate Limiting"], Str ": When evaluating large datasets, the number of LLM judge calls can trigger rate limits from providers. DeepEval includes exponential backoff retry logic (1-second initial delay, 2x base, 5-second cap), but throughput-sensitive pipelines may need additional rate management."], Para [Strong [Str "Threshold Sensitivity"], Str ": The default 0.5 threshold is permissive. Teams must calibrate thresholds based on their quality requirements, and scores near the threshold boundary may produce inconsistent pass/fail results across evaluation runs due to LLM judge non-determinism."], Header 2 ("changelog-highlights", ["unnumbered", "unlisted"], []) [Str "Changelog Highlights"], BulletList [[Plain [Strong [Str "50+ metrics library"], Str ": Expanded from initial release to over 50 research-backed metrics covering RAG, agentic, conversational, safety, and multimodal evaluation."]], [Plain [Strong [Str "DeepTeam separation"], Str ": Red-teaming capabilities extracted into a dedicated ", Code ("", [], []) "deepteam", Str " package with its own documentation and CLI, supporting 40+ vulnerabilities and 10+ attack methods."]], [Plain [Strong [Str "Multimodal support"], Str ": Added ", Code ("", [], []) "MLLMTestCase", Str " and ", Code ("", [], []) "MLLMImage", Str " for evaluating applications that produce or consume images alongside text."]], [Plain [Strong [Str "Agentic metrics"], Str ": Introduced Task Completion, Tool Correctness, Argument Correctness, Step Efficiency, Plan Adherence, and Plan Quality for evaluating autonomous agent behavior."]], [Plain [Strong [Str "DAG metrics"], Str ": Added Deep Acyclic Graph evaluation as a structured alternative to G-Eval for complex criteria."]], [Plain [Strong [Str "Arena G-Eval"], Str ": Comparative evaluation metric for head-to-head model comparison."]], [Plain [Strong [Str "Conversation Simulator"], Str ": Added ", Code ("", [], []) "ConversationSimulator", Str " for generating multi-turn test scenarios with configurable user profiles and stopping criteria."]], [Plain [Strong [Str "Synthesizer evolution"], Str ": Seven evolution techniques (Multicontext, Concretizing, Constrained, Comparative, Reasoning, Hypothetical, In-Breadth) for progressive dataset complexity."]]], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " DeepEval Documentation - https://deepeval.com/docs/getting-started"]], [Plain [Str "[", Str "2", Str "]", Str " DeepEval GitHub Repository - https://github.com/confident-ai/deepeval"]], [Plain [Str "[", Str "3", Str "]", Str " DeepTeam Red-Teaming Documentation - https://www.trydeepteam.com/docs/red-teaming-introduction"]], [Plain [Str "[", Str "4", Str "]", Str " Confident AI Platform - https://app.confident-ai.com"]]]]