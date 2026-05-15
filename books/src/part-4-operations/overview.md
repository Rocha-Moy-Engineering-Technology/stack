# Part IV — Operations & Quality

This part covers the operations layer around production deployments — safety, evaluation, observability, and workflow orchestration. Four groups span the lifecycle of a request: what happens before it hits the model, how its quality gets measured, what gets logged for diagnosis, and how it fits into a broader scheduled or automated pipeline.

## What's in this part

- **Guardrails** — input/output validation and content moderation (Guardrails AI, NeMo Guardrails, OpenAI Moderation, Lakera)
- **Evaluation** — LLM and RAG evaluation frameworks (Ragas, DeepEval, OpenAI Evals, promptfoo)
- **Observability** — tracing, monitoring, prompt management (LangSmith, Arize Phoenix, Weights & Biases, Helicone, Langfuse)
- **Workflow Orchestration** — pipeline scheduling and automation (Temporal, Prefect, Airflow, n8n, Activepieces, Node-RED)

## How to navigate this part

These four groups address concerns that have always existed in software engineering — safety, testing, observability, scheduling — but with LLM-specific characteristics that make off-the-shelf tools insufficient.

**Guardrails** sits at the boundary: input filtering (jailbreak detection, PII redaction, topic restriction) before the request reaches the model, and output validation (schema conformance, content moderation, hallucination detection) before the response reaches the user. **Evaluation** sits in development and CI: how do you know your prompt change didn't regress retrieval accuracy or factuality? Ragas specializes in RAG metrics; DeepEval has the widest metric catalog; promptfoo is the developer-tools CLI for prompt comparison; OpenAI Evals integrates with OpenAI's platform.

**Observability** is the production-runtime story: every LLM call produces a structured trace with cost, latency, token counts, prompt template version, and tool-call hierarchy. LangSmith, Langfuse, Arize Phoenix, Helicone, and Weights & Biases each lean toward a different deployment model (LangChain-integrated, open-source self-hosted, OpenTelemetry-native, zero-code proxy, or general ML experiment-tracking). **Workflow Orchestration** is the broader picture — Temporal for durable execution, Prefect and Airflow for Python-native scheduling, n8n/Activepieces/Node-RED for visual no-code or low-code flows.

For most teams, the path is: pick one observability platform (LangSmith if you're already on LangGraph, Langfuse if you want OSS), layer in guardrails as you encounter specific safety incidents, build out evaluation as your prompt library grows, and bring in workflow orchestration when your AI features become multi-step pipelines rather than single LLM calls.
