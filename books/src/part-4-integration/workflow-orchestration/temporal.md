# Temporal

> Durable execution platform for reliable distributed systems with automatic failure handling

| Field | Value |
|-------|-------|
| Group | Workflow Orchestration & Automation |
| Type | SDK |
| Open Source | Yes |
| GitHub | [https://github.com/temporalio/temporal](https://github.com/temporalio/temporal) |
| Stars | 18451 |
| Documentation | [Official Docs](https://docs.temporal.io/) |

## Overview

Temporal is an open-source durable execution platform that enables developers to build reliable distributed applications with crash-proof execution guarantees. Applications running on Temporal resume exactly where they left off after crashes, network failures, or infrastructure outages, whether the interruption lasted seconds, days, or years. The platform originated from Uber's Cadence project and is maintained by Temporal Technologies under the MIT license.

Rather than requiring developers to write defensive retry logic, error-handling boilerplate, and state-recovery mechanisms, Temporal shifts that burden to its server infrastructure. Developers write business logic as Workflows and Activities using familiar programming constructs in their language of choice, and the platform handles fault tolerance, state persistence, and automatic retries transparently. This approach is particularly valuable for mission-critical processes such as order fulfillment, customer onboarding, payment processing, and long-running data pipelines.

Temporal provides official Software Development Kits (SDKs) for Go, Java, PHP, Python, Ruby, TypeScript, and .NET, allowing teams to adopt the platform regardless of their primary language. Deployment options include self-hosted infrastructure and Temporal Cloud, a fully managed service offering USD 1,000 in free credits for new accounts.

## Core Concepts

**Workflow** -- A durable function that orchestrates the execution of business logic. Workflows are deterministic and can run for arbitrarily long durations. The Temporal Server persists Workflow state so that execution survives process restarts and infrastructure failures. Workflow code must remain deterministic: no threading, randomness, direct network input/output (I/O), global state mutation, or system clock access.

**Activity** -- A single unit of work that performs a business operation, typically involving side effects such as database writes, Application Programming Interface (API) calls, or file system operations. Activities are called from within Workflows and can be retried independently on failure.

**Worker** -- A process that hosts Workflow and Activity implementations and polls a Task Queue for work. Workers execute the actual code; the Temporal Server itself only orchestrates execution and persists state.

**Task Queue** -- A named queue that the Temporal Server uses to dispatch Workflow Tasks and Activity Tasks to Workers. Workers register against specific Task Queues, and multiple Workers can poll the same queue for horizontal scalability.

**Namespace** -- A logical isolation unit within the Temporal Server. Each Namespace has its own set of Workflows, Task Queues, and configuration. Namespaces provide multi-tenancy support for separating environments or teams.

**Workflow Execution** -- A single invocation of a Workflow with a unique Workflow Identifier (ID). Each execution maintains its own Event History, which is an append-only log of all events that occurred during the execution.

**Signal** -- An asynchronous message sent to a running Workflow to change its state or control its flow in real time. Signal handlers can execute Activities and Child Workflows.

**Query** -- A synchronous read-only request to inspect a running Workflow's internal state. Query handlers must not modify state or perform asynchronous operations.

**Update** -- A synchronous request sent to a running Workflow that can modify state and return a result. Updates support validation before execution.

**Retry Policy** -- Configuration that controls how Activities and Workflows are retried on failure, including initial interval, backoff coefficient, maximum interval, and maximum attempts.

## Installation and Setup

### Command-Line Interface (CLI) Installation

Install the Temporal CLI to run a local development server:

```bash
# macOS via Homebrew
brew install temporal

# Start the local development server
temporal server start-dev
```

The development server starts on port 7233 by default, with a Web User Interface (UI) accessible at `http://localhost:8233`.

### Python SDK Installation

```bash
pip install temporalio
```

### Go SDK Installation

```bash
go get go.temporal.io/sdk
```

### TypeScript SDK Installation

```bash
npm install @temporalio/client @temporalio/worker @temporalio/workflow @temporalio/activity
```

### Docker Compose (Self-Hosted)

For a production-like local setup, Temporal provides Docker Compose configurations:

```bash
git clone https://github.com/temporalio/docker-compose.git
cd docker-compose
docker compose up
```

This starts the Temporal Server along with its dependencies (PostgreSQL or Cassandra for persistence, Elasticsearch for visibility).

## Architecture

The Temporal Server consists of four independently scalable services that communicate through a membership protocol using Ringpop:

**Frontend Service** -- The gateway that handles rate limiting, routing, authorization, and API request validation. The Frontend Service is stateless and does not use sharding or partitioning.

**History Service** -- Maintains Workflow execution data including mutable state, queues, and timers. This is typically the most resource-intensive service and scales horizontally through sharding.

**Matching Service** -- Hosts Task Queues and dispatches tasks to appropriate Workers. It manages the assignment of Workflow Tasks and Activity Tasks to polling Workers.

**Worker Service** -- Runs internal background Workflows that the Temporal Server itself needs for system operations such as archival and replication.

A typical production deployment scales each service independently based on load characteristics. For example, a deployment might run 5 Frontend instances, 15 History instances, 17 Matching instances, and 3 Worker Service instances.

```text
+------------------+       +-------------------+
|   Client App     |       |   Worker Process  |
| (starts workflows|       | (executes code)   |
|  sends signals)  |       |                   |
+--------+---------+       +---------+---------+
         |                           |
         | gRPC                      | gRPC (polls)
         |                           |
+--------v---------------------------v---------+
|              Frontend Service                |
|         (rate limiting, routing)             |
+-----------+----------------+-----------------+
            |                |
   +--------v------+  +-----v--------+
   | History       |  | Matching     |
   | Service       |  | Service      |
   | (state, timers|  | (task queues,|
   |  event log)   |  |  dispatch)   |
   +--------+------+  +--------------+
            |
   +--------v------+
   | Persistence   |
   | (PostgreSQL,  |
   |  Cassandra,   |
   |  MySQL)       |
   +---------------+
```

Workers are external to the Temporal Server. They run within the application's infrastructure and communicate with the server over gRPC (Google Remote Procedure Call). This separation means that application code and its dependencies never run on the Temporal Server itself.

## Key Features and Functionality

### Durable Execution

Temporal persists the complete Event History of every Workflow Execution. If a Worker crashes mid-execution, the Workflow replays its history on another Worker and resumes from the exact point of failure. No application-level checkpointing is required.

### Automatic Failure Handling and Retries

Activities support configurable Retry Policies with exponential backoff:

```python
from datetime import timedelta
from temporalio.common import RetryPolicy

retry_policy = RetryPolicy(
    initial_interval=timedelta(seconds=1),
    backoff_coefficient=2.0,
    maximum_interval=timedelta(minutes=1),
    maximum_attempts=5,
)
```

### Activity Heartbeating

Long-running Activities can report progress via heartbeats. If a Worker crashes, the Activity is rescheduled on another Worker with the last heartbeat details, allowing it to resume from the last checkpoint rather than starting over.

### Child Workflows

Workflows can spawn Child Workflows for hierarchical decomposition of complex processes. Child Workflows have their own Event History and can be independently cancelled, terminated, or queried.

### Continue-As-New

For Workflows that accumulate large Event Histories (for example, long-running polling loops), Continue-As-New completes the current execution and starts a new one with a fresh history, passing forward any necessary state.

### Durable Timers

Workflows can sleep for arbitrary durations (seconds to years) using `workflow.sleep()`. These timers are persisted by the server and survive Worker restarts and outages.

### Cancellation and Termination

Workflows support graceful cancellation (which raises a cancellation error that the Workflow can catch and handle) and forced termination (which immediately stops execution).

### Workflow Reset

Completed or failed Workflow Executions can be reset to a specific point in their Event History and re-executed from there.

### Versioning

The Temporal SDKs provide patching and versioning mechanisms to safely deploy Workflow code changes without breaking in-progress executions. This allows gradual migration of running Workflows to new code paths.

### Observability

Temporal provides metrics, distributed tracing integration, and structured logging. The Temporal Web UI displays Workflow status, Event History, pending Activities, and allows manual operations such as signaling and termination.

### Data Encryption

Custom Data Converters and Payload Codecs enable encryption of all data at rest and in transit:

```typescript
const client = new Client({
  connection,
  dataConverter: await getDataConverter(),
});

const handle = await client.workflow.start(example, {
  args: ["Alice: Private message for Bob."],
  taskQueue: "encryption",
  workflowId: `my-business-id-${uuid()}`,
});
```

## Use Cases

- **Order Fulfillment** -- Coordinate inventory checks, payment processing, shipping, and notifications across multiple services with guaranteed completion.
- **User Onboarding** -- Orchestrate multi-step registration flows including email verification, profile setup, and third-party integrations that may span hours or days.
- **Payment Processing** -- Manage payment authorization, capture, settlement, and refund flows with automatic retry and idempotency guarantees.
- **Data Pipelines** -- Orchestrate Extract, Transform, Load (ETL) pipelines with per-step retry, timeout management, and progress tracking.
- **Subscription Management** -- Handle recurring billing cycles, trial expirations, plan upgrades, and cancellation flows over months or years.
- **Infrastructure Provisioning** -- Coordinate multi-step cloud resource provisioning with rollback capabilities on partial failures.
- **Machine Learning Pipelines** -- Manage long-running model training, evaluation, and deployment workflows with checkpointing and resource cleanup.
- **Human-in-the-Loop Processes** -- Workflows that pause waiting for human approval signals before proceeding, with durable timers for escalation deadlines.

## API Reference Summary

### Python SDK Decorators and Methods

| Decorator / Method | Purpose |
|---|---|
| `@workflow.defn` | Marks a class as a Workflow definition |
| `@workflow.run` | Marks the entry point method of a Workflow |
| `@workflow.signal` | Registers a method as a Signal handler |
| `@workflow.query` | Registers a method as a Query handler |
| `@workflow.update` | Registers a method as an Update handler |
| `@activity.defn` | Marks a function as an Activity definition |
| `workflow.execute_activity()` | Executes an Activity and awaits the result |
| `workflow.start_activity()` | Starts an Activity and returns a handle |
| `workflow.execute_child_workflow()` | Executes a Child Workflow and awaits the result |
| `workflow.start_child_workflow()` | Starts a Child Workflow and returns a handle |
| `workflow.sleep()` | Creates a durable timer |
| `workflow.wait_condition()` | Blocks until a condition is met |
| `Client.connect()` | Connects to the Temporal Server |
| `client.execute_workflow()` | Starts a Workflow and awaits the result |
| `client.start_workflow()` | Starts a Workflow and returns a handle |
| `handle.signal()` | Sends a Signal to a running Workflow |
| `handle.query()` | Queries a running Workflow's state |
| `handle.execute_update()` | Sends an Update and awaits the result |
| `handle.cancel()` | Requests cancellation of a Workflow |
| `handle.terminate()` | Terminates a Workflow immediately |
| `handle.result()` | Awaits the final result of a Workflow |

### Timeout Types

| Timeout | Scope | Description |
|---|---|---|
| Workflow Execution Timeout | Workflow | Maximum wall-clock duration for the entire Workflow Execution including retries |
| Workflow Run Timeout | Workflow | Maximum wall-clock duration for a single Workflow run |
| Workflow Task Timeout | Workflow | Maximum time for a Worker to process a single Workflow Task |
| Schedule-To-Close Timeout | Activity | Maximum time from scheduling to completion |
| Start-To-Close Timeout | Activity | Maximum time from start to completion on a Worker |
| Schedule-To-Start Timeout | Activity | Maximum time waiting in a Task Queue |
| Heartbeat Timeout | Activity | Maximum interval between heartbeats before the Activity is considered failed |

## Configuration and Customization

### Worker Configuration

```python
from temporalio.client import Client
from temporalio.worker import Worker

client = await Client.connect("localhost:7233")

worker = Worker(
    client,
    task_queue="my-task-queue",
    workflows=[MyWorkflow],
    activities=[my_activity],
    max_concurrent_activities=100,
    max_concurrent_workflow_tasks=100,
)
```

### Environment Variables

Common environment variables for connecting Workers to the Temporal Server:

```python
import os

TEMPORAL_ADDRESS = os.environ.get("TEMPORAL_ADDRESS", "localhost:7233")
TEMPORAL_NAMESPACE = os.environ.get("TEMPORAL_NAMESPACE", "default")
TEMPORAL_TASK_QUEUE = os.environ.get("TEMPORAL_TASK_QUEUE", "my-task-queue")
TEMPORAL_API_KEY = os.environ.get("TEMPORAL_API_KEY", "")
```

### Temporal Cloud Configuration

```bash
export TEMPORAL_PROFILE=cloud
temporal config set --profile cloud --prop address --value "<endpoint>"
temporal config set --profile cloud --prop namespace --value "<namespace>"
temporal config set --profile cloud --prop api_key --value "<api-key>"
```

### Server Deployment Configuration

For self-hosted deployments, individual services can be configured and scaled independently:

```bash
docker run \
    -e SERVICES=history \
    -e LOG_LEVEL=debug,info \
    -e DYNAMIC_CONFIG_FILE_PATH=config/dynamic_config.yaml \
    temporalio/server:1.29.3
```

## Integration Patterns

### Saga Pattern

Temporal naturally implements the Saga pattern for distributed transactions. Each step in a saga is an Activity, and compensating Activities are executed on failure:

```python
@workflow.defn
class OrderSagaWorkflow:
    @workflow.run
    async def run(self, order: OrderInput) -> str:
        compensations = []
        try:
            await workflow.execute_activity(
                reserve_inventory, order,
                start_to_close_timeout=timedelta(seconds=30),
            )
            compensations.append(release_inventory)

            await workflow.execute_activity(
                charge_payment, order,
                start_to_close_timeout=timedelta(seconds=30),
            )
            compensations.append(refund_payment)

            await workflow.execute_activity(
                ship_order, order,
                start_to_close_timeout=timedelta(seconds=60),
            )
            return "Order completed"
        except Exception:
            for compensation in reversed(compensations):
                await workflow.execute_activity(
                    compensation, order,
                    start_to_close_timeout=timedelta(seconds=30),
                )
            raise
```

### Human-in-the-Loop Approval

Workflows can wait for external Signals, enabling approval flows:

```python
@workflow.defn
class ApprovalWorkflow:
    def __init__(self):
        self.approved = False

    @workflow.run
    async def run(self, request: ApprovalRequest) -> str:
        await workflow.execute_activity(
            send_approval_email, request,
            start_to_close_timeout=timedelta(seconds=30),
        )
        # Wait up to 7 days for approval signal
        try:
            await workflow.wait_condition(
                lambda: self.approved, timeout=timedelta(days=7)
            )
            return "Approved"
        except asyncio.TimeoutError:
            return "Timed out waiting for approval"

    @workflow.signal
    def approve(self) -> None:
        self.approved = True
```

### Polling Pattern with Continue-As-New

For long-running polling Workflows, use Continue-As-New to prevent unbounded Event History growth:

```python
@workflow.defn
class PollingWorkflow:
    @workflow.run
    async def run(self, state: PollingState) -> None:
        result = await workflow.execute_activity(
            check_external_system, state,
            start_to_close_timeout=timedelta(seconds=30),
        )
        if result.is_complete:
            return
        state.iteration += 1
        await workflow.sleep(timedelta(minutes=5))
        workflow.continue_as_new(state)
```

### Microservice Orchestration

Temporal serves as a central orchestrator that coordinates calls across multiple microservices, replacing complex choreography with explicit, visible orchestration logic.

## Examples

### Complete Python Application

**Define an Activity:**

```python
from dataclasses import dataclass
from temporalio import activity

@dataclass
class GreetingInput:
    greeting: str
    name: str

@activity.defn
async def compose_greeting(input: GreetingInput) -> str:
    return f"{input.greeting}, {input.name}!"
```

**Define a Workflow:**

```python
from datetime import timedelta
from temporalio import workflow
from temporalio.common import RetryPolicy

with workflow.unsafe.imports_passed_through():
    from activities import compose_greeting, GreetingInput

@workflow.defn
class GreetingWorkflow:
    @workflow.run
    async def run(self, name: str) -> str:
        return await workflow.execute_activity(
            compose_greeting,
            GreetingInput("Hello", name),
            start_to_close_timeout=timedelta(seconds=10),
            retry_policy=RetryPolicy(maximum_attempts=3),
        )
```

**Run the Worker:**

```python
import asyncio
from temporalio.client import Client
from temporalio.worker import Worker

async def main():
    client = await Client.connect("localhost:7233")
    worker = Worker(
        client,
        task_queue="greeting-task-queue",
        workflows=[GreetingWorkflow],
        activities=[compose_greeting],
    )
    await worker.run()

if __name__ == "__main__":
    asyncio.run(main())
```

**Start the Workflow from a Client:**

```python
import asyncio
from temporalio.client import Client

async def main():
    client = await Client.connect("localhost:7233")
    result = await client.execute_workflow(
        GreetingWorkflow.run,
        "World",
        id="greeting-workflow-001",
        task_queue="greeting-task-queue",
    )
    print(f"Workflow result: {result}")

if __name__ == "__main__":
    asyncio.run(main())
```

### Message Passing Example

```python
from dataclasses import dataclass, field
from temporalio import workflow

@dataclass
class Language:
    name: str

@workflow.defn
class GreetingBotWorkflow:
    def __init__(self):
        self.language = Language("English")
        self.approved_for_release = False
        self.greetings = {"English": "Hello", "Spanish": "Hola", "French": "Bonjour"}

    @workflow.run
    async def run(self) -> str:
        await workflow.wait_condition(lambda: self.approved_for_release)
        return f"Released with language: {self.language.name}"

    @workflow.signal
    def approve(self) -> None:
        self.approved_for_release = True

    @workflow.query
    def get_language(self) -> str:
        return self.language.name

    @workflow.update
    def set_language(self, language: Language) -> Language:
        previous = self.language
        self.language = language
        return previous

    @set_language.validator
    def validate_language(self, language: Language) -> None:
        if language.name not in self.greetings:
            raise ValueError(f"{language.name} is not supported")
```

### Go SDK Workflow with Retry Policy

```go
func LoanApplicationWorkflow(ctx workflow.Context, applicantName string, loanAmount int) (string, error) {
    ao := workflow.ActivityOptions{
        ScheduleToCloseTimeout: time.Hour,
        HeartbeatTimeout:       time.Minute,
        RetryPolicy: &temporal.RetryPolicy{
            InitialInterval:    time.Second,
            BackoffCoefficient: 2,
            MaximumInterval:    time.Minute,
            MaximumAttempts:    5,
        },
    }
    ctx = workflow.WithActivityOptions(ctx, ao)

    var creditCheckResult string
    err := workflow.ExecuteActivity(ctx, LoanCreditCheckActivity, loanAmount).Get(ctx, &creditCheckResult)
    if err != nil {
        return "", err
    }
    return creditCheckResult, nil
}
```

### Testing Workflows (Python)

**Activity Testing with Heartbeats:**

```python
from temporalio.testing import ActivityEnvironment

async def test_activity_heartbeats():
    env = ActivityEnvironment()
    heartbeats = []
    env.on_heartbeat = lambda *args: heartbeats.append(args[0])
    result = await env.run(my_activity, "test-input")
    assert heartbeats == ["step-1", "step-2"]
    assert result == "expected-output"
```

**Workflow Testing with Time Skipping:**

```python
from temporalio.testing import WorkflowEnvironment
from temporalio.worker import Worker

async def test_long_running_workflow():
    async with await WorkflowEnvironment.start_time_skipping() as env:
        async with Worker(
            env.client,
            task_queue="test-queue",
            workflows=[MyWorkflow],
            activities=[my_activity],
        ):
            result = await env.client.execute_workflow(
                MyWorkflow.run,
                "input",
                id="test-workflow",
                task_queue="test-queue",
            )
            assert result == "expected"
```

**Workflow Replay Testing for Determinism Validation:**

```python
from temporalio.worker import Replayer

async def test_workflow_replay():
    replayer = Replayer(workflows=[MyWorkflow])
    await replayer.replay_workflows(histories)
```

## Limitations and Considerations

- **Event History Size** -- Each Workflow Execution has an Event History size limit (approximately 50,000 events by default). Long-running Workflows must use Continue-As-New to keep the history bounded.
- **Determinism Requirement** -- Workflow code must be deterministic. No direct I/O, threading, random number generation, or system clock access is permitted inside Workflow definitions. This constraint requires a mental model shift for developers.
- **Payload Size Limits** -- Individual Activity arguments are limited to 2 Megabytes (MB) per argument, with a 4 MB total message size. Large data must be passed by reference (for example, using object storage Uniform Resource Locators (URLs)).
- **Operational Complexity** -- Self-hosted deployments require managing the Temporal Server (four services), a persistence backend (PostgreSQL, MySQL, or Cassandra), and optionally Elasticsearch for advanced visibility. This adds significant infrastructure overhead.
- **Worker Registration Consistency** -- All Workers polling the same Task Queue must register identical Workflow and Activity types. Inconsistent registration can cause task failures.
- **Versioning Discipline** -- Changing Workflow logic requires careful use of versioning or patching APIs to avoid breaking in-progress executions. Incorrect changes can cause non-determinism errors.
- **Learning Curve** -- The determinism requirements, Event History model, and distinction between Workflows and Activities require meaningful investment to learn properly.
- **Cold Start Latency** -- Workflow replay (rebuilding state from Event History) introduces latency proportional to history length when a Workflow is loaded onto a new Worker.

## Changelog Highlights

- **v1.29.3** (February 2026) -- Latest stable release with bug fixes and stability improvements.
- **v1.25+** -- Introduction of Workflow Updates for synchronous, trackable interactions with running Workflows.
- **v1.20+** -- Enhanced multi-cluster replication and improved Namespace management.
- **v1.17+** -- Schedule support for cron-like recurring Workflow executions managed server-side.
- **v1.0** -- Initial stable release establishing the core Workflow, Activity, and Worker model with full durability guarantees.

The project maintains an active release cadence with 152 total releases as of February 2026, contributed to by over 253 contributors across 8,541 commits.

## Citations

- [Temporal Official Documentation](https://docs.temporal.io/)
- [Temporal GitHub Repository](https://github.com/temporalio/temporal)
- [Temporal Python SDK Documentation](https://docs.temporal.io/develop/python/core-application)
- [Temporal Python SDK Message Passing](https://docs.temporal.io/develop/python/message-passing)
- [Temporal Python SDK Testing Suite](https://docs.temporal.io/develop/python/testing-suite)
- [Temporal Server Architecture](https://docs.temporal.io/temporal-service/temporal-server)
- [Temporal Self-Hosted Deployment Guide](https://docs.temporal.io/self-hosted-guide/deployment)
- [Temporal Converters and Encryption](https://docs.temporal.io/develop/typescript/converters-and-encryption)
- [Temporal Cloud](https://temporal.io/cloud)
