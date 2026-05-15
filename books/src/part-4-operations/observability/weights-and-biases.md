# Weights & Biases

> ML developer platform for experiment tracking and model monitoring

| Field | Value |
|-------|-------|
| Group | Observability |
| Type | API/SDK/UI |
| Open Source | No |
| GitHub | N/A |
| Stars | N/A |
| Documentation | [Official Docs](https://docs.wandb.ai/) |

## Overview

Weights & Biases (W&B) is a machine learning developer platform for experiment tracking, hyperparameter optimization, model versioning, and Large Language Model (LLM) observability. The platform provides a unified system of record for ML workflows, covering the full lifecycle from training through evaluation and deployment.

W&B is organized into four product areas. W&B Models manages traditional ML and deep learning development with experiment tracking, hyperparameter sweeps, interactive reports, and a model registry. W&B Weave provides LLM-specific observability through tracing, output evaluation, cost estimation, and an inference playground for comparing language models. W&B Inference offers an OpenAI-compatible API for accessing open-source foundation models. W&B Training (public preview) enables post-training of LLMs using serverless Reinforcement Learning (RL) with managed GPU infrastructure.

The platform is commercial and cloud-hosted, with self-hosted deployment options available for enterprise customers. W&B integrates with all major ML frameworks including PyTorch, TensorFlow, Keras, Hugging Face Transformers, PyTorch Lightning, XGBoost, and scikit-learn.

## Core Concepts

**Run** is the fundamental unit of tracking in W&B. A Run represents a single execution of a training script, evaluation pipeline, or any tracked computation. Each Run captures hyperparameters, metrics, system resource usage, and output artifacts. Runs are created with `wandb.init()` and finalized with `wandb.finish()`.

**Project** is a collection of Runs grouped by a shared objective. Projects provide a workspace where teams can compare experiments, visualize metrics, and collaborate on model development.

**Config** stores hyperparameters and settings for a Run. The `wandb.config` object captures the input configuration (learning rate, batch size, architecture choices) that produced specific results, enabling reproducibility and comparison across experiments.

**Metrics** are time-series values logged during a Run using `wandb.log()`. Metrics can be scalars (loss, accuracy), media (images, audio, video), tables, histograms, or custom objects. W&B automatically generates interactive charts from logged metrics.

**Artifacts** are versioned datasets, models, and intermediate outputs tracked as inputs and outputs of Runs. Artifacts enable lineage tracking, showing which data produced which model, and support deduplication through content-addressable storage.

**Sweeps** automate hyperparameter search across a defined parameter space using strategies such as grid search, random search, and Bayesian optimization. A sweep controller manages the search process and distributes work to agents running on one or more machines.

**Reports** are collaborative documents that embed live charts, metrics, and visualizations from experiments. Reports allow teams to annotate results, share findings, and create reproducible documentation of model development.

**Registry** is a curated central repository of artifact versions within an organization. The registry supports model lifecycle management including promotion, tagging, lineage auditing, and automated downstream processes such as model CI/CD.

**Weave Ops** are versioned, tracked functions in the Weave product. Decorated with `@weave.op`, they automatically capture code, inputs, outputs, timing, and execution metadata for LLM application observability.

**Weave Calls** represent logged executions of Weave Ops. Each call records input arguments, output values, latency, parent-child relationships for nested calls, and any errors that occurred.

## Architecture

W&B follows a client-server architecture with a thin SDK that streams data to a cloud or self-hosted backend:

```
Application Code
    |
wandb SDK (Python)          Instruments training loops, logs metrics/artifacts
    |
Local Process               Buffers data, handles retries, manages uploads
    |
W&B Backend API             Receives and stores experiment data
    |
W&B Dashboard (UI)          Visualizes metrics, compares runs, manages artifacts
    |
W&B Registry                Curates promoted models and datasets
```

**SDK Layer**: The Python SDK (`wandb` package) provides the `init()`, `log()`, `finish()`, and artifact APIs. On `wandb.init()`, a background process starts that handles data serialization, buffering, and asynchronous upload. On exit, the SDK waits for all pending uploads to complete.

**Backend Layer**: The W&B backend stores experiment metadata, time-series metrics, system metrics, and artifact references. The backend exposes a REST API and a GraphQL API for programmatic access.

**Dashboard Layer**: The web UI renders interactive charts, comparison tables, parallel coordinate plots, and parameter importance visualizations. Dashboards update in real time as new data arrives from running experiments.

**Weave Layer**: Weave operates as a separate tracing and evaluation subsystem. It intercepts function calls decorated with `@weave.op`, serializes inputs and outputs, and streams trace data to the Weave backend for visualization and analysis.

**Storage Layer**: Artifacts use content-addressable storage with deduplication. Large files are uploaded to cloud object storage (S3, GCS, or Azure Blob) and referenced by hash. The registry layer maintains pointers to artifact versions without duplicating data.

## Key Features and Functionality

**Experiment Tracking**: Log metrics, hyperparameters, and system resources with minimal code changes. The SDK captures GPU utilization, memory usage, and other system metrics automatically alongside user-defined metrics.

```python
import wandb

run = wandb.init(project="image-classifier")
run.config.update({
    "learning_rate": 0.001,
    "epochs": 10,
    "batch_size": 32,
    "architecture": "resnet50",
})

for epoch in range(10):
    train_loss = train_one_epoch()
    val_loss = evaluate()
    run.log({
        "train_loss": train_loss,
        "val_loss": val_loss,
        "epoch": epoch,
    })

run.finish()
```

**Hyperparameter Sweeps**: Define a search space and optimization strategy, then let W&B explore parameter combinations automatically:

```python
import wandb

sweep_configuration = {
    "method": "bayes",
    "metric": {"goal": "minimize", "name": "val_loss"},
    "parameters": {
        "learning_rate": {"min": 0.0001, "max": 0.1},
        "batch_size": {"values": [16, 32, 64, 128]},
        "optimizer": {"values": ["adam", "sgd", "rmsprop"]},
    },
}

sweep_id = wandb.sweep(sweep=sweep_configuration, project="my-sweep")
wandb.agent(sweep_id, function=train, count=50)
```

**Artifact Versioning and Lineage**: Track datasets and models as versioned artifacts with automatic lineage graphs showing data provenance:

```python
import wandb

with wandb.init(project="data-pipeline") as run:
    artifact = wandb.Artifact(name="training-data", type="dataset")
    artifact.add_file("./data/train.csv")
    run.log_artifact(artifact)
```

**Model Registry**: Promote trained models to a central registry with aliases, tags, and access controls for organizational governance.

**Media Logging**: Log images, audio, video, 3D objects, HTML, and custom visualizations alongside scalar metrics:

```python
run.log({
    "examples": wandb.Table(
        columns=["image", "prediction", "ground_truth"],
        data=[[wandb.Image(img), pred, label] for img, pred, label in samples]
    )
})
```

**Interactive Reports**: Create collaborative documents embedding live charts, metric comparisons, and narrative descriptions of experiments. Reports can be exported as LaTeX or PDF.

**Weave Tracing**: Instrument LLM applications with automatic tracing of inputs, outputs, latency, and costs:

```python
import weave
from openai import OpenAI

weave.init("my-llm-app")
client = OpenAI()

@weave.op
def summarize(text: str) -> str:
    response = client.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "user", "content": f"Summarize: {text}"}],
    )
    return response.choices[0].message.content
```

**Weave Evaluations**: Systematically test LLM outputs using built-in and custom scorers for hallucination detection, summarization quality, toxicity, and embedding similarity.

## Use Cases

**Deep Learning Research**: Track thousands of training runs across GPU clusters, compare architectures and hyperparameters, and share reproducible results through reports and artifact lineage.

**Hyperparameter Optimization**: Automate parameter search using Bayesian optimization to find optimal configurations without manual trial-and-error, distributing sweep agents across multiple machines for parallel exploration.

**Model Lifecycle Management**: Version models from initial training through staging and production using the registry, with automated CI/CD pipelines triggered by artifact state changes.

**LLM Application Observability**: Trace production LLM applications to monitor latency, token usage, costs, and output quality. Identify regressions and debug failures by inspecting individual traces.

**LLM Evaluation Pipelines**: Build systematic evaluation suites for LLM applications using Weave scorers to measure response quality, factual accuracy, and safety before deploying updates.

**Dataset Versioning**: Track training, validation, and test datasets as artifacts with full lineage, ensuring every model can be traced back to the exact data that produced it.

**Team Collaboration**: Share experiment results through interactive reports, compare team members' runs in shared project dashboards, and establish organizational best practices through the registry.

## API Reference Summary

**Core SDK Functions**:

- `wandb.init(project, entity, config, name, tags, group)` -- Initialize a Run; returns a `wandb.Run` object. Parameters include project name, team entity, initial config dict, display name, tags for filtering, and group for logical run grouping.
- `wandb.log(data, step, commit)` -- Log a dictionary of metrics. Accepts scalars, `wandb.Image`, `wandb.Table`, `wandb.Histogram`, `wandb.Audio`, `wandb.Video`, and other media types.
- `wandb.finish(exit_code, quiet)` -- Finalize the current Run, flush all pending data, and wait for uploads to complete.
- `wandb.config` -- Object for storing and retrieving hyperparameters. Supports dictionary-style updates and attribute access.

**Sweep Functions**:

- `wandb.sweep(sweep, project, entity)` -- Create a sweep from a configuration dictionary; returns a sweep ID.
- `wandb.agent(sweep_id, function, count, entity, project)` -- Launch a sweep agent that executes the training function with sampled hyperparameters.

**Artifact API**:

- `wandb.Artifact(name, type, description, metadata)` -- Create a new artifact with a name and type (dataset, model, etc.).
- `artifact.add_file(local_path, name)` -- Add a local file to the artifact.
- `artifact.add_dir(local_path, name)` -- Add a directory to the artifact.
- `artifact.add_reference(uri, name)` -- Add a reference to an external file (S3, GCS).
- `run.log_artifact(artifact)` -- Log an artifact as an output of the current Run.
- `run.use_artifact(artifact_or_name)` -- Declare an artifact as an input to the current Run; returns the artifact.
- `artifact.download(root)` -- Download artifact contents to a local directory.
- `run.link_artifact(artifact, target_path)` -- Link an artifact version to a registry collection.

**Media Types**:

- `wandb.Image(data, caption)` -- Log images from arrays, PIL images, or file paths.
- `wandb.Table(columns, data)` -- Log tabular data with typed columns.
- `wandb.Audio(data, sample_rate, caption)` -- Log audio clips.
- `wandb.Video(data, fps, caption)` -- Log video files.
- `wandb.Histogram(sequence)` -- Log distribution data.
- `wandb.Html(data)` -- Log arbitrary HTML.

**Weave API**:

- `weave.init(project_name)` -- Initialize Weave tracing for a project.
- `@weave.op` -- Decorator to make a function a tracked Weave Op, capturing inputs, outputs, and execution metadata.
- `weave.Evaluation(dataset, scorers)` -- Create an evaluation pipeline with a dataset and scoring functions.
- `weave.Scorer` -- Base class for class-based evaluation scorers.

**Public API** (programmatic access):

- `wandb.Api()` -- Create a client for querying runs, artifacts, and projects.
- `api.runs(path, filters, order)` -- Query runs with filtering and sorting.
- `api.artifact(name)` -- Retrieve a specific artifact version.

## Configuration and Customization

**Run Configuration**: Pass hyperparameters at initialization or update incrementally:

```python
run = wandb.init(
    project="my-project",
    config={
        "learning_rate": 0.001,
        "architecture": "transformer",
        "dataset": "wikitext-103",
    },
)

# Update config after initialization
run.config.update({"dropout": 0.1})
```

**Environment Variables**: Control SDK behavior without code changes:

```bash
export WANDB_PROJECT="my-project"       # Default project name
export WANDB_ENTITY="my-team"           # Default team entity
export WANDB_MODE="offline"             # Run without cloud sync
export WANDB_DIR="/tmp/wandb"           # Local storage directory
export WANDB_SILENT="true"              # Suppress SDK output
export WANDB_DISABLE_CODE="true"        # Disable code saving
```

**Offline Mode**: Run experiments without network connectivity, then sync later:

```python
import os
os.environ["WANDB_MODE"] = "offline"

run = wandb.init(project="offline-experiment")
# ... log metrics as usual ...
run.finish()
```

Sync offline runs when connectivity is restored:

```bash
wandb sync ./wandb/offline-run-*
```

**Sweep Configuration Options**: Customize search strategies and early termination:

```python
sweep_config = {
    "method": "bayes",           # bayes, grid, random
    "metric": {
        "goal": "minimize",
        "name": "val_loss",
    },
    "parameters": {
        "learning_rate": {
            "distribution": "log_uniform_values",
            "min": 1e-5,
            "max": 1e-1,
        },
        "epochs": {"value": 20},  # Fixed parameter
    },
    "early_terminate": {
        "type": "hyperband",
        "min_iter": 3,
        "eta": 3,
    },
}
```

**Alerts**: Configure programmatic alerts when metrics cross thresholds:

```python
run.alert(
    title="Training diverging",
    text=f"Loss exceeded threshold: {loss}",
    level=wandb.AlertLevel.WARN,
)
```

## Integration Patterns

**PyTorch Training Loop**: Add W&B tracking to a standard PyTorch training loop with minimal changes:

```python
import wandb
import torch

run = wandb.init(project="pytorch-example")

model = MyModel()
optimizer = torch.optim.Adam(model.parameters(), lr=run.config.learning_rate)

for epoch in range(run.config.epochs):
    for batch in dataloader:
        loss = train_step(model, batch, optimizer)
        run.log({"batch_loss": loss.item()})

    val_metrics = evaluate(model, val_loader)
    run.log({"val_loss": val_metrics["loss"], "val_acc": val_metrics["accuracy"]})

run.finish()
```

**Hugging Face Transformers**: Use the built-in W&B callback:

```python
from transformers import TrainingArguments

training_args = TrainingArguments(
    output_dir="./results",
    report_to="wandb",
    run_name="bert-fine-tune",
    num_train_epochs=3,
)
```

**Keras Callback**: Integrate with Keras via the WandbCallback:

```python
from wandb.integration.keras import WandbCallback

model.fit(
    x_train, y_train,
    validation_data=(x_val, y_val),
    callbacks=[WandbCallback()],
)
```

**PyTorch Lightning Logger**: Attach W&B as a logger in Lightning:

```python
from lightning.pytorch.loggers import WandbLogger

wandb_logger = WandbLogger(project="lightning-example")
trainer = Trainer(logger=wandb_logger)
trainer.fit(model, datamodule)
```

**Weave with OpenAI**: Weave automatically traces OpenAI API calls when initialized:

```python
import weave
from openai import OpenAI

weave.init("openai-tracing")
client = OpenAI()

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Explain transformers."}],
)
```

**Weave with LangChain**: Trace LangChain chains and agents through Weave instrumentation:

```python
import weave

weave.init("langchain-tracing")

# LangChain operations are automatically traced
```

## Examples

**Complete Training Run with Artifacts**:

```python
import wandb

with wandb.init(project="artifact-demo", config={"lr": 0.01, "epochs": 5}) as run:
    # Log training data as input artifact
    dataset = run.use_artifact("training-data:latest")
    data_dir = dataset.download()

    # Train model
    model = train_model(data_dir, run.config)

    for epoch in range(run.config.epochs):
        metrics = evaluate(model)
        run.log({"epoch": epoch, **metrics})

    # Log trained model as output artifact
    model_artifact = wandb.Artifact("trained-model", type="model")
    model_artifact.add_file("model.pt")
    run.log_artifact(model_artifact)
```

**Hyperparameter Sweep with Bayesian Optimization**:

```python
import wandb

def train():
    with wandb.init() as run:
        lr = run.config.learning_rate
        batch_size = run.config.batch_size

        model = build_model(lr=lr)
        for epoch in range(10):
            loss = train_epoch(model, batch_size=batch_size)
            run.log({"loss": loss, "epoch": epoch})

sweep_config = {
    "method": "bayes",
    "metric": {"goal": "minimize", "name": "loss"},
    "parameters": {
        "learning_rate": {"min": 0.0001, "max": 0.01},
        "batch_size": {"values": [16, 32, 64]},
    },
}

if __name__ == "__main__":
    sweep_id = wandb.sweep(sweep_config, project="sweep-demo")
    wandb.agent(sweep_id, function=train, count=25)
```

**Weave Evaluation Pipeline**:

```python
import weave
from weave import Evaluation

weave.init("eval-demo")

@weave.op
def my_model(question: str) -> str:
    # LLM call here
    return answer

@weave.op
def correctness_scorer(output: str, expected: str) -> dict:
    return {"correct": output.strip().lower() == expected.strip().lower()}

dataset = [
    {"question": "What is 2+2?", "expected": "4"},
    {"question": "Capital of France?", "expected": "Paris"},
]

evaluation = Evaluation(dataset=dataset, scorers=[correctness_scorer])
results = evaluation.evaluate(my_model)
```

**Model Registry Workflow**:

```python
import wandb

with wandb.init(project="registry-demo") as run:
    # Train and log model
    model_artifact = wandb.Artifact("my-model", type="model")
    model_artifact.add_file("model.pt")
    run.log_artifact(model_artifact)

    # Link to registry
    run.link_artifact(
        artifact=model_artifact,
        target_path="my-org/wandb-registry-model/production-models",
    )
```

## Limitations and Considerations

**Commercial Platform**: W&B is a proprietary, closed-source platform. While a free tier is available for individual researchers, teams and organizations require paid subscriptions. Self-hosted deployments are available but require enterprise licensing.

**Cloud Dependency**: By default, all experiment data is transmitted to W&B cloud servers. Organizations with strict data residency or air-gapped requirements must deploy the self-hosted version, which involves additional infrastructure management.

**SDK Overhead**: The background upload process consumes CPU, memory, and network bandwidth. For very high-frequency logging (thousands of log calls per second) or resource-constrained environments, the overhead may be noticeable and require tuning of logging frequency.

**Vendor Lock-In**: Experiment data stored in W&B uses proprietary formats and APIs. While data can be exported via the Public API, migrating a large experiment history to another platform requires significant effort.

**Weave Maturity**: The Weave product for LLM observability is newer than the core experiment tracking platform. Its ecosystem of integrations and evaluation scorers is still expanding relative to established alternatives like LangSmith or Langfuse.

**Training Product Preview**: W&B Training (serverless RL for LLM post-training) is in public preview and may undergo breaking changes before general availability.

**Offline Mode Limitations**: While offline mode is supported, the experience is degraded. Interactive dashboards, sweeps coordination, team collaboration, and alerts are unavailable until runs are synced.

**Data Volume at Scale**: Organizations logging millions of runs with rich media (images, tables, audio) should plan for storage costs and query performance. Dashboard responsiveness can degrade with very large project histories.

## Changelog Highlights

- **W&B Weave**: LLM observability product providing tracing, evaluation, and cost estimation for language model applications, with automatic instrumentation for OpenAI, Anthropic, and Cohere SDKs.
- **W&B Inference**: OpenAI-compatible API for accessing open-source foundation models with built-in Weave integration for tracing and evaluation.
- **W&B Training (public preview)**: Serverless reinforcement learning for LLM post-training with managed GPU infrastructure and automatic scaling for multi-turn agentic tasks.
- **Model Registry**: Centralized artifact registry with organizational governance, tagging, lineage auditing, and automated CI/CD integration.
- **Weave Evaluations**: Systematic LLM output evaluation with built-in scorers for hallucination detection, summarization quality, toxicity, and embedding similarity.
- **Reports**: Collaborative experiment documentation with embedded live charts, LaTeX/PDF export, and programmatic creation via the Python SDK.

## Citations

- [1] Weights & Biases Documentation - https://docs.wandb.ai/
- [2] W&B Weave Documentation - https://docs.wandb.ai/weave/
- [3] W&B Models Documentation - https://docs.wandb.ai/models/
- [4] W&B Experiment Tracking Guide - https://docs.wandb.ai/guides/track
- [5] W&B Sweeps Guide - https://docs.wandb.ai/models/sweeps/walkthrough
- [6] W&B Artifacts Guide - https://docs.wandb.ai/guides/artifacts
- [7] W&B Registry Guide - https://docs.wandb.ai/models/registry

## Discovery Signals

> Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search.

### Keywords

- Weights & Biases
- W&B
- wandb
- experiment tracking
- hyperparameter sweeps
- Bayesian optimization
- artifact versioning
- model registry
- reports
- W&B Models
- W&B Weave
- W&B Inference
- W&B Training
- @weave.op
- runs
- projects
- config
- metrics
- artifacts
- PyTorch
- TensorFlow
- Keras
- Hugging Face
- PyTorch Lightning
- lineage
- offline mode
- WandbCallback

### Verb-Noun Tasks

- Initialize a run with `wandb.init()`
- Log scalars, images, audio, video, tables, and histograms
- Sweep hyperparameters with Bayesian optimization
- Track datasets and models as versioned artifacts
- Promote models through the registry with aliases and tags
- Create collaborative reports with embedded live charts
- Trace LLM calls automatically with `@weave.op`
- Build evaluation suites with weave Scorers
- Distribute sweep agents across multiple machines
- Run experiments in offline mode and sync later
- Visualize parallel coordinates and parameter importance
- Configure alerts when metrics cross thresholds

### User Intent Phrases

- I need ML experiment tracking with hyperparameter sweeps.
- How do I version datasets and models with lineage?
- I want a model registry for organizational governance.
- How do I add LLM observability alongside my ML training?
- I need to share experiment results as interactive reports.
- How do I run Bayesian hyperparameter optimization?
- I want to compare hundreds of PyTorch runs side-by-side.
- How do I track GPU utilization automatically?
- I want serverless RL post-training for LLMs.
- How do I trace OpenAI/Anthropic/Cohere calls automatically with Weave?

### Problem Statements

- Hyperparameter tuning by hand is unscalable.
- We lose track of which dataset produced which model.
- Reproducibility breaks when configs and runs aren't tied together.
- Model promotion lacks audit trails and governance.
- Separate tools for ML training and LLM tracing fragment our workflow.
- Sharing results across teams requires manual screenshots and notebooks.

### When to Pick This

- Pick this when your team does traditional ML/deep learning training (PyTorch, TF, Keras) alongside LLM work.
- Pick this over LangSmith/Langfuse/Arize Phoenix when ML experiment tracking is the primary need and LLM observability is secondary.
- Pick this over Helicone when you need a hyperparameter sweep controller and artifact registry, not API proxy logging.
- Pick this when you need a unified system of record across training, evaluation, deployment, and LLM tracing.
- Pick this when artifact lineage and model registry governance matter for compliance.
- Pick this when serverless RL post-training (W&B Training preview) fits the roadmap.

### Related Terms and Aliases

- wandb SDK
- W&B Models
- W&B Weave
- W&B Inference
- W&B Training
- WandbCallback
- WandbLogger
- model registry
- artifact lineage
- Weave Ops / Weave Calls
- ML experiment tracker
- hyperparameter sweep controller
- Bayesian search
- Hyperband early termination
- @weave.op decorator

