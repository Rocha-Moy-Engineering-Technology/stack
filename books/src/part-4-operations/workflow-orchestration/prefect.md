# Prefect

> Python workflow orchestration framework for scheduling and monitoring data pipelines

| Field | Value |
|-------|-------|
| Group | Workflow Orchestration |
| Type | SDK |
| Open Source | Yes |
| GitHub | [https://github.com/PrefectHQ/prefect](https://github.com/PrefectHQ/prefect) |
| Stars | 22402 |
| Documentation | [Official Docs](https://docs.prefect.io/) |

## Overview

Prefect is an open-source workflow orchestration engine that transforms standard Python functions into production-grade data pipelines. Built around two core decorators (`@flow` and `@task`), Prefect enables developers to add scheduling, retries, caching, monitoring, and event-driven automation to existing Python code without requiring a Domain-Specific Language (DSL), YAML configuration, or special syntax. The framework supports full type hints and async/await patterns natively.

Prefect follows a hybrid execution model where workflow metadata is managed by a central API server (self-hosted or Prefect Cloud) while the actual code execution remains within the user's infrastructure. This separation allows teams to orchestrate workloads across Docker, Kubernetes, and serverless platforms (AWS ECS, Azure ACI, Google Cloud Run (GCR), GCP Vertex AI) without surrendering control of their code or data.

The project is licensed under Apache 2.0, has over 400 contributors, and maintains a community of more than 25,000 practitioners. The codebase is primarily Python (79%) with TypeScript (20%) supporting the monitoring UI built on Vue.

## Core Concepts

- **Flows** -- The primary unit of orchestration. A flow is a Python function decorated with `@flow` that serves as the entry point for a workflow. Flows can call tasks, other flows (subflows), or plain Python functions. Every flow run is tracked, logged, and assigned a state by the orchestration engine.

- **Tasks** -- Individual units of work within a flow, decorated with `@task`. Tasks receive their own retry logic, caching behavior, timeout settings, and concurrency controls independent of the parent flow. Tasks can be submitted for concurrent execution or mapped across datasets.

- **States** -- Every flow run and task run transitions through a sequence of states that describe its lifecycle: `Scheduled`, `Pending`, `Running`, `Completed`, `Failed`, `Cancelled`, `Cancelling`, `Crashed`, `Paused`, and `Suspended`. States drive orchestration decisions such as retries, notifications, and automation triggers.

- **Deployments** -- Server-side representations of flows that store metadata for remote orchestration including timing, execution location, parameters, and infrastructure configuration. Deployments enable scheduled runs, event-based triggers, and API-driven execution.

- **Work Pools** -- Infrastructure templates that define where and how flow runs execute. Work pools support Docker, Kubernetes, and serverless platforms. Push work pools run flows directly on cloud infrastructure without requiring a persistent worker process.

- **Workers** -- Client-side processes that poll work pools for scheduled runs and execute them on the configured infrastructure. Required for hybrid work pools (Docker, Kubernetes) but unnecessary for push work pools.

- **Blocks** -- Reusable configuration objects that store settings, credentials, and connection details. Blocks extend Pydantic's `BaseModel` and integrate with the Prefect UI for visual management. Built-in block types include secret storage, cloud storage connectors, notification channels, and database connections.

- **Variables** -- Key-value pairs stored in the Prefect backend for sharing configuration across workflows. Variables support dynamic deployment configuration through template syntax.

- **Transactions** -- Units of work that execute at most once and produce a result record at a computed cache key address. Transactions support rollback hooks, commit hooks, isolation levels, and idempotency guarantees.

- **Events** -- Observable occurrences within the Prefect ecosystem (state changes, deployments, custom emissions) that trigger automations. Events carry resource metadata and support pattern matching with wildcards.

- **Automations** -- Reactive and proactive rules that execute predefined actions when trigger conditions are met. Triggers respond to state changes, metric thresholds, custom events, or the absence of expected events.

## Architecture

Prefect's architecture separates orchestration metadata from code execution.

```
+---------------------------+
|     Prefect API Server    |
|  (Cloud or Self-Hosted)   |
|  - State tracking         |
|  - Scheduling             |
|  - Event bus              |
|  - Automations engine     |
|  - REST API               |
|  - Monitoring UI          |
+----------+----------------+
           |
           | HTTPS / REST
           |
+----------v----------------+
|     User Infrastructure   |
|  - Workers poll for runs  |
|  - Code executes locally  |
|  - Results stored locally |
|    or in cloud storage    |
+---------------------------+
    |          |          |
    v          v          v
 Docker   Kubernetes  Serverless
                      (ECS/GCR)
```

**API Server** -- The central coordination point that tracks flow run states, manages schedules, processes events, evaluates automation triggers, and serves the monitoring dashboard. Runs as a self-hosted instance or as Prefect Cloud (managed SaaS).

**Workers** -- Lightweight polling processes that run within user infrastructure. Workers check assigned work pools for scheduled flow runs and submit them for execution on the configured compute platform.

**Push Work Pools** -- An alternative to workers for serverless environments. Push pools submit flow runs directly to cloud platforms (AWS ECS, GCR) without a persistent worker process.

**Result Storage** -- Flow and task results persist locally (`~/.prefect/storage/`) by default. For distributed execution, results can be stored in S3, Google Cloud Storage (GCS), Azure Blob, or any filesystem block.

**Technology Stack** -- Python 3.9+ for the orchestration engine and SDK. TypeScript and Vue for the monitoring UI. PostgreSQL or SQLite for the API server database. The REST API uses FastAPI internally.

## Key Features and Functionality

### Decorator-Based Workflow Definition

Prefect uses `@flow` and `@task` decorators to turn standard Python functions into orchestrated components. No DSL or configuration files are required.

```python
from prefect import flow, task

@task(retries=3, retry_delay_seconds=10, log_prints=True)
def extract_data(source: str) -> list[dict]:
    """Extract records from a data source."""
    print(f"Extracting from {source}")
    return [{"id": 1, "value": 42}, {"id": 2, "value": 84}]

@task(timeout_seconds=300)
def transform_data(records: list[dict]) -> list[dict]:
    """Apply transformations to extracted records."""
    return [{"id": r["id"], "value": r["value"] * 2} for r in records]

@task
def load_data(records: list[dict]) -> int:
    """Load transformed records into the destination."""
    print(f"Loading {len(records)} records")
    return len(records)

@flow(name="etl-pipeline", retries=2, retry_delay_seconds=60)
def etl_pipeline(source: str = "production_db") -> int:
    raw = extract_data(source)
    transformed = transform_data(raw)
    count = load_data(transformed)
    return count
```

### Dynamic Task Mapping

Tasks can be mapped across iterable inputs for parallel execution. Tasks are created at runtime based on the data, not statically defined in a DAG.

```python
from prefect import flow, task

@task
def process_customer(customer_id: int) -> dict:
    return {"customer_id": customer_id, "status": "processed"}

@flow
def batch_processing(customer_ids: list[int]):
    # .map() creates one task run per item, executed concurrently
    futures = process_customer.map(customer_ids)
    results = [f.result() for f in futures]
    return results
```

### Concurrent and Parallel Task Execution

Prefect provides multiple task runners for different execution strategies.

```python
from prefect import flow, task
from prefect.task_runners import ThreadPoolTaskRunner

@flow(task_runner=ThreadPoolTaskRunner(max_workers=4))
def parallel_flow():
    # Tasks submitted with .submit() run concurrently
    future_a = task_a.submit()
    future_b = task_b.submit()
    future_c = task_c.submit()
    return future_a.result(), future_b.result(), future_c.result()
```

### Caching and Result Persistence

Tasks support cache policies that prevent redundant re-execution.

```python
from datetime import timedelta
from prefect import flow, task
from prefect.cache_policies import INPUTS

@task(cache_policy=INPUTS, cache_expiration=timedelta(hours=1))
def expensive_computation(data: str) -> dict:
    """Result is cached based on input parameters for 1 hour."""
    return {"result": process(data)}
```

### Transactions with Rollback

Transactions group tasks into atomic units with rollback capabilities.

```python
from prefect import flow, task
from prefect.transactions import transaction
import os

@task
def write_file(contents: str):
    with open("output.txt", "w") as f:
        f.write(contents)

@write_file.on_rollback
def cleanup_file(txn):
    if os.path.exists("output.txt"):
        os.unlink("output.txt")

@task
def validate_output():
    with open("output.txt") as f:
        if len(f.read()) == 0:
            raise ValueError("Empty output")

@flow
def safe_pipeline(contents: str):
    with transaction():
        write_file(contents)
        validate_output()  # If this fails, write_file rolls back
```

### Event-Driven Automations

Automations react to events and execute actions such as notifications, state changes, or deployment triggers.

```python
from prefect.events import emit_event

# Emit a custom event from within a flow or task
emit_event(
    event="data.quality.check.passed",
    resource={"prefect.resource.id": "dataset.customers"},
    payload={"row_count": 15000, "null_percentage": 0.02},
)
```

Automation triggers support 16 action types including cancelling flow runs, pausing deployments, sending Slack/Teams/email notifications, and calling external webhooks.

### Secrets and Variables

```python
from prefect.variables import Variable
from prefect.blocks.system import Secret

# Variables for non-sensitive configuration
Variable.set("environment", "production")
env = Variable.get("environment", default="development")

# Secrets for sensitive credentials (values obfuscated in logs and UI)
secret = Secret(value="sk-abc123...")
secret.save("api-key")

# Retrieve later
loaded = Secret.load("api-key")
api_key = loaded.get()
```

### Subflows

Flows can call other flows, creating a hierarchical execution structure.

```python
from prefect import flow

@flow
def child_flow(data: list[int]) -> float:
    return sum(data) / len(data)

@flow
def parent_flow():
    result_a = child_flow([1, 2, 3])
    result_b = child_flow([4, 5, 6])
    return result_a + result_b
```

## Use Cases

- **Extract, Transform, Load (ETL) Pipelines** -- Schedule and monitor data ingestion workflows with retry logic, caching, and failure notifications. Prefect handles task dependencies, parallel extraction from multiple sources, and checkpoint-based resumption after failures.

- **Machine Learning (ML) Training Pipelines** -- Orchestrate data preprocessing, feature engineering, model training, evaluation, and artifact logging. Task mapping enables hyperparameter sweeps across compute clusters using Dask or Ray runners.

- **Data Quality Monitoring** -- Run scheduled validation checks on datasets with proactive automation triggers when expected events fail to occur (for example, an upstream pipeline does not complete within its window).

- **Event-Driven Data Processing** -- Trigger flows in response to file uploads, database changes, or external webhook events. Composite triggers combine multiple conditions with temporal constraints.

- **CI/CD for Data Pipelines** -- Version deployments with Git commit hashes, promote flows across environments using variables and parameter overrides, and integrate with GitHub, GitLab, Bitbucket, or Azure DevOps.

- **Human-in-the-Loop Workflows** -- Pause flow runs to await human approval or input, then resume execution from the paused state. Useful for data annotation review, model deployment gates, or compliance sign-offs.

## API Reference Summary

### Python SDK

```python
from prefect import flow, task, get_client, get_run_logger
from prefect.variables import Variable
from prefect.blocks.core import Block
from prefect.blocks.system import Secret
from prefect.events import emit_event
from prefect.transactions import transaction
from prefect.cache_policies import INPUTS, TASK_SOURCE, RUN_ID, FLOW_PARAMETERS, NO_CACHE
from prefect.task_runners import ThreadPoolTaskRunner
from prefect.concurrency.sync import rate_limit
```

### Flow Decorator Parameters

```python
@flow(
    name="my-flow",               # Display name (defaults to function name)
    description="...",            # Description (defaults to docstring)
    retries=3,                    # Retry attempts on failure
    retry_delay_seconds=30,       # Delay between retries
    timeout_seconds=3600,         # Maximum runtime
    task_runner=ThreadPoolTaskRunner(max_workers=8),
    validate_parameters=True,     # Pydantic parameter validation
    version="1.0.0",              # Version identifier
    flow_run_name="run-{date}",   # Dynamic run naming
    log_prints=True,              # Capture print() as log messages
    persist_result=True,          # Enable result persistence
    result_storage=...,           # Custom result storage block
    result_serializer="json",     # Result serialization format
)
```

### Task Decorator Parameters

```python
@task(
    name="my-task",               # Display name
    description="...",            # Description
    retries=3,                    # Retry attempts
    retry_delay_seconds=10,       # Delay between retries
    timeout_seconds=300,          # Maximum runtime
    cache_policy=INPUTS,          # Cache key generation policy
    cache_expiration=timedelta(hours=1),
    cache_key_fn=custom_fn,       # Custom cache key function
    tags={"etl", "production"},   # Tags for filtering and monitoring
    task_run_name="process-{id}", # Dynamic run naming
    log_prints=True,              # Capture print() as log messages
    persist_result=True,          # Enable result persistence
    result_storage_key="{parameters[name]}.json",
)
```

### REST API Client

```python
from prefect import get_client

# Async usage (default)
async with get_client() as client:
    runs = await client.read_flow_runs()
    await client.create_flow_run_from_deployment(deployment_id="...")
    await client.set_flow_run_state(flow_run_id="...", state=...)
    events = await client.read_events(filter=...)

# Synchronous usage
with get_client(sync_client=True) as client:
    runs = client.read_flow_runs()
```

### REST API Endpoints

The API base URL follows the pattern `https://api.prefect.cloud/api/accounts/{account_id}/workspaces/{workspace_id}` for Cloud, or `http://localhost:4200/api` for self-hosted. Authentication uses bearer tokens via the `Authorization` header.

## Configuration and Customization

### Environment Variables

```bash
# API connection
PREFECT_API_URL="http://localhost:4200/api"
PREFECT_API_KEY="pnu_..."

# Result persistence
PREFECT_RESULTS_PERSIST_BY_DEFAULT=true
PREFECT_LOCAL_STORAGE_PATH="/custom/storage/path"
PREFECT_DEFAULT_RESULT_STORAGE_BLOCK="s3-bucket/my-bucket"
PREFECT_RESULTS_DEFAULT_SERIALIZER="json"

# Task defaults
PREFECT_TASKS_DEFAULT_PERSIST_RESULT=true

# Logging
PREFECT_LOGGING_LEVEL="DEBUG"

# Server
PREFECT_SERVER_API_HOST="0.0.0.0"
PREFECT_SERVER_API_PORT="4200"
```

### Profiles

Prefect supports named configuration profiles for switching between environments.

```bash
# Create a profile
prefect profile create production
prefect profile use production
prefect config set PREFECT_API_URL="https://api.prefect.cloud/api/..."

# Switch back
prefect profile use default
```

### Custom Blocks

```python
from prefect.blocks.core import Block
from pydantic import SecretStr

class DatabaseCredentials(Block):
    _block_type_name = "Database Credentials"
    _description = "Credentials for database connections"

    host: str
    port: int = 5432
    username: str
    password: SecretStr
    database: str

    def connection_string(self) -> str:
        return f"postgresql://{self.username}:{self.password.get_secret_value()}@{self.host}:{self.port}/{self.database}"

# Register and save
DatabaseCredentials.register_type_and_schema()
creds = DatabaseCredentials(host="db.example.com", username="app", password="secret", database="prod")
creds.save("production-db")

# Load elsewhere
db = DatabaseCredentials.load("production-db")
```

### Deployment Configuration (prefect.yaml)

```yaml
deployments:
  - name: daily-etl
    entrypoint: flows/etl.py:etl_pipeline
    work_pool:
      name: my-docker-pool
    schedule:
      cron: "0 8 * * *"
      timezone: "America/New_York"
    parameters:
      source: "production_db"
    tags:
      - production
      - etl
    concurrency_limit: 1
    version: "{{ prefect.variables.deployment_version }}"
```

## Integration Patterns

### Prefect with Dask for Distributed Execution

```python
from prefect import flow, task
from prefect.task_runners import DaskTaskRunner

@task
def train_model(hyperparams: dict) -> float:
    # Training logic here
    return accuracy

@flow(task_runner=DaskTaskRunner(address="tcp://scheduler:8786"))
def hyperparameter_search(param_grid: list[dict]):
    futures = train_model.map(param_grid)
    results = [f.result() for f in futures]
    best = max(results)
    return best
```

### Prefect with Ray for Parallel ML Workloads

```python
from prefect import flow, task
from prefect.task_runners import RayTaskRunner

@task
def process_partition(partition_id: int) -> dict:
    return {"partition": partition_id, "rows_processed": 10000}

@flow(task_runner=RayTaskRunner())
def distributed_processing(partitions: list[int]):
    futures = process_partition.map(partitions)
    return [f.result() for f in futures]
```

### Prefect with Docker Work Pool

```bash
# Create a Docker work pool
prefect work-pool create --type docker my-docker-pool

# Start a worker for the pool
prefect worker start --pool my-docker-pool
```

```python
from prefect import flow

@flow(log_prints=True)
def containerized_flow():
    print("Running inside Docker")

if __name__ == "__main__":
    containerized_flow.deploy(
        name="docker-deployment",
        work_pool_name="my-docker-pool",
        image="my-registry/my-flow:latest",
        cron="0 */6 * * *",
    )
```

### Prefect with Kubernetes

```bash
# Create a Kubernetes work pool
prefect work-pool create --type kubernetes my-k8s-pool

# Start a worker
prefect worker start --pool my-k8s-pool
```

### Webhook-Triggered Flows

Prefect Cloud supports webhook endpoints that emit events, which can trigger automations to start flow runs. For self-hosted setups, the REST API enables programmatic flow run creation.

```python
from prefect import get_client

async def trigger_flow_from_webhook(deployment_id: str, params: dict):
    async with get_client() as client:
        await client.create_flow_run_from_deployment(
            deployment_id=deployment_id,
            parameters=params,
        )
```

## Examples

### Complete ETL Pipeline with Error Handling

```python
from prefect import flow, task, get_run_logger
from prefect.transactions import transaction
from datetime import timedelta
from prefect.cache_policies import INPUTS

@task(retries=3, retry_delay_seconds=[10, 30, 60], log_prints=True)
def fetch_api_data(endpoint: str) -> list[dict]:
    logger = get_run_logger()
    logger.info(f"Fetching data from {endpoint}")
    import httpx
    response = httpx.get(endpoint, timeout=30)
    response.raise_for_status()
    return response.json()

@task(cache_policy=INPUTS, cache_expiration=timedelta(minutes=30))
def deduplicate(records: list[dict], key: str) -> list[dict]:
    seen = set()
    unique = []
    for record in records:
        k = record[key]
        if k not in seen:
            seen.add(k)
            unique.append(record)
    return unique

@task
def write_to_database(records: list[dict]) -> int:
    # Database write logic
    return len(records)

@write_to_database.on_rollback
def rollback_write(txn):
    batch_id = txn.get("batch_id")
    # Delete the batch from the database
    print(f"Rolling back batch {batch_id}")

@flow(name="api-etl", retries=1, timeout_seconds=1800)
def api_etl_pipeline(endpoint: str, dedup_key: str = "id"):
    with transaction():
        raw = fetch_api_data(endpoint)
        cleaned = deduplicate(raw, dedup_key)
        count = write_to_database(cleaned)
    return count
```

### Scheduled Deployment with Parameterization

```python
from prefect import flow

@flow(log_prints=True)
def data_sync(source: str, target: str, batch_size: int = 1000):
    print(f"Syncing {source} -> {target} in batches of {batch_size}")
    # Sync logic here

if __name__ == "__main__":
    data_sync.serve(
        name="hourly-sync",
        cron="0 * * * *",
        parameters={"source": "warehouse", "target": "analytics", "batch_size": 5000},
        tags=["sync", "production"],
    )
```

### Async Flow with Concurrent Tasks

```python
import asyncio
from prefect import flow, task

@task
async def fetch_user(user_id: int) -> dict:
    await asyncio.sleep(0.1)  # Simulate API call
    return {"id": user_id, "name": f"User {user_id}"}

@task
async def enrich_user(user: dict) -> dict:
    await asyncio.sleep(0.05)  # Simulate enrichment
    user["enriched"] = True
    return user

@flow
async def user_pipeline(user_ids: list[int]):
    users = await asyncio.gather(*[fetch_user(uid) for uid in user_ids])
    enriched = await asyncio.gather(*[enrich_user(u) for u in users])
    return enriched
```

### Using Variables for Environment-Specific Configuration

```python
from prefect import flow
from prefect.variables import Variable

@flow
def configurable_pipeline():
    db_host = Variable.get("db_host", default="localhost")
    batch_size = Variable.get("batch_size", default=100)
    print(f"Connecting to {db_host} with batch size {batch_size}")
    # Pipeline logic using runtime configuration
```

## Limitations and Considerations

- **Python-Only SDK** -- Prefect flows and tasks must be written in Python. Workflows requiring Go, Java, or other languages need wrapper tasks that shell out to external processes.

- **Sync Task Timeout Limitations** -- When using the default `ThreadPoolTaskRunner`, timeouts cannot interrupt blocking synchronous operations such as `time.sleep()` or long network requests. Use `ProcessPoolTaskRunner`, async tasks, or libraries with native timeout parameters for reliable interruption.

- **Metric Triggers Require Cloud** -- Metric-based automation triggers (average duration, lateness, completion percentage) are available only in Prefect Cloud, not in the open-source self-hosted server.

- **Result Serialization Constraints** -- Task results must be serializable (pickle or JSON). Large objects, open file handles, database connections, and other non-serializable values require explicit handling or custom serializers.

- **Worker Infrastructure** -- Hybrid work pools (Docker, Kubernetes) require running a persistent worker process within the user's infrastructure. This adds operational overhead compared to push work pools or the simpler `flow.serve()` pattern.

- **Database Backend** -- The self-hosted Prefect server defaults to SQLite, which is suitable for development but not production workloads. Production deployments should use PostgreSQL.

- **State Explosion with Large Map Operations** -- Task mapping across very large iterables creates a flow run state entry per item. Thousands of mapped tasks can impact API server performance and UI responsiveness.

- **Single-Threaded Event Loop** -- The Prefect orchestration engine uses a single-threaded async event loop. CPU-intensive orchestration logic (complex automation evaluations, large parameter validation) can block the event loop.

## Changelog Highlights

- **2018-2021** -- Initial release of Prefect as a Pythonic workflow orchestration framework. Introduced the core `@flow` and `@task` decorator model with a DAG-based execution engine.

- **2022 (Prefect 2.0)** -- Major rewrite removing the DAG-only constraint. Flows became native Python functions with dynamic task creation at runtime. Introduced work pools, blocks, and the hybrid execution model. Replaced the Prefect Server with a new API-first architecture.

- **2024 (Prefect 3.0)** -- Open-sourced events and automations (previously Cloud-only). Achieved a 90% performance improvement in the orchestration engine. Introduced transactions with rollback and commit hooks. Added `ProcessPoolTaskRunner` for true parallelism. Enhanced caching with configurable cache policies (`INPUTS`, `TASK_SOURCE`, `RUN_ID`, `FLOW_PARAMETERS`, `NO_CACHE`).

- **2025-2026 (3.x Series)** -- Continued iteration on the 3.x line with the latest release being v3.6.18 (February 2026). Ongoing improvements to concurrency controls, result persistence, client API ergonomics, and deployment workflows.

## Citations

- Prefect Official Documentation -- [https://docs.prefect.io/](https://docs.prefect.io/)
- Prefect GitHub Repository -- [https://github.com/PrefectHQ/prefect](https://github.com/PrefectHQ/prefect)
- Prefect 3.0 Release Announcement -- [https://www.prefect.io/blog/prefect-3-is-here](https://www.prefect.io/blog/prefect-3-is-here)
- Prefect Cloud Documentation -- [https://docs.prefect.io/v3/manage/cloud](https://docs.prefect.io/v3/manage/cloud)
- Prefect REST API Reference -- [https://docs.prefect.io/v3/api-ref/rest-api](https://docs.prefect.io/v3/api-ref/rest-api)
- Prefect Community Slack -- [https://prefect.io/slack](https://prefect.io/slack)

## Discovery Signals

> Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search.

### Keywords

- Prefect
- @flow
- @task
- Python-native orchestration
- flow run
- task run
- states
- deployments
- work pools
- workers
- blocks
- variables
- transactions
- events
- automations
- task mapping
- cache policies
- ThreadPoolTaskRunner
- DaskTaskRunner
- RayTaskRunner
- Prefect Cloud
- self-hosted
- hybrid execution
- push work pools
- subflows
- async/await

### Verb-Noun Tasks

- Turn Python functions into orchestrated flows with `@flow` / `@task`
- Map a task across an iterable for parallel execution
- Configure cache policies (INPUTS, TASK_SOURCE, RUN_ID, FLOW_PARAMETERS)
- Wrap tasks in transactions with `on_rollback` hooks
- Emit custom events and trigger automations
- Deploy flows to Docker, Kubernetes, or serverless (ECS, GCR)
- Use Dask or Ray runners for distributed execution
- Store credentials and config in Blocks
- Schedule deployments with cron via prefect.yaml
- Pause flows for human approval and resume from paused state
- Call subflows for hierarchical workflow composition
- Run flows on Prefect Cloud or self-hosted server

### User Intent Phrases

- I want Python-native workflow orchestration without YAML or DSL.
- How do I add retries and caching to my Python functions?
- I need dynamic task mapping for hyperparameter sweeps.
- How do I run flows on Kubernetes without managing a worker?
- I want event-driven automations that react to flow states.
- How do I roll back side effects when a flow fails?
- I need human-in-the-loop approval gates in my pipeline.
- How do I integrate Dask or Ray with my orchestration layer?
- What's a modern alternative to Airflow that uses regular Python?
- How do I run flows hybrid — metadata in the cloud, code on-prem?

### Problem Statements

- Airflow DAGs feel boilerplate-heavy compared to plain Python.
- We need dynamic task creation at runtime, not static DAGs.
- Side effects in pipelines need rollback semantics.
- We want full type hints and async/await, not declarative quirks.
- Hybrid execution is needed so code never leaves our infrastructure.
- Caching should be configurable per task, not global.

### When to Pick This

- Pick this when you want the most Pythonic developer experience for workflow orchestration.
- Pick this over Airflow when you need dynamic task creation, async-native flows, and lighter ceremony.
- Pick this over Temporal when Python-only workflows with state persistence are enough and you don't need cross-language durable execution.
- Pick this over n8n/Activepieces/Node-RED when you want code-first, not visual builders.
- Pick this when transactions with rollback hooks are required for safe pipelines.
- Pick this when push work pools (serverless on ECS/GCR) eliminate the need for persistent workers.

### Related Terms and Aliases

- prefect.io
- Prefect 3.0
- Prefect Cloud
- Prefect Server
- flow.serve()
- flow.deploy()
- prefect.yaml
- work pool
- push pool
- hybrid execution
- @flow decorator
- @task decorator
- emit_event
- transaction()
- Pythonic workflow framework
- DAG-free orchestration

