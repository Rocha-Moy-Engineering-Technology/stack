# Ray

| Field          | Value                                                                |
|----------------|----------------------------------------------------------------------|
| **Name**       | Ray                                                                  |
| **Group**      | GPU Compute & Cloud Platforms                                        |
| **Type**       | SDK/Infra                                                            |
| **Open Source** | Yes                                                                 |
| **GitHub**     | [ray-project/ray](https://github.com/ray-project/ray)               |
| **Stars**      | 41,428                                                               |
| **Docs**       | [Official Docs](https://docs.ray.io/en/latest/)                     |

## Overview

Ray is an open-source distributed computing framework designed to scale Python applications and machine learning workloads from a single machine to large clusters. It provides both high-level libraries for common ML tasks (training, tuning, serving, data processing, reinforcement learning) and low-level primitives for general-purpose distributed computing. Ray abstracts away the complexity of cluster management, task scheduling, and fault tolerance, allowing developers to parallelize existing Python code with minimal changes. The framework is used extensively in production environments for distributed training, hyperparameter optimization, batch inference, and online model serving.

## Core Concepts

- **Ray Core** provides the foundational distributed computing primitives. Tasks are stateless remote function calls decorated with `@ray.remote` that execute asynchronously on cluster workers. Actors are stateful distributed objects, also decorated with `@ray.remote`, that maintain state across method invocations and run on dedicated worker processes. Objects are immutable data units stored in Ray's distributed object store, referenced by Object References (ObjectRefs), with automatic spilling to disk when memory is exhausted.

- **Ray Data** is a distributed data processing library built on Ray Core for scalable data ingestion, preprocessing, and batch inference pipelines. It provides a streaming execution model that processes data in windows rather than materializing entire datasets in memory, making it suitable for large-scale ETL and feature engineering workflows.

- **Ray Train** provides distributed training capabilities across multiple ML frameworks including PyTorch, XGBoost, JAX, and TensorFlow. It handles data parallelism, model checkpointing, and fault tolerance, allowing training jobs to resume from checkpoints after node failures.

- **Ray Tune** is a hyperparameter tuning library that supports search space definitions, Population-Based Training (PBT), trial scheduling, and early stopping. It integrates with external optimization libraries such as Optuna and BayesOpt for advanced search algorithms.

- **Ray Serve** is a model serving framework with FastAPI integration, multi-model composition through deployment graphs, dynamic request batching, autoscaling based on load, and gRPC support. It enables building inference services that combine multiple models and business logic in a single application.

- **Ray RLlib** is a reinforcement learning library with pre-configured algorithms (Proximal Policy Optimization (PPO), Deep Q-Network (DQN), and others), multi-agent training support, and extensible custom RL module definitions.

## Installation and Setup

Ray supports multiple installation profiles depending on the intended workload.

**pip (ML workloads with all libraries):**

```bash
pip install -U "ray[data,train,tune,serve]"
```

**pip (general distributed computing):**

```bash
pip install -U "ray[default]"
```

**pip (minimal core only):**

```bash
pip install -U "ray"
```

**Available extras:** `ray[default]`, `ray[data]`, `ray[train]`, `ray[tune]`, `ray[serve]`, `ray[rllib]`.

**Conda:**

```bash
conda install -c conda-forge "ray-default"
```

**Docker (CPU):**

```bash
docker run --shm-size=2G -t -i rayproject/ray
```

**Docker (GPU):**

```bash
docker run --shm-size=2G -t -i --gpus all rayproject/ray:latest-gpu
```

**Supported Python versions:** 3.10, 3.11, 3.12, 3.13 (beta).

**Supported platforms:** Linux (x86_64, aarch64), macOS (Apple Silicon M1+), Windows (beta).

The `--shm-size=2G` flag in Docker is required because Ray's object store uses shared memory (`/dev/shm`) for efficient inter-process data transfer.

## Architecture

Ray clusters consist of a head node and zero or more worker nodes.

- **Head node** runs the Global Control Store (GCS), which is a centralized metadata service that tracks cluster state, actor locations, and object directory information. The head node also hosts the Ray Dashboard and the autoscaler.

- **Worker nodes** each run a Raylet process that handles local task scheduling, resource management, and object store operations. The Raylet consists of a task scheduler and a node manager.

- **Object store** is a shared-memory (plasma-based) system on each node that enables zero-copy data sharing between tasks and actors on the same node. Objects are transferred between nodes via direct peer-to-peer communication when remote access is needed.

- **Dashboard** provides a web-based interface for monitoring cluster health, viewing task and actor states, inspecting logs, and profiling workloads.

The scheduling model is distributed: each Raylet can schedule tasks locally or forward them to other nodes based on resource availability. This avoids a single-point bottleneck for task dispatch.

## Key Features and Functionality

- **Task parallelism** with `@ray.remote` decorator converts Python functions into distributed tasks that execute asynchronously across the cluster and return futures (ObjectRefs).

- **Stateful actors** maintain persistent state across invocations and support concurrent method calls with configurable concurrency limits.

- **Automatic object spilling** moves objects from the in-memory object store to local disk or external storage when memory pressure is detected.

- **Fault tolerance** operates at both the node level (GCS fault tolerance, actor reconstruction) and the application level (task retries, checkpoint-based recovery in Ray Train).

- **Autoscaling** dynamically adds or removes cluster nodes based on pending resource demands, supported on Kubernetes via KubeRay and on cloud VMs via the cluster launcher.

- **Resource isolation** through cgroup v2 support enables CPU and memory limits per task or actor.

- **Token-based authentication** secures multi-tenant cluster access.

- **Namespace isolation** provides logical separation of actors and named resources within a shared cluster.

- **Cross-language support** includes a Java API for interoperating with JVM-based systems.

- **Ray Compiled Graph (beta)** optimizes Directed Acyclic Graph (DAG) execution by pre-compiling task graphs for reduced scheduling overhead.

- **Observability** through Ray Dashboard for web-based cluster visualization, Ray Distributed Debugger, State API/CLI for cluster queries, Prometheus metrics collection, structured logging, and profiling support.

## Use Cases

- **Distributed model training** across multiple GPUs and nodes using Ray Train with PyTorch, XGBoost, or JAX, including automatic checkpointing and fault recovery.

- **Hyperparameter optimization** using Ray Tune to run parallel trials with early stopping, Population-Based Training, and integration with search libraries like Optuna.

- **Online model serving** with Ray Serve for deploying inference endpoints that compose multiple models, apply dynamic batching, and autoscale based on request volume.

- **Batch inference** using Ray Data to process large datasets through ML models in a streaming fashion without materializing entire datasets in memory.

- **Data preprocessing** pipelines that chain transformations, feature engineering, and data validation steps across a cluster using Ray Data.

- **Reinforcement learning** experiments using RLlib with built-in algorithm implementations and multi-agent environment support.

- **General-purpose distributed computing** for parallelizing CPU-bound or I/O-bound Python workloads that do not involve ML.

## API Reference Summary

**Ray Core:**

```python
import ray

ray.init()                              # Initialize local cluster or connect to existing
ray.shutdown()                          # Disconnect from the cluster

@ray.remote                             # Decorator for remote functions (tasks) and classes (actors)
ray.get(object_ref)                     # Block and retrieve the value of an ObjectRef
ray.put(value)                          # Store a value in the object store, returns ObjectRef
ray.wait(object_refs, num_returns=1)    # Wait for a subset of futures to complete
```

**Tasks:**

```python
@ray.remote
def my_function(x):
    return x * x

future = my_function.remote(42)         # Non-blocking remote invocation
result = ray.get(future)                # Blocking retrieval
```

**Actors:**

```python
@ray.remote
class Counter:
    def __init__(self):
        self.count = 0

    def increment(self):
        self.count += 1
        return self.count

counter = Counter.remote()              # Instantiate remote actor
future = counter.increment.remote()     # Non-blocking method call
value = ray.get(future)                 # Blocking retrieval
```

**Ray Data:**

```python
import ray.data

ds = ray.data.read_parquet("s3://bucket/path")
ds = ds.map(transform_fn)
ds = ds.filter(filter_fn)
```

**Ray Train:**

```python
from ray.train.torch import TorchTrainer
from ray.train import ScalingConfig

trainer = TorchTrainer(
    train_loop_per_worker,
    scaling_config=ScalingConfig(num_workers=4, use_gpu=True),
)
result = trainer.fit()
```

**Ray Tune:**

```python
from ray import tune

tuner = tune.Tuner(
    trainable,
    param_space={"lr": tune.loguniform(1e-4, 1e-1)},
)
results = tuner.fit()
```

**Ray Serve:**

```python
from ray import serve

@serve.deployment
class MyModel:
    def __call__(self, request):
        return {"result": "prediction"}

app = MyModel.bind()
serve.run(app)
```

## Configuration and Customization

**Cluster configuration** is managed through YAML files for the cluster launcher or through KubeRay Custom Resource Definitions (CRDs) for Kubernetes deployments.

**Resource specification** on tasks and actors controls scheduling:

```python
@ray.remote(num_cpus=2, num_gpus=1, memory=4 * 1024 * 1024 * 1024)
def gpu_task():
    pass
```

**Custom resources** can be defined for specialized hardware:

```python
ray.init(resources={"TPU": 4})

@ray.remote(resources={"TPU": 1})
def tpu_task():
    pass
```

**Runtime environment** specification allows per-task or per-actor dependency isolation:

```python
@ray.remote(runtime_env={"pip": ["numpy==1.24.0"], "env_vars": {"KEY": "value"}})
def isolated_task():
    pass
```

**Ray Serve configuration** supports YAML-based deployment definitions for production environments, specifying replicas, autoscaling policies, resource limits, and health check intervals.

## Integration Patterns

- **Kubernetes** via KubeRay operator with three CRDs: RayCluster for long-running clusters, RayJob for batch job submission, and RayService for serving workloads with zero-downtime upgrades.

- **Cloud VMs** via the Ray cluster launcher supporting AWS EC2, Google Cloud Platform (GCP) Compute Engine, Azure Virtual Machines, and VMware vSphere.

- **ML frameworks** through Ray Train adapters for PyTorch (TorchTrainer), XGBoost (XGBoostTrainer), LightGBM (LightGBMTrainer), TensorFlow (TensorflowTrainer), and JAX.

- **Hyperparameter search** integrations with Optuna, BayesOpt, HyperOpt, and Ax through Ray Tune's search algorithm interface.

- **FastAPI** integration in Ray Serve allows defining HTTP endpoints with standard FastAPI decorators while leveraging Ray's distributed serving infrastructure.

- **Prometheus** metrics export for cluster and application-level monitoring.

- **Spark** interoperability through the Ray on Spark integration for running Ray workloads within existing Spark infrastructure.

## Examples

**Scaling a Python function across a cluster:**

```python
import ray

ray.init()

@ray.remote
def compute(x):
    return x ** 2

futures = [compute.remote(i) for i in range(1000)]
results = ray.get(futures)
```

**Distributed training with PyTorch:**

```python
import ray.train
from ray.train.torch import TorchTrainer
from ray.train import ScalingConfig

def train_loop(config):
    model = ray.train.torch.prepare_model(MyModel())
    dataloader = ray.train.torch.prepare_data_loader(train_dl)
    for epoch in range(config["epochs"]):
        for batch in dataloader:
            loss = train_step(model, batch)
        ray.train.report({"loss": loss.item()})

trainer = TorchTrainer(
    train_loop,
    train_loop_config={"epochs": 10},
    scaling_config=ScalingConfig(num_workers=4, use_gpu=True),
)
result = trainer.fit()
```

**Serving a model with Ray Serve:**

```python
from ray import serve
from fastapi import FastAPI

app = FastAPI()

@serve.deployment
@serve.ingress(app)
class ModelServer:
    def __init__(self):
        self.model = load_model()

    @app.post("/predict")
    async def predict(self, request):
        data = await request.json()
        return {"prediction": self.model(data["input"])}

serve.run(ModelServer.bind(), route_prefix="/")
```

## Limitations and Considerations

- **Shared memory requirement:** Ray's object store relies on `/dev/shm` for inter-process communication. Docker containers require `--shm-size` configuration, and systems with small shared memory partitions may encounter object store errors.

- **Head node as single point of failure:** While GCS fault tolerance exists, head node recovery adds latency and the cluster is unavailable during failover. Production deployments should plan for head node resilience.

- **Python-centric:** Although Java support exists, the primary development experience and most libraries (Train, Tune, Serve, Data, RLlib) are Python-only. Cross-language interoperability has limitations.

- **Serialization overhead:** All data passed between tasks and actors is serialized. Large objects benefit from the object store, but frequent small-message communication patterns can incur overhead.

- **Complexity of multi-library stacks:** Combining Ray Data, Train, Tune, and Serve in a single application introduces configuration complexity around resource allocation, scheduling priorities, and memory management.

- **Windows support is beta:** Full production support is limited to Linux and macOS. Windows users may encounter missing features or stability issues.

- **Cluster startup latency:** Autoscaling new nodes, especially on cloud VMs, introduces minutes-level delays before new capacity is available for task scheduling.

## Changelog Highlights

Ray follows a regular release cadence. Key recent developments include Ray Compiled Graph for optimized DAG execution (beta), cgroup v2 support for resource isolation, expanded Python 3.13 support (beta), and continued improvements to Ray Data's streaming execution model. The KubeRay operator has matured with stable CRDs for RayCluster, RayJob, and RayService.

## Citations

- [1] Ray Documentation - https://docs.ray.io/en/latest/
- [2] Ray GitHub - https://github.com/ray-project/ray
