[Header 1 ("openai-evals", [], []) [Str "OpenAI Evals"], BlockQuote [Para [Str "OpenAI framework for creating and running evaluations on LLM systems"]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.5)), (AlignDefault, (ColWidth 0.5))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Evaluation & Testing"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "SDK"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Yes"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "https://github.com/openai/evals"] ("https://github.com/openai/evals", "")]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "17876"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Documentation"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "Official Docs"] ("https://developers.openai.com/api/docs/guides/evals", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "OpenAI Evals is a framework for systematically testing whether Large Language Model (LLM) outputs meet defined style and content criteria. It operates on a principle similar to Behavior-Driven Development (BDD): define expected behavior before implementation, then verify model outputs against those expectations. The framework spans two complementary surfaces -- an open-source Python library for local evaluation execution and a cloud-based Evals Application Programming Interface (API) integrated into the OpenAI platform."], Para [Str "The open-source repository at ", Code ("", [], []) "github.com/openai/evals", Str " provides a framework for building custom evaluations and a registry of pre-built benchmarks. The API-based counterpart, introduced in April 2025, enables programmatic evaluation creation, execution, and result retrieval through RESTful endpoints. Both approaches share the same conceptual model: define a task with expected outcomes, run model outputs against test data, and analyze whether results meet the defined criteria."], Para [Str "Evals are particularly valuable during prompt engineering, model selection, and pre-deployment validation. They transform subjective quality assessments into repeatable, quantifiable tests that can be embedded into Continuous Integration / Continuous Deployment (CI/CD) pipelines."], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], Para [Strong [Str "Eval"], Str " is the top-level construct representing an evaluation definition. It specifies the data schema that test inputs must conform to and the testing criteria (graders) used to assess model outputs. An eval is reusable across multiple runs with different models, prompts, or data sets."], Para [Strong [Str "Eval Run"], Str " is a single execution of an eval against a specific model configuration and data set. Each run produces results including pass/fail counts, per-criteria breakdowns, and token usage metrics. Runs execute asynchronously on OpenAI infrastructure."], Para [Strong [Str "Data Source Configuration"], Str " defines the structure of test data using JavaScript Object Notation (JSON) Schema. It specifies the properties each test item must include (such as input text and expected labels) and whether model-generated output schemas should be available in grader templates."], Para [Strong [Str "Graders"], Str " are the evaluation functions that determine whether a model output meets requirements. OpenAI provides five built-in grader types: string check, text similarity, score model, label model (available via the API for fine-tuning workflows), and Python. Each grader returns a score between 0 and 1."], Para [Strong [Str "Template Syntax"], Str " uses double-brace notation to inject dynamic values into prompts and grader configurations. ", Code ("", [], []) "{{item.field_name}}", Str " references fields from the test data row, while ", Code ("", [], []) "{{sample.output_text}}", Str " references the model-generated response. This templating system connects test data, model prompts, and grading criteria into a unified evaluation pipeline."], Para [Strong [Str "Testing Criteria"], Str " is the collection of graders attached to an eval. Multiple criteria can assess different aspects of a single model output, enabling multi-dimensional quality evaluation from a single run."], Header 2 ("installation-and-setup", ["unnumbered", "unlisted"], []) [Str "Installation and Setup"], Header 3 ("open-source-library", ["unnumbered", "unlisted"], []) [Str "Open-Source Library"], Para [Str "The open-source library requires Python 3.9 or higher:"], CodeBlock ("", ["bash"], []) "pip install evals
", Para [Str "For development and contribution:"], CodeBlock ("", ["bash"], []) "git clone https://github.com/openai/evals.git
cd evals
pip install -e .
", Para [Str "The registry uses Git Large File Storage (LFS) for evaluation data:"], CodeBlock ("", ["bash"], []) "git lfs fetch --all
git lfs pull
", Para [Str "Set the OpenAI API key for running evaluations:"], CodeBlock ("", ["bash"], []) "export OPENAI_API_KEY=\"sk-...\"
", Header 3 ("api-based-evals-python-sdk", ["unnumbered", "unlisted"], []) [Str "API-Based Evals (Python SDK)"], Para [Str "The API-based evals use the standard OpenAI Python SDK:"], CodeBlock ("", ["bash"], []) "pip install openai
", CodeBlock ("", ["python"], []) "from openai import OpenAI

client = OpenAI()

eval_object = client.evals.create(
    name=\"ticket-classification\",
    data_source_config={
        \"type\": \"custom\",
        \"item_schema\": {
            \"type\": \"object\",
            \"properties\": {
                \"ticket_text\": {\"type\": \"string\"},
                \"correct_label\": {\"type\": \"string\"},
            },
            \"required\": [\"ticket_text\", \"correct_label\"],
        },
        \"include_sample_schema\": True,
    },
    testing_criteria=[
        {
            \"type\": \"string_check\",
            \"name\": \"classification_accuracy\",
            \"input\": \"{{sample.output_text}}\",
            \"reference\": \"{{item.correct_label}}\",
            \"operation\": \"eq\",
        }
    ],
)
", Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Para [Str "The OpenAI Evals architecture operates across two execution environments that share a common conceptual model:"], Para [Strong [Str "Open-source library"], Str " runs evaluations locally. Users define evals through YAML configuration files and JSON data sets, execute them via the command line against OpenAI models, and receive results in the terminal or export them to Weights & Biases for experiment tracking. The library includes a registry of community-contributed benchmarks and supports custom evaluation logic through the Completion Function Protocol."], Para [Strong [Str "API-based evals"], Str " run on OpenAI infrastructure. The evaluation lifecycle follows a clear sequence:"], OrderedList (1, DefaultStyle, DefaultDelim) [[Plain [Strong [Str "Eval creation"], Str " defines the data schema and testing criteria via ", Code ("", [], []) "POST /v1/evals"]], [Plain [Strong [Str "Data upload"], Str " sends test data as JSON Lines (JSONL) files via ", Code ("", [], []) "POST /v1/files", Str " with ", Code ("", [], []) "purpose: \"evals\""]], [Plain [Strong [Str "Run creation"], Str " executes the eval against a specific model and prompt template via ", Code ("", [], []) "POST /v1/evals/{eval_id}/runs"]], [Plain [Strong [Str "Asynchronous execution"], Str " generates model responses for each test item and applies graders"]], [Plain [Strong [Str "Result retrieval"], Str " returns pass/fail counts, per-criteria breakdowns, and a dashboard link via ", Code ("", [], []) "GET /v1/evals/{eval_id}/runs/{run_id}"]]], Para [Str "The grading system supports five evaluation strategies, each returning a normalized score between 0 and 1:"], BulletList [[Plain [Strong [Str "String check"], Str " -- deterministic string comparison (equality, containment)"]], [Plain [Strong [Str "Text similarity"], Str " -- numerical similarity metrics (BLEU, ROUGE, cosine embedding similarity)"]], [Plain [Strong [Str "Score model"], Str " -- LLM-as-judge producing numeric scores with reasoning"]], [Plain [Strong [Str "Python"], Str " -- custom grading logic executed in a sandboxed environment"]], [Plain [Strong [Str "Multi-grader"], Str " -- weighted combination of multiple graders into a single score"]]], Para [Str "The template engine connects all components by resolving ", Code ("", [], []) "{{item.*}}", Str " references against test data and ", Code ("", [], []) "{{sample.*}}", Str " references against model outputs at runtime."], Header 2 ("key-features-and-functionality", ["unnumbered", "unlisted"], []) [Str "Key Features and Functionality"], Para [Strong [Str "Deterministic String Grading"], Str " via the string check grader provides exact match (", Code ("", [], []) "eq", Str "), non-match (", Code ("", [], []) "ne", Str "), case-sensitive containment (", Code ("", [], []) "like", Str "), and case-insensitive containment (", Code ("", [], []) "ilike", Str ") operations. This is suited for classification tasks, keyword extraction, and any evaluation where the expected output is a known string."], Para [Strong [Str "Statistical Text Similarity"], Str " via the text similarity grader computes numerical closeness between model output and reference text. Supported metrics include ", Code ("", [], []) "fuzzy_match", Str " (via rapidfuzz), ", Code ("", [], []) "bleu", Str ", ", Code ("", [], []) "gleu", Str ", ", Code ("", [], []) "meteor", Str ", ", Code ("", [], []) "cosine", Str " (using ", Code ("", [], []) "text-embedding-3-large", Str "), and ", Code ("", [], []) "rouge_1", Str " through ", Code ("", [], []) "rouge_5", Str " plus ", Code ("", [], []) "rouge_l", Str ". A configurable ", Code ("", [], []) "pass_threshold", Str " determines the minimum similarity score for a passing result."], Para [Strong [Str "LLM-as-Judge Scoring"], Str " via the score model grader uses a specified model to evaluate outputs with structured reasoning. The grader accepts a message array with template variables, a scoring range, and a passing threshold. The response includes both a numeric score and reasoning steps, providing interpretable evaluation results."], Para [Strong [Str "Custom Python Grading"], Str " executes arbitrary Python code in a sandboxed environment with a 2-minute time limit, 2 GB memory, and no network access. The sandbox includes libraries such as numpy, scipy, pandas, scikit-learn, rapidfuzz, rouge-score, jsonschema, pydantic, and nltk. The grading function receives ", Code ("", [], []) "sample", Str " (model output) and ", Code ("", [], []) "item", Str " (test data) dictionaries and returns a float between 0 and 1."], Para [Strong [Str "Multi-Grader Composition"], Str " combines multiple graders with a mathematical expression for weighted scoring. Supported operators include addition, subtraction, multiplication, division, and exponentiation. Functions such as ", Code ("", [], []) "min", Str ", ", Code ("", [], []) "max", Str ", ", Code ("", [], []) "abs", Str ", ", Code ("", [], []) "floor", Str ", ", Code ("", [], []) "ceil", Str ", ", Code ("", [], []) "sqrt", Str ", ", Code ("", [], []) "log", Str ", and ", Code ("", [], []) "exp", Str " are available for complex scoring formulas."], Para [Strong [Str "Asynchronous Execution with Webhooks"], Str " enables non-blocking evaluation runs. Rather than polling for completion, developers subscribe to webhook events (", Code ("", [], []) "eval.run.succeeded", Str ", ", Code ("", [], []) "eval.run.failed", Str ", ", Code ("", [], []) "eval.run.canceled", Str ") for automatic notification when runs finish."], Para [Strong [Str "Dashboard Visualization"], Str " at ", Code ("", [], []) "platform.openai.com/evaluations", Str " provides a visual interface for configuring evals, monitoring runs, and analyzing results alongside the programmatic API."], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], BulletList [[Plain [Strong [Str "Prompt engineering iteration"], Str ": Define expected outputs, run evals against prompt variants, compare pass rates to identify the most effective prompt"]], [Plain [Strong [Str "Model selection and comparison"], Str ": Run the same eval across different models (such as gpt-4.1 versus gpt-4o-mini) to quantify performance differences for a specific task"]], [Plain [Strong [Str "Pre-deployment validation"], Str ": Embed eval runs in CI/CD pipelines to gate deployments on minimum quality thresholds"]], [Plain [Strong [Str "Classification accuracy testing"], Str ": Verify that models correctly categorize inputs (support tickets, sentiment, intent) using string check graders against known labels"]], [Plain [Strong [Str "Open-ended response quality"], Str ": Assess summarization, translation, or generation quality using text similarity metrics or LLM-as-judge scoring"]], [Plain [Strong [Str "Tool calling verification"], Str ": Validate that models invoke the correct functions with the correct arguments using multi-graders that check both function names and argument structures"]], [Plain [Strong [Str "Structured output validation"], Str ": Confirm that JSON outputs conform to expected schemas and contain correct field values using Python graders with jsonschema validation"]], [Plain [Strong [Str "Regression detection"], Str ": Re-run established eval suites after model updates or prompt changes to detect quality regressions before they reach production"]]], Header 2 ("api-reference-summary", ["unnumbered", "unlisted"], []) [Str "API Reference Summary"], Para [Strong [Str "Eval Management"], Str ":"], BulletList [[Plain [Code ("", [], []) "POST /v1/evals", Str " -- Create an eval with data source config and testing criteria"]], [Plain [Code ("", [], []) "GET /v1/evals/{eval_id}", Str " -- Retrieve eval definition"]], [Plain [Code ("", [], []) "DELETE /v1/evals/{eval_id}", Str " -- Delete an eval"]]], Para [Strong [Str "Data Upload"], Str ":"], BulletList [[Plain [Code ("", [], []) "POST /v1/files", Str " -- Upload JSONL test data with ", Code ("", [], []) "purpose: \"evals\""]]], Para [Strong [Str "Run Management"], Str ":"], BulletList [[Plain [Code ("", [], []) "POST /v1/evals/{eval_id}/runs", Str " -- Create and start an eval run"]], [Plain [Code ("", [], []) "GET /v1/evals/{eval_id}/runs/{run_id}", Str " -- Retrieve run status and results"]], [Plain [Code ("", [], []) "GET /v1/evals/{eval_id}/runs", Str " -- List all runs for an eval"]]], Para [Strong [Str "Run Results Structure"], Str ":"], BulletList [[Plain [Code ("", [], []) "result_counts", Str " -- Total, passed, failed, and errored item counts"]], [Plain [Code ("", [], []) "per_testing_criteria_results", Str " -- Pass rate broken down by each grader"]], [Plain [Code ("", [], []) "per_model_usage", Str " -- Token consumption and API call counts"]], [Plain [Code ("", [], []) "report_url", Str " -- Link to dashboard visualization"]]], Para [Strong [Str "Python SDK Methods"], Str ":"], BulletList [[Plain [Code ("", [], []) "client.evals.create(name, data_source_config, testing_criteria)", Str " -- Create eval"]], [Plain [Code ("", [], []) "client.evals.runs.create(eval_id, name, data_source)", Str " -- Start run"]], [Plain [Code ("", [], []) "client.evals.runs.retrieve(eval_id, run_id)", Str " -- Get results"]]], Para [Strong [Str "Webhook Events"], Str ":"], BulletList [[Plain [Code ("", [], []) "eval.run.succeeded", Str " -- Run completed with results"]], [Plain [Code ("", [], []) "eval.run.failed", Str " -- Run encountered an error"]], [Plain [Code ("", [], []) "eval.run.canceled", Str " -- Run was canceled"]]], Header 2 ("configuration-and-customization", ["unnumbered", "unlisted"], []) [Str "Configuration and Customization"], Header 3 ("data-source-configuration", ["unnumbered", "unlisted"], []) [Str "Data Source Configuration"], Para [Str "The data source defines the schema for test items. Three configuration types are available:"], CodeBlock ("", ["python"], []) "# Custom data source with explicit schema
data_source_config = {
    \"type\": \"custom\",
    \"item_schema\": {
        \"type\": \"object\",
        \"properties\": {
            \"input_text\": {\"type\": \"string\"},
            \"expected_output\": {\"type\": \"string\"},
            \"category\": {\"type\": \"string\"},
        },
        \"required\": [\"input_text\", \"expected_output\"],
    },
    \"include_sample_schema\": True,
}

# Logs data source for production traffic evaluation
data_source_config = {
    \"type\": \"logs\",
    \"metadata\": {\"environment\": \"production\"},
}
", Header 3 ("grader-configuration", ["unnumbered", "unlisted"], []) [Str "Grader Configuration"], Para [Str "String check grader for exact classification:"], CodeBlock ("", ["python"], []) "{
    \"type\": \"string_check\",
    \"name\": \"exact_match\",
    \"input\": \"{{sample.output_text}}\",
    \"reference\": \"{{item.expected_label}}\",
    \"operation\": \"eq\",
}
", Para [Str "Text similarity grader with threshold:"], CodeBlock ("", ["python"], []) "{
    \"type\": \"text_similarity\",
    \"name\": \"summary_quality\",
    \"input\": \"{{sample.output_text}}\",
    \"reference\": \"{{item.reference_summary}}\",
    \"evaluation_metric\": \"rouge_l\",
    \"pass_threshold\": 0.7,
}
", Para [Str "Score model grader with LLM judge:"], CodeBlock ("", ["python"], []) "{
    \"type\": \"score_model\",
    \"name\": \"relevance_score\",
    \"model\": \"gpt-4o-2024-08-06\",
    \"input\": [
        {
            \"role\": \"system\",
            \"content\": \"Score the answer relevance from 0 to 1. \"
                       \"1 = fully relevant, 0.5 = partially relevant, \"
                       \"0 = irrelevant.\",
        },
        {
            \"role\": \"user\",
            \"content\": \"Question: {{item.question}}\\n\"
                       \"Answer: {{sample.output_text}}\",
        },
    ],
    \"range\": [0, 1],
    \"pass_threshold\": 0.7,
}
", Para [Str "Python grader with custom logic:"], CodeBlock ("", ["python"], []) "{
    \"type\": \"python\",
    \"name\": \"json_structure_check\",
    \"image_tag\": \"2025-05-08\",
    \"source\": \"\"\"
import json

def grade(sample, item):
    try:
        output = json.loads(sample[\"output_text\"])
        required_keys = set(item.get(\"required_keys\", []))
        present_keys = set(output.keys())
        if required_keys.issubset(present_keys):
            return 1.0
        return len(required_keys & present_keys) / len(required_keys)
    except (json.JSONDecodeError, KeyError):
        return 0.0
\"\"\",
}
", Header 3 ("run-data-source-configuration", ["unnumbered", "unlisted"], []) [Str "Run Data Source Configuration"], Para [Str "When creating a run, the data source specifies the model, prompt template, and test data file:"], CodeBlock ("", ["python"], []) "data_source = {
    \"type\": \"responses\",
    \"model\": \"gpt-4.1\",
    \"input_messages\": {
        \"type\": \"template\",
        \"template\": [
            {
                \"role\": \"developer\",
                \"content\": \"Classify the ticket as Hardware, \"
                           \"Software, or Other. Reply with only \"
                           \"the category label.\",
            },
            {
                \"role\": \"user\",
                \"content\": \"{{item.ticket_text}}\",
            },
        ],
    },
    \"source\": {
        \"type\": \"file_content\",
        \"file_id\": uploaded_file.id,
    },
}
", Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Para [Strong [Str "CI/CD Pipeline Integration"], Str ": Embed eval runs as quality gates in deployment pipelines. Upload test data, trigger eval runs, poll or listen via webhooks for completion, and fail the pipeline if pass rates drop below thresholds:"], CodeBlock ("", ["python"], []) "import time
from openai import OpenAI

client = OpenAI()

run = client.evals.runs.create(
    eval_id=\"eval_abc123\",
    name=\"ci-run\",
    data_source=data_source,
)

while True:
    result = client.evals.runs.retrieve(
        eval_id=\"eval_abc123\",
        run_id=run.id,
    )
    if result.status in (\"completed\", \"failed\", \"canceled\"):
        break
    time.sleep(10)

total = result.result_counts.total
passed = result.result_counts.passed
pass_rate = passed / total if total > 0 else 0

if pass_rate < 0.95:
    raise SystemExit(
        f\"Eval pass rate {pass_rate:.1%} below 95% threshold\"
    )
", Para [Strong [Str "Webhook-Driven Automation"], Str ": Subscribe to eval completion events rather than polling. Configure a webhook endpoint in the OpenAI dashboard to receive ", Code ("", [], []) "eval.run.succeeded", Str " and ", Code ("", [], []) "eval.run.failed", Str " events, then trigger downstream actions (notifications, deployments, rollbacks) based on results."], Para [Strong [Str "A/B Model Comparison"], Str ": Create a single eval definition and run it against multiple models to produce comparable metrics:"], CodeBlock ("", ["python"], []) "models = [\"gpt-4.1\", \"gpt-4o-mini\", \"gpt-4o\"]

for model in models:
    data_source[\"model\"] = model
    run = client.evals.runs.create(
        eval_id=eval_id,
        name=f\"comparison-{model}\",
        data_source=data_source,
    )
", Para [Strong [Str "Open-Source Library with Weights & Biases"], Str ": The open-source library integrates with Weights & Biases (W&B) for experiment tracking, enabling longitudinal analysis of eval results across prompt iterations and model versions."], Para [Strong [Str "Multi-Grader for Complex Validation"], Str ": Combine deterministic checks with similarity scoring for structured outputs that require both exact field matches and approximate text quality:"], CodeBlock ("", ["python"], []) "{
    \"type\": \"multi\",
    \"graders\": {
        \"category\": {
            \"type\": \"string_check\",
            \"input\": \"{{sample.output_json.category}}\",
            \"reference\": \"{{item.expected_category}}\",
            \"operation\": \"eq\",
        },
        \"explanation\": {
            \"type\": \"text_similarity\",
            \"input\": \"{{sample.output_json.explanation}}\",
            \"reference\": \"{{item.reference_explanation}}\",
            \"evaluation_metric\": \"rouge_l\",
            \"pass_threshold\": 0.5,
        },
    },
    \"calculate_output\": \"0.6 * category + 0.4 * explanation\",
}
", Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Para [Strong [Str "IT Support Ticket Classification"], Str ":"], CodeBlock ("", ["python"], []) "from openai import OpenAI

client = OpenAI()

# Step 1: Create the eval
eval_obj = client.evals.create(
    name=\"it-ticket-classifier\",
    data_source_config={
        \"type\": \"custom\",
        \"item_schema\": {
            \"type\": \"object\",
            \"properties\": {
                \"ticket_text\": {\"type\": \"string\"},
                \"correct_label\": {\"type\": \"string\"},
            },
            \"required\": [\"ticket_text\", \"correct_label\"],
        },
        \"include_sample_schema\": True,
    },
    testing_criteria=[
        {
            \"type\": \"string_check\",
            \"name\": \"label_match\",
            \"input\": \"{{sample.output_text}}\",
            \"reference\": \"{{item.correct_label}}\",
            \"operation\": \"eq\",
        }
    ],
)

# Step 2: Upload test data (JSONL format)
data_file = client.files.create(
    file=open(\"tickets.jsonl\", \"rb\"),
    purpose=\"evals\",
)

# Step 3: Run the eval
run = client.evals.runs.create(
    eval_id=eval_obj.id,
    name=\"gpt-4.1-classification-run\",
    data_source={
        \"type\": \"responses\",
        \"model\": \"gpt-4.1\",
        \"input_messages\": {
            \"type\": \"template\",
            \"template\": [
                {
                    \"role\": \"developer\",
                    \"content\": \"Classify the following IT support \"
                               \"ticket into exactly one category: \"
                               \"Hardware, Software, or Other. \"
                               \"Respond with only the category name.\",
                },
                {
                    \"role\": \"user\",
                    \"content\": \"{{item.ticket_text}}\",
                },
            ],
        },
        \"source\": {
            \"type\": \"file_content\",
            \"file_id\": data_file.id,
        },
    },
)

# Step 4: Retrieve results
import time

while True:
    result = client.evals.runs.retrieve(
        eval_id=eval_obj.id, run_id=run.id,
    )
    if result.status in (\"completed\", \"failed\", \"canceled\"):
        break
    time.sleep(5)

print(f\"Total: {result.result_counts.total}\")
print(f\"Passed: {result.result_counts.passed}\")
print(f\"Failed: {result.result_counts.failed}\")
print(f\"Dashboard: {result.report_url}\")
", Para [Str "Test data file (", Code ("", [], []) "tickets.jsonl", Str "):"], CodeBlock ("", ["jsonl"], []) "{\"ticket_text\": \"My laptop screen is cracked and won't display\", \"correct_label\": \"Hardware\"}
{\"ticket_text\": \"Excel keeps crashing when I open large files\", \"correct_label\": \"Software\"}
{\"ticket_text\": \"I need a new employee badge for building access\", \"correct_label\": \"Other\"}
{\"ticket_text\": \"The printer on floor 3 is jamming constantly\", \"correct_label\": \"Hardware\"}
{\"ticket_text\": \"VPN client fails to connect after the update\", \"correct_label\": \"Software\"}
", Para [Strong [Str "Python Grader for Custom Validation"], Str ":"], CodeBlock ("", ["python"], []) "{
    \"type\": \"python\",
    \"name\": \"json_field_validator\",
    \"image_tag\": \"2025-05-08\",
    \"source\": \"\"\"
import json

def grade(sample, item):
    try:
        output = json.loads(sample[\"output_text\"])
        expected = json.loads(item[\"expected_json\"])

        matches = 0
        total = len(expected)

        for key, value in expected.items():
            if key in output and str(output[key]).lower() == str(value).lower():
                matches += 1

        return matches / total if total > 0 else 0.0
    except (json.JSONDecodeError, KeyError):
        return 0.0
\"\"\",
}
", Para [Strong [Str "Tool Calling Verification with Multi-Grader"], Str ":"], CodeBlock ("", ["python"], []) "{
    \"type\": \"multi\",
    \"graders\": {
        \"function_name\": {
            \"type\": \"string_check\",
            \"input\": \"{{sample.output_tools[0].function.name}}\",
            \"reference\": \"{{item.expected_function}}\",
            \"operation\": \"eq\",
        },
        \"arguments\": {
            \"type\": \"python\",
            \"name\": \"arg_check\",
            \"image_tag\": \"2025-05-08\",
            \"source\": \"\"\"
import json

def grade(sample, item):
    try:
        actual = json.loads(
            sample[\"output_tools\"][0][\"function\"][\"arguments\"]
        )
        expected = json.loads(item[\"expected_arguments\"])
        return 1.0 if actual == expected else 0.0
    except (json.JSONDecodeError, KeyError, IndexError):
        return 0.0
\"\"\",
        },
    },
    \"calculate_output\": \"0.5 * function_name + 0.5 * arguments\",
}
", Header 2 ("limitations-and-considerations", ["unnumbered", "unlisted"], []) [Str "Limitations and Considerations"], BulletList [[Plain [Strong [Str "OpenAI model dependency"], Str ": The API-based evals only evaluate OpenAI models. Third-party models cannot be tested through the Evals API, though the open-source library can be extended for other providers."]], [Plain [Strong [Str "Asynchronous-only execution"], Str ": Eval runs are asynchronous with no synchronous mode. Applications must implement polling or webhook handling to retrieve results."]], [Plain [Strong [Str "Python grader sandbox constraints"], Str ": The sandboxed environment has a 2-minute execution limit, 2 GB memory cap, no network access, and a fixed set of available libraries. Complex grading logic that requires external API calls or large data processing is not feasible."]], [Plain [Strong [Str "Score model grader reliability"], Str ": LLM-as-judge approaches are susceptible to grader hacking, where models learn to exploit weaknesses in the judge model. OpenAI recommends cross-validating score model grader results against human expert evaluations to detect this phenomenon."]], [Plain [Strong [Str "Data set balance"], Str ": Imbalanced test data can lead to artificially high pass rates if models learn to guess the majority label. Test data sets should be balanced across expected output categories."]], [Plain [Strong [Str "Template syntax rigidity"], Str ": The double-brace template system supports field access and array indexing but does not provide conditional logic, loops, or string manipulation within templates. Complex data transformations must be handled in prompt design or Python graders."]], [Plain [Strong [Str "Cost considerations"], Str ": Each eval run generates model API calls for every test item plus any score model or label model grader invocations. Large test data sets with LLM-as-judge grading can accumulate significant token costs."]], [Plain [Strong [Str "Open-source library divergence"], Str ": The open-source ", Code ("", [], []) "evals", Str " library and the API-based Evals system are separate implementations. Features, grader types, and configuration formats do not have a one-to-one mapping between the two."]]], Header 2 ("changelog-highlights", ["unnumbered", "unlisted"], []) [Str "Changelog Highlights"], Para [Str "The OpenAI Evals ecosystem has evolved through two major phases. The open-source library launched as the initial framework for community-contributed benchmarks and custom evaluation logic, establishing the conceptual foundations of systematic LLM testing. In April 2025, OpenAI introduced the API-based Evals system, bringing programmatic eval creation, cloud-hosted execution, five built-in grader types (string check, text similarity, score model, Python, and multi-grader), webhook-driven automation, and dashboard visualization. The Python grader sandbox introduced versioned image tags (such as ", Code ("", [], []) "2025-05-08", Str ") to pin the available library set, and the data source configuration expanded to support custom schemas, logs-based sources, and the deprecated stored completions type."], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " ", Link ("", [], []) [Str "OpenAI Evals API Guide"] ("https://developers.openai.com/api/docs/guides/evals", "")]], [Plain [Str "[", Str "2", Str "]", Str " ", Link ("", [], []) [Str "OpenAI Graders Guide"] ("https://developers.openai.com/api/docs/guides/graders/", "")]], [Plain [Str "[", Str "3", Str "]", Str " ", Link ("", [], []) [Str "OpenAI Evals API Reference"] ("https://platform.openai.com/docs/api-reference/evals", "")]], [Plain [Str "[", Str "4", Str "]", Str " ", Link ("", [], []) [Str "OpenAI Graders API Reference"] ("https://platform.openai.com/docs/api-reference/graders", "")]], [Plain [Str "[", Str "5", Str "]", Str " ", Link ("", [], []) [Str "OpenAI Evals GitHub Repository"] ("https://github.com/openai/evals", "")]], [Plain [Str "[", Str "6", Str "]", Str " ", Link ("", [], []) [Str "Create Eval API Reference"] ("https://developers.openai.com/api/reference/resources/evals/methods/create", "")]]]]