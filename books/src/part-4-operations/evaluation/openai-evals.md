# OpenAI Evals

> OpenAI framework for creating and running evaluations on LLM systems

| Field | Value |
|-------|-------|
| Group | Evaluation & Testing |
| Type | SDK |
| Open Source | Yes |
| GitHub | [https://github.com/openai/evals](https://github.com/openai/evals) |
| Stars | 17876 |
| Documentation | [Official Docs](https://developers.openai.com/api/docs/guides/evals) |

## Overview

OpenAI Evals is a framework for systematically testing whether Large Language Model (LLM) outputs meet defined style and content criteria. It operates on a principle similar to Behavior-Driven Development (BDD): define expected behavior before implementation, then verify model outputs against those expectations. The framework spans two complementary surfaces -- an open-source Python library for local evaluation execution and a cloud-based Evals Application Programming Interface (API) integrated into the OpenAI platform.

The open-source repository at `github.com/openai/evals` provides a framework for building custom evaluations and a registry of pre-built benchmarks. The API-based counterpart, introduced in April 2025, enables programmatic evaluation creation, execution, and result retrieval through RESTful endpoints. Both approaches share the same conceptual model: define a task with expected outcomes, run model outputs against test data, and analyze whether results meet the defined criteria.

Evals are particularly valuable during prompt engineering, model selection, and pre-deployment validation. They transform subjective quality assessments into repeatable, quantifiable tests that can be embedded into Continuous Integration / Continuous Deployment (CI/CD) pipelines.

## Core Concepts

**Eval** is the top-level construct representing an evaluation definition. It specifies the data schema that test inputs must conform to and the testing criteria (graders) used to assess model outputs. An eval is reusable across multiple runs with different models, prompts, or data sets.

**Eval Run** is a single execution of an eval against a specific model configuration and data set. Each run produces results including pass/fail counts, per-criteria breakdowns, and token usage metrics. Runs execute asynchronously on OpenAI infrastructure.

**Data Source Configuration** defines the structure of test data using JavaScript Object Notation (JSON) Schema. It specifies the properties each test item must include (such as input text and expected labels) and whether model-generated output schemas should be available in grader templates.

**Graders** are the evaluation functions that determine whether a model output meets requirements. OpenAI provides five built-in grader types: string check, text similarity, score model, label model (available via the API for fine-tuning workflows), and Python. Each grader returns a score between 0 and 1.

**Template Syntax** uses double-brace notation to inject dynamic values into prompts and grader configurations. `{{item.field_name}}` references fields from the test data row, while `{{sample.output_text}}` references the model-generated response. This templating system connects test data, model prompts, and grading criteria into a unified evaluation pipeline.

**Testing Criteria** is the collection of graders attached to an eval. Multiple criteria can assess different aspects of a single model output, enabling multi-dimensional quality evaluation from a single run.

## Installation and Setup

### Open-Source Library

The open-source library requires Python 3.9 or higher:

```bash
pip install evals
```

For development and contribution:

```bash
git clone https://github.com/openai/evals.git
cd evals
pip install -e .
```

The registry uses Git Large File Storage (LFS) for evaluation data:

```bash
git lfs fetch --all
git lfs pull
```

Set the OpenAI API key for running evaluations:

```bash
export OPENAI_API_KEY="sk-..."
```

### API-Based Evals (Python SDK)

The API-based evals use the standard OpenAI Python SDK:

```bash
pip install openai
```

```python
from openai import OpenAI

client = OpenAI()

eval_object = client.evals.create(
    name="ticket-classification",
    data_source_config={
        "type": "custom",
        "item_schema": {
            "type": "object",
            "properties": {
                "ticket_text": {"type": "string"},
                "correct_label": {"type": "string"},
            },
            "required": ["ticket_text", "correct_label"],
        },
        "include_sample_schema": True,
    },
    testing_criteria=[
        {
            "type": "string_check",
            "name": "classification_accuracy",
            "input": "{{sample.output_text}}",
            "reference": "{{item.correct_label}}",
            "operation": "eq",
        }
    ],
)
```

## Architecture

The OpenAI Evals architecture operates across two execution environments that share a common conceptual model:

**Open-source library** runs evaluations locally. Users define evals through YAML configuration files and JSON data sets, execute them via the command line against OpenAI models, and receive results in the terminal or export them to Weights & Biases for experiment tracking. The library includes a registry of community-contributed benchmarks and supports custom evaluation logic through the Completion Function Protocol.

**API-based evals** run on OpenAI infrastructure. The evaluation lifecycle follows a clear sequence:

1. **Eval creation** defines the data schema and testing criteria via `POST /v1/evals`
2. **Data upload** sends test data as JSON Lines (JSONL) files via `POST /v1/files` with `purpose: "evals"`
3. **Run creation** executes the eval against a specific model and prompt template via `POST /v1/evals/{eval_id}/runs`
4. **Asynchronous execution** generates model responses for each test item and applies graders
5. **Result retrieval** returns pass/fail counts, per-criteria breakdowns, and a dashboard link via `GET /v1/evals/{eval_id}/runs/{run_id}`

The grading system supports five evaluation strategies, each returning a normalized score between 0 and 1:

- **String check** -- deterministic string comparison (equality, containment)
- **Text similarity** -- numerical similarity metrics (BLEU, ROUGE, cosine embedding similarity)
- **Score model** -- LLM-as-judge producing numeric scores with reasoning
- **Python** -- custom grading logic executed in a sandboxed environment
- **Multi-grader** -- weighted combination of multiple graders into a single score

The template engine connects all components by resolving `{{item.*}}` references against test data and `{{sample.*}}` references against model outputs at runtime.

## Key Features and Functionality

**Deterministic String Grading** via the string check grader provides exact match (`eq`), non-match (`ne`), case-sensitive containment (`like`), and case-insensitive containment (`ilike`) operations. This is suited for classification tasks, keyword extraction, and any evaluation where the expected output is a known string.

**Statistical Text Similarity** via the text similarity grader computes numerical closeness between model output and reference text. Supported metrics include `fuzzy_match` (via rapidfuzz), `bleu`, `gleu`, `meteor`, `cosine` (using `text-embedding-3-large`), and `rouge_1` through `rouge_5` plus `rouge_l`. A configurable `pass_threshold` determines the minimum similarity score for a passing result.

**LLM-as-Judge Scoring** via the score model grader uses a specified model to evaluate outputs with structured reasoning. The grader accepts a message array with template variables, a scoring range, and a passing threshold. The response includes both a numeric score and reasoning steps, providing interpretable evaluation results.

**Custom Python Grading** executes arbitrary Python code in a sandboxed environment with a 2-minute time limit, 2 GB memory, and no network access. The sandbox includes libraries such as numpy, scipy, pandas, scikit-learn, rapidfuzz, rouge-score, jsonschema, pydantic, and nltk. The grading function receives `sample` (model output) and `item` (test data) dictionaries and returns a float between 0 and 1.

**Multi-Grader Composition** combines multiple graders with a mathematical expression for weighted scoring. Supported operators include addition, subtraction, multiplication, division, and exponentiation. Functions such as `min`, `max`, `abs`, `floor`, `ceil`, `sqrt`, `log`, and `exp` are available for complex scoring formulas.

**Asynchronous Execution with Webhooks** enables non-blocking evaluation runs. Rather than polling for completion, developers subscribe to webhook events (`eval.run.succeeded`, `eval.run.failed`, `eval.run.canceled`) for automatic notification when runs finish.

**Dashboard Visualization** at `platform.openai.com/evaluations` provides a visual interface for configuring evals, monitoring runs, and analyzing results alongside the programmatic API.

## Use Cases

- **Prompt engineering iteration**: Define expected outputs, run evals against prompt variants, compare pass rates to identify the most effective prompt
- **Model selection and comparison**: Run the same eval across different models (such as gpt-4.1 versus gpt-4o-mini) to quantify performance differences for a specific task
- **Pre-deployment validation**: Embed eval runs in CI/CD pipelines to gate deployments on minimum quality thresholds
- **Classification accuracy testing**: Verify that models correctly categorize inputs (support tickets, sentiment, intent) using string check graders against known labels
- **Open-ended response quality**: Assess summarization, translation, or generation quality using text similarity metrics or LLM-as-judge scoring
- **Tool calling verification**: Validate that models invoke the correct functions with the correct arguments using multi-graders that check both function names and argument structures
- **Structured output validation**: Confirm that JSON outputs conform to expected schemas and contain correct field values using Python graders with jsonschema validation
- **Regression detection**: Re-run established eval suites after model updates or prompt changes to detect quality regressions before they reach production

## API Reference Summary

**Eval Management**:

- `POST /v1/evals` -- Create an eval with data source config and testing criteria
- `GET /v1/evals/{eval_id}` -- Retrieve eval definition
- `DELETE /v1/evals/{eval_id}` -- Delete an eval

**Data Upload**:

- `POST /v1/files` -- Upload JSONL test data with `purpose: "evals"`

**Run Management**:

- `POST /v1/evals/{eval_id}/runs` -- Create and start an eval run
- `GET /v1/evals/{eval_id}/runs/{run_id}` -- Retrieve run status and results
- `GET /v1/evals/{eval_id}/runs` -- List all runs for an eval

**Run Results Structure**:

- `result_counts` -- Total, passed, failed, and errored item counts
- `per_testing_criteria_results` -- Pass rate broken down by each grader
- `per_model_usage` -- Token consumption and API call counts
- `report_url` -- Link to dashboard visualization

**Python SDK Methods**:

- `client.evals.create(name, data_source_config, testing_criteria)` -- Create eval
- `client.evals.runs.create(eval_id, name, data_source)` -- Start run
- `client.evals.runs.retrieve(eval_id, run_id)` -- Get results

**Webhook Events**:

- `eval.run.succeeded` -- Run completed with results
- `eval.run.failed` -- Run encountered an error
- `eval.run.canceled` -- Run was canceled

## Configuration and Customization

### Data Source Configuration

The data source defines the schema for test items. Three configuration types are available:

```python
# Custom data source with explicit schema
data_source_config = {
    "type": "custom",
    "item_schema": {
        "type": "object",
        "properties": {
            "input_text": {"type": "string"},
            "expected_output": {"type": "string"},
            "category": {"type": "string"},
        },
        "required": ["input_text", "expected_output"],
    },
    "include_sample_schema": True,
}

# Logs data source for production traffic evaluation
data_source_config = {
    "type": "logs",
    "metadata": {"environment": "production"},
}
```

### Grader Configuration

String check grader for exact classification:

```python
{
    "type": "string_check",
    "name": "exact_match",
    "input": "{{sample.output_text}}",
    "reference": "{{item.expected_label}}",
    "operation": "eq",
}
```

Text similarity grader with threshold:

```python
{
    "type": "text_similarity",
    "name": "summary_quality",
    "input": "{{sample.output_text}}",
    "reference": "{{item.reference_summary}}",
    "evaluation_metric": "rouge_l",
    "pass_threshold": 0.7,
}
```

Score model grader with LLM judge:

```python
{
    "type": "score_model",
    "name": "relevance_score",
    "model": "gpt-4o-2024-08-06",
    "input": [
        {
            "role": "system",
            "content": "Score the answer relevance from 0 to 1. "
                       "1 = fully relevant, 0.5 = partially relevant, "
                       "0 = irrelevant.",
        },
        {
            "role": "user",
            "content": "Question: {{item.question}}\n"
                       "Answer: {{sample.output_text}}",
        },
    ],
    "range": [0, 1],
    "pass_threshold": 0.7,
}
```

Python grader with custom logic:

```python
{
    "type": "python",
    "name": "json_structure_check",
    "image_tag": "2025-05-08",
    "source": """
import json

def grade(sample, item):
    try:
        output = json.loads(sample["output_text"])
        required_keys = set(item.get("required_keys", []))
        present_keys = set(output.keys())
        if required_keys.issubset(present_keys):
            return 1.0
        return len(required_keys & present_keys) / len(required_keys)
    except (json.JSONDecodeError, KeyError):
        return 0.0
""",
}
```

### Run Data Source Configuration

When creating a run, the data source specifies the model, prompt template, and test data file:

```python
data_source = {
    "type": "responses",
    "model": "gpt-4.1",
    "input_messages": {
        "type": "template",
        "template": [
            {
                "role": "developer",
                "content": "Classify the ticket as Hardware, "
                           "Software, or Other. Reply with only "
                           "the category label.",
            },
            {
                "role": "user",
                "content": "{{item.ticket_text}}",
            },
        ],
    },
    "source": {
        "type": "file_content",
        "file_id": uploaded_file.id,
    },
}
```

## Integration Patterns

**CI/CD Pipeline Integration**: Embed eval runs as quality gates in deployment pipelines. Upload test data, trigger eval runs, poll or listen via webhooks for completion, and fail the pipeline if pass rates drop below thresholds:

```python
import time
from openai import OpenAI

client = OpenAI()

run = client.evals.runs.create(
    eval_id="eval_abc123",
    name="ci-run",
    data_source=data_source,
)

while True:
    result = client.evals.runs.retrieve(
        eval_id="eval_abc123",
        run_id=run.id,
    )
    if result.status in ("completed", "failed", "canceled"):
        break
    time.sleep(10)

total = result.result_counts.total
passed = result.result_counts.passed
pass_rate = passed / total if total > 0 else 0

if pass_rate < 0.95:
    raise SystemExit(
        f"Eval pass rate {pass_rate:.1%} below 95% threshold"
    )
```

**Webhook-Driven Automation**: Subscribe to eval completion events rather than polling. Configure a webhook endpoint in the OpenAI dashboard to receive `eval.run.succeeded` and `eval.run.failed` events, then trigger downstream actions (notifications, deployments, rollbacks) based on results.

**A/B Model Comparison**: Create a single eval definition and run it against multiple models to produce comparable metrics:

```python
models = ["gpt-4.1", "gpt-4o-mini", "gpt-4o"]

for model in models:
    data_source["model"] = model
    run = client.evals.runs.create(
        eval_id=eval_id,
        name=f"comparison-{model}",
        data_source=data_source,
    )
```

**Open-Source Library with Weights & Biases**: The open-source library integrates with Weights & Biases (W&B) for experiment tracking, enabling longitudinal analysis of eval results across prompt iterations and model versions.

**Multi-Grader for Complex Validation**: Combine deterministic checks with similarity scoring for structured outputs that require both exact field matches and approximate text quality:

```python
{
    "type": "multi",
    "graders": {
        "category": {
            "type": "string_check",
            "input": "{{sample.output_json.category}}",
            "reference": "{{item.expected_category}}",
            "operation": "eq",
        },
        "explanation": {
            "type": "text_similarity",
            "input": "{{sample.output_json.explanation}}",
            "reference": "{{item.reference_explanation}}",
            "evaluation_metric": "rouge_l",
            "pass_threshold": 0.5,
        },
    },
    "calculate_output": "0.6 * category + 0.4 * explanation",
}
```

## Examples

**IT Support Ticket Classification**:

```python
from openai import OpenAI

client = OpenAI()

# Step 1: Create the eval
eval_obj = client.evals.create(
    name="it-ticket-classifier",
    data_source_config={
        "type": "custom",
        "item_schema": {
            "type": "object",
            "properties": {
                "ticket_text": {"type": "string"},
                "correct_label": {"type": "string"},
            },
            "required": ["ticket_text", "correct_label"],
        },
        "include_sample_schema": True,
    },
    testing_criteria=[
        {
            "type": "string_check",
            "name": "label_match",
            "input": "{{sample.output_text}}",
            "reference": "{{item.correct_label}}",
            "operation": "eq",
        }
    ],
)

# Step 2: Upload test data (JSONL format)
data_file = client.files.create(
    file=open("tickets.jsonl", "rb"),
    purpose="evals",
)

# Step 3: Run the eval
run = client.evals.runs.create(
    eval_id=eval_obj.id,
    name="gpt-4.1-classification-run",
    data_source={
        "type": "responses",
        "model": "gpt-4.1",
        "input_messages": {
            "type": "template",
            "template": [
                {
                    "role": "developer",
                    "content": "Classify the following IT support "
                               "ticket into exactly one category: "
                               "Hardware, Software, or Other. "
                               "Respond with only the category name.",
                },
                {
                    "role": "user",
                    "content": "{{item.ticket_text}}",
                },
            ],
        },
        "source": {
            "type": "file_content",
            "file_id": data_file.id,
        },
    },
)

# Step 4: Retrieve results
import time

while True:
    result = client.evals.runs.retrieve(
        eval_id=eval_obj.id, run_id=run.id,
    )
    if result.status in ("completed", "failed", "canceled"):
        break
    time.sleep(5)

print(f"Total: {result.result_counts.total}")
print(f"Passed: {result.result_counts.passed}")
print(f"Failed: {result.result_counts.failed}")
print(f"Dashboard: {result.report_url}")
```

Test data file (`tickets.jsonl`):

```jsonl
{"ticket_text": "My laptop screen is cracked and won't display", "correct_label": "Hardware"}
{"ticket_text": "Excel keeps crashing when I open large files", "correct_label": "Software"}
{"ticket_text": "I need a new employee badge for building access", "correct_label": "Other"}
{"ticket_text": "The printer on floor 3 is jamming constantly", "correct_label": "Hardware"}
{"ticket_text": "VPN client fails to connect after the update", "correct_label": "Software"}
```

**Python Grader for Custom Validation**:

```python
{
    "type": "python",
    "name": "json_field_validator",
    "image_tag": "2025-05-08",
    "source": """
import json

def grade(sample, item):
    try:
        output = json.loads(sample["output_text"])
        expected = json.loads(item["expected_json"])

        matches = 0
        total = len(expected)

        for key, value in expected.items():
            if key in output and str(output[key]).lower() == str(value).lower():
                matches += 1

        return matches / total if total > 0 else 0.0
    except (json.JSONDecodeError, KeyError):
        return 0.0
""",
}
```

**Tool Calling Verification with Multi-Grader**:

```python
{
    "type": "multi",
    "graders": {
        "function_name": {
            "type": "string_check",
            "input": "{{sample.output_tools[0].function.name}}",
            "reference": "{{item.expected_function}}",
            "operation": "eq",
        },
        "arguments": {
            "type": "python",
            "name": "arg_check",
            "image_tag": "2025-05-08",
            "source": """
import json

def grade(sample, item):
    try:
        actual = json.loads(
            sample["output_tools"][0]["function"]["arguments"]
        )
        expected = json.loads(item["expected_arguments"])
        return 1.0 if actual == expected else 0.0
    except (json.JSONDecodeError, KeyError, IndexError):
        return 0.0
""",
        },
    },
    "calculate_output": "0.5 * function_name + 0.5 * arguments",
}
```

## Limitations and Considerations

- **OpenAI model dependency**: The API-based evals only evaluate OpenAI models. Third-party models cannot be tested through the Evals API, though the open-source library can be extended for other providers.
- **Asynchronous-only execution**: Eval runs are asynchronous with no synchronous mode. Applications must implement polling or webhook handling to retrieve results.
- **Python grader sandbox constraints**: The sandboxed environment has a 2-minute execution limit, 2 GB memory cap, no network access, and a fixed set of available libraries. Complex grading logic that requires external API calls or large data processing is not feasible.
- **Score model grader reliability**: LLM-as-judge approaches are susceptible to grader hacking, where models learn to exploit weaknesses in the judge model. OpenAI recommends cross-validating score model grader results against human expert evaluations to detect this phenomenon.
- **Data set balance**: Imbalanced test data can lead to artificially high pass rates if models learn to guess the majority label. Test data sets should be balanced across expected output categories.
- **Template syntax rigidity**: The double-brace template system supports field access and array indexing but does not provide conditional logic, loops, or string manipulation within templates. Complex data transformations must be handled in prompt design or Python graders.
- **Cost considerations**: Each eval run generates model API calls for every test item plus any score model or label model grader invocations. Large test data sets with LLM-as-judge grading can accumulate significant token costs.
- **Open-source library divergence**: The open-source `evals` library and the API-based Evals system are separate implementations. Features, grader types, and configuration formats do not have a one-to-one mapping between the two.

## Changelog Highlights

The OpenAI Evals ecosystem has evolved through two major phases. The open-source library launched as the initial framework for community-contributed benchmarks and custom evaluation logic, establishing the conceptual foundations of systematic LLM testing. In April 2025, OpenAI introduced the API-based Evals system, bringing programmatic eval creation, cloud-hosted execution, five built-in grader types (string check, text similarity, score model, Python, and multi-grader), webhook-driven automation, and dashboard visualization. The Python grader sandbox introduced versioned image tags (such as `2025-05-08`) to pin the available library set, and the data source configuration expanded to support custom schemas, logs-based sources, and the deprecated stored completions type.

## Citations

- [1] [OpenAI Evals API Guide](https://developers.openai.com/api/docs/guides/evals)
- [2] [OpenAI Graders Guide](https://developers.openai.com/api/docs/guides/graders/)
- [3] [OpenAI Evals API Reference](https://platform.openai.com/docs/api-reference/evals)
- [4] [OpenAI Graders API Reference](https://platform.openai.com/docs/api-reference/graders)
- [5] [OpenAI Evals GitHub Repository](https://github.com/openai/evals)
- [6] [Create Eval API Reference](https://developers.openai.com/api/reference/resources/evals/methods/create)
