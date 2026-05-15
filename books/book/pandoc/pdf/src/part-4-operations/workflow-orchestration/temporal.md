[Header 1 ("temporal", [], []) [Str "Temporal"], BlockQuote [Para [Str "Durable execution platform for reliable distributed systems with automatic failure handling"]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.5)), (AlignDefault, (ColWidth 0.5))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Workflow Orchestration"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "SDK"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Yes"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "https://github.com/temporalio/temporal"] ("https://github.com/temporalio/temporal", "")]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "20256"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Documentation"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "Official Docs"] ("https://docs.temporal.io/", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "Temporal is an open-source durable execution platform that enables developers to build reliable distributed applications with crash-proof execution guarantees. Applications running on Temporal resume exactly where they left off after crashes, network failures, or infrastructure outages, whether the interruption lasted seconds, days, or years. The platform originated from Uber's Cadence project and is maintained by Temporal Technologies under the MIT license."], Para [Str "Rather than requiring developers to write defensive retry logic, error-handling boilerplate, and state-recovery mechanisms, Temporal shifts that burden to its server infrastructure. Developers write business logic as Workflows and Activities using familiar programming constructs in their language of choice, and the platform handles fault tolerance, state persistence, and automatic retries transparently. This approach is particularly valuable for mission-critical processes such as order fulfillment, customer onboarding, payment processing, and long-running data pipelines."], Para [Str "Temporal provides official Software Development Kits (SDKs) for Go, Java, PHP, Python, Ruby, TypeScript, and .NET, allowing teams to adopt the platform regardless of their primary language. Deployment options include self-hosted infrastructure and Temporal Cloud, a fully managed service offering USD 1,000 in free credits for new accounts."], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], Para [Strong [Str "Workflow"], Str " -- A durable function that orchestrates the execution of business logic. Workflows are deterministic and can run for arbitrarily long durations. The Temporal Server persists Workflow state so that execution survives process restarts and infrastructure failures. Workflow code must remain deterministic: no threading, randomness, direct network input/output (I/O), global state mutation, or system clock access."], Para [Strong [Str "Activity"], Str " -- A single unit of work that performs a business operation, typically involving side effects such as database writes, Application Programming Interface (API) calls, or file system operations. Activities are called from within Workflows and can be retried independently on failure."], Para [Strong [Str "Worker"], Str " -- A process that hosts Workflow and Activity implementations and polls a Task Queue for work. Workers execute the actual code; the Temporal Server itself only orchestrates execution and persists state."], Para [Strong [Str "Task Queue"], Str " -- A named queue that the Temporal Server uses to dispatch Workflow Tasks and Activity Tasks to Workers. Workers register against specific Task Queues, and multiple Workers can poll the same queue for horizontal scalability."], Para [Strong [Str "Namespace"], Str " -- A logical isolation unit within the Temporal Server. Each Namespace has its own set of Workflows, Task Queues, and configuration. Namespaces provide multi-tenancy support for separating environments or teams."], Para [Strong [Str "Workflow Execution"], Str " -- A single invocation of a Workflow with a unique Workflow Identifier (ID). Each execution maintains its own Event History, which is an append-only log of all events that occurred during the execution."], Para [Strong [Str "Signal"], Str " -- An asynchronous message sent to a running Workflow to change its state or control its flow in real time. Signal handlers can execute Activities and Child Workflows."], Para [Strong [Str "Query"], Str " -- A synchronous read-only request to inspect a running Workflow's internal state. Query handlers must not modify state or perform asynchronous operations."], Para [Strong [Str "Update"], Str " -- A synchronous request sent to a running Workflow that can modify state and return a result. Updates support validation before execution."], Para [Strong [Str "Retry Policy"], Str " -- Configuration that controls how Activities and Workflows are retried on failure, including initial interval, backoff coefficient, maximum interval, and maximum attempts."], Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Para [Str "The Temporal Server consists of four independently scalable services that communicate through a membership protocol using Ringpop:"], Para [Strong [Str "Frontend Service"], Str " -- The gateway that handles rate limiting, routing, authorization, and API request validation. The Frontend Service is stateless and does not use sharding or partitioning."], Para [Strong [Str "History Service"], Str " -- Maintains Workflow execution data including mutable state, queues, and timers. This is typically the most resource-intensive service and scales horizontally through sharding."], Para [Strong [Str "Matching Service"], Str " -- Hosts Task Queues and dispatches tasks to appropriate Workers. It manages the assignment of Workflow Tasks and Activity Tasks to polling Workers."], Para [Strong [Str "Worker Service"], Str " -- Runs internal background Workflows that the Temporal Server itself needs for system operations such as archival and replication."], Para [Str "A typical production deployment scales each service independently based on load characteristics. For example, a deployment might run 5 Frontend instances, 15 History instances, 17 Matching instances, and 3 Worker Service instances."], CodeBlock ("", ["text"], []) "+------------------+       +-------------------+
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
", Para [Str "Workers are external to the Temporal Server. They run within the application's infrastructure and communicate with the server over gRPC (Google Remote Procedure Call). This separation means that application code and its dependencies never run on the Temporal Server itself."], Header 2 ("key-features-and-functionality", ["unnumbered", "unlisted"], []) [Str "Key Features and Functionality"], Header 3 ("durable-execution", ["unnumbered", "unlisted"], []) [Str "Durable Execution"], Para [Str "Temporal persists the complete Event History of every Workflow Execution. If a Worker crashes mid-execution, the Workflow replays its history on another Worker and resumes from the exact point of failure. No application-level checkpointing is required."], Header 3 ("automatic-failure-handling-and-retries", ["unnumbered", "unlisted"], []) [Str "Automatic Failure Handling and Retries"], Para [Str "Activities support configurable Retry Policies with exponential backoff:"], CodeBlock ("", ["python"], []) "from datetime import timedelta
from temporalio.common import RetryPolicy

retry_policy = RetryPolicy(
    initial_interval=timedelta(seconds=1),
    backoff_coefficient=2.0,
    maximum_interval=timedelta(minutes=1),
    maximum_attempts=5,
)
", Header 3 ("activity-heartbeating", ["unnumbered", "unlisted"], []) [Str "Activity Heartbeating"], Para [Str "Long-running Activities can report progress via heartbeats. If a Worker crashes, the Activity is rescheduled on another Worker with the last heartbeat details, allowing it to resume from the last checkpoint rather than starting over."], Header 3 ("child-workflows", ["unnumbered", "unlisted"], []) [Str "Child Workflows"], Para [Str "Workflows can spawn Child Workflows for hierarchical decomposition of complex processes. Child Workflows have their own Event History and can be independently cancelled, terminated, or queried."], Header 3 ("continue-as-new", ["unnumbered", "unlisted"], []) [Str "Continue-As-New"], Para [Str "For Workflows that accumulate large Event Histories (for example, long-running polling loops), Continue-As-New completes the current execution and starts a new one with a fresh history, passing forward any necessary state."], Header 3 ("durable-timers", ["unnumbered", "unlisted"], []) [Str "Durable Timers"], Para [Str "Workflows can sleep for arbitrary durations (seconds to years) using ", Code ("", [], []) "workflow.sleep()", Str ". These timers are persisted by the server and survive Worker restarts and outages."], Header 3 ("cancellation-and-termination", ["unnumbered", "unlisted"], []) [Str "Cancellation and Termination"], Para [Str "Workflows support graceful cancellation (which raises a cancellation error that the Workflow can catch and handle) and forced termination (which immediately stops execution)."], Header 3 ("workflow-reset", ["unnumbered", "unlisted"], []) [Str "Workflow Reset"], Para [Str "Completed or failed Workflow Executions can be reset to a specific point in their Event History and re-executed from there."], Header 3 ("versioning", ["unnumbered", "unlisted"], []) [Str "Versioning"], Para [Str "The Temporal SDKs provide patching and versioning mechanisms to safely deploy Workflow code changes without breaking in-progress executions. This allows gradual migration of running Workflows to new code paths."], Header 3 ("observability", ["unnumbered", "unlisted"], []) [Str "Observability"], Para [Str "Temporal provides metrics, distributed tracing integration, and structured logging. The Temporal Web UI displays Workflow status, Event History, pending Activities, and allows manual operations such as signaling and termination."], Header 3 ("data-encryption", ["unnumbered", "unlisted"], []) [Str "Data Encryption"], Para [Str "Custom Data Converters and Payload Codecs enable encryption of all data at rest and in transit:"], CodeBlock ("", ["typescript"], []) "const client = new Client({
  connection,
  dataConverter: await getDataConverter(),
});

const handle = await client.workflow.start(example, {
  args: [\"Alice: Private message for Bob.\"],
  taskQueue: \"encryption\",
  workflowId: `my-business-id-${uuid()}`,
});
", Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], BulletList [[Plain [Strong [Str "Order Fulfillment"], Str " -- Coordinate inventory checks, payment processing, shipping, and notifications across multiple services with guaranteed completion."]], [Plain [Strong [Str "User Onboarding"], Str " -- Orchestrate multi-step registration flows including email verification, profile setup, and third-party integrations that may span hours or days."]], [Plain [Strong [Str "Payment Processing"], Str " -- Manage payment authorization, capture, settlement, and refund flows with automatic retry and idempotency guarantees."]], [Plain [Strong [Str "Data Pipelines"], Str " -- Orchestrate Extract, Transform, Load (ETL) pipelines with per-step retry, timeout management, and progress tracking."]], [Plain [Strong [Str "Subscription Management"], Str " -- Handle recurring billing cycles, trial expirations, plan upgrades, and cancellation flows over months or years."]], [Plain [Strong [Str "Infrastructure Provisioning"], Str " -- Coordinate multi-step cloud resource provisioning with rollback capabilities on partial failures."]], [Plain [Strong [Str "Machine Learning Pipelines"], Str " -- Manage long-running model training, evaluation, and deployment workflows with checkpointing and resource cleanup."]], [Plain [Strong [Str "Human-in-the-Loop Processes"], Str " -- Workflows that pause waiting for human approval signals before proceeding, with durable timers for escalation deadlines."]]], Header 2 ("api-reference-summary", ["unnumbered", "unlisted"], []) [Str "API Reference Summary"], Header 3 ("python-sdk-decorators-and-methods", ["unnumbered", "unlisted"], []) [Str "Python SDK Decorators and Methods"], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.5)), (AlignDefault, (ColWidth 0.5))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Decorator / Method"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Purpose"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "@workflow.defn"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Marks a class as a Workflow definition"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "@workflow.run"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Marks the entry point method of a Workflow"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "@workflow.signal"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Registers a method as a Signal handler"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "@workflow.query"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Registers a method as a Query handler"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "@workflow.update"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Registers a method as an Update handler"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "@activity.defn"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Marks a function as an Activity definition"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "workflow.execute_activity()"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Executes an Activity and awaits the result"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "workflow.start_activity()"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Starts an Activity and returns a handle"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "workflow.execute_child_workflow()"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Executes a Child Workflow and awaits the result"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "workflow.start_child_workflow()"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Starts a Child Workflow and returns a handle"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "workflow.sleep()"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Creates a durable timer"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "workflow.wait_condition()"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Blocks until a condition is met"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "Client.connect()"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Connects to the Temporal Server"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "client.execute_workflow()"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Starts a Workflow and awaits the result"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "client.start_workflow()"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Starts a Workflow and returns a handle"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "handle.signal()"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Sends a Signal to a running Workflow"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "handle.query()"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Queries a running Workflow's state"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "handle.execute_update()"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Sends an Update and awaits the result"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "handle.cancel()"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Requests cancellation of a Workflow"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "handle.terminate()"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Terminates a Workflow immediately"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "handle.result()"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Awaits the final result of a Workflow"]]]])] (TableFoot ("", [], []) []), Header 3 ("timeout-types", ["unnumbered", "unlisted"], []) [Str "Timeout Types"], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.3333333333333333)), (AlignDefault, (ColWidth 0.3333333333333333)), (AlignDefault, (ColWidth 0.3333333333333333))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Timeout"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Scope"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Description"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Workflow Execution Timeout"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Workflow"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Maximum wall-clock duration for the entire Workflow Execution including retries"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Workflow Run Timeout"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Workflow"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Maximum wall-clock duration for a single Workflow run"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Workflow Task Timeout"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Workflow"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Maximum time for a Worker to process a single Workflow Task"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Schedule-To-Close Timeout"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Activity"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Maximum time from scheduling to completion"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Start-To-Close Timeout"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Activity"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Maximum time from start to completion on a Worker"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Schedule-To-Start Timeout"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Activity"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Maximum time waiting in a Task Queue"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Heartbeat Timeout"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Activity"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Maximum interval between heartbeats before the Activity is considered failed"]]]])] (TableFoot ("", [], []) []), Header 2 ("configuration-and-customization", ["unnumbered", "unlisted"], []) [Str "Configuration and Customization"], Header 3 ("worker-configuration", ["unnumbered", "unlisted"], []) [Str "Worker Configuration"], CodeBlock ("", ["python"], []) "from temporalio.client import Client
from temporalio.worker import Worker

client = await Client.connect(\"localhost:7233\")

worker = Worker(
    client,
    task_queue=\"my-task-queue\",
    workflows=[MyWorkflow],
    activities=[my_activity],
    max_concurrent_activities=100,
    max_concurrent_workflow_tasks=100,
)
", Header 3 ("environment-variables", ["unnumbered", "unlisted"], []) [Str "Environment Variables"], Para [Str "Common environment variables for connecting Workers to the Temporal Server:"], CodeBlock ("", ["python"], []) "import os

TEMPORAL_ADDRESS = os.environ.get(\"TEMPORAL_ADDRESS\", \"localhost:7233\")
TEMPORAL_NAMESPACE = os.environ.get(\"TEMPORAL_NAMESPACE\", \"default\")
TEMPORAL_TASK_QUEUE = os.environ.get(\"TEMPORAL_TASK_QUEUE\", \"my-task-queue\")
TEMPORAL_API_KEY = os.environ.get(\"TEMPORAL_API_KEY\", \"\")
", Header 3 ("temporal-cloud-configuration", ["unnumbered", "unlisted"], []) [Str "Temporal Cloud Configuration"], CodeBlock ("", ["bash"], []) "export TEMPORAL_PROFILE=cloud
temporal config set --profile cloud --prop address --value \"<endpoint>\"
temporal config set --profile cloud --prop namespace --value \"<namespace>\"
temporal config set --profile cloud --prop api_key --value \"<api-key>\"
", Header 3 ("server-deployment-configuration", ["unnumbered", "unlisted"], []) [Str "Server Deployment Configuration"], Para [Str "For self-hosted deployments, individual services can be configured and scaled independently:"], CodeBlock ("", ["bash"], []) "docker run \\
    -e SERVICES=history \\
    -e LOG_LEVEL=debug,info \\
    -e DYNAMIC_CONFIG_FILE_PATH=config/dynamic_config.yaml \\
    temporalio/server:1.29.3
", Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Header 3 ("saga-pattern", ["unnumbered", "unlisted"], []) [Str "Saga Pattern"], Para [Str "Temporal naturally implements the Saga pattern for distributed transactions. Each step in a saga is an Activity, and compensating Activities are executed on failure:"], CodeBlock ("", ["python"], []) "@workflow.defn
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
            return \"Order completed\"
        except Exception:
            for compensation in reversed(compensations):
                await workflow.execute_activity(
                    compensation, order,
                    start_to_close_timeout=timedelta(seconds=30),
                )
            raise
", Header 3 ("human-in-the-loop-approval", ["unnumbered", "unlisted"], []) [Str "Human-in-the-Loop Approval"], Para [Str "Workflows can wait for external Signals, enabling approval flows:"], CodeBlock ("", ["python"], []) "@workflow.defn
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
            return \"Approved\"
        except asyncio.TimeoutError:
            return \"Timed out waiting for approval\"

    @workflow.signal
    def approve(self) -> None:
        self.approved = True
", Header 3 ("polling-pattern-with-continue-as-new", ["unnumbered", "unlisted"], []) [Str "Polling Pattern with Continue-As-New"], Para [Str "For long-running polling Workflows, use Continue-As-New to prevent unbounded Event History growth:"], CodeBlock ("", ["python"], []) "@workflow.defn
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
", Header 3 ("microservice-orchestration", ["unnumbered", "unlisted"], []) [Str "Microservice Orchestration"], Para [Str "Temporal serves as a central orchestrator that coordinates calls across multiple microservices, replacing complex choreography with explicit, visible orchestration logic."], Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Header 3 ("complete-python-application", ["unnumbered", "unlisted"], []) [Str "Complete Python Application"], Para [Strong [Str "Define an Activity:"]], CodeBlock ("", ["python"], []) "from dataclasses import dataclass
from temporalio import activity

@dataclass
class GreetingInput:
    greeting: str
    name: str

@activity.defn
async def compose_greeting(input: GreetingInput) -> str:
    return f\"{input.greeting}, {input.name}!\"
", Para [Strong [Str "Define a Workflow:"]], CodeBlock ("", ["python"], []) "from datetime import timedelta
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
            GreetingInput(\"Hello\", name),
            start_to_close_timeout=timedelta(seconds=10),
            retry_policy=RetryPolicy(maximum_attempts=3),
        )
", Para [Strong [Str "Run the Worker:"]], CodeBlock ("", ["python"], []) "import asyncio
from temporalio.client import Client
from temporalio.worker import Worker

async def main():
    client = await Client.connect(\"localhost:7233\")
    worker = Worker(
        client,
        task_queue=\"greeting-task-queue\",
        workflows=[GreetingWorkflow],
        activities=[compose_greeting],
    )
    await worker.run()

if __name__ == \"__main__\":
    asyncio.run(main())
", Para [Strong [Str "Start the Workflow from a Client:"]], CodeBlock ("", ["python"], []) "import asyncio
from temporalio.client import Client

async def main():
    client = await Client.connect(\"localhost:7233\")
    result = await client.execute_workflow(
        GreetingWorkflow.run,
        \"World\",
        id=\"greeting-workflow-001\",
        task_queue=\"greeting-task-queue\",
    )
    print(f\"Workflow result: {result}\")

if __name__ == \"__main__\":
    asyncio.run(main())
", Header 3 ("message-passing-example", ["unnumbered", "unlisted"], []) [Str "Message Passing Example"], CodeBlock ("", ["python"], []) "from dataclasses import dataclass, field
from temporalio import workflow

@dataclass
class Language:
    name: str

@workflow.defn
class GreetingBotWorkflow:
    def __init__(self):
        self.language = Language(\"English\")
        self.approved_for_release = False
        self.greetings = {\"English\": \"Hello\", \"Spanish\": \"Hola\", \"French\": \"Bonjour\"}

    @workflow.run
    async def run(self) -> str:
        await workflow.wait_condition(lambda: self.approved_for_release)
        return f\"Released with language: {self.language.name}\"

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
            raise ValueError(f\"{language.name} is not supported\")
", Header 3 ("go-sdk-workflow-with-retry-policy", ["unnumbered", "unlisted"], []) [Str "Go SDK Workflow with Retry Policy"], CodeBlock ("", ["go"], []) "func LoanApplicationWorkflow(ctx workflow.Context, applicantName string, loanAmount int) (string, error) {
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
        return \"\", err
    }
    return creditCheckResult, nil
}
", Header 3 ("testing-workflows-python", ["unnumbered", "unlisted"], []) [Str "Testing Workflows (Python)"], Para [Strong [Str "Activity Testing with Heartbeats:"]], CodeBlock ("", ["python"], []) "from temporalio.testing import ActivityEnvironment

async def test_activity_heartbeats():
    env = ActivityEnvironment()
    heartbeats = []
    env.on_heartbeat = lambda *args: heartbeats.append(args[0])
    result = await env.run(my_activity, \"test-input\")
    assert heartbeats == [\"step-1\", \"step-2\"]
    assert result == \"expected-output\"
", Para [Strong [Str "Workflow Testing with Time Skipping:"]], CodeBlock ("", ["python"], []) "from temporalio.testing import WorkflowEnvironment
from temporalio.worker import Worker

async def test_long_running_workflow():
    async with await WorkflowEnvironment.start_time_skipping() as env:
        async with Worker(
            env.client,
            task_queue=\"test-queue\",
            workflows=[MyWorkflow],
            activities=[my_activity],
        ):
            result = await env.client.execute_workflow(
                MyWorkflow.run,
                \"input\",
                id=\"test-workflow\",
                task_queue=\"test-queue\",
            )
            assert result == \"expected\"
", Para [Strong [Str "Workflow Replay Testing for Determinism Validation:"]], CodeBlock ("", ["python"], []) "from temporalio.worker import Replayer

async def test_workflow_replay():
    replayer = Replayer(workflows=[MyWorkflow])
    await replayer.replay_workflows(histories)
", Header 2 ("limitations-and-considerations", ["unnumbered", "unlisted"], []) [Str "Limitations and Considerations"], BulletList [[Plain [Strong [Str "Event History Size"], Str " -- Each Workflow Execution has an Event History size limit (approximately 50,000 events by default). Long-running Workflows must use Continue-As-New to keep the history bounded."]], [Plain [Strong [Str "Determinism Requirement"], Str " -- Workflow code must be deterministic. No direct I/O, threading, random number generation, or system clock access is permitted inside Workflow definitions. This constraint requires a mental model shift for developers."]], [Plain [Strong [Str "Payload Size Limits"], Str " -- Individual Activity arguments are limited to 2 Megabytes (MB) per argument, with a 4 MB total message size. Large data must be passed by reference (for example, using object storage Uniform Resource Locators (URLs))."]], [Plain [Strong [Str "Operational Complexity"], Str " -- Self-hosted deployments require managing the Temporal Server (four services), a persistence backend (PostgreSQL, MySQL, or Cassandra), and optionally Elasticsearch for advanced visibility. This adds significant infrastructure overhead."]], [Plain [Strong [Str "Worker Registration Consistency"], Str " -- All Workers polling the same Task Queue must register identical Workflow and Activity types. Inconsistent registration can cause task failures."]], [Plain [Strong [Str "Versioning Discipline"], Str " -- Changing Workflow logic requires careful use of versioning or patching APIs to avoid breaking in-progress executions. Incorrect changes can cause non-determinism errors."]], [Plain [Strong [Str "Learning Curve"], Str " -- The determinism requirements, Event History model, and distinction between Workflows and Activities require meaningful investment to learn properly."]], [Plain [Strong [Str "Cold Start Latency"], Str " -- Workflow replay (rebuilding state from Event History) introduces latency proportional to history length when a Workflow is loaded onto a new Worker."]]], Header 2 ("changelog-highlights", ["unnumbered", "unlisted"], []) [Str "Changelog Highlights"], BulletList [[Plain [Strong [Str "v1.29.3"], Str " (February 2026) -- Latest stable release with bug fixes and stability improvements."]], [Plain [Strong [Str "v1.25+"], Str " -- Introduction of Workflow Updates for synchronous, trackable interactions with running Workflows."]], [Plain [Strong [Str "v1.20+"], Str " -- Enhanced multi-cluster replication and improved Namespace management."]], [Plain [Strong [Str "v1.17+"], Str " -- Schedule support for cron-like recurring Workflow executions managed server-side."]], [Plain [Strong [Str "v1.0"], Str " -- Initial stable release establishing the core Workflow, Activity, and Worker model with full durability guarantees."]]], Para [Str "The project maintains an active release cadence with 152 total releases as of February 2026, contributed to by over 253 contributors across 8,541 commits."], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Link ("", [], []) [Str "Temporal Official Documentation"] ("https://docs.temporal.io/", "")]], [Plain [Link ("", [], []) [Str "Temporal GitHub Repository"] ("https://github.com/temporalio/temporal", "")]], [Plain [Link ("", [], []) [Str "Temporal Python SDK Documentation"] ("https://docs.temporal.io/develop/python/core-application", "")]], [Plain [Link ("", [], []) [Str "Temporal Python SDK Message Passing"] ("https://docs.temporal.io/develop/python/message-passing", "")]], [Plain [Link ("", [], []) [Str "Temporal Python SDK Testing Suite"] ("https://docs.temporal.io/develop/python/testing-suite", "")]], [Plain [Link ("", [], []) [Str "Temporal Server Architecture"] ("https://docs.temporal.io/temporal-service/temporal-server", "")]], [Plain [Link ("", [], []) [Str "Temporal Self-Hosted Deployment Guide"] ("https://docs.temporal.io/self-hosted-guide/deployment", "")]], [Plain [Link ("", [], []) [Str "Temporal Converters and Encryption"] ("https://docs.temporal.io/develop/typescript/converters-and-encryption", "")]], [Plain [Link ("", [], []) [Str "Temporal Cloud"] ("https://temporal.io/cloud", "")]]], Header 2 ("discovery-signals", ["unnumbered", "unlisted"], []) [Str "Discovery Signals"], BlockQuote [Para [Str "Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search."]], Header 3 ("keywords", ["unnumbered", "unlisted"], []) [Str "Keywords"], BulletList [[Plain [Str "Temporal"]], [Plain [Str "durable execution"]], [Plain [Str "crash-proof workflows"]], [Plain [Str "workflow"]], [Plain [Str "activity"]], [Plain [Str "worker"]], [Plain [Str "task queue"]], [Plain [Str "namespace"]], [Plain [Str "event history"]], [Plain [Str "signal"]], [Plain [Str "query"]], [Plain [Str "update"]], [Plain [Str "retry policy"]], [Plain [Str "heartbeat"]], [Plain [Str "continue-as-new"]], [Plain [Str "durable timer"]], [Plain [Str "child workflow"]], [Plain [Str "saga pattern"]], [Plain [Str "workflow replay"]], [Plain [Str "determinism"]], [Plain [Str "versioning"]], [Plain [Str "multi-language SDK"]], [Plain [Str "Go SDK"]], [Plain [Str "Python SDK"]], [Plain [Str "TypeScript SDK"]], [Plain [Str "Temporal Cloud"]], [Plain [Str "Cadence"]]], Header 3 ("verb-noun-tasks", ["unnumbered", "unlisted"], []) [Str "Verb-Noun Tasks"], BulletList [[Plain [Str "Define workflows with ", Code ("", [], []) "@workflow.defn", Str " and activities with ", Code ("", [], []) "@activity.defn"]], [Plain [Str "Execute activities with retry policies and timeouts"]], [Plain [Str "Send signals to running workflows"]], [Plain [Str "Query workflow state synchronously"]], [Plain [Str "Update workflow state with validation"]], [Plain [Str "Sleep durable timers from seconds to years"]], [Plain [Str "Continue-As-New to bound event history"]], [Plain [Str "Implement saga compensation on partial failures"]], [Plain [Str "Test workflows with time skipping"]], [Plain [Str "Replay event histories to validate determinism"]], [Plain [Str "Encrypt payloads with custom Data Converters"]], [Plain [Str "Spawn child workflows for hierarchical decomposition"]]], Header 3 ("user-intent-phrases", ["unnumbered", "unlisted"], []) [Str "User Intent Phrases"], BulletList [[Plain [Str "I need workflows that survive process crashes and resume exactly where they left off."]], [Plain [Str "How do I implement a saga with compensating actions?"]], [Plain [Str "I want durable timers that last days or years."]], [Plain [Str "How do I orchestrate microservices reliably without writing retry boilerplate?"]], [Plain [Str "I need human-in-the-loop approval gates that wait for signals."]], [Plain [Str "How do I write reliable order fulfillment or payment workflows?"]], [Plain [Str "What's a polyglot durable execution platform (Go, Java, Python, TypeScript, .NET, Ruby, PHP)?"]], [Plain [Str "How do I version workflow code without breaking in-flight executions?"]], [Plain [Str "I want to replay production workflows for debugging."]], [Plain [Str "How do I test long-running workflows quickly with time skipping?"]]], Header 3 ("problem-statements", ["unnumbered", "unlisted"], []) [Str "Problem Statements"], BulletList [[Plain [Str "Distributed transactions break when processes crash mid-flight."]], [Plain [Str "Retry logic and state recovery boilerplate is duplicated across services."]], [Plain [Str "Cron + queues + state machines is fragile compared to a unified abstraction."]], [Plain [Str "Long-running business processes (subscriptions, onboarding) span days or months."]], [Plain [Str "Microservice choreography is invisible; orchestration is explicit but lacks tooling."]], [Plain [Str "Cross-language teams need one workflow engine that supports all their stacks."]]], Header 3 ("when-to-pick-this", ["unnumbered", "unlisted"], []) [Str "When to Pick This"], BulletList [[Plain [Str "Pick this when business processes must be crash-proof and resumable across hours, days, or years."]], [Plain [Str "Pick this over Airflow/Prefect when each workflow instance needs durable per-instance state (orders, onboardings, payments), not scheduled batch DAGs."]], [Plain [Str "Pick this over n8n/Activepieces/Node-RED when code-first durability and multi-language SDKs matter."]], [Plain [Str "Pick this when saga patterns with explicit compensation are required."]], [Plain [Str "Pick this when teams use Go, Java, .NET, or Ruby in addition to Python/TypeScript."]], [Plain [Str "Avoid this when workflows are short, schedule-driven batch ETL — Airflow or Prefect are simpler."]]], Header 3 ("related-terms-and-aliases", ["unnumbered", "unlisted"], []) [Str "Related Terms and Aliases"], BulletList [[Plain [Str "Temporal Technologies"]], [Plain [Str "Uber Cadence (predecessor)"]], [Plain [Str "temporal.io"]], [Plain [Str "Workflow Execution"]], [Plain [Str "Event History"]], [Plain [Str "Workflow Task / Activity Task"]], [Plain [Str "workflow.execute_activity"]], [Plain [Str "workflow.sleep"]], [Plain [Str "workflow.continue_as_new"]], [Plain [Str "@workflow.signal / @workflow.query / @workflow.update"]], [Plain [Str "Frontend / History / Matching / Worker Service"]], [Plain [Str "saga orchestration"]], [Plain [Str "durable function"]], [Plain [Str "workflow-as-code"]], [Plain [Str "Temporal Web UI"]]]]