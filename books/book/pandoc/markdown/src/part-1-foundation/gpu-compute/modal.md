[Header 1 ("modal", [], []) [Str "Modal"], Table ("", [], []) (Caption Nothing []) [(AlignDefault, ColWidthDefault), (AlignDefault, ColWidthDefault)] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Strong [Str "Group"]]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GPU Compute & Cloud Platforms"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Strong [Str "Type"]]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "API/SDK/Infra"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Strong [Str "Open Source"]]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "No"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Strong [Str "GitHub"]]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "N/A"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Strong [Str "Stars"]]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "N/A"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Strong [Str "Docs"]]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "Official Docs"] ("https://modal.com/docs", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "Modal is a serverless GPU compute platform purpose-built for AI and machine learning workloads. It takes Python code, packages it into a container, and executes it in the cloud with automatic horizontal scaling. The platform follows a code-first approach that eliminates YAML configuration files entirely, offering sub-second cold starts, per-second billing, and multi-cloud infrastructure. Modal targets teams that need on-demand GPU access without managing infrastructure, containers, or orchestration layers."], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], BulletList [[Plain [Strong [Str "App"], Str ": Top-level container that groups related functions, images, volumes, and other resources into a single deployable unit."]], [Plain [Strong [Str "Function"], Str ": A Python function decorated with ", Code ("", [], []) "@app.function()", Str " that runs remotely in the cloud. Functions are the primary unit of execution and can be invoked synchronously, asynchronously, or mapped over inputs in parallel."]], [Plain [Strong [Str "Image"], Str ": A container image definition that specifies the runtime environment for functions. Images are built incrementally using a builder pattern (e.g., ", Code ("", [], []) "modal.Image.debian_slim().pip_install(\"torch\")", Str "), and layers are cached for fast rebuilds."]], [Plain [Strong [Str "Volume"], Str ": Persistent storage that can be mounted into function containers. Volumes survive across function invocations and deployments, making them suitable for storing model weights, datasets, and checkpoints."]], [Plain [Strong [Str "Secret"], Str ": Environment variable management for sensitive data such as API keys, database credentials, and tokens. Secrets are injected into function containers at runtime without being embedded in code or images."]], [Plain [Strong [Str "Sandbox"], Str ": Isolated execution environments for running untrusted or experimental code with resource limits and timeouts."]]], Header 2 ("installation-and-setup", ["unnumbered", "unlisted"], []) [Str "Installation and Setup"], Para [Str "Modal requires Python 3.9 or later. Installation and authentication are handled through the CLI:"], CodeBlock ("", ["bash"], []) "pip install modal
modal setup  # opens browser for authentication
", Para [Str "The ", Code ("", [], []) "modal setup", Str " command creates a local token that authenticates all subsequent CLI and SDK operations. No additional configuration files are required."], Para [Str "Running a Modal app locally for testing:"], CodeBlock ("", ["bash"], []) "modal run my_app.py
", Para [Str "Deploying a Modal app as a persistent service:"], CodeBlock ("", ["bash"], []) "modal deploy my_app.py
", Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Para [Str "Modal operates on a serverless execution model. When a function is invoked, Modal performs the following sequence:"], OrderedList (1, DefaultStyle, DefaultDelim) [[Plain [Strong [Str "Image resolution"], Str ": The platform checks whether the specified container image exists in its cache. If not, it builds the image from the declarative definition."]], [Plain [Strong [Str "Container scheduling"], Str ": A container is scheduled on available infrastructure matching the requested resources (CPU, memory, GPU type)."]], [Plain [Strong [Str "Code injection"], Str ": The decorated function code is serialized and injected into the container at runtime."]], [Plain [Strong [Str "Execution"], Str ": The function runs inside the container with access to mounted volumes, secrets, and network resources."]], [Plain [Strong [Str "Scaling"], Str ": Additional containers are spawned automatically based on incoming request volume, scaling from zero to thousands of concurrent instances."]], [Plain [Strong [Str "Teardown"], Str ": Idle containers are terminated after a configurable timeout, and billing stops immediately."]]], Para [Str "The platform abstracts away container registries, orchestration systems, load balancers, and GPU drivers. Users interact exclusively through Python decorators and the Modal SDK."], Header 2 ("key-features", ["unnumbered", "unlisted"], []) [Str "Key Features"], BulletList [[Plain [Strong [Str "Sub-second cold starts"], Str ": Containers launch in under one second through aggressive image caching and snapshot-based initialization."]], [Plain [Strong [Str "Per-second billing"], Str ": Compute charges are measured per second of actual usage, with no minimum billing increments for idle time."]], [Plain [Strong [Str "Web endpoints"], Str ": Functions can be exposed as HTTP endpoints using ", Code ("", [], []) "@app.function()", Str " combined with ", Code ("", [], []) "@modal.web_endpoint()", Str ", supporting REST APIs and webhook receivers."]], [Plain [Strong [Str "Streaming responses"], Str ": Server-sent events and streaming HTTP responses are supported natively for real-time inference applications."]], [Plain [Strong [Str "Volume mounts"], Str ": Persistent volumes can be attached to functions for reading and writing data that persists across invocations."]], [Plain [Strong [Str "Cloud bucket integrations"], Str ": Direct mounting of S3 and GCS buckets into function containers without manual credential wiring."]], [Plain [Strong [Str "Scheduled jobs"], Str ": Functions can be triggered on cron schedules using ", Code ("", [], []) "@modal.Cron(\"0 * * * *\")", Str " or periodic intervals."]], [Plain [Strong [Str "Secret management"], Str ": Secrets are defined once in the Modal dashboard and referenced by name in code, with automatic injection at runtime."]], [Plain [Strong [Str "GPU health monitoring"], Str ": The platform monitors GPU health and automatically migrates workloads away from degraded hardware."]], [Plain [Strong [Str "Preemption handling"], Str ": Functions can register callbacks to handle preemption events gracefully, saving state before container termination."]], [Plain [Strong [Str "Multi-node training"], Str ": Distributed training across multiple GPU nodes is available in closed beta."]]], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], BulletList [[Plain [Strong [Str "Model training"], Str ": GPU-accelerated training jobs that scale from a single GPU to multi-GPU configurations without infrastructure changes."]], [Plain [Strong [Str "Batch inference"], Str ": Processing large datasets through ML models by mapping a function over thousands of inputs in parallel."]], [Plain [Strong [Str "Real-time inference APIs"], Str ": Deploying model serving endpoints with automatic scaling based on request volume."]], [Plain [Strong [Str "Data preprocessing"], Str ": Running CPU or GPU-intensive data pipelines on demand without maintaining persistent compute clusters."]], [Plain [Strong [Str "Fine-tuning"], Str ": Running fine-tuning jobs on large language models with configurable GPU types and memory."]], [Plain [Strong [Str "Scheduled ETL"], Str ": Periodic data extraction, transformation, and loading jobs triggered by cron schedules."]]], Header 2 ("api-reference-summary", ["unnumbered", "unlisted"], []) [Str "API Reference Summary"], Para [Strong [Str "App definition"], Str ":"], CodeBlock ("", ["python"], []) "import modal

app = modal.App(\"my-app\")
", Para [Strong [Str "Function decorator"], Str ":"], CodeBlock ("", ["python"], []) "@app.function(gpu=\"A100\", timeout=3600, memory=32768)
def my_function(input_data):
    return process(input_data)
", Para [Strong [Str "Image builder"], Str ":"], CodeBlock ("", ["python"], []) "image = (
    modal.Image.debian_slim(python_version=\"3.11\")
    .pip_install(\"torch\", \"transformers\")
    .apt_install(\"ffmpeg\")
)

@app.function(image=image, gpu=\"H100\")
def inference(prompt):
    pass
", Para [Strong [Str "Volume"], Str ":"], CodeBlock ("", ["python"], []) "volume = modal.Volume.from_name(\"my-volume\", create_if_missing=True)

@app.function(volumes={\"/data\": volume})
def write_data():
    with open(\"/data/output.txt\", \"w\") as f:
        f.write(\"result\")
    volume.commit()
", Para [Strong [Str "Web endpoint"], Str ":"], CodeBlock ("", ["python"], []) "@app.function()
@modal.web_endpoint(method=\"POST\")
def predict(request: dict):
    return {\"result\": run_model(request[\"input\"])}
", Para [Strong [Str "Parallel map"], Str ":"], CodeBlock ("", ["python"], []) "@app.function(gpu=\"T4\")
def process_item(item):
    return transform(item)

@app.local_entrypoint()
def main():
    items = list(range(1000))
    results = list(process_item.map(items))
", Para [Strong [Str "Secrets"], Str ":"], CodeBlock ("", ["python"], []) "@app.function(secrets=[modal.Secret.from_name(\"my-api-key\")])
def call_api():
    import os
    key = os.environ[\"API_KEY\"]
", Para [Strong [Str "Scheduled function"], Str ":"], CodeBlock ("", ["python"], []) "@app.function(schedule=modal.Cron(\"0 */6 * * *\"))
def periodic_job():
    pass
", Header 2 ("configuration", ["unnumbered", "unlisted"], []) [Str "Configuration"], Para [Strong [Str "GPU selection"], Str ": GPUs are requested through the ", Code ("", [], []) "gpu", Str " parameter on the function decorator. Available GPU types and their memory:"], Table ("", [], []) (Caption Nothing []) [(AlignDefault, ColWidthDefault), (AlignDefault, ColWidthDefault), (AlignDefault, ColWidthDefault)] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GPU"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Memory"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Notes"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "T4"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "16 GB"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Budget inference"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "L4"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "24 GB"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "General purpose inference"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "A10"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "24 GB"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Balanced training/inference"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "A100"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "40/80 GB"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "May auto-upgrade to 80 GB"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "L40S"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "48 GB"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Recommended for inference (cost/perf)"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "H100"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "80 GB"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "High-end training, may upgrade to H200"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "H100!"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "80 GB"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Reserved H100 (no upgrade)"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "H200"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "141 GB"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Large model training"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "B200"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "192 GB"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Next-gen training"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "B200+"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "192 GB"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Opt-in for B300 access"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "B300"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "288 GB"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Latest generation"]]]])] (TableFoot ("", [], []) []), Para [Strong [Str "Multi-GPU"], Str ": Request multiple GPUs by appending a count to the GPU type string:"], CodeBlock ("", ["python"], []) "@app.function(gpu=\"H100:8\")  # 8x H100, up to 1,536 GB total
def distributed_training():
    pass
", Para [Strong [Str "GPU fallbacks"], Str ": Specify multiple GPU types as a prioritized list for availability:"], CodeBlock ("", ["python"], []) "@app.function(gpu=modal.gpu.Any([\"H100\", \"A100-80GB\"]))
def flexible_training():
    pass
", Para [Strong [Str "Resource limits"], Str ": CPU, memory, and timeout are configured per function:"], CodeBlock ("", ["python"], []) "@app.function(cpu=4, memory=65536, timeout=7200, gpu=\"A100\")
def heavy_job():
    pass
", Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Para [Strong [Str "SDKs"], Str ": Modal provides client SDKs in three languages for invoking deployed functions:"], BulletList [[Plain [Strong [Str "Python"], Str " (primary): Full SDK for defining and invoking functions, building images, and managing resources."]], [Plain [Strong [Str "JavaScript/TypeScript"], Str ": Client SDK for invoking Modal functions from Node.js applications."]], [Plain [Strong [Str "Go"], Str ": Client SDK for invoking Modal functions from Go services."]]], Para [Strong [Str "Webhook integration"], Str ": Web endpoints can serve as webhook receivers for external services, processing incoming HTTP requests with GPU-backed functions."], Para [Strong [Str "Pipeline chaining"], Str ": Functions can call other Modal functions directly, enabling multi-step pipelines where each stage runs on different hardware:"], CodeBlock ("", ["python"], []) "@app.function(gpu=\"A100\")
def generate_embeddings(text):
    return model.encode(text)

@app.function(cpu=2)
def store_results(embeddings):
    database.insert(embeddings)

@app.local_entrypoint()
def pipeline(text):
    embeddings = generate_embeddings.remote(text)
    store_results.remote(embeddings)
", Para [Strong [Str "Cloud storage"], Str ": Volumes and cloud bucket mounts provide persistent storage across function invocations, enabling workflows that accumulate state over time."], Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Para [Strong [Str "Basic GPU function"], Str ":"], CodeBlock ("", ["python"], []) "import modal

app = modal.App(\"gpu-example\")

@app.function(gpu=\"A100\")
def train_model():
    import torch
    device = torch.device(\"cuda\")
    tensor = torch.randn(1000, 1000, device=device)
    result = torch.matmul(tensor, tensor.T)
    return result.shape
", Para [Strong [Str "Model serving endpoint"], Str ":"], CodeBlock ("", ["python"], []) "import modal

app = modal.App(\"inference-api\")

image = modal.Image.debian_slim().pip_install(\"transformers\", \"torch\")

@app.cls(image=image, gpu=\"L40S\")
class ModelServer:
    @modal.enter()
    def load_model(self):
        from transformers import pipeline
        self.pipe = pipeline(\"text-generation\", model=\"meta-llama/Llama-2-7b-hf\", device=\"cuda\")

    @modal.web_endpoint(method=\"POST\")
    def generate(self, request: dict):
        result = self.pipe(request[\"prompt\"], max_new_tokens=256)
        return {\"output\": result[0][\"generated_text\"]}
", Para [Strong [Str "Batch processing with parallel map"], Str ":"], CodeBlock ("", ["python"], []) "import modal

app = modal.App(\"batch-processing\")
volume = modal.Volume.from_name(\"results\", create_if_missing=True)

@app.function(gpu=\"T4\", volumes={\"/output\": volume})
def process_image(image_path: str):
    result = run_inference(image_path)
    with open(f\"/output/{image_path}.json\", \"w\") as f:
        json.dump(result, f)
    volume.commit()

@app.local_entrypoint()
def main():
    image_paths = get_all_image_paths()
    list(process_image.map(image_paths))
", Header 2 ("limitations", ["unnumbered", "unlisted"], []) [Str "Limitations"], BulletList [[Plain [Strong [Str "Closed source"], Str ": The platform is proprietary with no self-hosted deployment option. All workloads run on Modal-managed infrastructure."]], [Plain [Strong [Str "Vendor lock-in"], Str ": The decorator-based API is Modal-specific. Migrating to another platform requires rewriting the infrastructure layer."]], [Plain [Strong [Str "Multi-node training"], Str ": Distributed training across multiple nodes is in closed beta and not generally available."]], [Plain [Strong [Str "Execution time limits"], Str ": Functions have maximum timeout constraints that may not suit extremely long-running workloads."]], [Plain [Strong [Str "Cold start variability"], Str ": While sub-second cold starts are typical, complex images with large dependencies may take longer on first invocation."]], [Plain [Strong [Str "Regional availability"], Str ": Infrastructure availability varies by region, which can affect GPU type availability and latency."]], [Plain [Strong [Str "No raw VM access"], Str ": Users cannot SSH into containers or access the underlying virtual machines directly."]]], Header 2 ("changelog-highlights", ["unnumbered", "unlisted"], []) [Str "Changelog Highlights"], Para [Str "Modal is a continuously deployed platform without traditional versioned releases. The Python SDK receives frequent updates through PyPI. Notable platform capabilities include the addition of B200 and B300 GPU support, the introduction of cloud bucket mounts, and the ongoing closed beta for multi-node training."], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " ", Link ("", [], []) [Str "Modal Documentation"] ("https://modal.com/docs/guide", "")]], [Plain [Str "[", Str "2", Str "]", Str " ", Link ("", [], []) [Str "Modal GPU Guide"] ("https://modal.com/docs/guide/gpu", "")]]]]