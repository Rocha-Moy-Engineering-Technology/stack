[Header 1 ("promptfoo", [], []) [Str "promptfoo"], BlockQuote [Para [Str "Open-source CLI and library for evaluating and red-teaming LLM applications"]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.5)), (AlignDefault, (ColWidth 0.5))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Evaluation & Testing"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "SDK"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Yes"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "https://github.com/promptfoo/promptfoo"] ("https://github.com/promptfoo/promptfoo", "")]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "10600"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Documentation"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "Official Docs"] ("https://www.promptfoo.dev/docs/intro/", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "promptfoo is an open-source command-line interface (CLI) and Node.js library for systematically evaluating and red-teaming Large Language Model (LLM) applications. Rather than relying on manual trial-and-error prompt engineering, promptfoo brings test-driven development (TDD) practices to LLM workflows by defining test cases declaratively in YAML, running them against one or more providers, and automatically scoring outputs against deterministic and model-graded assertions."], Para [Str "The tool is language-agnostic and provider-agnostic: it supports over 70 LLM providers including OpenAI, Anthropic, Google Gemini, Azure OpenAI, AWS Bedrock, Mistral, HuggingFace, Ollama, and custom HTTP/WebSocket APIs. All execution happens locally on the developer's machine, communicating directly with LLM APIs without routing through intermediary services. This local-first design gives teams full control over their data and credentials."], Para [Str "promptfoo operates across three usage modes. As a CLI, it runs evaluations from the terminal with caching, concurrency, and live reloading for rapid iteration. As a Node.js library, it exposes a programmatic ", Code ("", [], []) "evaluate()", Str " function for integration into custom applications and test suites. As a CI/CD component, it plugs into GitHub Actions, GitLab CI, Jenkins, and other pipelines to enforce quality gates and security scanning on every code change. A built-in web viewer provides matrix-style side-by-side comparison of prompt-provider-test combinations, making it straightforward to identify regressions and compare model performance."], Para [Str "Beyond functional evaluation, promptfoo includes a dedicated red-teaming and penetration-testing subsystem. This generates adversarial inputs targeting prompt injection, jailbreaking, harmful content generation, PII leakage, hallucination, and other vulnerability categories. Red-team results produce quantitative risk scores and structured reports suitable for security audits."], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], Para [Strong [Str "Prompts"], Str " are the text templates sent to LLM providers. promptfoo supports inline prompt strings, file references using ", Code ("", [], []) "file://", Str " paths, and Nunjucks templating with double-brace variable substitution (", Code ("", [], []) "{{variable}}", Str "). Multiple prompts can be evaluated side-by-side in a single run, creating a matrix of prompt-provider combinations."], Para [Strong [Str "Providers"], Str " represent the LLM backends that process prompts. A provider is identified by a string in the format ", Code ("", [], []) "provider_name:model_name", Str " (for example, ", Code ("", [], []) "openai:gpt-4o", Str " or ", Code ("", [], []) "anthropic:messages:claude-sonnet-4-20250514", Str "). Providers can also be configured as objects with parameters like temperature, max tokens, and API keys, or referenced from external files and custom scripts."], Para [Strong [Str "Tests"], Str " are individual evaluation cases consisting of input variables, optional assertions, and metadata. Each test supplies a set of variables that are substituted into the prompt template before being sent to providers. When multiple variable values are provided as arrays, promptfoo automatically generates combinatorial test runs."], Para [Strong [Str "Assertions"], Str " define the pass/fail criteria for test outputs. promptfoo provides two categories: deterministic assertions (exact match, contains, regex, JSON validation, cost, latency) and model-graded assertions (semantic similarity, LLM rubric, factuality, context faithfulness, answer relevance). Assertions can be weighted, negated with a ", Code ("", [], []) "not-", Str " prefix, and grouped into assertion sets with collective thresholds."], Para [Strong [Str "defaultTest"], Str " is a configuration block that sets shared properties (variables, assertions, options) applied to every test case. This reduces duplication when all tests share common validation criteria."], Para [Strong [Str "Scenarios"], Str " group a set of variable data with a set of tests, creating a test matrix. For example, three input phrases tested across four languages automatically produce twelve test runs without manually defining each combination."], Para [Strong [Str "Evaluation"], Str " is the process of running all prompt-provider-test combinations, collecting outputs, scoring them against assertions, and producing a summary with pass/fail counts, scores, and token usage statistics."], Para [Strong [Str "Red Teaming"], Str " is an automated adversarial testing process that generates malicious inputs, runs them through the target application, and analyzes responses for vulnerabilities using plugins that target specific risk categories."], Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Para [Str "promptfoo follows a declarative configuration architecture where the evaluation pipeline is defined in YAML and executed by the runtime engine. The architecture consists of five stages:"], OrderedList (1, DefaultStyle, DefaultDelim) [[Para [Strong [Str "Configuration loading"], Str " parses ", Code ("", [], []) "promptfooconfig.yaml", Str ", resolves file references for prompts, providers, and test data, expands variable arrays into combinatorial test cases, and merges ", Code ("", [], []) "defaultTest", Str " properties into each test case."]], [Para [Strong [Str "Provider initialization"], Str " creates API client instances for each specified provider, configuring authentication, model parameters, and any custom provider implementations (JavaScript/Python functions, HTTP endpoints, shell commands)."]], [Para [Strong [Str "Evaluation execution"], Str " sends each prompt-variable combination to each provider with configurable concurrency (default: 4 concurrent requests), caching results to disk to avoid redundant API calls. Outputs pass through optional transform functions before assertion evaluation."]], [Para [Strong [Str "Assertion scoring"], Str " applies each test's assertions to the provider output, producing per-assertion pass/fail results and numerical scores. Model-graded assertions invoke a separate LLM call (configurable via the assertion's ", Code ("", [], []) "provider", Str " field) to evaluate output quality. Scores are aggregated using weighted averaging or custom scoring functions."]], [Para [Strong [Str "Result aggregation"], Str " compiles all results into a summary structure containing per-test results, aggregate statistics (successes, failures, errors, token usage), and formatted output for CLI display, JSON export, or the web viewer."]]], Para [Str "The architecture is extensible at every stage: custom providers handle arbitrary backends, extension hooks (", Code ("", [], []) "beforeAll", Str ", ", Code ("", [], []) "afterAll", Str ", ", Code ("", [], []) "beforeEach", Str ", ", Code ("", [], []) "afterEach", Str ") inject lifecycle logic, and custom assertion functions implement domain-specific validation."], Header 2 ("key-features-and-functionality", ["unnumbered", "unlisted"], []) [Str "Key Features and Functionality"], Para [Strong [Str "Deterministic Assertions"], Str " provide programmatic validation without LLM calls. These include exact string matching (", Code ("", [], []) "equals", Str "), substring detection (", Code ("", [], []) "contains", Str ", ", Code ("", [], []) "icontains", Str "), regular expressions (", Code ("", [], []) "regex", Str "), JSON schema validation (", Code ("", [], []) "is-json", Str ", ", Code ("", [], []) "contains-json", Str "), SQL/XML/HTML validation, refusal detection (", Code ("", [], []) "is-refusal", Str "), text similarity metrics (ROUGE-N, BLEU, METEOR, Levenshtein distance), and operational constraints (", Code ("", [], []) "latency", Str ", ", Code ("", [], []) "cost", Str ")."], Para [Strong [Str "Model-Graded Assertions"], Str " use LLMs or embedding models to evaluate output quality. Semantic similarity (", Code ("", [], []) "similar", Str ") uses embedding cosine distance with a configurable threshold. LLM rubric (", Code ("", [], []) "llm-rubric", Str ") evaluates output against a free-text description of expected quality. Context faithfulness (", Code ("", [], []) "context-faithfulness", Str ") and context recall (", Code ("", [], []) "context-recall", Str ") evaluate Retrieval-Augmented Generation (RAG) pipeline quality. Factuality (", Code ("", [], []) "factuality", Str ") checks output against reference facts."], Para [Strong [Str "Caching and Concurrency"], Str " accelerate evaluation loops. LLM responses are cached to disk by default, so re-running evaluations with the same inputs skips redundant API calls. Concurrency is configurable via ", Code ("", [], []) "maxConcurrency", Str " (default: 4), and a delay between calls can be set to respect rate limits."], Para [Strong [Str "Live Reloading"], Str " watches configuration files and re-runs evaluations automatically when changes are detected, using the ", Code ("", [], []) "--watch", Str " flag. Combined with caching, this creates a rapid feedback loop during prompt development."], Para [Strong [Str "Red Teaming and Penetration Testing"], Str " generates adversarial inputs targeting model-layer threats (prompt injection, jailbreaking, harmful content, PII leakage, hallucination) and application-layer threats (indirect prompt injection, information leakage from RAG context, unauthorized tool access, data exfiltration). Results produce quantitative vulnerability scores and structured reports."], Para [Strong [Str "Side-by-Side Comparison"], Str " via the web viewer and CLI output presents a matrix view of all prompt-provider-test combinations with color-coded pass/fail indicators, assertion scores, and raw outputs. This makes it straightforward to compare multiple prompts or models on the same test suite."], Para [Strong [Str "Output Transforms"], Str " modify provider outputs before assertion evaluation. Transforms can be inline JavaScript expressions, external JavaScript/Python files, or provider-level transforms. The execution order is: provider transforms first, then test-level transforms."], Para [Strong [Str "External Test Data"], Str " can be loaded from CSV, JSON, YAML, or JavaScript/Python files using ", Code ("", [], []) "file://", Str " references. Glob patterns (", Code ("", [], []) "file://tests/*.yaml", Str ") enable batch loading, and Google Sheets integration is supported for collaborative test management."], Para [Strong [Str "Synthetic Test Generation"], Str " via ", Code ("", [], []) "promptfoo generate dataset", Str " creates test cases from prompt descriptions, and ", Code ("", [], []) "promptfoo generate assertions", Str " produces assertion sets from example outputs."], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], BulletList [[Plain [Strong [Str "Prompt engineering iteration"], Str ": Compare multiple prompt variants across models to find the highest-quality, most cost-effective combination for a specific task"]], [Plain [Strong [Str "Model migration evaluation"], Str ": When switching from one LLM provider or model version to another, run the existing test suite against the new target to quantify quality differences"]], [Plain [Strong [Str "RAG pipeline validation"], Str ": Use context-faithfulness, context-recall, and context-relevance assertions to verify that retrieval-augmented generation produces grounded, accurate outputs"]], [Plain [Strong [Str "Security auditing"], Str ": Run red-team scans to identify prompt injection vulnerabilities, harmful content generation risks, and data leakage paths before deploying to production"]], [Plain [Strong [Str "Regression testing in CI/CD"], Str ": Integrate evaluations into pull request workflows to catch prompt or model configuration changes that degrade output quality"]], [Plain [Strong [Str "Cost and latency optimization"], Str ": Use ", Code ("", [], []) "cost", Str " and ", Code ("", [], []) "latency", Str " assertions to enforce operational budgets and response time requirements across providers"]], [Plain [Strong [Str "Compliance verification"], Str ": Apply deterministic assertions (regex, contains, is-refusal) to verify that outputs conform to regulatory requirements such as content restrictions or mandatory disclosures"]]], Header 2 ("api-reference-summary", ["unnumbered", "unlisted"], []) [Str "API Reference Summary"], Para [Strong [Str "CLI Commands"], Str ":"], BulletList [[Plain [Code ("", [], []) "promptfoo init [directory]", Str " -- Initialize a new project with configuration files"]], [Plain [Code ("", [], []) "promptfoo eval", Str " -- Run evaluation (flags: ", Code ("", [], []) "-c", Str " config, ", Code ("", [], []) "-p", Str " prompts, ", Code ("", [], []) "-r", Str " providers, ", Code ("", [], []) "-t", Str " tests, ", Code ("", [], []) "-o", Str " output, ", Code ("", [], []) "--watch", Str ", ", Code ("", [], []) "--share", Str ", ", Code ("", [], []) "--resume", Str ")"]], [Plain [Code ("", [], []) "promptfoo view", Str " -- Launch browser-based result viewer (flag: ", Code ("", [], []) "-p", Str " port)"]], [Plain [Code ("", [], []) "promptfoo share [evalId]", Str " -- Generate shareable URL for results"]], [Plain [Code ("", [], []) "promptfoo validate", Str " -- Check configuration schema compliance"]], [Plain [Code ("", [], []) "promptfoo cache clear", Str " -- Clear cached LLM responses"]], [Plain [Code ("", [], []) "promptfoo list evals|prompts|datasets", Str " -- List stored resources"]], [Plain [Code ("", [], []) "promptfoo export eval <evalId>", Str " -- Export results to JSON"]], [Plain [Code ("", [], []) "promptfoo generate dataset", Str " -- Generate synthetic test cases"]], [Plain [Code ("", [], []) "promptfoo generate assertions", Str " -- Generate assertion sets from examples"]], [Plain [Code ("", [], []) "promptfoo redteam init", Str " -- Initialize red-team configuration"]], [Plain [Code ("", [], []) "promptfoo redteam run", Str " -- Execute red-team scan"]], [Plain [Code ("", [], []) "promptfoo redteam report", Str " -- Generate vulnerability report"]]], Para [Strong [Str "Node.js Library"], Str ":"], CodeBlock ("", ["typescript"], []) "import promptfoo from \"promptfoo\";

const results = await promptfoo.evaluate({
  prompts: [\"Translate to {{language}}: {{input}}\"],
  providers: [\"openai:gpt-4o\"],
  tests: [
    {
      vars: { language: \"French\", input: \"Hello\" },
      assert: [{ type: \"contains\", value: \"Bonjour\" }],
    },
  ],
});

// results.stats: { successes, failures, tokenUsage }
// results.results: per-test result array
// results.table: formatted tabular output
", Para [Strong [Code ("", [], []) "evaluate(testSuite, options)"], Str " accepts a ", Code ("", [], []) "TestSuiteConfiguration", Str " object and an optional ", Code ("", [], []) "EvaluateOptions", Str " object (with ", Code ("", [], []) "maxConcurrency", Str "). It returns an ", Code ("", [], []) "EvaluateSummary", Str " containing ", Code ("", [], []) "results", Str ", ", Code ("", [], []) "stats", Str ", ", Code ("", [], []) "table", Str ", and optionally ", Code ("", [], []) "shareableUrl", Str "."], Para [Strong [Code ("", [], []) "loadApiProvider(providerString, config)"], Str " loads a provider programmatically with optional overrides for ", Code ("", [], []) "apiHost", Str ", ", Code ("", [], []) "apiKey", Str ", and model parameters."], Para [Strong [Str "Custom Provider Function"], Str ":"], CodeBlock ("", ["typescript"], []) "async function customProvider(
  prompt: string,
  context: { vars: Record<string, string> }
): Promise<string | { output: string; error?: string }> {
  // Call custom backend
  return { output: \"response text\" };
}
", Para [Strong [Str "Custom Assertion Function"], Str ":"], CodeBlock ("", ["typescript"], []) "function customAssertion(
  output: string,
  testCase: TestCase,
  assertion: Assertion
): GradingResult {
  return { pass: true, score: 1.0, reason: \"Meets criteria\" };
}
", Header 2 ("configuration-and-customization", ["unnumbered", "unlisted"], []) [Str "Configuration and Customization"], Para [Str "The full configuration reference for ", Code ("", [], []) "promptfooconfig.yaml", Str ":"], CodeBlock ("", ["yaml"], []) "# Top-level metadata
description: \"Evaluation suite description\"
tags:
  env: staging
  team: nlp

# Prompts: inline strings, file references, or functions
prompts:
  - \"Answer the question: {{question}}\"
  - file://prompts/detailed.txt

# Providers: string shorthand or object with config
providers:
  - openai:gpt-4o
  - id: anthropic:messages:claude-sonnet-4-20250514
    config:
      temperature: 0.3
      max_tokens: 1024

# Default test properties applied to all tests
defaultTest:
  assert:
    - type: llm-rubric
      value: \"Response is helpful, accurate, and concise\"
  options:
    transform: output.trim()

# Test cases
tests:
  - description: \"Basic greeting\"
    vars:
      question: \"What is machine learning?\"
    assert:
      - type: contains
        value: \"algorithm\"
      - type: not-contains
        value: \"I don't know\"
      - type: latency
        threshold: 3000
      - type: cost
        threshold: 0.01

# Scenarios for combinatorial testing
scenarios:
  - description: \"Multi-language support\"
    config:
      - vars:
          language: Spanish
      - vars:
          language: French
      - vars:
          language: German
    tests:
      - vars:
          input: \"Hello world\"
        assert:
          - type: llm-rubric
            value: \"Is a correct translation\"

# Evaluation runtime options
evaluateOptions:
  maxConcurrency: 8
  repeat: 3
  delay: 100
  cache: true
  timeoutMs: 30000

# Output destination
outputPath: results.json

# Extension hooks
extensions:
  - file://hooks.js

# Environment variable overrides
env:
  OPENAI_API_KEY: sk-...
", Para [Strong [Str "Assertion weighting and sets"], Str ":"], CodeBlock ("", ["yaml"], []) "assert:
  - type: contains-json
    weight: 2
  - type: llm-rubric
    value: \"Response is factually accurate\"
    weight: 3
  - type: assert-set
    threshold: 0.5
    assert:
      - type: cost
        threshold: 0.005
      - type: latency
        threshold: 2000
", Para [Strong [Str "Named metrics"], Str " for dashboard aggregation:"], CodeBlock ("", ["yaml"], []) "assert:
  - type: llm-rubric
    value: \"Response is helpful\"
    metric: helpfulness
  - type: similar
    value: \"expected output\"
    threshold: 0.8
    metric: semantic_accuracy
", Para [Strong [Str "Variable transforms"], Str " for pre-processing inputs:"], CodeBlock ("", ["yaml"], []) "tests:
  - vars:
      document: file://data/contract.txt
    options:
      transformVars: \"{ ...vars, document: vars.document.substring(0, 5000) }\"
", Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Para [Strong [Str "GitHub Actions"], Str " for automated evaluation on pull requests:"], CodeBlock ("", ["yaml"], []) "name: LLM Evaluation
on:
  pull_request:
    paths:
      - \"prompts/**\"
      - \"promptfooconfig.yaml\"

jobs:
  evaluate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - name: Run evaluation
        env:
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
          PROMPTFOO_CACHE_PATH: .cache/promptfoo
        run: npx promptfoo@latest eval -c promptfooconfig.yaml -o results.json
      - name: Quality gate
        run: |
          PASS_RATE=$(jq '.results.stats.successes / (.results.stats.successes + .results.stats.failures) * 100' results.json)
          if (( $(echo \"$PASS_RATE < 90\" | bc -l) )); then
            echo \"Pass rate $PASS_RATE% is below 90% threshold\"
            exit 1
          fi
      - uses: actions/upload-artifact@v4
        with:
          name: eval-results
          path: results.json
", Para [Strong [Str "Red-team scanning on schedule"], Str ":"], CodeBlock ("", ["yaml"], []) "name: Security Scan
on:
  schedule:
    - cron: \"0 6 * * 1\"

jobs:
  redteam:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npx promptfoo@latest redteam run -o redteam-report.html
", Para [Strong [Str "Custom provider wrapping an existing API"], Str ":"], CodeBlock ("", ["javascript"], []) "// custom_provider.js
module.exports = async function (prompt, context) {
  const response = await fetch(\"https://my-api.example.com/generate\", {
    method: \"POST\",
    headers: { \"Content-Type\": \"application/json\" },
    body: JSON.stringify({ prompt, parameters: context.vars }),
  });
  const data = await response.json();
  return { output: data.text };
};
", CodeBlock ("", ["yaml"], []) "providers:
  - file://custom_provider.js
", Para [Strong [Str "Python custom assertion"], Str ":"], CodeBlock ("", ["python"], []) "# assert_no_pii.py
import re

def get_assert(output, context):
    pii_patterns = [
        r'\\b\\d{3}-\\d{2}-\\d{4}\\b',  # SSN
        r'\\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\\.[A-Z|a-z]{2,}\\b',  # Email
    ]
    for pattern in pii_patterns:
        if re.search(pattern, output):
            return {\"pass\": False, \"score\": 0, \"reason\": f\"PII detected: {pattern}\"}
    return {\"pass\": True, \"score\": 1.0, \"reason\": \"No PII detected\"}
", CodeBlock ("", ["yaml"], []) "assert:
  - type: python
    value: file://assert_no_pii.py
", Para [Strong [Str "Using promptfoo as a Node.js library in a test runner"], Str ":"], CodeBlock ("", ["typescript"], []) "import { describe, it, expect } from \"vitest\";
import promptfoo from \"promptfoo\";

describe(\"Summarization prompt\", () => {
  it(\"produces concise summaries\", async () => {
    const results = await promptfoo.evaluate({
      prompts: [\"Summarize the following text in 2 sentences: {{text}}\"],
      providers: [\"openai:gpt-4o\"],
      tests: [
        {
          vars: { text: \"Long article content here...\" },
          assert: [
            { type: \"javascript\", value: \"output.split('.').length <= 3\" },
            { type: \"llm-rubric\", value: \"Is a faithful summary of the input\" },
          ],
        },
      ],
    });
    expect(results.stats.failures).toBe(0);
  });
});
", Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Para [Strong [Str "Comparing multiple prompts across providers"], Str ":"], CodeBlock ("", ["yaml"], []) "description: \"Prompt comparison for customer support\"

prompts:
  - file://prompts/concise.txt
  - file://prompts/detailed.txt
  - file://prompts/structured.txt

providers:
  - openai:gpt-4o
  - anthropic:messages:claude-sonnet-4-20250514
  - openai:gpt-4o-mini

defaultTest:
  assert:
    - type: llm-rubric
      value: \"Response addresses the customer's issue directly and professionally\"
      weight: 3
    - type: cost
      threshold: 0.02
    - type: latency
      threshold: 5000

tests:
  - vars:
      issue: \"My order arrived damaged\"
    assert:
      - type: contains-any
        value:
          - \"refund\"
          - \"replacement\"
          - \"return\"
  - vars:
      issue: \"I was charged twice for the same item\"
    assert:
      - type: not-contains
        value: \"I don't know\"
  - vars:
      issue: \"How do I cancel my subscription?\"
    assert:
      - type: contains
        value: \"cancel\"
", Para [Strong [Str "RAG pipeline evaluation with context assertions"], Str ":"], CodeBlock ("", ["yaml"], []) "description: \"RAG quality evaluation\"

prompts:
  - |
    Context: {{context}}
    Question: {{question}}
    Answer the question using only the provided context.

providers:
  - openai:gpt-4o

tests:
  - vars:
      context: \"The company was founded in 2015 by Jane Smith in San Francisco.\"
      question: \"When was the company founded?\"
    assert:
      - type: contains
        value: \"2015\"
      - type: context-faithfulness
        threshold: 0.9
      - type: answer-relevance
        threshold: 0.8
      - type: not-contains
        value: \"I'm not sure\"
  - vars:
      context: \"Revenue grew 40% year-over-year to $50M in 2024.\"
      question: \"What was the revenue growth rate?\"
    assert:
      - type: contains
        value: \"40%\"
      - type: factuality
        value: \"Revenue grew 40% year-over-year\"
", Para [Strong [Str "Loading tests from CSV for large-scale evaluation"], Str ":"], CodeBlock ("", ["csv"], []) "question,expected_topic,max_cost
\"What is photosynthesis?\",biology,0.005
\"Explain the French Revolution\",history,0.008
\"How does TCP/IP work?\",networking,0.006
", CodeBlock ("", ["yaml"], []) "prompts:
  - \"Answer this question concisely: {{question}}\"

providers:
  - openai:gpt-4o-mini

tests: file://test_cases.csv

defaultTest:
  assert:
    - type: contains
      value: \"{{expected_topic}}\"
    - type: cost
      threshold: \"{{max_cost}}\"
", Para [Strong [Str "Red-team configuration"], Str ":"], CodeBlock ("", ["yaml"], []) "# promptfooconfig.redteam.yaml
description: \"Security red team scan\"

targets:
  - openai:gpt-4o

redteam:
  plugins:
    - harmful:hate
    - harmful:self-harm
    - pii:direct
    - pii:session
    - hijacking
    - jailbreak
    - prompt-injection
  strategies:
    - jailbreak
    - prompt-injection
", CodeBlock ("", ["bash"], []) "promptfoo redteam run -c promptfooconfig.redteam.yaml -o report.html
", Header 2 ("limitations-and-considerations", ["unnumbered", "unlisted"], []) [Str "Limitations and Considerations"], BulletList [[Plain [Strong [Str "Node.js dependency"], Str ": promptfoo is built on Node.js and distributed via npm. Projects not already using the Node.js ecosystem must add it as a toolchain dependency. The Python integration is limited to custom providers and assertions via file references, not a native Python SDK."]], [Plain [Strong [Str "YAML configuration complexity"], Str ": As evaluation suites grow with many prompts, providers, scenarios, and assertion types, the YAML configuration can become unwieldy. Splitting across multiple files and using ", Code ("", [], []) "$ref", Str " references helps but adds indirection."]], [Plain [Strong [Str "Model-graded assertion costs"], Str ": Assertions like ", Code ("", [], []) "llm-rubric", Str ", ", Code ("", [], []) "factuality", Str ", and ", Code ("", [], []) "context-faithfulness", Str " invoke additional LLM calls for each test case, which can significantly increase evaluation cost and duration for large test suites."]], [Plain [Strong [Str "Non-determinism in LLM outputs"], Str ": Even with caching, LLM responses vary across runs. Tests relying on exact matching (", Code ("", [], []) "equals", Str ", ", Code ("", [], []) "contains", Str ") can be brittle unless outputs are highly constrained. Semantic assertions (", Code ("", [], []) "similar", Str ", ", Code ("", [], []) "llm-rubric", Str ") are more robust but introduce their own variability."]], [Plain [Strong [Str "Red-team coverage"], Str ": Automated red teaming identifies known vulnerability patterns through plugins, but it does not guarantee discovery of novel attack vectors or application-specific risks that require domain expertise."]], [Plain [Strong [Str "Web viewer is local-only"], Str ": The built-in web viewer runs locally and does not provide a hosted dashboard for team collaboration without using the optional cloud sharing feature."]], [Plain [Strong [Str "TypeScript-first documentation"], Str ": Examples and documentation lean toward TypeScript/JavaScript. Developers working primarily in Python may find fewer native integration patterns."]]], Header 2 ("changelog-highlights", ["unnumbered", "unlisted"], []) [Str "Changelog Highlights"], Para [Str "promptfoo is under active development with frequent releases. Key evolutionary milestones include the introduction of the red-teaming subsystem with plugin-based vulnerability scanning, support for over 70 LLM providers including local model runners like Ollama and vLLM, the Model Context Protocol (MCP) server mode for agent integration, assertion sets with collective thresholds and weighted scoring, extension hooks for lifecycle customization, synthetic test generation and assertion generation commands, and the ", Code ("", [], []) "promptfoo scan-model", Str " command for ML model vulnerability scanning."], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " ", Link ("", [], []) [Str "promptfoo Documentation - Introduction"] ("https://www.promptfoo.dev/docs/intro/", "")]], [Plain [Str "[", Str "2", Str "]", Str " ", Link ("", [], []) [Str "promptfoo Documentation - Configuration Guide"] ("https://www.promptfoo.dev/docs/configuration/guide/", "")]], [Plain [Str "[", Str "3", Str "]", Str " ", Link ("", [], []) [Str "promptfoo Documentation - Configuration Reference"] ("https://www.promptfoo.dev/docs/configuration/reference/", "")]], [Plain [Str "[", Str "4", Str "]", Str " ", Link ("", [], []) [Str "promptfoo Documentation - Expected Outputs (Assertions)"] ("https://www.promptfoo.dev/docs/configuration/expected-outputs/", "")]], [Plain [Str "[", Str "5", Str "]", Str " ", Link ("", [], []) [Str "promptfoo Documentation - Providers"] ("https://www.promptfoo.dev/docs/providers/", "")]], [Plain [Str "[", Str "6", Str "]", Str " ", Link ("", [], []) [Str "promptfoo Documentation - CLI Commands"] ("https://www.promptfoo.dev/docs/usage/command-line/", "")]], [Plain [Str "[", Str "7", Str "]", Str " ", Link ("", [], []) [Str "promptfoo Documentation - Node.js Package"] ("https://www.promptfoo.dev/docs/usage/node-package/", "")]], [Plain [Str "[", Str "8", Str "]", Str " ", Link ("", [], []) [Str "promptfoo Documentation - Red Teaming"] ("https://www.promptfoo.dev/docs/red-team/", "")]], [Plain [Str "[", Str "9", Str "]", Str " ", Link ("", [], []) [Str "promptfoo Documentation - CI/CD Integration"] ("https://www.promptfoo.dev/docs/integrations/ci-cd/", "")]], [Plain [Str "[", Str "10", Str "]", Str " ", Link ("", [], []) [Str "promptfoo GitHub Repository"] ("https://github.com/promptfoo/promptfoo", "")]]]]