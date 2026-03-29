[Header 1 ("inferless", [], []) [Str "Inferless"], BlockQuote [Para [Str "Serverless GPU inference platform for deploying ML models"]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, ColWidthDefault), (AlignDefault, ColWidthDefault)] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Name"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Inferless"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GPU Infrastructure"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "API/Infra"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "No"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "N/A"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "N/A"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Docs"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "docs.inferless.com"] ("https://docs.inferless.com/", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "Inferless is a serverless GPU inference platform that deploys custom machine learning models with minimal cold start latency. The platform abstracts away infrastructure management, GPU provisioning, and container orchestration so teams can focus on model development. Inferless supports importing models from HuggingFace, GitHub, GitLab, AWS S3, Google Cloud Storage (GCS), DockerHub, Dockerfiles, and direct file uploads. The billing model charges per second of actual compute usage rather than reserved capacity, with fractional GPU support so multiple models and workloads can share GPUs with automatic rebalancing and node draining ", Str "[", Str "1", Str "]", Str "[", Str "2", Str "]", Str "."], Para [Str "The platform supports PyTorch, TensorFlow, ONNX, and custom Python functions without framework restrictions. It provides a web dashboard, a Command-Line Interface (CLI), a Python client library, and REST API endpoints for managing deployments. Built-in Prometheus metrics and Grafana dashboards track GPU utilization and system performance. Auto-scaling handles traffic from zero to thousands of GPUs based on requests per second ", Str "[", Str "1", Str "]", Str "[", Str "3", Str "]", Str "."], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], BulletList [[Plain [Strong [Str "Serverless GPU Inference"], Str ": Models run on GPU-backed infrastructure without users managing servers, containers, or orchestration layers. Resources are provisioned on demand and released when idle, following a pay-per-second billing model ", Str "[", Str "2", Str "]", Str "."]], [Plain [Strong [Str "Model Import"], Str ": Inferless supports seven integration methods for importing models: HuggingFace, GitHub/GitLab custom code, file upload, AWS S3, Google Cloud Buckets, DockerHub, and Dockerfile-based deployments ", Str "[", Str "4", Str "]", Str "."]], [Plain [Strong [Str "Autoscaling"], Str ": The platform automatically scales endpoints from zero instances up to a configured maximum based on requests per second. Min scale defines workers kept continuously running; max scale sets the maximum concurrent workers allowed ", Str "[", Str "5", Str "]", Str "."]], [Plain [Strong [Str "Shared and Dedicated GPU Instances"], Str ": Shared instances allow fractional GPU access (partial GPU memory and vCPUs) at lower cost, while dedicated instances provide full GPU resources with higher memory and compute allocations ", Str "[", Str "6", Str "]", Str "."]], [Plain [Strong [Str "NFS Volumes"], Str ": NFS-like writable volumes enable simultaneous connections across multiple replicas for storing model parameters, archiving datasets centrally, and establishing shared caches ", Str "[", Str "7", Str "]", Str "."]], [Plain [Strong [Str "Custom Runtimes"], Str ": YAML-based configuration specifies CUDA version, Python packages, system packages, and shell commands for building custom container environments without complex application server code ", Str "[", Str "8", Str "]", Str "."]], [Plain [Strong [Str "Dynamic Batching"], Str ": Inference requests are combined server-side into batches to improve throughput. Configurable via ", Code ("", [], []) "BATCH_SIZE", Str " and ", Code ("", [], []) "BATCH_WINDOW", Str " parameters in the input schema ", Str "[", Str "9", Str "]", Str "."]], [Plain [Strong [Str "Streaming Output"], Str ": Server-Sent Events (SSE) enable one-way server-to-client communication for real-time data streaming during inference, with automatic reconnection after connection loss ", Str "[", Str "10", Str "]", Str "."]], [Plain [Strong [Str "Secrets Manager"], Str ": Centralized storage for passwords, API keys, and tokens with encryption at rest and in transit, access control, and automatic rotation support ", Str "[", Str "11", Str "]", Str "."]], [Plain [Strong [Str "Remote Run"], Str ": Execute code on remote GPU servers directly from a local machine using annotations and the ", Code ("", [], []) "inferless remote-run", Str " command, supporting T4, A10, and A100 GPUs ", Str "[", Str "12", Str "]", Str "."]]], Header 2 ("installation", ["unnumbered", "unlisted"], []) [Str "Installation"], Para [Str "Inferless is a managed cloud platform with no local installation required for the core service. Interaction happens through the web dashboard, the CLI, or the Python client library."], Para [Strong [Str "CLI Installation:"]], CodeBlock ("", ["bash"], []) "pip install inferless-cli
", Para [Strong [Str "CLI Authentication:"]], CodeBlock ("", ["bash"], []) "# Retrieve CLI keys from https://console.inferless.com/user/settings?current-tab=keys
inferless login
# Paste CLI keys when prompted
", Para [Strong [Str "Python Client Installation:"]], CodeBlock ("", ["bash"], []) "pip install --upgrade inferless
", Para [Strong [Str "Quick Start Flow:"]], OrderedList (1, DefaultStyle, DefaultDelim) [[Plain [Str "Sign up for an Inferless account through the web dashboard."]], [Plain [Str "Install the CLI with ", Code ("", [], []) "pip install inferless-cli", Str "."]], [Plain [Str "Authenticate with ", Code ("", [], []) "inferless login", Str "."]], [Plain [Str "Scaffold a demo project: ", Code ("", [], []) "inferless scaffold --demo", Str "."]], [Plain [Str "Initialize the model: ", Code ("", [], []) "inferless init --name <modelname>", Str "."]], [Plain [Str "Deploy to GPU: ", Code ("", [], []) "inferless deploy --gpu T4", Str "."]]], Para [Str "The template repository at ", Link ("", [], []) [Str "github.com/inferless/template"] ("https://github.com/inferless/template", ""), Str " provides a reference implementation using the GPT Neo model with Pydantic request/response schemas ", Str "[", Str "3", Str "]", Str "."], Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Para [Str "Inferless follows a serverless architecture pattern for GPU inference with these primary components:"], CodeBlock ("", [""], []) "Model Source              Inferless Platform              Client
(HuggingFace,       -->  [ Import & Build Layer    ]
 S3, GCS, GitHub,        [ Custom Runtime Builder   ]
 DockerHub,              [ GPU Scheduling Engine    ]
 Dockerfile,             [ Autoscaler (0..N GPUs)   ]  -->  REST API / SSE
 File Upload)            [ Dynamic Batcher          ]
                         [ NFS Volume Storage       ]
                         [ Secrets Manager          ]
                         [ Prometheus + Grafana     ]
", BulletList [[Plain [Strong [Str "Import and Build Layer"], Str ": Fetches model artifacts from the configured source, builds the container with specified runtime dependencies, and prepares the model for deployment. Build progress is tracked with streaming logs via WebSockets ", Str "[", Str "13", Str "]", Str "."]], [Plain [Strong [Str "Custom Runtime Builder"], Str ": Constructs container images from YAML configuration specifying CUDA version (12.4.1, 12.1.1, or 11.8.0), system packages, Python packages, and shell commands. Supports runtime versioning with in-place updates ", Str "[", Str "8", Str "]", Str "[", Str "13", Str "]", Str "."]], [Plain [Strong [Str "GPU Scheduling Engine"], Str ": Allocates GPU resources (T4, A10, A100) to inference requests. Supports both shared instances (fractional GPU) and dedicated instances (full GPU) ", Str "[", Str "6", Str "]", Str "."]], [Plain [Strong [Str "Autoscaler"], Str ": Monitors request traffic and scales instances between zero and the configured maximum. Integrates warm pools to minimize cold start delays during scaling events ", Str "[", Str "5", Str "]", Str "[", Str "13", Str "]", Str "."]], [Plain [Strong [Str "Dynamic Batcher"], Str ": Combines concurrent inference requests into batches based on configurable batch size and time window, improving throughput for stateless models ", Str "[", Str "9", Str "]", Str "."]], [Plain [Strong [Str "NFS Volume Storage"], Str ": Persistent shared storage accessible across multiple replicas at ", Code ("", [], []) "/var/nfs-share/<volume-name>", Str ". Temporary storage available at ", Code ("", [], []) "/tmp", Str " (deleted when model stops) ", Str "[", Str "7", Str "]", Str "[", Str "14", Str "]", Str "."]], [Plain [Strong [Str "Monitoring Layer"], Str ": Built-in Prometheus metrics and Grafana dashboards for GPU utilization, latency tracking, and system performance observability ", Str "[", Str "1", Str "]", Str "."]]], Header 2 ("key-features", ["unnumbered", "unlisted"], []) [Str "Key Features"], BulletList [[Plain [Strong [Str "Per-Second Billing"], Str ": Charges based on total seconds models are running in a healthy state, rounded up to the nearest second. Billing components include model weight loading time, inference duration, and eviction timeout (5 seconds to 60 minutes of warm status) ", Str "[", Str "6", Str "]", Str "."]], [Plain [Strong [Str "Scale-to-Zero"], Str ": Endpoints scale down to zero instances when idle, eliminating costs during periods of no traffic. Scale-down timing is configurable to balance cost savings against cold start latency ", Str "[", Str "15", Str "]", Str "."]], [Plain [Strong [Str "Seven Integration Sources"], Str ": HuggingFace, GitHub/GitLab, file upload, AWS S3, Google Cloud Buckets, DockerHub, and Dockerfile imports ", Str "[", Str "4", Str "]", Str "."]], [Plain [Strong [Str "Container Concurrency"], Str ": Configure 1 to 100 simultaneous requests per container, with sequential processing or batch processing modes ", Str "[", Str "16", Str "]", Str "."]], [Plain [Strong [Str "Automatic Builds"], Str ": Webhook-driven automatic rebuilds when model sources update, supporting GitHub branch triggers and HuggingFace repo update webhooks ", Str "[", Str "17", Str "]", Str "."]], [Plain [Strong [Str "Version Management"], Str ": Complete build history tracking with automatic version deployment. Models remain on their current version unless explicitly updated ", Str "[", Str "18", Str "]", Str "."]], [Plain [Strong [Str "AWS SNS Alerts"], Str ": Integration for notifications on non-200 HTTP responses and high inference latency (exceeding 20 seconds) ", Str "[", Str "19", Str "]", Str "."]], [Plain [Strong [Str "AWS PrivateLink"], Str ": Private endpoint connectivity for secure, non-public-internet traffic between AWS infrastructure and Inferless deployments ", Str "[", Str "20", Str "]", Str "."]], [Plain [Strong [Str "Cold Start Optimization"], Str ": Custom-built orchestration engine, advanced router, and proprietary storage infrastructure minimize cold start latency, with initialization times as low as 3 seconds ", Str "[", Str "2", Str "]", Str "."]], [Plain [Strong [Str "Remote Run"], Str ": Execute code on remote GPU servers from local machines using ", Code ("", [], []) "inferless remote-run app.py -c config.yaml", Str ", with automatic file transfer (up to 10MB) excluding ", Code ("", [], []) ".git", Str ", ", Code ("", [], []) "*.pyc", Str ", and ", Code ("", [], []) "__pycache__", Str " ", Str "[", Str "12", Str "]", Str "."]], [Plain [Strong [Str "Cookbooks"], Str ": Pre-built deployment recipes for common use cases including PDF Q&A systems, voice chatbots, logo generators, ComfyUI API deployments, debugger agents, and MCP-based Google Maps agents ", Str "[", Str "20", Str "]", Str "."]]], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], BulletList [[Plain [Strong [Str "ML Model Serving"], Str ": Deploying trained models from HuggingFace, custom repositories, or cloud storage as production API endpoints with sub-second cold starts and automatic scaling ", Str "[", Str "2", Str "]", Str "."]], [Plain [Strong [Str "Large Language Model (LLM) Inference"], Str ": Hosting open-weight LLMs (Llama, Qwen, DeepSeek, Mistral, Gemma, Phi, Mixtral) with streaming SSE output and dynamic batching for throughput optimization ", Str "[", Str "10", Str "]", Str "[", Str "20", Str "]", Str "."]], [Plain [Strong [Str "Variable-Traffic Workloads"], Str ": Applications with unpredictable or bursty inference demand that benefit from scale-to-zero and request-based autoscaling without idle GPU costs ", Str "[", Str "5", Str "]", Str "."]], [Plain [Strong [Str "CI/CD Model Deployment"], Str ": Automatic rebuilds triggered by GitHub pushes or HuggingFace webhooks, enabling continuous deployment pipelines without manual infrastructure management ", Str "[", Str "17", Str "]", Str "."]], [Plain [Strong [Str "Multi-Model Management"], Str ": Organizations deploying multiple models that need centralized endpoint management, shared NFS volumes for model weights, and unified monitoring dashboards ", Str "[", Str "7", Str "]", Str "."]], [Plain [Strong [Str "Audio and Video Processing"], Str ": Transient workloads using ", Code ("", [], []) "/tmp", Str " storage for intermediate files during audio/video inference pipelines, with NFS volumes for persistent output storage ", Str "[", Str "14", Str "]", Str "."]], [Plain [Strong [Str "AI Agent Infrastructure"], Str ": Serverless backend for AI agents requiring on-demand GPU compute, demonstrated in cookbooks for debugger agents and MCP-based tool agents ", Str "[", Str "20", Str "]", Str "."]]], Header 2 ("api-reference", ["unnumbered", "unlisted"], []) [Str "API Reference"], Para [Strong [Str "REST API Endpoint Pattern:"]], Para [Str "After deploying a model, Inferless provides a unique API endpoint accessible via the dashboard's API tab. Authentication uses Workspace API keys managed through workspace settings."], CodeBlock ("", ["bash"], []) "curl -X POST \"https://<your-endpoint-url>/v1/predict\" \\
  -H \"Content-Type: application/json\" \\
  -H \"Authorization: Bearer <workspace-api-key>\" \\
  -d '{
    \"input\": {
      \"prompt\": \"Explain serverless inference in one sentence.\"
    }
  }'
", Para [Strong [Str "Python Client (Synchronous):"]], CodeBlock ("", ["python"], []) "import inferless

result = inferless.call(
    url=\"https://<your-endpoint-url>\",
    workspace_key=\"<workspace-api-key>\",
    data={\"prompt\": \"Hello, world!\"}
)
", Para [Strong [Str "Python Client (Asynchronous):"]], CodeBlock ("", ["python"], []) "import inferless

def on_complete(error, response):
    if error:
        print(f\"Error: {error}\")
    else:
        print(f\"Result: {response}\")

inferless.call_async(
    url=\"https://<your-endpoint-url>\",
    workspace_key=\"<workspace-api-key>\",
    data={\"prompt\": \"Hello, world!\"},
    callback=on_complete
)
", Para [Strong [Str "CLI Commands:"]], CodeBlock ("", ["bash"], []) "# Scaffold a demo project
inferless scaffold --demo

# Initialize a model
inferless init --name <modelname>

# Deploy with GPU selection
inferless deploy --gpu T4

# Deploy with region and runtime
inferless deploy --gpu t4 --region <region> --runtime <runtime_name>

# Deploy with volume mount
inferless deploy --gpu t4 --volume <volume_name> --volume-mount-path <path>

# Remote run on GPU
inferless remote-run app.py -c config.yaml

# Volume management
inferless volume create --name <volume_name>
inferless volume cp --source <local_path> --destination <remote_path>
inferless volume ls
inferless volume rm
inferless volume list
inferless volume select --id <volume_id>

# Runtime management
inferless runtime list
inferless runtime upload
inferless runtime patch
inferless runtime version-list
", Para [Strong [Str "Input Schema Definition (", Code ("", [], []) "input_schema.py", Str "):"]], CodeBlock ("", ["python"], []) "INPUT_SCHEMA = {
    \"prompt\": {
        \"datatype\": \"STRING\",
        \"required\": True,
        \"shape\": [1],
        \"example\": [\"There is a fine house in the forest\"]
    },
    \"num_steps\": {
        \"datatype\": \"INT8\",
        \"required\": False,
        \"shape\": [1],
        \"example\": [50]
    },
}
", Para [Str "Supported datatypes: STRING, BOOL, INT8, INT16, INT32, INT64, FP16, FP32, FP64, UINT8, UINT16, UINT32, UINT64, BYTES, BF16. Shape ", Code ("", [], []) "[1]", Str " returns a single variable; arrays greater than 1 return arrays; ", Code ("", [], []) "-1", Str " indicates variable length ", Str "[", Str "21", Str "]", Str "."], Para [Strong [Str "Output Format:"]], Para [Str "Outputs are returned as dictionaries from the ", Code ("", [], []) "infer()", Str " function without explicit schema configuration:"], CodeBlock ("", ["python"], []) "# Simple output
return {\"label_1\": 0.398, \"label_2\": 0.563}

# Array output
return {\"generated_images_base64\": [img_str1, img_str2]}

# Dynamic keys (JSON stringified)
return {\"result\": json.dumps({\"label_x\": 0.4554})}
", Header 2 ("configuration", ["unnumbered", "unlisted"], []) [Str "Configuration"], Para [Strong [Str "Runtime Configuration (", Code ("", [], []) "inferless_runtime_config.yaml", Str "):"]], CodeBlock ("", ["yaml"], []) "cuda_version: \"12.4.1\"  # Options: \"12.4.1\", \"12.1.1\", \"11.8.0\" (default: 12.1.1)
system_packages:
  - libssl-dev
  - opencv
  - ffmpeg
python_packages:
  - transformers==4.41.1
  - torch==2.1.2
  - numpy
  - pandas
run_commands:
  - ln -sf /usr/lib/x86_64-linux-gnu/libcuda.so /usr/lib/libcuda.so
", Para [Strong [Str "Model App Structure (", Code ("", [], []) "app.py", Str "):"]], CodeBlock ("", ["python"], []) "class InferlessPythonModel:
    def initialize(self):
        \"\"\"Load model weights and initialize pipeline (runs once on cold start).\"\"\"
        from transformers import pipeline
        self.generator = pipeline(\"text-generation\", model=\"EleutherAI/gpt-neo-125M\", device=0)

    def infer(self, inputs):
        \"\"\"Run inference on input data (runs per request).\"\"\"
        prompt = inputs[\"prompt\"]
        result = self.generator(prompt, max_length=100)
        return {\"generated_txt\": result[0][\"generated_text\"]}

    def finalize(self):
        \"\"\"Cleanup resources (runs on shutdown).\"\"\"
        pass
", Para [Strong [Str "Dynamic Batching Configuration:"]], CodeBlock ("", ["python"], []) "# In input_schema.py
BATCH_SIZE = 4
BATCH_WINDOW = 5000  # milliseconds

INPUT_SCHEMA = {
    \"prompt\": {
        \"datatype\": \"STRING\",
        \"required\": True,
        \"shape\": [1],
        \"example\": [\"Hello\"]
    },
}
", Para [Str "With batching enabled, the ", Code ("", [], []) "infer()", Str " method receives a list of dictionaries and must return a list of dictionaries ", Str "[", Str "9", Str "]", Str "."], Para [Strong [Str "Streaming SSE Configuration:"]], CodeBlock ("", ["python"], []) "# In input_schema.py
IS_STREAMING_OUTPUT = True

INPUT_SCHEMA = {
    \"prompt\": {
        \"datatype\": \"STRING\",
        \"required\": True,
        \"shape\": [1],
        \"example\": [\"Hello\"]
    },
}
", Para [Str "The ", Code ("", [], []) "infer()", Str " method receives a ", Code ("", [], []) "stream_output_handler", Str " parameter. Call ", Code ("", [], []) "send_streamed_output()", Str " for partial outputs and ", Code ("", [], []) "finalise_streamed_output()", Str " to close the stream. SSE inputs are limited to INT, STRING, and BOOLEAN datatypes with shape ", Code ("", [], []) "[1]", Str " ", Str "[", Str "10", Str "]", Str "."], Para [Strong [Str "Model Settings (Dashboard):"]], BulletList [[Plain [Strong [Str "Scale Down Timeout"], Str ": Controls how quickly idle containers terminate (balance cost vs. cold start latency) ", Str "[", Str "15", Str "]", Str "."]], [Plain [Strong [Str "Inference Timeout"], Str ": Maximum execution duration in seconds for inference requests ", Str "[", Str "15", Str "]", Str "."]], [Plain [Strong [Str "Container Concurrency"], Str ": 1 to 100 simultaneous requests per container ", Str "[", Str "16", Str "]", Str "."]], [Plain [Strong [Str "GPU Type"], Str ": T4, A10, or A100 in shared or dedicated configurations ", Str "[", Str "6", Str "]", Str "."]], [Plain [Strong [Str "Min/Max Replicas"], Str ": Scaling boundaries for autoscaler ", Str "[", Str "5", Str "]", Str "."]]], Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], BulletList [[Plain [Strong [Str "HuggingFace"], Str ": Direct model import via model identifier with optional webhook-based automatic rebuilds on model updates. Supports Transformer, ONNX, and custom model types ", Str "[", Str "4", Str "]", Str "[", Str "17", Str "]", Str "."]], [Plain [Strong [Str "GitHub/GitLab"], Str ": Deploy custom code from repositories with branch-specific automatic builds on push. Requires ", Code ("", [], []) "app.py", Str ", ", Code ("", [], []) "input_schema.py", Str ", and ", Code ("", [], []) "inferless_runtime_config.yaml", Str " ", Str "[", Str "3", Str "]", Str "[", Str "17", Str "]", Str "."]], [Plain [Strong [Str "AWS S3"], Str ": Import model artifacts from S3 buckets using AWS credentials configured through the secrets manager ", Str "[", Str "4", Str "]", Str "."]], [Plain [Strong [Str "Google Cloud Storage"], Str ": Import from GCS buckets for teams using the Google Cloud ecosystem ", Str "[", Str "4", Str "]", Str "."]], [Plain [Strong [Str "DockerHub"], Str ": Deploy pre-built Docker containers directly from DockerHub registries ", Str "[", Str "4", Str "]", Str "."]], [Plain [Strong [Str "Dockerfile"], Str ": Build and deploy from Dockerfiles for full container customization ", Str "[", Str "4", Str "]", Str "."]], [Plain [Strong [Str "AWS SNS"], Str ": Alert integration for model health monitoring with notifications on HTTP errors and latency spikes ", Str "[", Str "19", Str "]", Str "."]], [Plain [Strong [Str "AWS PrivateLink"], Str ": Private endpoint connectivity for secure traffic between AWS infrastructure and Inferless ", Str "[", Str "20", Str "]", Str "."]], [Plain [Strong [Str "Python Client"], Str ": Synchronous and asynchronous API calls from Python applications using the ", Code ("", [], []) "inferless", Str " package with Workspace API key authentication ", Str "[", Str "22", Str "]", Str "."]], [Plain [Strong [Str "REST API"], Str ": Standard HTTP endpoints compatible with any programming language or HTTP client for direct inference calls ", Str "[", Str "23", Str "]", Str "."]], [Plain [Strong [Str "Prometheus and Grafana"], Str ": Built-in metrics export for GPU utilization and performance monitoring integration ", Str "[", Str "1", Str "]", Str "."]]], Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Para [Strong [Str "Deploying from HuggingFace via Dashboard:"]], OrderedList (1, DefaultStyle, DefaultDelim) [[Plain [Str "Select \"HuggingFace\" from the workspace dashboard."]], [Plain [Str "Enter model name, type (Transformer), task (Text generation), and HuggingFace model identifier."]], [Plain [Str "Customize ", Code ("", [], []) "app.py", Str " and ", Code ("", [], []) "input_schema.py", Str " as needed."]], [Plain [Str "Select GPU type (T4/A10/A100), set min and max replicas."]], [Plain [Str "Configure runtime dependencies, volumes, and secrets."]], [Plain [Str "Review and submit. Monitor build progress (typically 5-10 minutes) ", Str "[", Str "5", Str "]", Str "."]]], Para [Strong [Str "Deploying from CLI:"]], CodeBlock ("", ["bash"], []) "# Create a new project from template
inferless scaffold --demo

# Initialize with model name
inferless init --name my-llm-model

# Deploy on A10 GPU
inferless deploy --gpu A10

# Attach a volume for model weights
inferless volume create --name model-weights
inferless volume cp --source ./weights --destination /model-weights
inferless deploy --gpu A10 --volume model-weights --volume-mount-path /var/nfs-share/model-weights
", Para [Strong [Str "Calling a Deployed Endpoint (Python):"]], CodeBlock ("", ["python"], []) "import inferless

# Synchronous call
result = inferless.call(
    url=\"https://your-endpoint.inferless.com\",
    workspace_key=\"your-workspace-api-key\",
    data={\"prompt\": \"Explain serverless inference in one sentence.\"}
)
print(result)
", Para [Strong [Str "Streaming SSE Example (", Code ("", [], []) "app.py", Str "):"]], CodeBlock ("", ["python"], []) "class InferlessPythonModel:
    def initialize(self):
        from transformers import AutoModelForCausalLM, AutoTokenizer
        self.tokenizer = AutoTokenizer.from_pretrained(\"model-name\")
        self.model = AutoModelForCausalLM.from_pretrained(\"model-name\", device_map=\"cuda\")

    def infer(self, inputs, stream_output_handler):
        prompt = inputs[\"prompt\"]
        # Generate tokens iteratively
        for token in self.generate_stream(prompt):
            stream_output_handler.send_streamed_output({\"token\": token})
        stream_output_handler.finalise_streamed_output()

    def finalize(self):
        pass
", Para [Str "Reference streaming template: ", Link ("", [], []) [Str "github.com/inferless/inferless_template_streaming"] ("https://github.com/inferless/inferless_template_streaming", ""), Str " ", Str "[", Str "10", Str "]", Str "."], Para [Strong [Str "Remote Run Example:"]], CodeBlock ("", ["python"], []) "import inferless

@inferless.method(gpu=\"T4\")
def generate(prompt):
    from transformers import pipeline
    generator = pipeline(\"text-generation\", model=\"gpt2\", device=0)
    return generator(prompt, max_length=100)
", CodeBlock ("", ["bash"], []) "inferless remote-run app.py -c config.yaml
", Header 2 ("limitations", ["unnumbered", "unlisted"], []) [Str "Limitations"], BulletList [[Plain [Strong [Str "Closed Source"], Str ": The platform is proprietary with no self-hosted option; all inference runs on Inferless-managed infrastructure ", Str "[", Str "1", Str "]", Str "."]], [Plain [Strong [Str "Vendor Lock-In"], Str ": Deployment configurations (", Code ("", [], []) "app.py", Str ", ", Code ("", [], []) "input_schema.py", Str ", ", Code ("", [], []) "inferless_runtime_config.yaml", Str ") are specific to the Inferless platform and not portable to other serving solutions."]], [Plain [Strong [Str "Cold Start Latency"], Str ": Scale-to-zero introduces cold start delays when the first request arrives after a period of inactivity, though the platform optimizes for initialization times as low as 3 seconds ", Str "[", Str "2", Str "]", Str "."]], [Plain [Strong [Str "GPU Selection"], Str ": Limited to T4, A10, and A100 GPUs. No H100 or other GPU types are listed in the current pricing ", Str "[", Str "6", Str "]", Str "."]], [Plain [Strong [Str "Python Version"], Str ": Remote run supports only Python 3.10; other versions may face compatibility issues ", Str "[", Str "12", Str "]", Str "."]], [Plain [Strong [Str "SSE Datatype Restrictions"], Str ": Streaming output inputs are limited to INT, STRING, and BOOLEAN datatypes with shape ", Code ("", [], []) "[1]", Str ". Multiple inputs require JSON serialization as strings ", Str "[", Str "10", Str "]", Str "."]], [Plain [Strong [Str "File Transfer Limit"], Str ": Remote run file transfer is capped at 10MB per working directory ", Str "[", Str "12", Str "]", Str "."]], [Plain [Strong [Str "Secret Scope"], Str ": Secrets are user-level and can only be updated by the person who performed the model import ", Str "[", Str "11", Str "]", Str "."]], [Plain [Strong [Str "Root Filesystem"], Str ": The platform restricts root file system access. Persistent storage requires NFS volumes; temporary storage at ", Code ("", [], []) "/tmp", Str " is deleted when the model stops ", Str "[", Str "14", Str "]", Str "."]]], Header 2 ("changelog", ["unnumbered", "unlisted"], []) [Str "Changelog"], Para [Str "Notable platform updates through the changelog (2023-2025):"], BulletList [[Plain [Strong [Str "June 2025"], Str ": Runtime version switching for deployed models, streaming build logs via WebSockets, improved autoscaler with warm pool integration for faster cold start recovery ", Str "[", Str "13", Str "]", Str "."]], [Plain [Strong [Str "May 2025"], Str ": Continued platform performance improvements and dashboard enhancements ", Str "[", Str "20", Str "]", Str "."]], [Plain [Strong [Str "April 2025"], Str ": CLI and dashboard updates ", Str "[", Str "20", Str "]", Str "."]], [Plain [Strong [Str "March 2025"], Str ": Feature releases and optimizations ", Str "[", Str "20", Str "]", Str "."]], [Plain [Strong [Str "February 2025"], Str ": Platform stability improvements ", Str "[", Str "20", Str "]", Str "."]], [Plain [Strong [Str "January 2025"], Str ": New year platform updates ", Str "[", Str "20", Str "]", Str "."]], [Plain [Strong [Str "December 2024"], Str ": End-of-year feature releases ", Str "[", Str "20", Str "]", Str "."]], [Plain [Strong [Str "November 2024"], Str ": Two update cycles with infrastructure improvements ", Str "[", Str "20", Str "]", Str "."]], [Plain [Strong [Str "October 2024"], Str ": Platform enhancements ", Str "[", Str "20", Str "]", Str "."]], [Plain [Strong [Str "September 2024"], Str ": Infrastructure updates ", Str "[", Str "20", Str "]", Str "."]], [Plain [Strong [Str "July 2024"], Str ": Mid-year feature releases ", Str "[", Str "20", Str "]", Str "."]], [Plain [Strong [Str "June 2024"], Str ": Two update cycles ", Str "[", Str "20", Str "]", Str "."]], [Plain [Strong [Str "May 2024"], Str ": Platform improvements ", Str "[", Str "20", Str "]", Str "."]], [Plain [Strong [Str "April 2024"], Str ": Two update cycles ", Str "[", Str "20", Str "]", Str "."]], [Plain [Strong [Str "March 2024"], Str ": Two update cycles ", Str "[", Str "20", Str "]", Str "."]], [Plain [Strong [Str "February 2024"], Str ": Two update cycles ", Str "[", Str "20", Str "]", Str "."]], [Plain [Strong [Str "January 2024"], Str ": Three update cycles ", Str "[", Str "20", Str "]", Str "."]], [Plain [Strong [Str "December 2023"], Str ": Three update cycles marking the initial public changelog ", Str "[", Str "20", Str "]", Str "."]]], Para [Str "Refer to the ", Link ("", [], []) [Str "Inferless Changelog"] ("https://docs.inferless.com/changelog/overview", ""), Str " for complete details on each release."], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " What is Inferless - https://docs.inferless.com/introduction/what-is-inferless"]], [Plain [Str "[", Str "2", Str "]", Str " Serverless GPUs for AI/ML Inference - https://www.inferless.com/serverless-gpu"]], [Plain [Str "[", Str "3", Str "]", Str " Quickstart Guide - https://docs.inferless.com/introduction/quickstart"]], [Plain [Str "[", Str "4", Str "]", Str " Integrations - https://docs.inferless.com/introduction/integrations"]], [Plain [Str "[", Str "5", Str "]", Str " Deploy ML Models - https://docs.inferless.com/getting-started/deploy-ml"]], [Plain [Str "[", Str "6", Str "]", Str " Pricing - https://www.inferless.com/pricing"]], [Plain [Str "[", Str "7", Str "]", Str " Working with NFS Volumes - https://docs.inferless.com/concepts/working-with-nfs-volumes"]], [Plain [Str "[", Str "8", Str "]", Str " Building Custom Images - https://docs.inferless.com/concepts/building-custom-images"]], [Plain [Str "[", Str "9", Str "]", Str " Dynamic Batching - https://docs.inferless.com/concepts/dynamic-batching"]], [Plain [Str "[", Str "10", Str "]", Str " Streaming with SSE - https://docs.inferless.com/concepts/streaming-with-sse"]], [Plain [Str "[", Str "11", Str "]", Str " Managing Secrets - https://docs.inferless.com/concepts/managing-secrets-on-inferless"]], [Plain [Str "[", Str "12", Str "]", Str " Remote Run - https://docs.inferless.com/concepts/remote-run"]], [Plain [Str "[", Str "13", Str "]", Str " Changelog June 2025 - https://docs.inferless.com/changelog/June-2025/30th-June"]], [Plain [Str "[", Str "14", Str "]", Str " Working with Files - https://docs.inferless.com/concepts/working-with-files"]], [Plain [Str "[", Str "15", Str "]", Str " Model Settings - https://docs.inferless.com/api-reference/model-endpoint/configuring-the-model-settings"]], [Plain [Str "[", Str "16", Str "]", Str " Processing Concurrent Requests - https://docs.inferless.com/concepts/processing-concurrent-requests"]], [Plain [Str "[", Str "17", Str "]", Str " Automatic Builds - https://docs.inferless.com/concepts/setting-up-automatic-builds"]], [Plain [Str "[", Str "18", Str "]", Str " Version Management - https://docs.inferless.com/api-reference/version-management"]], [Plain [Str "[", Str "19", Str "]", Str " AWS SNS Alerts - https://docs.inferless.com/integrations/aws-sns/aws-sns"]], [Plain [Str "[", Str "20", Str "]", Str " Inferless Documentation - https://docs.inferless.com/"]], [Plain [Str "[", Str "21", Str "]", Str " Input/Output Schema - https://docs.inferless.com/concepts/configuring-the-input-output-schema"]], [Plain [Str "[", Str "22", Str "]", Str " Python Client - https://docs.inferless.com/api-reference/model-endpoint/inferless-python-client"]], [Plain [Str "[", Str "23", Str "]", Str " Model Endpoint - https://docs.inferless.com/api-reference/model-endpoint/model-endpoint"]]]]