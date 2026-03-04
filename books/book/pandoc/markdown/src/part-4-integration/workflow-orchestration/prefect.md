[Header 1 ("prefect", [], []) [Str "Prefect"], BlockQuote [Para [Str "Python workflow orchestration framework for scheduling and monitoring data pipelines"]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.5)), (AlignDefault, (ColWidth 0.5))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Workflow Orchestration & Automation"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "SDK"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Yes"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "https://github.com/PrefectHQ/prefect"] ("https://github.com/PrefectHQ/prefect", "")]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "21652"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Documentation"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "Official Docs"] ("https://docs.prefect.io/", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "Prefect is an open-source workflow orchestration engine that transforms standard Python functions into production-grade data pipelines. Built around two core decorators (", Code ("", [], []) "@flow", Str " and ", Code ("", [], []) "@task", Str "), Prefect enables developers to add scheduling, retries, caching, monitoring, and event-driven automation to existing Python code without requiring a Domain-Specific Language (DSL), YAML configuration, or special syntax. The framework supports full type hints and async/await patterns natively."], Para [Str "Prefect follows a hybrid execution model where workflow metadata is managed by a central API server (self-hosted or Prefect Cloud) while the actual code execution remains within the user's infrastructure. This separation allows teams to orchestrate workloads across Docker, Kubernetes, and serverless platforms (AWS ECS, Azure ACI, Google Cloud Run (GCR), GCP Vertex AI) without surrendering control of their code or data."], Para [Str "The project is licensed under Apache 2.0, has over 400 contributors, and maintains a community of more than 25,000 practitioners. The codebase is primarily Python (79%) with TypeScript (20%) supporting the monitoring UI built on Vue."], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], BulletList [[Para [Strong [Str "Flows"], Str " -- The primary unit of orchestration. A flow is a Python function decorated with ", Code ("", [], []) "@flow", Str " that serves as the entry point for a workflow. Flows can call tasks, other flows (subflows), or plain Python functions. Every flow run is tracked, logged, and assigned a state by the orchestration engine."]], [Para [Strong [Str "Tasks"], Str " -- Individual units of work within a flow, decorated with ", Code ("", [], []) "@task", Str ". Tasks receive their own retry logic, caching behavior, timeout settings, and concurrency controls independent of the parent flow. Tasks can be submitted for concurrent execution or mapped across datasets."]], [Para [Strong [Str "States"], Str " -- Every flow run and task run transitions through a sequence of states that describe its lifecycle: ", Code ("", [], []) "Scheduled", Str ", ", Code ("", [], []) "Pending", Str ", ", Code ("", [], []) "Running", Str ", ", Code ("", [], []) "Completed", Str ", ", Code ("", [], []) "Failed", Str ", ", Code ("", [], []) "Cancelled", Str ", ", Code ("", [], []) "Cancelling", Str ", ", Code ("", [], []) "Crashed", Str ", ", Code ("", [], []) "Paused", Str ", and ", Code ("", [], []) "Suspended", Str ". States drive orchestration decisions such as retries, notifications, and automation triggers."]], [Para [Strong [Str "Deployments"], Str " -- Server-side representations of flows that store metadata for remote orchestration including timing, execution location, parameters, and infrastructure configuration. Deployments enable scheduled runs, event-based triggers, and API-driven execution."]], [Para [Strong [Str "Work Pools"], Str " -- Infrastructure templates that define where and how flow runs execute. Work pools support Docker, Kubernetes, and serverless platforms. Push work pools run flows directly on cloud infrastructure without requiring a persistent worker process."]], [Para [Strong [Str "Workers"], Str " -- Client-side processes that poll work pools for scheduled runs and execute them on the configured infrastructure. Required for hybrid work pools (Docker, Kubernetes) but unnecessary for push work pools."]], [Para [Strong [Str "Blocks"], Str " -- Reusable configuration objects that store settings, credentials, and connection details. Blocks extend Pydantic's ", Code ("", [], []) "BaseModel", Str " and integrate with the Prefect UI for visual management. Built-in block types include secret storage, cloud storage connectors, notification channels, and database connections."]], [Para [Strong [Str "Variables"], Str " -- Key-value pairs stored in the Prefect backend for sharing configuration across workflows. Variables support dynamic deployment configuration through template syntax."]], [Para [Strong [Str "Transactions"], Str " -- Units of work that execute at most once and produce a result record at a computed cache key address. Transactions support rollback hooks, commit hooks, isolation levels, and idempotency guarantees."]], [Para [Strong [Str "Events"], Str " -- Observable occurrences within the Prefect ecosystem (state changes, deployments, custom emissions) that trigger automations. Events carry resource metadata and support pattern matching with wildcards."]], [Para [Strong [Str "Automations"], Str " -- Reactive and proactive rules that execute predefined actions when trigger conditions are met. Triggers respond to state changes, metric thresholds, custom events, or the absence of expected events."]]], Header 2 ("installation-and-setup", ["unnumbered", "unlisted"], []) [Str "Installation and Setup"], Header 3 ("install-prefect", ["unnumbered", "unlisted"], []) [Str "Install Prefect"], CodeBlock ("", ["bash"], []) "# Using pip
pip install prefect

# Using uv
uv pip install prefect

# With distributed task runner extras
pip install \"prefect[dask]\"
pip install \"prefect[ray]\"
", Header 3 ("start-the-prefect-server-self-hosted", ["unnumbered", "unlisted"], []) [Str "Start the Prefect Server (Self-Hosted)"], CodeBlock ("", ["bash"], []) "# Start the API server and UI
prefect server start

# Or use Docker
docker run -p 4200:4200 -d --rm prefecthq/prefect:3-python3.12
", Para [Str "The UI is available at ", Code ("", [], []) "http://localhost:4200", Str " after starting the server."], Header 3 ("connect-to-prefect-cloud", ["unnumbered", "unlisted"], []) [Str "Connect to Prefect Cloud"], CodeBlock ("", ["bash"], []) "# Login to Prefect Cloud
uvx prefect-cloud login

# Or set the API URL and key manually
prefect config set PREFECT_API_URL=\"https://api.prefect.cloud/api/accounts/<ACCOUNT_ID>/workspaces/<WORKSPACE_ID>\"
prefect config set PREFECT_API_KEY=\"<YOUR_API_KEY>\"
", Header 3 ("verify-installation", ["unnumbered", "unlisted"], []) [Str "Verify Installation"], CodeBlock ("", ["python"], []) "from prefect import flow

@flow
def hello():
    return \"Prefect is working!\"

hello()
", Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Para [Str "Prefect's architecture separates orchestration metadata from code execution."], CodeBlock ("", [""], []) "+---------------------------+
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
", Para [Strong [Str "API Server"], Str " -- The central coordination point that tracks flow run states, manages schedules, processes events, evaluates automation triggers, and serves the monitoring dashboard. Runs as a self-hosted instance or as Prefect Cloud (managed SaaS)."], Para [Strong [Str "Workers"], Str " -- Lightweight polling processes that run within user infrastructure. Workers check assigned work pools for scheduled flow runs and submit them for execution on the configured compute platform."], Para [Strong [Str "Push Work Pools"], Str " -- An alternative to workers for serverless environments. Push pools submit flow runs directly to cloud platforms (AWS ECS, GCR) without a persistent worker process."], Para [Strong [Str "Result Storage"], Str " -- Flow and task results persist locally (", Code ("", [], []) "~/.prefect/storage/", Str ") by default. For distributed execution, results can be stored in S3, Google Cloud Storage (GCS), Azure Blob, or any filesystem block."], Para [Strong [Str "Technology Stack"], Str " -- Python 3.9+ for the orchestration engine and SDK. TypeScript and Vue for the monitoring UI. PostgreSQL or SQLite for the API server database. The REST API uses FastAPI internally."], Header 2 ("key-features-and-functionality", ["unnumbered", "unlisted"], []) [Str "Key Features and Functionality"], Header 3 ("decorator-based-workflow-definition", ["unnumbered", "unlisted"], []) [Str "Decorator-Based Workflow Definition"], Para [Str "Prefect uses ", Code ("", [], []) "@flow", Str " and ", Code ("", [], []) "@task", Str " decorators to turn standard Python functions into orchestrated components. No DSL or configuration files are required."], CodeBlock ("", ["python"], []) "from prefect import flow, task

@task(retries=3, retry_delay_seconds=10, log_prints=True)
def extract_data(source: str) -> list[dict]:
    \"\"\"Extract records from a data source.\"\"\"
    print(f\"Extracting from {source}\")
    return [{\"id\": 1, \"value\": 42}, {\"id\": 2, \"value\": 84}]

@task(timeout_seconds=300)
def transform_data(records: list[dict]) -> list[dict]:
    \"\"\"Apply transformations to extracted records.\"\"\"
    return [{\"id\": r[\"id\"], \"value\": r[\"value\"] * 2} for r in records]

@task
def load_data(records: list[dict]) -> int:
    \"\"\"Load transformed records into the destination.\"\"\"
    print(f\"Loading {len(records)} records\")
    return len(records)

@flow(name=\"etl-pipeline\", retries=2, retry_delay_seconds=60)
def etl_pipeline(source: str = \"production_db\") -> int:
    raw = extract_data(source)
    transformed = transform_data(raw)
    count = load_data(transformed)
    return count
", Header 3 ("dynamic-task-mapping", ["unnumbered", "unlisted"], []) [Str "Dynamic Task Mapping"], Para [Str "Tasks can be mapped across iterable inputs for parallel execution. Tasks are created at runtime based on the data, not statically defined in a DAG."], CodeBlock ("", ["python"], []) "from prefect import flow, task

@task
def process_customer(customer_id: int) -> dict:
    return {\"customer_id\": customer_id, \"status\": \"processed\"}

@flow
def batch_processing(customer_ids: list[int]):
    # .map() creates one task run per item, executed concurrently
    futures = process_customer.map(customer_ids)
    results = [f.result() for f in futures]
    return results
", Header 3 ("concurrent-and-parallel-task-execution", ["unnumbered", "unlisted"], []) [Str "Concurrent and Parallel Task Execution"], Para [Str "Prefect provides multiple task runners for different execution strategies."], CodeBlock ("", ["python"], []) "from prefect import flow, task
from prefect.task_runners import ThreadPoolTaskRunner

@flow(task_runner=ThreadPoolTaskRunner(max_workers=4))
def parallel_flow():
    # Tasks submitted with .submit() run concurrently
    future_a = task_a.submit()
    future_b = task_b.submit()
    future_c = task_c.submit()
    return future_a.result(), future_b.result(), future_c.result()
", Header 3 ("caching-and-result-persistence", ["unnumbered", "unlisted"], []) [Str "Caching and Result Persistence"], Para [Str "Tasks support cache policies that prevent redundant re-execution."], CodeBlock ("", ["python"], []) "from datetime import timedelta
from prefect import flow, task
from prefect.cache_policies import INPUTS

@task(cache_policy=INPUTS, cache_expiration=timedelta(hours=1))
def expensive_computation(data: str) -> dict:
    \"\"\"Result is cached based on input parameters for 1 hour.\"\"\"
    return {\"result\": process(data)}
", Header 3 ("transactions-with-rollback", ["unnumbered", "unlisted"], []) [Str "Transactions with Rollback"], Para [Str "Transactions group tasks into atomic units with rollback capabilities."], CodeBlock ("", ["python"], []) "from prefect import flow, task
from prefect.transactions import transaction
import os

@task
def write_file(contents: str):
    with open(\"output.txt\", \"w\") as f:
        f.write(contents)

@write_file.on_rollback
def cleanup_file(txn):
    if os.path.exists(\"output.txt\"):
        os.unlink(\"output.txt\")

@task
def validate_output():
    with open(\"output.txt\") as f:
        if len(f.read()) == 0:
            raise ValueError(\"Empty output\")

@flow
def safe_pipeline(contents: str):
    with transaction():
        write_file(contents)
        validate_output()  # If this fails, write_file rolls back
", Header 3 ("event-driven-automations", ["unnumbered", "unlisted"], []) [Str "Event-Driven Automations"], Para [Str "Automations react to events and execute actions such as notifications, state changes, or deployment triggers."], CodeBlock ("", ["python"], []) "from prefect.events import emit_event

# Emit a custom event from within a flow or task
emit_event(
    event=\"data.quality.check.passed\",
    resource={\"prefect.resource.id\": \"dataset.customers\"},
    payload={\"row_count\": 15000, \"null_percentage\": 0.02},
)
", Para [Str "Automation triggers support 16 action types including cancelling flow runs, pausing deployments, sending Slack/Teams/email notifications, and calling external webhooks."], Header 3 ("secrets-and-variables", ["unnumbered", "unlisted"], []) [Str "Secrets and Variables"], CodeBlock ("", ["python"], []) "from prefect.variables import Variable
from prefect.blocks.system import Secret

# Variables for non-sensitive configuration
Variable.set(\"environment\", \"production\")
env = Variable.get(\"environment\", default=\"development\")

# Secrets for sensitive credentials (values obfuscated in logs and UI)
secret = Secret(value=\"sk-abc123...\")
secret.save(\"api-key\")

# Retrieve later
loaded = Secret.load(\"api-key\")
api_key = loaded.get()
", Header 3 ("subflows", ["unnumbered", "unlisted"], []) [Str "Subflows"], Para [Str "Flows can call other flows, creating a hierarchical execution structure."], CodeBlock ("", ["python"], []) "from prefect import flow

@flow
def child_flow(data: list[int]) -> float:
    return sum(data) / len(data)

@flow
def parent_flow():
    result_a = child_flow([1, 2, 3])
    result_b = child_flow([4, 5, 6])
    return result_a + result_b
", Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], BulletList [[Para [Strong [Str "Extract, Transform, Load (ETL) Pipelines"], Str " -- Schedule and monitor data ingestion workflows with retry logic, caching, and failure notifications. Prefect handles task dependencies, parallel extraction from multiple sources, and checkpoint-based resumption after failures."]], [Para [Strong [Str "Machine Learning (ML) Training Pipelines"], Str " -- Orchestrate data preprocessing, feature engineering, model training, evaluation, and artifact logging. Task mapping enables hyperparameter sweeps across compute clusters using Dask or Ray runners."]], [Para [Strong [Str "Data Quality Monitoring"], Str " -- Run scheduled validation checks on datasets with proactive automation triggers when expected events fail to occur (for example, an upstream pipeline does not complete within its window)."]], [Para [Strong [Str "Event-Driven Data Processing"], Str " -- Trigger flows in response to file uploads, database changes, or external webhook events. Composite triggers combine multiple conditions with temporal constraints."]], [Para [Strong [Str "CI/CD for Data Pipelines"], Str " -- Version deployments with Git commit hashes, promote flows across environments using variables and parameter overrides, and integrate with GitHub, GitLab, Bitbucket, or Azure DevOps."]], [Para [Strong [Str "Human-in-the-Loop Workflows"], Str " -- Pause flow runs to await human approval or input, then resume execution from the paused state. Useful for data annotation review, model deployment gates, or compliance sign-offs."]]], Header 2 ("api-reference-summary", ["unnumbered", "unlisted"], []) [Str "API Reference Summary"], Header 3 ("python-sdk", ["unnumbered", "unlisted"], []) [Str "Python SDK"], CodeBlock ("", ["python"], []) "from prefect import flow, task, get_client, get_run_logger
from prefect.variables import Variable
from prefect.blocks.core import Block
from prefect.blocks.system import Secret
from prefect.events import emit_event
from prefect.transactions import transaction
from prefect.cache_policies import INPUTS, TASK_SOURCE, RUN_ID, FLOW_PARAMETERS, NO_CACHE
from prefect.task_runners import ThreadPoolTaskRunner
from prefect.concurrency.sync import rate_limit
", Header 3 ("flow-decorator-parameters", ["unnumbered", "unlisted"], []) [Str "Flow Decorator Parameters"], CodeBlock ("", ["python"], []) "@flow(
    name=\"my-flow\",               # Display name (defaults to function name)
    description=\"...\",            # Description (defaults to docstring)
    retries=3,                    # Retry attempts on failure
    retry_delay_seconds=30,       # Delay between retries
    timeout_seconds=3600,         # Maximum runtime
    task_runner=ThreadPoolTaskRunner(max_workers=8),
    validate_parameters=True,     # Pydantic parameter validation
    version=\"1.0.0\",              # Version identifier
    flow_run_name=\"run-{date}\",   # Dynamic run naming
    log_prints=True,              # Capture print() as log messages
    persist_result=True,          # Enable result persistence
    result_storage=...,           # Custom result storage block
    result_serializer=\"json\",     # Result serialization format
)
", Header 3 ("task-decorator-parameters", ["unnumbered", "unlisted"], []) [Str "Task Decorator Parameters"], CodeBlock ("", ["python"], []) "@task(
    name=\"my-task\",               # Display name
    description=\"...\",            # Description
    retries=3,                    # Retry attempts
    retry_delay_seconds=10,       # Delay between retries
    timeout_seconds=300,          # Maximum runtime
    cache_policy=INPUTS,          # Cache key generation policy
    cache_expiration=timedelta(hours=1),
    cache_key_fn=custom_fn,       # Custom cache key function
    tags={\"etl\", \"production\"},   # Tags for filtering and monitoring
    task_run_name=\"process-{id}\", # Dynamic run naming
    log_prints=True,              # Capture print() as log messages
    persist_result=True,          # Enable result persistence
    result_storage_key=\"{parameters[name]}.json\",
)
", Header 3 ("rest-api-client", ["unnumbered", "unlisted"], []) [Str "REST API Client"], CodeBlock ("", ["python"], []) "from prefect import get_client

# Async usage (default)
async with get_client() as client:
    runs = await client.read_flow_runs()
    await client.create_flow_run_from_deployment(deployment_id=\"...\")
    await client.set_flow_run_state(flow_run_id=\"...\", state=...)
    events = await client.read_events(filter=...)

# Synchronous usage
with get_client(sync_client=True) as client:
    runs = client.read_flow_runs()
", Header 3 ("rest-api-endpoints", ["unnumbered", "unlisted"], []) [Str "REST API Endpoints"], Para [Str "The API base URL follows the pattern ", Code ("", [], []) "https://api.prefect.cloud/api/accounts/{account_id}/workspaces/{workspace_id}", Str " for Cloud, or ", Code ("", [], []) "http://localhost:4200/api", Str " for self-hosted. Authentication uses bearer tokens via the ", Code ("", [], []) "Authorization", Str " header."], Header 2 ("configuration-and-customization", ["unnumbered", "unlisted"], []) [Str "Configuration and Customization"], Header 3 ("environment-variables", ["unnumbered", "unlisted"], []) [Str "Environment Variables"], CodeBlock ("", ["bash"], []) "# API connection
PREFECT_API_URL=\"http://localhost:4200/api\"
PREFECT_API_KEY=\"pnu_...\"

# Result persistence
PREFECT_RESULTS_PERSIST_BY_DEFAULT=true
PREFECT_LOCAL_STORAGE_PATH=\"/custom/storage/path\"
PREFECT_DEFAULT_RESULT_STORAGE_BLOCK=\"s3-bucket/my-bucket\"
PREFECT_RESULTS_DEFAULT_SERIALIZER=\"json\"

# Task defaults
PREFECT_TASKS_DEFAULT_PERSIST_RESULT=true

# Logging
PREFECT_LOGGING_LEVEL=\"DEBUG\"

# Server
PREFECT_SERVER_API_HOST=\"0.0.0.0\"
PREFECT_SERVER_API_PORT=\"4200\"
", Header 3 ("profiles", ["unnumbered", "unlisted"], []) [Str "Profiles"], Para [Str "Prefect supports named configuration profiles for switching between environments."], CodeBlock ("", ["bash"], []) "# Create a profile
prefect profile create production
prefect profile use production
prefect config set PREFECT_API_URL=\"https://api.prefect.cloud/api/...\"

# Switch back
prefect profile use default
", Header 3 ("custom-blocks", ["unnumbered", "unlisted"], []) [Str "Custom Blocks"], CodeBlock ("", ["python"], []) "from prefect.blocks.core import Block
from pydantic import SecretStr

class DatabaseCredentials(Block):
    _block_type_name = \"Database Credentials\"
    _description = \"Credentials for database connections\"

    host: str
    port: int = 5432
    username: str
    password: SecretStr
    database: str

    def connection_string(self) -> str:
        return f\"postgresql://{self.username}:{self.password.get_secret_value()}@{self.host}:{self.port}/{self.database}\"

# Register and save
DatabaseCredentials.register_type_and_schema()
creds = DatabaseCredentials(host=\"db.example.com\", username=\"app\", password=\"secret\", database=\"prod\")
creds.save(\"production-db\")

# Load elsewhere
db = DatabaseCredentials.load(\"production-db\")
", Header 3 ("deployment-configuration-prefectyaml", ["unnumbered", "unlisted"], []) [Str "Deployment Configuration (prefect.yaml)"], CodeBlock ("", ["yaml"], []) "deployments:
  - name: daily-etl
    entrypoint: flows/etl.py:etl_pipeline
    work_pool:
      name: my-docker-pool
    schedule:
      cron: \"0 8 * * *\"
      timezone: \"America/New_York\"
    parameters:
      source: \"production_db\"
    tags:
      - production
      - etl
    concurrency_limit: 1
    version: \"{{ prefect.variables.deployment_version }}\"
", Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Header 3 ("prefect-with-dask-for-distributed-execution", ["unnumbered", "unlisted"], []) [Str "Prefect with Dask for Distributed Execution"], CodeBlock ("", ["python"], []) "from prefect import flow, task
from prefect.task_runners import DaskTaskRunner

@task
def train_model(hyperparams: dict) -> float:
    # Training logic here
    return accuracy

@flow(task_runner=DaskTaskRunner(address=\"tcp://scheduler:8786\"))
def hyperparameter_search(param_grid: list[dict]):
    futures = train_model.map(param_grid)
    results = [f.result() for f in futures]
    best = max(results)
    return best
", Header 3 ("prefect-with-ray-for-parallel-ml-workloads", ["unnumbered", "unlisted"], []) [Str "Prefect with Ray for Parallel ML Workloads"], CodeBlock ("", ["python"], []) "from prefect import flow, task
from prefect.task_runners import RayTaskRunner

@task
def process_partition(partition_id: int) -> dict:
    return {\"partition\": partition_id, \"rows_processed\": 10000}

@flow(task_runner=RayTaskRunner())
def distributed_processing(partitions: list[int]):
    futures = process_partition.map(partitions)
    return [f.result() for f in futures]
", Header 3 ("prefect-with-docker-work-pool", ["unnumbered", "unlisted"], []) [Str "Prefect with Docker Work Pool"], CodeBlock ("", ["bash"], []) "# Create a Docker work pool
prefect work-pool create --type docker my-docker-pool

# Start a worker for the pool
prefect worker start --pool my-docker-pool
", CodeBlock ("", ["python"], []) "from prefect import flow

@flow(log_prints=True)
def containerized_flow():
    print(\"Running inside Docker\")

if __name__ == \"__main__\":
    containerized_flow.deploy(
        name=\"docker-deployment\",
        work_pool_name=\"my-docker-pool\",
        image=\"my-registry/my-flow:latest\",
        cron=\"0 */6 * * *\",
    )
", Header 3 ("prefect-with-kubernetes", ["unnumbered", "unlisted"], []) [Str "Prefect with Kubernetes"], CodeBlock ("", ["bash"], []) "# Create a Kubernetes work pool
prefect work-pool create --type kubernetes my-k8s-pool

# Start a worker
prefect worker start --pool my-k8s-pool
", Header 3 ("webhook-triggered-flows", ["unnumbered", "unlisted"], []) [Str "Webhook-Triggered Flows"], Para [Str "Prefect Cloud supports webhook endpoints that emit events, which can trigger automations to start flow runs. For self-hosted setups, the REST API enables programmatic flow run creation."], CodeBlock ("", ["python"], []) "from prefect import get_client

async def trigger_flow_from_webhook(deployment_id: str, params: dict):
    async with get_client() as client:
        await client.create_flow_run_from_deployment(
            deployment_id=deployment_id,
            parameters=params,
        )
", Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Header 3 ("complete-etl-pipeline-with-error-handling", ["unnumbered", "unlisted"], []) [Str "Complete ETL Pipeline with Error Handling"], CodeBlock ("", ["python"], []) "from prefect import flow, task, get_run_logger
from prefect.transactions import transaction
from datetime import timedelta
from prefect.cache_policies import INPUTS

@task(retries=3, retry_delay_seconds=[10, 30, 60], log_prints=True)
def fetch_api_data(endpoint: str) -> list[dict]:
    logger = get_run_logger()
    logger.info(f\"Fetching data from {endpoint}\")
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
    batch_id = txn.get(\"batch_id\")
    # Delete the batch from the database
    print(f\"Rolling back batch {batch_id}\")

@flow(name=\"api-etl\", retries=1, timeout_seconds=1800)
def api_etl_pipeline(endpoint: str, dedup_key: str = \"id\"):
    with transaction():
        raw = fetch_api_data(endpoint)
        cleaned = deduplicate(raw, dedup_key)
        count = write_to_database(cleaned)
    return count
", Header 3 ("scheduled-deployment-with-parameterization", ["unnumbered", "unlisted"], []) [Str "Scheduled Deployment with Parameterization"], CodeBlock ("", ["python"], []) "from prefect import flow

@flow(log_prints=True)
def data_sync(source: str, target: str, batch_size: int = 1000):
    print(f\"Syncing {source} -> {target} in batches of {batch_size}\")
    # Sync logic here

if __name__ == \"__main__\":
    data_sync.serve(
        name=\"hourly-sync\",
        cron=\"0 * * * *\",
        parameters={\"source\": \"warehouse\", \"target\": \"analytics\", \"batch_size\": 5000},
        tags=[\"sync\", \"production\"],
    )
", Header 3 ("async-flow-with-concurrent-tasks", ["unnumbered", "unlisted"], []) [Str "Async Flow with Concurrent Tasks"], CodeBlock ("", ["python"], []) "import asyncio
from prefect import flow, task

@task
async def fetch_user(user_id: int) -> dict:
    await asyncio.sleep(0.1)  # Simulate API call
    return {\"id\": user_id, \"name\": f\"User {user_id}\"}

@task
async def enrich_user(user: dict) -> dict:
    await asyncio.sleep(0.05)  # Simulate enrichment
    user[\"enriched\"] = True
    return user

@flow
async def user_pipeline(user_ids: list[int]):
    users = await asyncio.gather(*[fetch_user(uid) for uid in user_ids])
    enriched = await asyncio.gather(*[enrich_user(u) for u in users])
    return enriched
", Header 3 ("using-variables-for-environment-specific-configuration", ["unnumbered", "unlisted"], []) [Str "Using Variables for Environment-Specific Configuration"], CodeBlock ("", ["python"], []) "from prefect import flow
from prefect.variables import Variable

@flow
def configurable_pipeline():
    db_host = Variable.get(\"db_host\", default=\"localhost\")
    batch_size = Variable.get(\"batch_size\", default=100)
    print(f\"Connecting to {db_host} with batch size {batch_size}\")
    # Pipeline logic using runtime configuration
", Header 2 ("limitations-and-considerations", ["unnumbered", "unlisted"], []) [Str "Limitations and Considerations"], BulletList [[Para [Strong [Str "Python-Only SDK"], Str " -- Prefect flows and tasks must be written in Python. Workflows requiring Go, Java, or other languages need wrapper tasks that shell out to external processes."]], [Para [Strong [Str "Sync Task Timeout Limitations"], Str " -- When using the default ", Code ("", [], []) "ThreadPoolTaskRunner", Str ", timeouts cannot interrupt blocking synchronous operations such as ", Code ("", [], []) "time.sleep()", Str " or long network requests. Use ", Code ("", [], []) "ProcessPoolTaskRunner", Str ", async tasks, or libraries with native timeout parameters for reliable interruption."]], [Para [Strong [Str "Metric Triggers Require Cloud"], Str " -- Metric-based automation triggers (average duration, lateness, completion percentage) are available only in Prefect Cloud, not in the open-source self-hosted server."]], [Para [Strong [Str "Result Serialization Constraints"], Str " -- Task results must be serializable (pickle or JSON). Large objects, open file handles, database connections, and other non-serializable values require explicit handling or custom serializers."]], [Para [Strong [Str "Worker Infrastructure"], Str " -- Hybrid work pools (Docker, Kubernetes) require running a persistent worker process within the user's infrastructure. This adds operational overhead compared to push work pools or the simpler ", Code ("", [], []) "flow.serve()", Str " pattern."]], [Para [Strong [Str "Database Backend"], Str " -- The self-hosted Prefect server defaults to SQLite, which is suitable for development but not production workloads. Production deployments should use PostgreSQL."]], [Para [Strong [Str "State Explosion with Large Map Operations"], Str " -- Task mapping across very large iterables creates a flow run state entry per item. Thousands of mapped tasks can impact API server performance and UI responsiveness."]], [Para [Strong [Str "Single-Threaded Event Loop"], Str " -- The Prefect orchestration engine uses a single-threaded async event loop. CPU-intensive orchestration logic (complex automation evaluations, large parameter validation) can block the event loop."]]], Header 2 ("changelog-highlights", ["unnumbered", "unlisted"], []) [Str "Changelog Highlights"], BulletList [[Para [Strong [Str "2018-2021"], Str " -- Initial release of Prefect as a Pythonic workflow orchestration framework. Introduced the core ", Code ("", [], []) "@flow", Str " and ", Code ("", [], []) "@task", Str " decorator model with a DAG-based execution engine."]], [Para [Strong [Str "2022 (Prefect 2.0)"], Str " -- Major rewrite removing the DAG-only constraint. Flows became native Python functions with dynamic task creation at runtime. Introduced work pools, blocks, and the hybrid execution model. Replaced the Prefect Server with a new API-first architecture."]], [Para [Strong [Str "2024 (Prefect 3.0)"], Str " -- Open-sourced events and automations (previously Cloud-only). Achieved a 90% performance improvement in the orchestration engine. Introduced transactions with rollback and commit hooks. Added ", Code ("", [], []) "ProcessPoolTaskRunner", Str " for true parallelism. Enhanced caching with configurable cache policies (", Code ("", [], []) "INPUTS", Str ", ", Code ("", [], []) "TASK_SOURCE", Str ", ", Code ("", [], []) "RUN_ID", Str ", ", Code ("", [], []) "FLOW_PARAMETERS", Str ", ", Code ("", [], []) "NO_CACHE", Str ")."]], [Para [Strong [Str "2025-2026 (3.x Series)"], Str " -- Continued iteration on the 3.x line with the latest release being v3.6.18 (February 2026). Ongoing improvements to concurrency controls, result persistence, client API ergonomics, and deployment workflows."]]], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "Prefect Official Documentation -- ", Link ("", [], []) [Str "https://docs.prefect.io/"] ("https://docs.prefect.io/", "")]], [Plain [Str "Prefect GitHub Repository -- ", Link ("", [], []) [Str "https://github.com/PrefectHQ/prefect"] ("https://github.com/PrefectHQ/prefect", "")]], [Plain [Str "Prefect 3.0 Release Announcement -- ", Link ("", [], []) [Str "https://www.prefect.io/blog/prefect-3-is-here"] ("https://www.prefect.io/blog/prefect-3-is-here", "")]], [Plain [Str "Prefect Cloud Documentation -- ", Link ("", [], []) [Str "https://docs.prefect.io/v3/manage/cloud"] ("https://docs.prefect.io/v3/manage/cloud", "")]], [Plain [Str "Prefect REST API Reference -- ", Link ("", [], []) [Str "https://docs.prefect.io/v3/api-ref/rest-api"] ("https://docs.prefect.io/v3/api-ref/rest-api", "")]], [Plain [Str "Prefect Community Slack -- ", Link ("", [], []) [Str "https://prefect.io/slack"] ("https://prefect.io/slack", "")]]]]