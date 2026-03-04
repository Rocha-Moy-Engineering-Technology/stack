[Header 1 ("ray", [], []) [Str "Ray"], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.18604651162790697)), (AlignDefault, (ColWidth 0.813953488372093))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Strong [Str "Name"]]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Ray"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Strong [Str "Group"]]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GPU Compute & Cloud Platforms"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Strong [Str "Type"]]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "SDK/Infra"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Strong [Str "Open Source"]]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Yes"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Strong [Str "GitHub"]]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "ray-project/ray"] ("https://github.com/ray-project/ray", "")]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Strong [Str "Stars"]]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "41,428"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Strong [Str "Docs"]]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "Official Docs"] ("https://docs.ray.io/en/latest/", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "Ray is an open-source distributed computing framework designed to scale Python applications and machine learning workloads from a single machine to large clusters. It provides both high-level libraries for common ML tasks (training, tuning, serving, data processing, reinforcement learning) and low-level primitives for general-purpose distributed computing. Ray abstracts away the complexity of cluster management, task scheduling, and fault tolerance, allowing developers to parallelize existing Python code with minimal changes. The framework is used extensively in production environments for distributed training, hyperparameter optimization, batch inference, and online model serving."], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], BulletList [[Para [Strong [Str "Ray Core"], Str " provides the foundational distributed computing primitives. Tasks are stateless remote function calls decorated with ", Code ("", [], []) "@ray.remote", Str " that execute asynchronously on cluster workers. Actors are stateful distributed objects, also decorated with ", Code ("", [], []) "@ray.remote", Str ", that maintain state across method invocations and run on dedicated worker processes. Objects are immutable data units stored in Ray's distributed object store, referenced by Object References (ObjectRefs), with automatic spilling to disk when memory is exhausted."]], [Para [Strong [Str "Ray Data"], Str " is a distributed data processing library built on Ray Core for scalable data ingestion, preprocessing, and batch inference pipelines. It provides a streaming execution model that processes data in windows rather than materializing entire datasets in memory, making it suitable for large-scale ETL and feature engineering workflows."]], [Para [Strong [Str "Ray Train"], Str " provides distributed training capabilities across multiple ML frameworks including PyTorch, XGBoost, JAX, and TensorFlow. It handles data parallelism, model checkpointing, and fault tolerance, allowing training jobs to resume from checkpoints after node failures."]], [Para [Strong [Str "Ray Tune"], Str " is a hyperparameter tuning library that supports search space definitions, Population-Based Training (PBT), trial scheduling, and early stopping. It integrates with external optimization libraries such as Optuna and BayesOpt for advanced search algorithms."]], [Para [Strong [Str "Ray Serve"], Str " is a model serving framework with FastAPI integration, multi-model composition through deployment graphs, dynamic request batching, autoscaling based on load, and gRPC support. It enables building inference services that combine multiple models and business logic in a single application."]], [Para [Strong [Str "Ray RLlib"], Str " is a reinforcement learning library with pre-configured algorithms (Proximal Policy Optimization (PPO), Deep Q-Network (DQN), and others), multi-agent training support, and extensible custom RL module definitions."]]], Header 2 ("installation-and-setup", ["unnumbered", "unlisted"], []) [Str "Installation and Setup"], Para [Str "Ray supports multiple installation profiles depending on the intended workload."], Para [Strong [Str "pip (ML workloads with all libraries):"]], CodeBlock ("", ["bash"], []) "pip install -U \"ray[data,train,tune,serve]\"
", Para [Strong [Str "pip (general distributed computing):"]], CodeBlock ("", ["bash"], []) "pip install -U \"ray[default]\"
", Para [Strong [Str "pip (minimal core only):"]], CodeBlock ("", ["bash"], []) "pip install -U \"ray\"
", Para [Strong [Str "Available extras:"], Str " ", Code ("", [], []) "ray[default]", Str ", ", Code ("", [], []) "ray[data]", Str ", ", Code ("", [], []) "ray[train]", Str ", ", Code ("", [], []) "ray[tune]", Str ", ", Code ("", [], []) "ray[serve]", Str ", ", Code ("", [], []) "ray[rllib]", Str "."], Para [Strong [Str "Conda:"]], CodeBlock ("", ["bash"], []) "conda install -c conda-forge \"ray-default\"
", Para [Strong [Str "Docker (CPU):"]], CodeBlock ("", ["bash"], []) "docker run --shm-size=2G -t -i rayproject/ray
", Para [Strong [Str "Docker (GPU):"]], CodeBlock ("", ["bash"], []) "docker run --shm-size=2G -t -i --gpus all rayproject/ray:latest-gpu
", Para [Strong [Str "Supported Python versions:"], Str " 3.10, 3.11, 3.12, 3.13 (beta)."], Para [Strong [Str "Supported platforms:"], Str " Linux (x86_64, aarch64), macOS (Apple Silicon M1+), Windows (beta)."], Para [Str "The ", Code ("", [], []) "--shm-size=2G", Str " flag in Docker is required because Ray's object store uses shared memory (", Code ("", [], []) "/dev/shm", Str ") for efficient inter-process data transfer."], Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Para [Str "Ray clusters consist of a head node and zero or more worker nodes."], BulletList [[Para [Strong [Str "Head node"], Str " runs the Global Control Store (GCS), which is a centralized metadata service that tracks cluster state, actor locations, and object directory information. The head node also hosts the Ray Dashboard and the autoscaler."]], [Para [Strong [Str "Worker nodes"], Str " each run a Raylet process that handles local task scheduling, resource management, and object store operations. The Raylet consists of a task scheduler and a node manager."]], [Para [Strong [Str "Object store"], Str " is a shared-memory (plasma-based) system on each node that enables zero-copy data sharing between tasks and actors on the same node. Objects are transferred between nodes via direct peer-to-peer communication when remote access is needed."]], [Para [Strong [Str "Dashboard"], Str " provides a web-based interface for monitoring cluster health, viewing task and actor states, inspecting logs, and profiling workloads."]]], Para [Str "The scheduling model is distributed: each Raylet can schedule tasks locally or forward them to other nodes based on resource availability. This avoids a single-point bottleneck for task dispatch."], Header 2 ("key-features-and-functionality", ["unnumbered", "unlisted"], []) [Str "Key Features and Functionality"], BulletList [[Para [Strong [Str "Task parallelism"], Str " with ", Code ("", [], []) "@ray.remote", Str " decorator converts Python functions into distributed tasks that execute asynchronously across the cluster and return futures (ObjectRefs)."]], [Para [Strong [Str "Stateful actors"], Str " maintain persistent state across invocations and support concurrent method calls with configurable concurrency limits."]], [Para [Strong [Str "Automatic object spilling"], Str " moves objects from the in-memory object store to local disk or external storage when memory pressure is detected."]], [Para [Strong [Str "Fault tolerance"], Str " operates at both the node level (GCS fault tolerance, actor reconstruction) and the application level (task retries, checkpoint-based recovery in Ray Train)."]], [Para [Strong [Str "Autoscaling"], Str " dynamically adds or removes cluster nodes based on pending resource demands, supported on Kubernetes via KubeRay and on cloud VMs via the cluster launcher."]], [Para [Strong [Str "Resource isolation"], Str " through cgroup v2 support enables CPU and memory limits per task or actor."]], [Para [Strong [Str "Token-based authentication"], Str " secures multi-tenant cluster access."]], [Para [Strong [Str "Namespace isolation"], Str " provides logical separation of actors and named resources within a shared cluster."]], [Para [Strong [Str "Cross-language support"], Str " includes a Java API for interoperating with JVM-based systems."]], [Para [Strong [Str "Ray Compiled Graph (beta)"], Str " optimizes Directed Acyclic Graph (DAG) execution by pre-compiling task graphs for reduced scheduling overhead."]], [Para [Strong [Str "Observability"], Str " through Ray Dashboard for web-based cluster visualization, Ray Distributed Debugger, State API/CLI for cluster queries, Prometheus metrics collection, structured logging, and profiling support."]]], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], BulletList [[Para [Strong [Str "Distributed model training"], Str " across multiple GPUs and nodes using Ray Train with PyTorch, XGBoost, or JAX, including automatic checkpointing and fault recovery."]], [Para [Strong [Str "Hyperparameter optimization"], Str " using Ray Tune to run parallel trials with early stopping, Population-Based Training, and integration with search libraries like Optuna."]], [Para [Strong [Str "Online model serving"], Str " with Ray Serve for deploying inference endpoints that compose multiple models, apply dynamic batching, and autoscale based on request volume."]], [Para [Strong [Str "Batch inference"], Str " using Ray Data to process large datasets through ML models in a streaming fashion without materializing entire datasets in memory."]], [Para [Strong [Str "Data preprocessing"], Str " pipelines that chain transformations, feature engineering, and data validation steps across a cluster using Ray Data."]], [Para [Strong [Str "Reinforcement learning"], Str " experiments using RLlib with built-in algorithm implementations and multi-agent environment support."]], [Para [Strong [Str "General-purpose distributed computing"], Str " for parallelizing CPU-bound or I/O-bound Python workloads that do not involve ML."]]], Header 2 ("api-reference-summary", ["unnumbered", "unlisted"], []) [Str "API Reference Summary"], Para [Strong [Str "Ray Core:"]], CodeBlock ("", ["python"], []) "import ray

ray.init()                              # Initialize local cluster or connect to existing
ray.shutdown()                          # Disconnect from the cluster

@ray.remote                             # Decorator for remote functions (tasks) and classes (actors)
ray.get(object_ref)                     # Block and retrieve the value of an ObjectRef
ray.put(value)                          # Store a value in the object store, returns ObjectRef
ray.wait(object_refs, num_returns=1)    # Wait for a subset of futures to complete
", Para [Strong [Str "Tasks:"]], CodeBlock ("", ["python"], []) "@ray.remote
def my_function(x):
    return x * x

future = my_function.remote(42)         # Non-blocking remote invocation
result = ray.get(future)                # Blocking retrieval
", Para [Strong [Str "Actors:"]], CodeBlock ("", ["python"], []) "@ray.remote
class Counter:
    def __init__(self):
        self.count = 0

    def increment(self):
        self.count += 1
        return self.count

counter = Counter.remote()              # Instantiate remote actor
future = counter.increment.remote()     # Non-blocking method call
value = ray.get(future)                 # Blocking retrieval
", Para [Strong [Str "Ray Data:"]], CodeBlock ("", ["python"], []) "import ray.data

ds = ray.data.read_parquet(\"s3://bucket/path\")
ds = ds.map(transform_fn)
ds = ds.filter(filter_fn)
", Para [Strong [Str "Ray Train:"]], CodeBlock ("", ["python"], []) "from ray.train.torch import TorchTrainer
from ray.train import ScalingConfig

trainer = TorchTrainer(
    train_loop_per_worker,
    scaling_config=ScalingConfig(num_workers=4, use_gpu=True),
)
result = trainer.fit()
", Para [Strong [Str "Ray Tune:"]], CodeBlock ("", ["python"], []) "from ray import tune

tuner = tune.Tuner(
    trainable,
    param_space={\"lr\": tune.loguniform(1e-4, 1e-1)},
)
results = tuner.fit()
", Para [Strong [Str "Ray Serve:"]], CodeBlock ("", ["python"], []) "from ray import serve

@serve.deployment
class MyModel:
    def __call__(self, request):
        return {\"result\": \"prediction\"}

app = MyModel.bind()
serve.run(app)
", Header 2 ("configuration-and-customization", ["unnumbered", "unlisted"], []) [Str "Configuration and Customization"], Para [Strong [Str "Cluster configuration"], Str " is managed through YAML files for the cluster launcher or through KubeRay Custom Resource Definitions (CRDs) for Kubernetes deployments."], Para [Strong [Str "Resource specification"], Str " on tasks and actors controls scheduling:"], CodeBlock ("", ["python"], []) "@ray.remote(num_cpus=2, num_gpus=1, memory=4 * 1024 * 1024 * 1024)
def gpu_task():
    pass
", Para [Strong [Str "Custom resources"], Str " can be defined for specialized hardware:"], CodeBlock ("", ["python"], []) "ray.init(resources={\"TPU\": 4})

@ray.remote(resources={\"TPU\": 1})
def tpu_task():
    pass
", Para [Strong [Str "Runtime environment"], Str " specification allows per-task or per-actor dependency isolation:"], CodeBlock ("", ["python"], []) "@ray.remote(runtime_env={\"pip\": [\"numpy==1.24.0\"], \"env_vars\": {\"KEY\": \"value\"}})
def isolated_task():
    pass
", Para [Strong [Str "Ray Serve configuration"], Str " supports YAML-based deployment definitions for production environments, specifying replicas, autoscaling policies, resource limits, and health check intervals."], Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], BulletList [[Para [Strong [Str "Kubernetes"], Str " via KubeRay operator with three CRDs: RayCluster for long-running clusters, RayJob for batch job submission, and RayService for serving workloads with zero-downtime upgrades."]], [Para [Strong [Str "Cloud VMs"], Str " via the Ray cluster launcher supporting AWS EC2, Google Cloud Platform (GCP) Compute Engine, Azure Virtual Machines, and VMware vSphere."]], [Para [Strong [Str "ML frameworks"], Str " through Ray Train adapters for PyTorch (TorchTrainer), XGBoost (XGBoostTrainer), LightGBM (LightGBMTrainer), TensorFlow (TensorflowTrainer), and JAX."]], [Para [Strong [Str "Hyperparameter search"], Str " integrations with Optuna, BayesOpt, HyperOpt, and Ax through Ray Tune's search algorithm interface."]], [Para [Strong [Str "FastAPI"], Str " integration in Ray Serve allows defining HTTP endpoints with standard FastAPI decorators while leveraging Ray's distributed serving infrastructure."]], [Para [Strong [Str "Prometheus"], Str " metrics export for cluster and application-level monitoring."]], [Para [Strong [Str "Spark"], Str " interoperability through the Ray on Spark integration for running Ray workloads within existing Spark infrastructure."]]], Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Para [Strong [Str "Scaling a Python function across a cluster:"]], CodeBlock ("", ["python"], []) "import ray

ray.init()

@ray.remote
def compute(x):
    return x ** 2

futures = [compute.remote(i) for i in range(1000)]
results = ray.get(futures)
", Para [Strong [Str "Distributed training with PyTorch:"]], CodeBlock ("", ["python"], []) "import ray.train
from ray.train.torch import TorchTrainer
from ray.train import ScalingConfig

def train_loop(config):
    model = ray.train.torch.prepare_model(MyModel())
    dataloader = ray.train.torch.prepare_data_loader(train_dl)
    for epoch in range(config[\"epochs\"]):
        for batch in dataloader:
            loss = train_step(model, batch)
        ray.train.report({\"loss\": loss.item()})

trainer = TorchTrainer(
    train_loop,
    train_loop_config={\"epochs\": 10},
    scaling_config=ScalingConfig(num_workers=4, use_gpu=True),
)
result = trainer.fit()
", Para [Strong [Str "Serving a model with Ray Serve:"]], CodeBlock ("", ["python"], []) "from ray import serve
from fastapi import FastAPI

app = FastAPI()

@serve.deployment
@serve.ingress(app)
class ModelServer:
    def __init__(self):
        self.model = load_model()

    @app.post(\"/predict\")
    async def predict(self, request):
        data = await request.json()
        return {\"prediction\": self.model(data[\"input\"])}

serve.run(ModelServer.bind(), route_prefix=\"/\")
", Header 2 ("limitations-and-considerations", ["unnumbered", "unlisted"], []) [Str "Limitations and Considerations"], BulletList [[Para [Strong [Str "Shared memory requirement:"], Str " Ray's object store relies on ", Code ("", [], []) "/dev/shm", Str " for inter-process communication. Docker containers require ", Code ("", [], []) "--shm-size", Str " configuration, and systems with small shared memory partitions may encounter object store errors."]], [Para [Strong [Str "Head node as single point of failure:"], Str " While GCS fault tolerance exists, head node recovery adds latency and the cluster is unavailable during failover. Production deployments should plan for head node resilience."]], [Para [Strong [Str "Python-centric:"], Str " Although Java support exists, the primary development experience and most libraries (Train, Tune, Serve, Data, RLlib) are Python-only. Cross-language interoperability has limitations."]], [Para [Strong [Str "Serialization overhead:"], Str " All data passed between tasks and actors is serialized. Large objects benefit from the object store, but frequent small-message communication patterns can incur overhead."]], [Para [Strong [Str "Complexity of multi-library stacks:"], Str " Combining Ray Data, Train, Tune, and Serve in a single application introduces configuration complexity around resource allocation, scheduling priorities, and memory management."]], [Para [Strong [Str "Windows support is beta:"], Str " Full production support is limited to Linux and macOS. Windows users may encounter missing features or stability issues."]], [Para [Strong [Str "Cluster startup latency:"], Str " Autoscaling new nodes, especially on cloud VMs, introduces minutes-level delays before new capacity is available for task scheduling."]]], Header 2 ("changelog-highlights", ["unnumbered", "unlisted"], []) [Str "Changelog Highlights"], Para [Str "Ray follows a regular release cadence. Key recent developments include Ray Compiled Graph for optimized DAG execution (beta), cgroup v2 support for resource isolation, expanded Python 3.13 support (beta), and continued improvements to Ray Data's streaming execution model. The KubeRay operator has matured with stable CRDs for RayCluster, RayJob, and RayService."], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " Ray Documentation - https://docs.ray.io/en/latest/"]], [Plain [Str "[", Str "2", Str "]", Str " Ray GitHub - https://github.com/ray-project/ray"]]]]