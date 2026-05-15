[Header 1 ("weights--biases", [], []) [Str "Weights & Biases"], BlockQuote [Para [Str "ML developer platform for experiment tracking and model monitoring"]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, ColWidthDefault), (AlignDefault, ColWidthDefault)] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Observability"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "API/SDK/UI"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "No"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "N/A"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "N/A"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Documentation"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "Official Docs"] ("https://docs.wandb.ai/", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "Weights & Biases (W&B) is a machine learning developer platform for experiment tracking, hyperparameter optimization, model versioning, and Large Language Model (LLM) observability. The platform provides a unified system of record for ML workflows, covering the full lifecycle from training through evaluation and deployment."], Para [Str "W&B is organized into four product areas. W&B Models manages traditional ML and deep learning development with experiment tracking, hyperparameter sweeps, interactive reports, and a model registry. W&B Weave provides LLM-specific observability through tracing, output evaluation, cost estimation, and an inference playground for comparing language models. W&B Inference offers an OpenAI-compatible API for accessing open-source foundation models. W&B Training (public preview) enables post-training of LLMs using serverless Reinforcement Learning (RL) with managed GPU infrastructure."], Para [Str "The platform is commercial and cloud-hosted, with self-hosted deployment options available for enterprise customers. W&B integrates with all major ML frameworks including PyTorch, TensorFlow, Keras, Hugging Face Transformers, PyTorch Lightning, XGBoost, and scikit-learn."], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], Para [Strong [Str "Run"], Str " is the fundamental unit of tracking in W&B. A Run represents a single execution of a training script, evaluation pipeline, or any tracked computation. Each Run captures hyperparameters, metrics, system resource usage, and output artifacts. Runs are created with ", Code ("", [], []) "wandb.init()", Str " and finalized with ", Code ("", [], []) "wandb.finish()", Str "."], Para [Strong [Str "Project"], Str " is a collection of Runs grouped by a shared objective. Projects provide a workspace where teams can compare experiments, visualize metrics, and collaborate on model development."], Para [Strong [Str "Config"], Str " stores hyperparameters and settings for a Run. The ", Code ("", [], []) "wandb.config", Str " object captures the input configuration (learning rate, batch size, architecture choices) that produced specific results, enabling reproducibility and comparison across experiments."], Para [Strong [Str "Metrics"], Str " are time-series values logged during a Run using ", Code ("", [], []) "wandb.log()", Str ". Metrics can be scalars (loss, accuracy), media (images, audio, video), tables, histograms, or custom objects. W&B automatically generates interactive charts from logged metrics."], Para [Strong [Str "Artifacts"], Str " are versioned datasets, models, and intermediate outputs tracked as inputs and outputs of Runs. Artifacts enable lineage tracking, showing which data produced which model, and support deduplication through content-addressable storage."], Para [Strong [Str "Sweeps"], Str " automate hyperparameter search across a defined parameter space using strategies such as grid search, random search, and Bayesian optimization. A sweep controller manages the search process and distributes work to agents running on one or more machines."], Para [Strong [Str "Reports"], Str " are collaborative documents that embed live charts, metrics, and visualizations from experiments. Reports allow teams to annotate results, share findings, and create reproducible documentation of model development."], Para [Strong [Str "Registry"], Str " is a curated central repository of artifact versions within an organization. The registry supports model lifecycle management including promotion, tagging, lineage auditing, and automated downstream processes such as model CI/CD."], Para [Strong [Str "Weave Ops"], Str " are versioned, tracked functions in the Weave product. Decorated with ", Code ("", [], []) "@weave.op", Str ", they automatically capture code, inputs, outputs, timing, and execution metadata for LLM application observability."], Para [Strong [Str "Weave Calls"], Str " represent logged executions of Weave Ops. Each call records input arguments, output values, latency, parent-child relationships for nested calls, and any errors that occurred."], Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Para [Str "W&B follows a client-server architecture with a thin SDK that streams data to a cloud or self-hosted backend:"], CodeBlock ("", [""], []) "Application Code
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
", Para [Strong [Str "SDK Layer"], Str ": The Python SDK (", Code ("", [], []) "wandb", Str " package) provides the ", Code ("", [], []) "init()", Str ", ", Code ("", [], []) "log()", Str ", ", Code ("", [], []) "finish()", Str ", and artifact APIs. On ", Code ("", [], []) "wandb.init()", Str ", a background process starts that handles data serialization, buffering, and asynchronous upload. On exit, the SDK waits for all pending uploads to complete."], Para [Strong [Str "Backend Layer"], Str ": The W&B backend stores experiment metadata, time-series metrics, system metrics, and artifact references. The backend exposes a REST API and a GraphQL API for programmatic access."], Para [Strong [Str "Dashboard Layer"], Str ": The web UI renders interactive charts, comparison tables, parallel coordinate plots, and parameter importance visualizations. Dashboards update in real time as new data arrives from running experiments."], Para [Strong [Str "Weave Layer"], Str ": Weave operates as a separate tracing and evaluation subsystem. It intercepts function calls decorated with ", Code ("", [], []) "@weave.op", Str ", serializes inputs and outputs, and streams trace data to the Weave backend for visualization and analysis."], Para [Strong [Str "Storage Layer"], Str ": Artifacts use content-addressable storage with deduplication. Large files are uploaded to cloud object storage (S3, GCS, or Azure Blob) and referenced by hash. The registry layer maintains pointers to artifact versions without duplicating data."], Header 2 ("key-features-and-functionality", ["unnumbered", "unlisted"], []) [Str "Key Features and Functionality"], Para [Strong [Str "Experiment Tracking"], Str ": Log metrics, hyperparameters, and system resources with minimal code changes. The SDK captures GPU utilization, memory usage, and other system metrics automatically alongside user-defined metrics."], CodeBlock ("", ["python"], []) "import wandb

run = wandb.init(project=\"image-classifier\")
run.config.update({
    \"learning_rate\": 0.001,
    \"epochs\": 10,
    \"batch_size\": 32,
    \"architecture\": \"resnet50\",
})

for epoch in range(10):
    train_loss = train_one_epoch()
    val_loss = evaluate()
    run.log({
        \"train_loss\": train_loss,
        \"val_loss\": val_loss,
        \"epoch\": epoch,
    })

run.finish()
", Para [Strong [Str "Hyperparameter Sweeps"], Str ": Define a search space and optimization strategy, then let W&B explore parameter combinations automatically:"], CodeBlock ("", ["python"], []) "import wandb

sweep_configuration = {
    \"method\": \"bayes\",
    \"metric\": {\"goal\": \"minimize\", \"name\": \"val_loss\"},
    \"parameters\": {
        \"learning_rate\": {\"min\": 0.0001, \"max\": 0.1},
        \"batch_size\": {\"values\": [16, 32, 64, 128]},
        \"optimizer\": {\"values\": [\"adam\", \"sgd\", \"rmsprop\"]},
    },
}

sweep_id = wandb.sweep(sweep=sweep_configuration, project=\"my-sweep\")
wandb.agent(sweep_id, function=train, count=50)
", Para [Strong [Str "Artifact Versioning and Lineage"], Str ": Track datasets and models as versioned artifacts with automatic lineage graphs showing data provenance:"], CodeBlock ("", ["python"], []) "import wandb

with wandb.init(project=\"data-pipeline\") as run:
    artifact = wandb.Artifact(name=\"training-data\", type=\"dataset\")
    artifact.add_file(\"./data/train.csv\")
    run.log_artifact(artifact)
", Para [Strong [Str "Model Registry"], Str ": Promote trained models to a central registry with aliases, tags, and access controls for organizational governance."], Para [Strong [Str "Media Logging"], Str ": Log images, audio, video, 3D objects, HTML, and custom visualizations alongside scalar metrics:"], CodeBlock ("", ["python"], []) "run.log({
    \"examples\": wandb.Table(
        columns=[\"image\", \"prediction\", \"ground_truth\"],
        data=[[wandb.Image(img), pred, label] for img, pred, label in samples]
    )
})
", Para [Strong [Str "Interactive Reports"], Str ": Create collaborative documents embedding live charts, metric comparisons, and narrative descriptions of experiments. Reports can be exported as LaTeX or PDF."], Para [Strong [Str "Weave Tracing"], Str ": Instrument LLM applications with automatic tracing of inputs, outputs, latency, and costs:"], CodeBlock ("", ["python"], []) "import weave
from openai import OpenAI

weave.init(\"my-llm-app\")
client = OpenAI()

@weave.op
def summarize(text: str) -> str:
    response = client.chat.completions.create(
        model=\"gpt-4o\",
        messages=[{\"role\": \"user\", \"content\": f\"Summarize: {text}\"}],
    )
    return response.choices[0].message.content
", Para [Strong [Str "Weave Evaluations"], Str ": Systematically test LLM outputs using built-in and custom scorers for hallucination detection, summarization quality, toxicity, and embedding similarity."], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], Para [Strong [Str "Deep Learning Research"], Str ": Track thousands of training runs across GPU clusters, compare architectures and hyperparameters, and share reproducible results through reports and artifact lineage."], Para [Strong [Str "Hyperparameter Optimization"], Str ": Automate parameter search using Bayesian optimization to find optimal configurations without manual trial-and-error, distributing sweep agents across multiple machines for parallel exploration."], Para [Strong [Str "Model Lifecycle Management"], Str ": Version models from initial training through staging and production using the registry, with automated CI/CD pipelines triggered by artifact state changes."], Para [Strong [Str "LLM Application Observability"], Str ": Trace production LLM applications to monitor latency, token usage, costs, and output quality. Identify regressions and debug failures by inspecting individual traces."], Para [Strong [Str "LLM Evaluation Pipelines"], Str ": Build systematic evaluation suites for LLM applications using Weave scorers to measure response quality, factual accuracy, and safety before deploying updates."], Para [Strong [Str "Dataset Versioning"], Str ": Track training, validation, and test datasets as artifacts with full lineage, ensuring every model can be traced back to the exact data that produced it."], Para [Strong [Str "Team Collaboration"], Str ": Share experiment results through interactive reports, compare team members' runs in shared project dashboards, and establish organizational best practices through the registry."], Header 2 ("api-reference-summary", ["unnumbered", "unlisted"], []) [Str "API Reference Summary"], Para [Strong [Str "Core SDK Functions"], Str ":"], BulletList [[Plain [Code ("", [], []) "wandb.init(project, entity, config, name, tags, group)", Str " -- Initialize a Run; returns a ", Code ("", [], []) "wandb.Run", Str " object. Parameters include project name, team entity, initial config dict, display name, tags for filtering, and group for logical run grouping."]], [Plain [Code ("", [], []) "wandb.log(data, step, commit)", Str " -- Log a dictionary of metrics. Accepts scalars, ", Code ("", [], []) "wandb.Image", Str ", ", Code ("", [], []) "wandb.Table", Str ", ", Code ("", [], []) "wandb.Histogram", Str ", ", Code ("", [], []) "wandb.Audio", Str ", ", Code ("", [], []) "wandb.Video", Str ", and other media types."]], [Plain [Code ("", [], []) "wandb.finish(exit_code, quiet)", Str " -- Finalize the current Run, flush all pending data, and wait for uploads to complete."]], [Plain [Code ("", [], []) "wandb.config", Str " -- Object for storing and retrieving hyperparameters. Supports dictionary-style updates and attribute access."]]], Para [Strong [Str "Sweep Functions"], Str ":"], BulletList [[Plain [Code ("", [], []) "wandb.sweep(sweep, project, entity)", Str " -- Create a sweep from a configuration dictionary; returns a sweep ID."]], [Plain [Code ("", [], []) "wandb.agent(sweep_id, function, count, entity, project)", Str " -- Launch a sweep agent that executes the training function with sampled hyperparameters."]]], Para [Strong [Str "Artifact API"], Str ":"], BulletList [[Plain [Code ("", [], []) "wandb.Artifact(name, type, description, metadata)", Str " -- Create a new artifact with a name and type (dataset, model, etc.)."]], [Plain [Code ("", [], []) "artifact.add_file(local_path, name)", Str " -- Add a local file to the artifact."]], [Plain [Code ("", [], []) "artifact.add_dir(local_path, name)", Str " -- Add a directory to the artifact."]], [Plain [Code ("", [], []) "artifact.add_reference(uri, name)", Str " -- Add a reference to an external file (S3, GCS)."]], [Plain [Code ("", [], []) "run.log_artifact(artifact)", Str " -- Log an artifact as an output of the current Run."]], [Plain [Code ("", [], []) "run.use_artifact(artifact_or_name)", Str " -- Declare an artifact as an input to the current Run; returns the artifact."]], [Plain [Code ("", [], []) "artifact.download(root)", Str " -- Download artifact contents to a local directory."]], [Plain [Code ("", [], []) "run.link_artifact(artifact, target_path)", Str " -- Link an artifact version to a registry collection."]]], Para [Strong [Str "Media Types"], Str ":"], BulletList [[Plain [Code ("", [], []) "wandb.Image(data, caption)", Str " -- Log images from arrays, PIL images, or file paths."]], [Plain [Code ("", [], []) "wandb.Table(columns, data)", Str " -- Log tabular data with typed columns."]], [Plain [Code ("", [], []) "wandb.Audio(data, sample_rate, caption)", Str " -- Log audio clips."]], [Plain [Code ("", [], []) "wandb.Video(data, fps, caption)", Str " -- Log video files."]], [Plain [Code ("", [], []) "wandb.Histogram(sequence)", Str " -- Log distribution data."]], [Plain [Code ("", [], []) "wandb.Html(data)", Str " -- Log arbitrary HTML."]]], Para [Strong [Str "Weave API"], Str ":"], BulletList [[Plain [Code ("", [], []) "weave.init(project_name)", Str " -- Initialize Weave tracing for a project."]], [Plain [Code ("", [], []) "@weave.op", Str " -- Decorator to make a function a tracked Weave Op, capturing inputs, outputs, and execution metadata."]], [Plain [Code ("", [], []) "weave.Evaluation(dataset, scorers)", Str " -- Create an evaluation pipeline with a dataset and scoring functions."]], [Plain [Code ("", [], []) "weave.Scorer", Str " -- Base class for class-based evaluation scorers."]]], Para [Strong [Str "Public API"], Str " (programmatic access):"], BulletList [[Plain [Code ("", [], []) "wandb.Api()", Str " -- Create a client for querying runs, artifacts, and projects."]], [Plain [Code ("", [], []) "api.runs(path, filters, order)", Str " -- Query runs with filtering and sorting."]], [Plain [Code ("", [], []) "api.artifact(name)", Str " -- Retrieve a specific artifact version."]]], Header 2 ("configuration-and-customization", ["unnumbered", "unlisted"], []) [Str "Configuration and Customization"], Para [Strong [Str "Run Configuration"], Str ": Pass hyperparameters at initialization or update incrementally:"], CodeBlock ("", ["python"], []) "run = wandb.init(
    project=\"my-project\",
    config={
        \"learning_rate\": 0.001,
        \"architecture\": \"transformer\",
        \"dataset\": \"wikitext-103\",
    },
)

# Update config after initialization
run.config.update({\"dropout\": 0.1})
", Para [Strong [Str "Environment Variables"], Str ": Control SDK behavior without code changes:"], CodeBlock ("", ["bash"], []) "export WANDB_PROJECT=\"my-project\"       # Default project name
export WANDB_ENTITY=\"my-team\"           # Default team entity
export WANDB_MODE=\"offline\"             # Run without cloud sync
export WANDB_DIR=\"/tmp/wandb\"           # Local storage directory
export WANDB_SILENT=\"true\"              # Suppress SDK output
export WANDB_DISABLE_CODE=\"true\"        # Disable code saving
", Para [Strong [Str "Offline Mode"], Str ": Run experiments without network connectivity, then sync later:"], CodeBlock ("", ["python"], []) "import os
os.environ[\"WANDB_MODE\"] = \"offline\"

run = wandb.init(project=\"offline-experiment\")
# ... log metrics as usual ...
run.finish()
", Para [Str "Sync offline runs when connectivity is restored:"], CodeBlock ("", ["bash"], []) "wandb sync ./wandb/offline-run-*
", Para [Strong [Str "Sweep Configuration Options"], Str ": Customize search strategies and early termination:"], CodeBlock ("", ["python"], []) "sweep_config = {
    \"method\": \"bayes\",           # bayes, grid, random
    \"metric\": {
        \"goal\": \"minimize\",
        \"name\": \"val_loss\",
    },
    \"parameters\": {
        \"learning_rate\": {
            \"distribution\": \"log_uniform_values\",
            \"min\": 1e-5,
            \"max\": 1e-1,
        },
        \"epochs\": {\"value\": 20},  # Fixed parameter
    },
    \"early_terminate\": {
        \"type\": \"hyperband\",
        \"min_iter\": 3,
        \"eta\": 3,
    },
}
", Para [Strong [Str "Alerts"], Str ": Configure programmatic alerts when metrics cross thresholds:"], CodeBlock ("", ["python"], []) "run.alert(
    title=\"Training diverging\",
    text=f\"Loss exceeded threshold: {loss}\",
    level=wandb.AlertLevel.WARN,
)
", Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Para [Strong [Str "PyTorch Training Loop"], Str ": Add W&B tracking to a standard PyTorch training loop with minimal changes:"], CodeBlock ("", ["python"], []) "import wandb
import torch

run = wandb.init(project=\"pytorch-example\")

model = MyModel()
optimizer = torch.optim.Adam(model.parameters(), lr=run.config.learning_rate)

for epoch in range(run.config.epochs):
    for batch in dataloader:
        loss = train_step(model, batch, optimizer)
        run.log({\"batch_loss\": loss.item()})

    val_metrics = evaluate(model, val_loader)
    run.log({\"val_loss\": val_metrics[\"loss\"], \"val_acc\": val_metrics[\"accuracy\"]})

run.finish()
", Para [Strong [Str "Hugging Face Transformers"], Str ": Use the built-in W&B callback:"], CodeBlock ("", ["python"], []) "from transformers import TrainingArguments

training_args = TrainingArguments(
    output_dir=\"./results\",
    report_to=\"wandb\",
    run_name=\"bert-fine-tune\",
    num_train_epochs=3,
)
", Para [Strong [Str "Keras Callback"], Str ": Integrate with Keras via the WandbCallback:"], CodeBlock ("", ["python"], []) "from wandb.integration.keras import WandbCallback

model.fit(
    x_train, y_train,
    validation_data=(x_val, y_val),
    callbacks=[WandbCallback()],
)
", Para [Strong [Str "PyTorch Lightning Logger"], Str ": Attach W&B as a logger in Lightning:"], CodeBlock ("", ["python"], []) "from lightning.pytorch.loggers import WandbLogger

wandb_logger = WandbLogger(project=\"lightning-example\")
trainer = Trainer(logger=wandb_logger)
trainer.fit(model, datamodule)
", Para [Strong [Str "Weave with OpenAI"], Str ": Weave automatically traces OpenAI API calls when initialized:"], CodeBlock ("", ["python"], []) "import weave
from openai import OpenAI

weave.init(\"openai-tracing\")
client = OpenAI()

response = client.chat.completions.create(
    model=\"gpt-4o\",
    messages=[{\"role\": \"user\", \"content\": \"Explain transformers.\"}],
)
", Para [Strong [Str "Weave with LangChain"], Str ": Trace LangChain chains and agents through Weave instrumentation:"], CodeBlock ("", ["python"], []) "import weave

weave.init(\"langchain-tracing\")

# LangChain operations are automatically traced
", Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Para [Strong [Str "Complete Training Run with Artifacts"], Str ":"], CodeBlock ("", ["python"], []) "import wandb

with wandb.init(project=\"artifact-demo\", config={\"lr\": 0.01, \"epochs\": 5}) as run:
    # Log training data as input artifact
    dataset = run.use_artifact(\"training-data:latest\")
    data_dir = dataset.download()

    # Train model
    model = train_model(data_dir, run.config)

    for epoch in range(run.config.epochs):
        metrics = evaluate(model)
        run.log({\"epoch\": epoch, **metrics})

    # Log trained model as output artifact
    model_artifact = wandb.Artifact(\"trained-model\", type=\"model\")
    model_artifact.add_file(\"model.pt\")
    run.log_artifact(model_artifact)
", Para [Strong [Str "Hyperparameter Sweep with Bayesian Optimization"], Str ":"], CodeBlock ("", ["python"], []) "import wandb

def train():
    with wandb.init() as run:
        lr = run.config.learning_rate
        batch_size = run.config.batch_size

        model = build_model(lr=lr)
        for epoch in range(10):
            loss = train_epoch(model, batch_size=batch_size)
            run.log({\"loss\": loss, \"epoch\": epoch})

sweep_config = {
    \"method\": \"bayes\",
    \"metric\": {\"goal\": \"minimize\", \"name\": \"loss\"},
    \"parameters\": {
        \"learning_rate\": {\"min\": 0.0001, \"max\": 0.01},
        \"batch_size\": {\"values\": [16, 32, 64]},
    },
}

if __name__ == \"__main__\":
    sweep_id = wandb.sweep(sweep_config, project=\"sweep-demo\")
    wandb.agent(sweep_id, function=train, count=25)
", Para [Strong [Str "Weave Evaluation Pipeline"], Str ":"], CodeBlock ("", ["python"], []) "import weave
from weave import Evaluation

weave.init(\"eval-demo\")

@weave.op
def my_model(question: str) -> str:
    # LLM call here
    return answer

@weave.op
def correctness_scorer(output: str, expected: str) -> dict:
    return {\"correct\": output.strip().lower() == expected.strip().lower()}

dataset = [
    {\"question\": \"What is 2+2?\", \"expected\": \"4\"},
    {\"question\": \"Capital of France?\", \"expected\": \"Paris\"},
]

evaluation = Evaluation(dataset=dataset, scorers=[correctness_scorer])
results = evaluation.evaluate(my_model)
", Para [Strong [Str "Model Registry Workflow"], Str ":"], CodeBlock ("", ["python"], []) "import wandb

with wandb.init(project=\"registry-demo\") as run:
    # Train and log model
    model_artifact = wandb.Artifact(\"my-model\", type=\"model\")
    model_artifact.add_file(\"model.pt\")
    run.log_artifact(model_artifact)

    # Link to registry
    run.link_artifact(
        artifact=model_artifact,
        target_path=\"my-org/wandb-registry-model/production-models\",
    )
", Header 2 ("limitations-and-considerations", ["unnumbered", "unlisted"], []) [Str "Limitations and Considerations"], Para [Strong [Str "Commercial Platform"], Str ": W&B is a proprietary, closed-source platform. While a free tier is available for individual researchers, teams and organizations require paid subscriptions. Self-hosted deployments are available but require enterprise licensing."], Para [Strong [Str "Cloud Dependency"], Str ": By default, all experiment data is transmitted to W&B cloud servers. Organizations with strict data residency or air-gapped requirements must deploy the self-hosted version, which involves additional infrastructure management."], Para [Strong [Str "SDK Overhead"], Str ": The background upload process consumes CPU, memory, and network bandwidth. For very high-frequency logging (thousands of log calls per second) or resource-constrained environments, the overhead may be noticeable and require tuning of logging frequency."], Para [Strong [Str "Vendor Lock-In"], Str ": Experiment data stored in W&B uses proprietary formats and APIs. While data can be exported via the Public API, migrating a large experiment history to another platform requires significant effort."], Para [Strong [Str "Weave Maturity"], Str ": The Weave product for LLM observability is newer than the core experiment tracking platform. Its ecosystem of integrations and evaluation scorers is still expanding relative to established alternatives like LangSmith or Langfuse."], Para [Strong [Str "Training Product Preview"], Str ": W&B Training (serverless RL for LLM post-training) is in public preview and may undergo breaking changes before general availability."], Para [Strong [Str "Offline Mode Limitations"], Str ": While offline mode is supported, the experience is degraded. Interactive dashboards, sweeps coordination, team collaboration, and alerts are unavailable until runs are synced."], Para [Strong [Str "Data Volume at Scale"], Str ": Organizations logging millions of runs with rich media (images, tables, audio) should plan for storage costs and query performance. Dashboard responsiveness can degrade with very large project histories."], Header 2 ("changelog-highlights", ["unnumbered", "unlisted"], []) [Str "Changelog Highlights"], BulletList [[Plain [Strong [Str "W&B Weave"], Str ": LLM observability product providing tracing, evaluation, and cost estimation for language model applications, with automatic instrumentation for OpenAI, Anthropic, and Cohere SDKs."]], [Plain [Strong [Str "W&B Inference"], Str ": OpenAI-compatible API for accessing open-source foundation models with built-in Weave integration for tracing and evaluation."]], [Plain [Strong [Str "W&B Training (public preview)"], Str ": Serverless reinforcement learning for LLM post-training with managed GPU infrastructure and automatic scaling for multi-turn agentic tasks."]], [Plain [Strong [Str "Model Registry"], Str ": Centralized artifact registry with organizational governance, tagging, lineage auditing, and automated CI/CD integration."]], [Plain [Strong [Str "Weave Evaluations"], Str ": Systematic LLM output evaluation with built-in scorers for hallucination detection, summarization quality, toxicity, and embedding similarity."]], [Plain [Strong [Str "Reports"], Str ": Collaborative experiment documentation with embedded live charts, LaTeX/PDF export, and programmatic creation via the Python SDK."]]], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " Weights & Biases Documentation - https://docs.wandb.ai/"]], [Plain [Str "[", Str "2", Str "]", Str " W&B Weave Documentation - https://docs.wandb.ai/weave/"]], [Plain [Str "[", Str "3", Str "]", Str " W&B Models Documentation - https://docs.wandb.ai/models/"]], [Plain [Str "[", Str "4", Str "]", Str " W&B Experiment Tracking Guide - https://docs.wandb.ai/guides/track"]], [Plain [Str "[", Str "5", Str "]", Str " W&B Sweeps Guide - https://docs.wandb.ai/models/sweeps/walkthrough"]], [Plain [Str "[", Str "6", Str "]", Str " W&B Artifacts Guide - https://docs.wandb.ai/guides/artifacts"]], [Plain [Str "[", Str "7", Str "]", Str " W&B Registry Guide - https://docs.wandb.ai/models/registry"]]], Header 2 ("discovery-signals", ["unnumbered", "unlisted"], []) [Str "Discovery Signals"], BlockQuote [Para [Str "Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search."]], Header 3 ("keywords", ["unnumbered", "unlisted"], []) [Str "Keywords"], BulletList [[Plain [Str "Weights & Biases"]], [Plain [Str "W&B"]], [Plain [Str "wandb"]], [Plain [Str "experiment tracking"]], [Plain [Str "hyperparameter sweeps"]], [Plain [Str "Bayesian optimization"]], [Plain [Str "artifact versioning"]], [Plain [Str "model registry"]], [Plain [Str "reports"]], [Plain [Str "W&B Models"]], [Plain [Str "W&B Weave"]], [Plain [Str "W&B Inference"]], [Plain [Str "W&B Training"]], [Plain [Str "@weave.op"]], [Plain [Str "runs"]], [Plain [Str "projects"]], [Plain [Str "config"]], [Plain [Str "metrics"]], [Plain [Str "artifacts"]], [Plain [Str "PyTorch"]], [Plain [Str "TensorFlow"]], [Plain [Str "Keras"]], [Plain [Str "Hugging Face"]], [Plain [Str "PyTorch Lightning"]], [Plain [Str "lineage"]], [Plain [Str "offline mode"]], [Plain [Str "WandbCallback"]]], Header 3 ("verb-noun-tasks", ["unnumbered", "unlisted"], []) [Str "Verb-Noun Tasks"], BulletList [[Plain [Str "Initialize a run with ", Code ("", [], []) "wandb.init()"]], [Plain [Str "Log scalars, images, audio, video, tables, and histograms"]], [Plain [Str "Sweep hyperparameters with Bayesian optimization"]], [Plain [Str "Track datasets and models as versioned artifacts"]], [Plain [Str "Promote models through the registry with aliases and tags"]], [Plain [Str "Create collaborative reports with embedded live charts"]], [Plain [Str "Trace LLM calls automatically with ", Code ("", [], []) "@weave.op"]], [Plain [Str "Build evaluation suites with weave Scorers"]], [Plain [Str "Distribute sweep agents across multiple machines"]], [Plain [Str "Run experiments in offline mode and sync later"]], [Plain [Str "Visualize parallel coordinates and parameter importance"]], [Plain [Str "Configure alerts when metrics cross thresholds"]]], Header 3 ("user-intent-phrases", ["unnumbered", "unlisted"], []) [Str "User Intent Phrases"], BulletList [[Plain [Str "I need ML experiment tracking with hyperparameter sweeps."]], [Plain [Str "How do I version datasets and models with lineage?"]], [Plain [Str "I want a model registry for organizational governance."]], [Plain [Str "How do I add LLM observability alongside my ML training?"]], [Plain [Str "I need to share experiment results as interactive reports."]], [Plain [Str "How do I run Bayesian hyperparameter optimization?"]], [Plain [Str "I want to compare hundreds of PyTorch runs side-by-side."]], [Plain [Str "How do I track GPU utilization automatically?"]], [Plain [Str "I want serverless RL post-training for LLMs."]], [Plain [Str "How do I trace OpenAI/Anthropic/Cohere calls automatically with Weave?"]]], Header 3 ("problem-statements", ["unnumbered", "unlisted"], []) [Str "Problem Statements"], BulletList [[Plain [Str "Hyperparameter tuning by hand is unscalable."]], [Plain [Str "We lose track of which dataset produced which model."]], [Plain [Str "Reproducibility breaks when configs and runs aren't tied together."]], [Plain [Str "Model promotion lacks audit trails and governance."]], [Plain [Str "Separate tools for ML training and LLM tracing fragment our workflow."]], [Plain [Str "Sharing results across teams requires manual screenshots and notebooks."]]], Header 3 ("when-to-pick-this", ["unnumbered", "unlisted"], []) [Str "When to Pick This"], BulletList [[Plain [Str "Pick this when your team does traditional ML/deep learning training (PyTorch, TF, Keras) alongside LLM work."]], [Plain [Str "Pick this over LangSmith/Langfuse/Arize Phoenix when ML experiment tracking is the primary need and LLM observability is secondary."]], [Plain [Str "Pick this over Helicone when you need a hyperparameter sweep controller and artifact registry, not API proxy logging."]], [Plain [Str "Pick this when you need a unified system of record across training, evaluation, deployment, and LLM tracing."]], [Plain [Str "Pick this when artifact lineage and model registry governance matter for compliance."]], [Plain [Str "Pick this when serverless RL post-training (W&B Training preview) fits the roadmap."]]], Header 3 ("related-terms-and-aliases", ["unnumbered", "unlisted"], []) [Str "Related Terms and Aliases"], BulletList [[Plain [Str "wandb SDK"]], [Plain [Str "W&B Models"]], [Plain [Str "W&B Weave"]], [Plain [Str "W&B Inference"]], [Plain [Str "W&B Training"]], [Plain [Str "WandbCallback"]], [Plain [Str "WandbLogger"]], [Plain [Str "model registry"]], [Plain [Str "artifact lineage"]], [Plain [Str "Weave Ops / Weave Calls"]], [Plain [Str "ML experiment tracker"]], [Plain [Str "hyperparameter sweep controller"]], [Plain [Str "Bayesian search"]], [Plain [Str "Hyperband early termination"]], [Plain [Str "@weave.op decorator"]]]]