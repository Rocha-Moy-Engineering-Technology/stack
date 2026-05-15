# promptfoo

> Open-source CLI and library for evaluating and red-teaming LLM applications

| Field | Value |
|-------|-------|
| Group | Evaluation |
| Type | SDK |
| Open Source | Yes |
| GitHub | [https://github.com/promptfoo/promptfoo](https://github.com/promptfoo/promptfoo) |
| Stars | 21253 |
| Documentation | [Official Docs](https://www.promptfoo.dev/docs/intro/) |

## Overview

promptfoo is an open-source command-line interface (CLI) and Node.js library for systematically evaluating and red-teaming Large Language Model (LLM) applications. Rather than relying on manual trial-and-error prompt engineering, promptfoo brings test-driven development (TDD) practices to LLM workflows by defining test cases declaratively in YAML, running them against one or more providers, and automatically scoring outputs against deterministic and model-graded assertions.

The tool is language-agnostic and provider-agnostic: it supports over 70 LLM providers including OpenAI, Anthropic, Google Gemini, Azure OpenAI, AWS Bedrock, Mistral, HuggingFace, Ollama, and custom HTTP/WebSocket APIs. All execution happens locally on the developer's machine, communicating directly with LLM APIs without routing through intermediary services. This local-first design gives teams full control over their data and credentials.

promptfoo operates across three usage modes. As a CLI, it runs evaluations from the terminal with caching, concurrency, and live reloading for rapid iteration. As a Node.js library, it exposes a programmatic `evaluate()` function for integration into custom applications and test suites. As a CI/CD component, it plugs into GitHub Actions, GitLab CI, Jenkins, and other pipelines to enforce quality gates and security scanning on every code change. A built-in web viewer provides matrix-style side-by-side comparison of prompt-provider-test combinations, making it straightforward to identify regressions and compare model performance.

Beyond functional evaluation, promptfoo includes a dedicated red-teaming and penetration-testing subsystem. This generates adversarial inputs targeting prompt injection, jailbreaking, harmful content generation, PII leakage, hallucination, and other vulnerability categories. Red-team results produce quantitative risk scores and structured reports suitable for security audits.

## Core Concepts

**Prompts** are the text templates sent to LLM providers. promptfoo supports inline prompt strings, file references using `file://` paths, and Nunjucks templating with double-brace variable substitution (`{{variable}}`). Multiple prompts can be evaluated side-by-side in a single run, creating a matrix of prompt-provider combinations.

**Providers** represent the LLM backends that process prompts. A provider is identified by a string in the format `provider_name:model_name` (for example, `openai:gpt-4o` or `anthropic:messages:claude-sonnet-4-20250514`). Providers can also be configured as objects with parameters like temperature, max tokens, and API keys, or referenced from external files and custom scripts.

**Tests** are individual evaluation cases consisting of input variables, optional assertions, and metadata. Each test supplies a set of variables that are substituted into the prompt template before being sent to providers. When multiple variable values are provided as arrays, promptfoo automatically generates combinatorial test runs.

**Assertions** define the pass/fail criteria for test outputs. promptfoo provides two categories: deterministic assertions (exact match, contains, regex, JSON validation, cost, latency) and model-graded assertions (semantic similarity, LLM rubric, factuality, context faithfulness, answer relevance). Assertions can be weighted, negated with a `not-` prefix, and grouped into assertion sets with collective thresholds.

**defaultTest** is a configuration block that sets shared properties (variables, assertions, options) applied to every test case. This reduces duplication when all tests share common validation criteria.

**Scenarios** group a set of variable data with a set of tests, creating a test matrix. For example, three input phrases tested across four languages automatically produce twelve test runs without manually defining each combination.

**Evaluation** is the process of running all prompt-provider-test combinations, collecting outputs, scoring them against assertions, and producing a summary with pass/fail counts, scores, and token usage statistics.

**Red Teaming** is an automated adversarial testing process that generates malicious inputs, runs them through the target application, and analyzes responses for vulnerabilities using plugins that target specific risk categories.

## Architecture

promptfoo follows a declarative configuration architecture where the evaluation pipeline is defined in YAML and executed by the runtime engine. The architecture consists of five stages:

1. **Configuration loading** parses `promptfooconfig.yaml`, resolves file references for prompts, providers, and test data, expands variable arrays into combinatorial test cases, and merges `defaultTest` properties into each test case.

2. **Provider initialization** creates API client instances for each specified provider, configuring authentication, model parameters, and any custom provider implementations (JavaScript/Python functions, HTTP endpoints, shell commands).

3. **Evaluation execution** sends each prompt-variable combination to each provider with configurable concurrency (default: 4 concurrent requests), caching results to disk to avoid redundant API calls. Outputs pass through optional transform functions before assertion evaluation.

4. **Assertion scoring** applies each test's assertions to the provider output, producing per-assertion pass/fail results and numerical scores. Model-graded assertions invoke a separate LLM call (configurable via the assertion's `provider` field) to evaluate output quality. Scores are aggregated using weighted averaging or custom scoring functions.

5. **Result aggregation** compiles all results into a summary structure containing per-test results, aggregate statistics (successes, failures, errors, token usage), and formatted output for CLI display, JSON export, or the web viewer.

The architecture is extensible at every stage: custom providers handle arbitrary backends, extension hooks (`beforeAll`, `afterAll`, `beforeEach`, `afterEach`) inject lifecycle logic, and custom assertion functions implement domain-specific validation.

## Key Features and Functionality

**Deterministic Assertions** provide programmatic validation without LLM calls. These include exact string matching (`equals`), substring detection (`contains`, `icontains`), regular expressions (`regex`), JSON schema validation (`is-json`, `contains-json`), SQL/XML/HTML validation, refusal detection (`is-refusal`), text similarity metrics (ROUGE-N, BLEU, METEOR, Levenshtein distance), and operational constraints (`latency`, `cost`).

**Model-Graded Assertions** use LLMs or embedding models to evaluate output quality. Semantic similarity (`similar`) uses embedding cosine distance with a configurable threshold. LLM rubric (`llm-rubric`) evaluates output against a free-text description of expected quality. Context faithfulness (`context-faithfulness`) and context recall (`context-recall`) evaluate Retrieval-Augmented Generation (RAG) pipeline quality. Factuality (`factuality`) checks output against reference facts.

**Caching and Concurrency** accelerate evaluation loops. LLM responses are cached to disk by default, so re-running evaluations with the same inputs skips redundant API calls. Concurrency is configurable via `maxConcurrency` (default: 4), and a delay between calls can be set to respect rate limits.

**Live Reloading** watches configuration files and re-runs evaluations automatically when changes are detected, using the `--watch` flag. Combined with caching, this creates a rapid feedback loop during prompt development.

**Red Teaming and Penetration Testing** generates adversarial inputs targeting model-layer threats (prompt injection, jailbreaking, harmful content, PII leakage, hallucination) and application-layer threats (indirect prompt injection, information leakage from RAG context, unauthorized tool access, data exfiltration). Results produce quantitative vulnerability scores and structured reports.

**Side-by-Side Comparison** via the web viewer and CLI output presents a matrix view of all prompt-provider-test combinations with color-coded pass/fail indicators, assertion scores, and raw outputs. This makes it straightforward to compare multiple prompts or models on the same test suite.

**Output Transforms** modify provider outputs before assertion evaluation. Transforms can be inline JavaScript expressions, external JavaScript/Python files, or provider-level transforms. The execution order is: provider transforms first, then test-level transforms.

**External Test Data** can be loaded from CSV, JSON, YAML, or JavaScript/Python files using `file://` references. Glob patterns (`file://tests/*.yaml`) enable batch loading, and Google Sheets integration is supported for collaborative test management.

**Synthetic Test Generation** via `promptfoo generate dataset` creates test cases from prompt descriptions, and `promptfoo generate assertions` produces assertion sets from example outputs.

## Use Cases

- **Prompt engineering iteration**: Compare multiple prompt variants across models to find the highest-quality, most cost-effective combination for a specific task
- **Model migration evaluation**: When switching from one LLM provider or model version to another, run the existing test suite against the new target to quantify quality differences
- **RAG pipeline validation**: Use context-faithfulness, context-recall, and context-relevance assertions to verify that retrieval-augmented generation produces grounded, accurate outputs
- **Security auditing**: Run red-team scans to identify prompt injection vulnerabilities, harmful content generation risks, and data leakage paths before deploying to production
- **Regression testing in CI/CD**: Integrate evaluations into pull request workflows to catch prompt or model configuration changes that degrade output quality
- **Cost and latency optimization**: Use `cost` and `latency` assertions to enforce operational budgets and response time requirements across providers
- **Compliance verification**: Apply deterministic assertions (regex, contains, is-refusal) to verify that outputs conform to regulatory requirements such as content restrictions or mandatory disclosures

## API Reference Summary

**CLI Commands**:

- `promptfoo init [directory]` -- Initialize a new project with configuration files
- `promptfoo eval` -- Run evaluation (flags: `-c` config, `-p` prompts, `-r` providers, `-t` tests, `-o` output, `--watch`, `--share`, `--resume`)
- `promptfoo view` -- Launch browser-based result viewer (flag: `-p` port)
- `promptfoo share [evalId]` -- Generate shareable URL for results
- `promptfoo validate` -- Check configuration schema compliance
- `promptfoo cache clear` -- Clear cached LLM responses
- `promptfoo list evals|prompts|datasets` -- List stored resources
- `promptfoo export eval <evalId>` -- Export results to JSON
- `promptfoo generate dataset` -- Generate synthetic test cases
- `promptfoo generate assertions` -- Generate assertion sets from examples
- `promptfoo redteam init` -- Initialize red-team configuration
- `promptfoo redteam run` -- Execute red-team scan
- `promptfoo redteam report` -- Generate vulnerability report

**Node.js Library**:

```typescript
import promptfoo from "promptfoo";

const results = await promptfoo.evaluate({
  prompts: ["Translate to {{language}}: {{input}}"],
  providers: ["openai:gpt-4o"],
  tests: [
    {
      vars: { language: "French", input: "Hello" },
      assert: [{ type: "contains", value: "Bonjour" }],
    },
  ],
});

// results.stats: { successes, failures, tokenUsage }
// results.results: per-test result array
// results.table: formatted tabular output
```

**`evaluate(testSuite, options)`** accepts a `TestSuiteConfiguration` object and an optional `EvaluateOptions` object (with `maxConcurrency`). It returns an `EvaluateSummary` containing `results`, `stats`, `table`, and optionally `shareableUrl`.

**`loadApiProvider(providerString, config)`** loads a provider programmatically with optional overrides for `apiHost`, `apiKey`, and model parameters.

**Custom Provider Function**:

```typescript
async function customProvider(
  prompt: string,
  context: { vars: Record<string, string> }
): Promise<string | { output: string; error?: string }> {
  // Call custom backend
  return { output: "response text" };
}
```

**Custom Assertion Function**:

```typescript
function customAssertion(
  output: string,
  testCase: TestCase,
  assertion: Assertion
): GradingResult {
  return { pass: true, score: 1.0, reason: "Meets criteria" };
}
```

## Configuration and Customization

The full configuration reference for `promptfooconfig.yaml`:

```yaml
# Top-level metadata
description: "Evaluation suite description"
tags:
  env: staging
  team: nlp

# Prompts: inline strings, file references, or functions
prompts:
  - "Answer the question: {{question}}"
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
      value: "Response is helpful, accurate, and concise"
  options:
    transform: output.trim()

# Test cases
tests:
  - description: "Basic greeting"
    vars:
      question: "What is machine learning?"
    assert:
      - type: contains
        value: "algorithm"
      - type: not-contains
        value: "I don't know"
      - type: latency
        threshold: 3000
      - type: cost
        threshold: 0.01

# Scenarios for combinatorial testing
scenarios:
  - description: "Multi-language support"
    config:
      - vars:
          language: Spanish
      - vars:
          language: French
      - vars:
          language: German
    tests:
      - vars:
          input: "Hello world"
        assert:
          - type: llm-rubric
            value: "Is a correct translation"

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
```

**Assertion weighting and sets**:

```yaml
assert:
  - type: contains-json
    weight: 2
  - type: llm-rubric
    value: "Response is factually accurate"
    weight: 3
  - type: assert-set
    threshold: 0.5
    assert:
      - type: cost
        threshold: 0.005
      - type: latency
        threshold: 2000
```

**Named metrics** for dashboard aggregation:

```yaml
assert:
  - type: llm-rubric
    value: "Response is helpful"
    metric: helpfulness
  - type: similar
    value: "expected output"
    threshold: 0.8
    metric: semantic_accuracy
```

**Variable transforms** for pre-processing inputs:

```yaml
tests:
  - vars:
      document: file://data/contract.txt
    options:
      transformVars: "{ ...vars, document: vars.document.substring(0, 5000) }"
```

## Integration Patterns

**GitHub Actions** for automated evaluation on pull requests:

```yaml
name: LLM Evaluation
on:
  pull_request:
    paths:
      - "prompts/**"
      - "promptfooconfig.yaml"

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
          if (( $(echo "$PASS_RATE < 90" | bc -l) )); then
            echo "Pass rate $PASS_RATE% is below 90% threshold"
            exit 1
          fi
      - uses: actions/upload-artifact@v4
        with:
          name: eval-results
          path: results.json
```

**Red-team scanning on schedule**:

```yaml
name: Security Scan
on:
  schedule:
    - cron: "0 6 * * 1"

jobs:
  redteam:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npx promptfoo@latest redteam run -o redteam-report.html
```

**Custom provider wrapping an existing API**:

```javascript
// custom_provider.js
module.exports = async function (prompt, context) {
  const response = await fetch("https://my-api.example.com/generate", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ prompt, parameters: context.vars }),
  });
  const data = await response.json();
  return { output: data.text };
};
```

```yaml
providers:
  - file://custom_provider.js
```

**Python custom assertion**:

```python
# assert_no_pii.py
import re

def get_assert(output, context):
    pii_patterns = [
        r'\b\d{3}-\d{2}-\d{4}\b',  # SSN
        r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b',  # Email
    ]
    for pattern in pii_patterns:
        if re.search(pattern, output):
            return {"pass": False, "score": 0, "reason": f"PII detected: {pattern}"}
    return {"pass": True, "score": 1.0, "reason": "No PII detected"}
```

```yaml
assert:
  - type: python
    value: file://assert_no_pii.py
```

**Using promptfoo as a Node.js library in a test runner**:

```typescript
import { describe, it, expect } from "vitest";
import promptfoo from "promptfoo";

describe("Summarization prompt", () => {
  it("produces concise summaries", async () => {
    const results = await promptfoo.evaluate({
      prompts: ["Summarize the following text in 2 sentences: {{text}}"],
      providers: ["openai:gpt-4o"],
      tests: [
        {
          vars: { text: "Long article content here..." },
          assert: [
            { type: "javascript", value: "output.split('.').length <= 3" },
            { type: "llm-rubric", value: "Is a faithful summary of the input" },
          ],
        },
      ],
    });
    expect(results.stats.failures).toBe(0);
  });
});
```

## Examples

**Comparing multiple prompts across providers**:

```yaml
description: "Prompt comparison for customer support"

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
      value: "Response addresses the customer's issue directly and professionally"
      weight: 3
    - type: cost
      threshold: 0.02
    - type: latency
      threshold: 5000

tests:
  - vars:
      issue: "My order arrived damaged"
    assert:
      - type: contains-any
        value:
          - "refund"
          - "replacement"
          - "return"
  - vars:
      issue: "I was charged twice for the same item"
    assert:
      - type: not-contains
        value: "I don't know"
  - vars:
      issue: "How do I cancel my subscription?"
    assert:
      - type: contains
        value: "cancel"
```

**RAG pipeline evaluation with context assertions**:

```yaml
description: "RAG quality evaluation"

prompts:
  - |
    Context: {{context}}
    Question: {{question}}
    Answer the question using only the provided context.

providers:
  - openai:gpt-4o

tests:
  - vars:
      context: "The company was founded in 2015 by Jane Smith in San Francisco."
      question: "When was the company founded?"
    assert:
      - type: contains
        value: "2015"
      - type: context-faithfulness
        threshold: 0.9
      - type: answer-relevance
        threshold: 0.8
      - type: not-contains
        value: "I'm not sure"
  - vars:
      context: "Revenue grew 40% year-over-year to $50M in 2024."
      question: "What was the revenue growth rate?"
    assert:
      - type: contains
        value: "40%"
      - type: factuality
        value: "Revenue grew 40% year-over-year"
```

**Loading tests from CSV for large-scale evaluation**:

```csv
question,expected_topic,max_cost
"What is photosynthesis?",biology,0.005
"Explain the French Revolution",history,0.008
"How does TCP/IP work?",networking,0.006
```

```yaml
prompts:
  - "Answer this question concisely: {{question}}"

providers:
  - openai:gpt-4o-mini

tests: file://test_cases.csv

defaultTest:
  assert:
    - type: contains
      value: "{{expected_topic}}"
    - type: cost
      threshold: "{{max_cost}}"
```

**Red-team configuration**:

```yaml
# promptfooconfig.redteam.yaml
description: "Security red team scan"

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
```

```bash
promptfoo redteam run -c promptfooconfig.redteam.yaml -o report.html
```

## Limitations and Considerations

- **Node.js dependency**: promptfoo is built on Node.js and distributed via npm. Projects not already using the Node.js ecosystem must add it as a toolchain dependency. The Python integration is limited to custom providers and assertions via file references, not a native Python SDK.
- **YAML configuration complexity**: As evaluation suites grow with many prompts, providers, scenarios, and assertion types, the YAML configuration can become unwieldy. Splitting across multiple files and using `$ref` references helps but adds indirection.
- **Model-graded assertion costs**: Assertions like `llm-rubric`, `factuality`, and `context-faithfulness` invoke additional LLM calls for each test case, which can significantly increase evaluation cost and duration for large test suites.
- **Non-determinism in LLM outputs**: Even with caching, LLM responses vary across runs. Tests relying on exact matching (`equals`, `contains`) can be brittle unless outputs are highly constrained. Semantic assertions (`similar`, `llm-rubric`) are more robust but introduce their own variability.
- **Red-team coverage**: Automated red teaming identifies known vulnerability patterns through plugins, but it does not guarantee discovery of novel attack vectors or application-specific risks that require domain expertise.
- **Web viewer is local-only**: The built-in web viewer runs locally and does not provide a hosted dashboard for team collaboration without using the optional cloud sharing feature.
- **TypeScript-first documentation**: Examples and documentation lean toward TypeScript/JavaScript. Developers working primarily in Python may find fewer native integration patterns.

## Changelog Highlights

promptfoo is under active development with frequent releases. Key evolutionary milestones include the introduction of the red-teaming subsystem with plugin-based vulnerability scanning, support for over 70 LLM providers including local model runners like Ollama and vLLM, the Model Context Protocol (MCP) server mode for agent integration, assertion sets with collective thresholds and weighted scoring, extension hooks for lifecycle customization, synthetic test generation and assertion generation commands, and the `promptfoo scan-model` command for ML model vulnerability scanning.

## Citations

- [1] [promptfoo Documentation - Introduction](https://www.promptfoo.dev/docs/intro/)
- [2] [promptfoo Documentation - Configuration Guide](https://www.promptfoo.dev/docs/configuration/guide/)
- [3] [promptfoo Documentation - Configuration Reference](https://www.promptfoo.dev/docs/configuration/reference/)
- [4] [promptfoo Documentation - Expected Outputs (Assertions)](https://www.promptfoo.dev/docs/configuration/expected-outputs/)
- [5] [promptfoo Documentation - Providers](https://www.promptfoo.dev/docs/providers/)
- [6] [promptfoo Documentation - CLI Commands](https://www.promptfoo.dev/docs/usage/command-line/)
- [7] [promptfoo Documentation - Node.js Package](https://www.promptfoo.dev/docs/usage/node-package/)
- [8] [promptfoo Documentation - Red Teaming](https://www.promptfoo.dev/docs/red-team/)
- [9] [promptfoo Documentation - CI/CD Integration](https://www.promptfoo.dev/docs/integrations/ci-cd/)
- [10] [promptfoo GitHub Repository](https://github.com/promptfoo/promptfoo)

## Discovery Signals

> Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search.

### Keywords

- promptfoo
- prompt comparison
- side-by-side evaluation
- YAML configuration
- Node.js library
- CLI evaluator
- deterministic assertions
- model-graded assertions
- llm-rubric
- context-faithfulness
- semantic similarity
- regex assertion
- JSON validation
- cost threshold
- latency threshold
- red teaming
- penetration testing
- web viewer
- 70+ providers
- Nunjucks templating
- scenarios
- defaultTest
- GitHub Actions
- assertion sets
- promptfooconfig.yaml

### Verb-Noun Tasks

- Compare multiple prompts across multiple models
- Define test cases declaratively in YAML
- Assert exact match, contains, regex, JSON schema, or refusal
- Score outputs using LLM-graded rubrics
- Enforce cost and latency budgets per test
- Cache LLM responses between evaluation runs
- Launch the local web viewer for matrix-style comparison
- Run a red-team scan against a target provider
- Integrate evaluation into GitHub Actions for PR gating
- Load test data from CSV, JSON, or Google Sheets
- Generate synthetic test cases via promptfoo generate dataset
- Wrap a custom backend with a JavaScript or Python provider
- Apply weighted assertions and assertion sets
- Evaluate RAG with context-faithfulness and answer-relevance

### User Intent Phrases

- How do I run side-by-side prompt comparisons across providers?
- I need a CLI that runs LLM evals with YAML configuration.
- Add LLM evaluation to my GitHub Actions workflow.
- How do I red-team a chatbot for prompt injection and PII leakage?
- Test the same prompt against OpenAI, Anthropic, and Mistral.
- Enforce a maximum cost per LLM call in tests.
- Define a custom assertion in Python that runs locally.
- View results in a browser as a pass/fail matrix.
- How do I generate test cases automatically from a prompt?
- Compare gpt-4o-mini vs gpt-4o on the same suite.

### Problem Statements

- No deterministic harness for comparing prompt variants across models.
- Prompt changes silently regress quality with no automated detection.
- Eval costs explode without local caching of LLM responses.
- Switching models requires re-validating an entire test suite.
- Security testing for prompt injection is manual and inconsistent.
- Cost and latency budgets are not enforced as part of QA.
- Team has no shared, reproducible view of LLM test results.

### When to Pick This

- Pick this when you want a CLI-first, YAML-driven, side-by-side prompt comparison harness with a local web viewer.
- Pick this over Ragas when prompt comparison and CI gating across many providers matter more than RAG-specific metrics.
- Pick this over DeepEval when Node.js + YAML + CLI is preferred over Python + Pytest.
- Pick this over OpenAI Evals when provider-agnostic evaluation across 70+ providers and local execution are requirements.
- Pick this when integrated red-teaming with plugins (jailbreak, prompt-injection, PII, hijacking) is part of the workflow.
- Pick this when cost and latency thresholds must be first-class assertion types alongside quality.

### Related Terms and Aliases

- prompt eval CLI
- LLM test harness
- prompt regression suite
- prompt A/B testing
- YAML LLM tests
- promptfoo redteam
- llm-rubric grader
- assertion-based LLM testing
- LLM CI/CD gate
- prompt engineering toolkit
- side-by-side LLM viewer
- node-based LLM evaluator
