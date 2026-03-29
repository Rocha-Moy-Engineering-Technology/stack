[Header 1 ("airflow", [], []) [Str "Airflow"], BlockQuote [Para [Str "Apache workflow orchestration platform for scheduling and monitoring pipelines"]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.5)), (AlignDefault, (ColWidth 0.5))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Workflow Orchestration & Automation"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "SDK"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Yes"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "https://github.com/apache/airflow"] ("https://github.com/apache/airflow", "")]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "44357"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Documentation"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "Official Docs"] ("https://airflow.apache.org/docs/apache-airflow/stable/", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "Apache Airflow is an open-source platform for developing, scheduling, and monitoring batch-oriented workflows. Originally created at Airbnb in 2014, it became an Apache Software Foundation top-level project in 2019. Airflow uses Python to define workflows as Directed Acyclic Graphs (DAGs), enabling dynamic pipeline generation, parameterization, and full version control integration."], Para [Str "The platform follows a \"workflows as code\" philosophy where pipelines are defined in standard Python files. This approach provides dynamic generation capabilities, extensibility through a rich operator ecosystem, and flexibility via Jinja templating. Airflow excels at orchestrating finite, batch-oriented workflows with clear start and end points, scheduled execution windows, and complex dependency chains."], Para [Str "Airflow is not designed for continuously running, event-driven, or streaming workloads. It complements stream processing tools such as Apache Kafka rather than replacing them. For teams that require a code-first approach to pipeline orchestration with robust scheduling, backfill support, and a mature web interface, Airflow remains the industry standard."], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], Para [Strong [Str "DAG (Directed Acyclic Graph)"], Str " -- The fundamental abstraction in Airflow. A DAG encapsulates a complete workflow specification including schedule timing, tasks, task dependencies, callbacks, and operational parameters. DAGs are defined in Python files and discovered by the scheduler from a configured folder."], Para [Strong [Str "Task"], Str " -- A discrete unit of work within a DAG. Each task is an instance of an operator and runs on a worker. Tasks have states (queued, running, success, failed, skipped, up_for_retry) and maintain execution history across DAG runs."], Para [Strong [Str "Operator"], Str " -- A template for a predefined task. Airflow ships with built-in operators (BashOperator, PythonOperator, EmailOperator) and hundreds of provider-packaged operators for external systems (DockerOperator, KubernetesOperator, SQLExecuteQueryOperator). Operators define what a task does."], Para [Strong [Str "Sensor"], Str " -- A special type of operator that waits for a specific condition to be met before proceeding. Sensors poll at a defined interval and can operate in \"poke\" mode (occupying a worker slot) or \"reschedule\" mode (releasing the slot between checks)."], Para [Strong [Str "XCom (Cross-Communication)"], Str " -- A mechanism for tasks to exchange small amounts of data. Tasks push and pull XCom values stored in the metadata database. The TaskFlow API manages XCom automatically through function return values and arguments."], Para [Strong [Str "Hook"], Str " -- An interface to external platforms and databases (S3, PostgreSQL, Slack). Hooks abstract connection details and authentication, allowing operators to interact with external systems without hardcoding credentials."], Para [Strong [Str "Connection"], Str " -- A stored set of credentials and configuration for connecting to external systems. Connections are managed through the UI, CLI, or environment variables and referenced by a connection ID."], Para [Strong [Str "Variable"], Str " -- A generic key-value store for configuration data accessible across DAGs. Variables are stored in the metadata database and can be managed through the UI or CLI."], Para [Strong [Str "Pool"], Str " -- A mechanism to limit the parallelism of a set of tasks across DAGs. Pools define a number of slots, and tasks assigned to a pool will only run when a slot is available."], Para [Strong [Str "Dataset / Asset"], Str " -- A logical representation of data that enables data-aware scheduling. A producer DAG declares which datasets it updates, and consumer DAGs can be triggered when those datasets change rather than relying solely on time-based schedules. In Airflow 3.0, datasets were refactored into \"Assets\" with support for asset aliases."], Para [Strong [Str "TaskFlow API"], Str " -- A modern Python-native approach to writing DAGs introduced in Airflow 2.0. The ", Code ("", [], []) "@task", Str " decorator simplifies DAG authoring by automatically managing XCom data passing and dependency resolution between tasks."], Para [Strong [Str "Task Group"], Str " -- A UI-level grouping mechanism that organizes related tasks into collapsible sections in the graph view. Task groups do not affect execution behavior but improve readability of complex DAGs."], Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Para [Str "Airflow follows a distributed architecture with several core components that communicate through a shared metadata database and DAG file synchronization."], Para [Strong [Str "Scheduler"], Str " -- The central orchestration engine. The scheduler triggers workflows based on their defined schedules, submits tasks to the executor, monitors running tasks, and manages retries. The executor runs as a configuration property within the scheduler process."], Para [Strong [Str "DAG Processor"], Str " -- Parses Python DAG files from the configured DAGs folder and serializes the resulting DAG structures into the metadata database. In Airflow 3.0, a standalone DAG processor is required for scheduler-managed backfills."], Para [Strong [Str "Webserver"], Str " -- Serves the web UI for inspecting, triggering, and debugging DAGs and tasks. Provides Graph View, Grid View, DAG overview, log inspection, manual triggering, and variable/connection management."], Para [Strong [Str "Metadata Database"], Str " -- Stores all state: DAG definitions, task instance statuses, XCom values, variables, connections, pools, and audit logs. PostgreSQL and MySQL are supported for production; SQLite is available for development only."], Para [Strong [Str "Worker"], Str " -- Executes tasks as assigned by the scheduler. Workers may be integrated with the scheduler in basic setups (LocalExecutor) or deployed as separate processes or containers (CeleryExecutor, KubernetesExecutor)."], Para [Strong [Str "Triggerer"], Str " -- An optional component that runs deferred tasks in an asyncio event loop. Required only when using deferrable operators, which release their worker slot while waiting for an external condition."], Header 3 ("executor-types", ["unnumbered", "unlisted"], []) [Str "Executor Types"], BulletList [[Plain [Strong [Str "LocalExecutor"], Str " -- Tasks run as subprocesses on the same machine as the scheduler. Suitable for small to medium workloads on a single node."]], [Plain [Strong [Str "CeleryExecutor"], Str " -- Tasks are distributed to long-running Celery worker processes via a message broker (Redis or RabbitMQ). Scales horizontally across multiple machines."]], [Plain [Strong [Str "KubernetesExecutor"], Str " -- Each task runs in a dedicated Kubernetes pod. Provides strong isolation and dynamic resource allocation. Pods are created on demand and destroyed after completion."]], [Plain [Strong [Str "Edge Executor"], Str " -- Introduced in Airflow 3.0 (AIP-69), supports distributed, event-driven, and edge-compute workflows. Enables task execution in remote environments."]]], Header 3 ("component-communication", ["unnumbered", "unlisted"], []) [Str "Component Communication"], BulletList [[Plain [Str "DAG files are synchronized across the scheduler, triggerer, and workers"]], [Plain [Str "All components communicate state through the metadata database"]], [Plain [Str "The scheduler sends control signals to workers via the executor"]], [Plain [Str "Installed packages and plugins must be deployed consistently to all relevant components"]]], Header 2 ("key-features-and-functionality", ["unnumbered", "unlisted"], []) [Str "Key Features and Functionality"], Para [Strong [Str "Dynamic DAG Generation"], Str " -- DAGs are Python code, so they can be generated programmatically using loops, conditionals, configuration files, or external data sources. This enables patterns like generating one DAG per customer or per data source from a configuration table."], Para [Strong [Str "Rich Scheduling"], Str " -- Supports cron expressions, preset schedules (", Code ("", [], []) "@daily", Str ", ", Code ("", [], []) "@hourly", Str ", ", Code ("", [], []) "@weekly", Str "), timedelta intervals, timetable objects for complex custom schedules, and data-aware scheduling via datasets/assets."], Para [Strong [Str "Backfill and Catchup"], Str " -- Automatically (or manually) execute DAG runs for historical date ranges. Selective task reruns allow reprocessing specific tasks without re-running the entire pipeline, reducing compute costs."], Para [Strong [Str "Web UI"], Str " -- Comprehensive interface with Graph View (dependency visualization), Grid View (historical run matrix), calendar view, Gantt chart, task log viewer, code viewer, and manual DAG/task triggering. Airflow 3.0 introduced a fully modernized React-based UI with 17 language translations."], Para [Strong [Str "Jinja Templating"], Str " -- Operator parameters support Jinja template rendering with access to execution context variables (execution date, DAG run information, task instance metadata). Enables parameterized, date-partitioned pipelines."], Para [Strong [Str "Connection and Secret Management"], Str " -- Centralized credential storage with support for external secret backends (HashiCorp Vault, AWS Secrets Manager, GCP Secret Manager). Fernet encryption protects stored passwords."], Para [Strong [Str "Extensible Provider Ecosystem"], Str " -- Over 80 official provider packages integrating with cloud platforms (AWS, GCP, Azure), databases (Postgres, MySQL, MongoDB), messaging systems (Kafka, RabbitMQ), and more. Community-contributed providers extend this further."], Para [Strong [Str "REST API"], Str " -- Full CRUD API for programmatic DAG management, triggering runs, querying task states, managing connections and variables, and building external integrations."], Para [Strong [Str "Multiple Executor Support"], Str " -- Since Airflow 2.10, a single deployment can use multiple executors concurrently. Individual tasks within a DAG can specify which executor to use, allowing mixed execution strategies."], Para [Strong [Str "OpenTelemetry Traces"], Str " -- Airflow 2.10 added native OpenTelemetry tracing for the scheduler, triggerer, executor, and processor, enabling deep observability through OTLP-compatible endpoints."], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], Para [Strong [Str "ETL/ELT Pipelines"], Str " -- Extract data from source systems, transform it, and load it into data warehouses or data lakes on a recurring schedule. Airflow manages dependencies between extraction, transformation, and loading stages."], Para [Strong [Str "Machine Learning Pipelines"], Str " -- Orchestrate data preprocessing, feature engineering, model training, evaluation, and deployment steps. Integrate with platforms like MLflow, SageMaker, or Vertex AI through provider operators."], Para [Strong [Str "Data Quality and Validation"], Str " -- Schedule recurring data quality checks, anomaly detection runs, and data freshness monitoring. Trigger alerts or corrective workflows when checks fail."], Para [Strong [Str "Report Generation"], Str " -- Automate the generation and distribution of business reports: query databases, produce visualizations, compile documents, and deliver via email or Slack."], Para [Strong [Str "Infrastructure Automation"], Str " -- Manage cloud resource provisioning, database maintenance tasks, log rotation, backup schedules, and environment cleanup through code-defined workflows."], Para [Strong [Str "Cross-System Data Synchronization"], Str " -- Coordinate data movement between multiple systems (databases, APIs, file systems, cloud storage) with dependency tracking and retry logic."], Header 2 ("api-reference-summary", ["unnumbered", "unlisted"], []) [Str "API Reference Summary"], Header 3 ("rest-api", ["unnumbered", "unlisted"], []) [Str "REST API"], Para [Str "Airflow exposes a REST API (v2 in Airflow 2.x, v2 redesigned in Airflow 3.x) for programmatic interaction:"], CodeBlock ("", ["bash"], []) "# List all DAGs
curl -X GET \"http://localhost:8080/api/v2/dags\" \\
    -H \"Authorization: Bearer <token>\"

# Trigger a DAG run
curl -X POST \"http://localhost:8080/api/v2/dags/{dag_id}/dagRuns\" \\
    -H \"Content-Type: application/json\" \\
    -d '{\"logical_date\": \"2025-01-01T00:00:00Z\", \"conf\": {\"key\": \"value\"}}'

# Get task instance status
curl -X GET \"http://localhost:8080/api/v2/dags/{dag_id}/dagRuns/{run_id}/taskInstances/{task_id}\"
", Para [Str "Key endpoint groups: DAGs, DAG Runs, Task Instances, Connections, Variables, Pools, Plugins, Health, Config, Users, and Permissions."], Header 3 ("cli", ["unnumbered", "unlisted"], []) [Str "CLI"], CodeBlock ("", ["bash"], []) "# DAG operations
airflow dags list                          # List all DAGs
airflow dags trigger <dag_id>              # Trigger a DAG run
airflow dags backfill <dag_id> -s <start> -e <end>  # Backfill date range
airflow dags test <dag_id> <date>          # Test a DAG without recording state

# Task operations
airflow tasks list <dag_id>                # List tasks in a DAG
airflow tasks test <dag_id> <task_id> <date>  # Test a single task
airflow tasks run <dag_id> <task_id> <date>   # Run a single task

# Database operations
airflow db migrate                         # Apply database migrations
airflow db check                           # Verify database connectivity

# User management
airflow users create --username admin --role Admin --email admin@example.com
airflow users list
", Header 3 ("python-api-stable", ["unnumbered", "unlisted"], []) [Str "Python API (Stable)"], CodeBlock ("", ["python"], []) "from airflow.sdk import DAG, dag, task
from airflow.providers.standard.operators.bash import BashOperator
from airflow.providers.standard.operators.python import PythonOperator
from airflow.providers.standard.sensors.filesystem import FileSensor
from airflow.models import Variable
", Header 2 ("configuration-and-customization", ["unnumbered", "unlisted"], []) [Str "Configuration and Customization"], Header 3 ("configuration-file", ["unnumbered", "unlisted"], []) [Str "Configuration File"], Para [Str "Airflow is configured through ", Code ("", [], []) "airflow.cfg", Str " (INI format) or environment variables with the pattern ", Code ("", [], []) "AIRFLOW__{SECTION}__{KEY}", Str ":"], CodeBlock ("", ["bash"], []) "# Core settings
export AIRFLOW__CORE__DAGS_FOLDER=/opt/airflow/dags
export AIRFLOW__CORE__EXECUTOR=CeleryExecutor
export AIRFLOW__CORE__PARALLELISM=32
export AIRFLOW__CORE__MAX_ACTIVE_RUNS_PER_DAG=16
export AIRFLOW__CORE__LOAD_EXAMPLES=False

# Database
export AIRFLOW__DATABASE__SQL_ALCHEMY_CONN=postgresql+psycopg2://user:pass@host:5432/airflow

# Webserver
export AIRFLOW__WEBSERVER__BASE_URL=https://airflow.example.com
export AIRFLOW__WEBSERVER__WEB_SERVER_PORT=8080

# Scheduler
export AIRFLOW__SCHEDULER__MIN_FILE_PROCESS_INTERVAL=30

# Logging
export AIRFLOW__LOGGING__BASE_LOG_FOLDER=/opt/airflow/logs
export AIRFLOW__LOGGING__REMOTE_LOGGING=True
export AIRFLOW__LOGGING__REMOTE_LOG_CONN_ID=aws_s3_logs

# Encryption
export AIRFLOW__CORE__FERNET_KEY=$(python3 -c \"from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())\")
", Header 3 ("custom-operators", ["unnumbered", "unlisted"], []) [Str "Custom Operators"], Para [Str "Create reusable operators by extending ", Code ("", [], []) "BaseOperator", Str ":"], CodeBlock ("", ["python"], []) "from airflow.models import BaseOperator

class MyCustomOperator(BaseOperator):
    template_fields = (\"my_param\",)

    def __init__(self, my_param: str, **kwargs):
        super().__init__(**kwargs)
        self.my_param = my_param

    def execute(self, context):
        self.log.info(\"Running with param: %s\", self.my_param)
        # Custom logic here
        return result
", Header 3 ("custom-hooks", ["unnumbered", "unlisted"], []) [Str "Custom Hooks"], CodeBlock ("", ["python"], []) "from airflow.hooks.base import BaseHook

class MyServiceHook(BaseHook):
    conn_name_attr = \"my_conn_id\"
    default_conn_name = \"my_service_default\"
    conn_type = \"my_service\"
    hook_name = \"My Service\"

    def __init__(self, my_conn_id: str = default_conn_name):
        super().__init__()
        self.my_conn_id = my_conn_id

    def get_conn(self):
        conn = self.get_connection(self.my_conn_id)
        return create_client(host=conn.host, token=conn.password)
", Header 3 ("plugins", ["unnumbered", "unlisted"], []) [Str "Plugins"], Para [Str "Extend Airflow functionality through the plugin system:"], CodeBlock ("", ["python"], []) "from airflow.plugins_manager import AirflowPlugin
from flask import Blueprint

my_blueprint = Blueprint(\"my_plugin\", __name__, url_prefix=\"/myplugin\")

class MyPlugin(AirflowPlugin):
    name = \"my_plugin\"
    flask_blueprints = [my_blueprint]
", Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Header 3 ("cloud-provider-integration", ["unnumbered", "unlisted"], []) [Str "Cloud Provider Integration"], CodeBlock ("", ["python"], []) "from airflow.providers.amazon.operators.s3 import S3CopyObjectOperator
from airflow.providers.google.cloud.operators.bigquery import BigQueryInsertJobOperator
from airflow.providers.amazon.operators.glue import GlueJobOperator
", Header 3 ("database-integration", ["unnumbered", "unlisted"], []) [Str "Database Integration"], CodeBlock ("", ["python"], []) "from airflow.providers.common.sql.operators.sql import SQLExecuteQueryOperator

query_task = SQLExecuteQueryOperator(
    task_id=\"run_query\",
    conn_id=\"my_postgres\",
    sql=\"SELECT COUNT(*) FROM orders WHERE date = '{{ ds }}'\",
)
", Header 3 ("container-integration", ["unnumbered", "unlisted"], []) [Str "Container Integration"], CodeBlock ("", ["python"], []) "from airflow.providers.docker.operators.docker import DockerOperator

docker_task = DockerOperator(
    task_id=\"run_container\",
    image=\"my-etl-image:latest\",
    command=\"python /app/process.py --date {{ ds }}\",
    docker_url=\"unix://var/run/docker.sock\",
    network_mode=\"bridge\",
)
", Header 3 ("kubernetes-pod-integration", ["unnumbered", "unlisted"], []) [Str "Kubernetes Pod Integration"], CodeBlock ("", ["python"], []) "from airflow.providers.cncf.kubernetes.operators.pod import KubernetesPodOperator

k8s_task = KubernetesPodOperator(
    task_id=\"run_pod\",
    name=\"etl-processor\",
    namespace=\"data-pipelines\",
    image=\"my-etl-image:latest\",
    arguments=[\"--date\", \"{{ ds }}\"],
    is_delete_operator_pod=True,
)
", Header 3 ("notification-integration", ["unnumbered", "unlisted"], []) [Str "Notification Integration"], CodeBlock ("", ["python"], []) "from airflow.providers.slack.notifications.slack import send_slack_notification

dag = DAG(
    dag_id=\"my_dag\",
    on_failure_callback=send_slack_notification(
        slack_conn_id=\"slack_default\",
        text=\"DAG {{ dag.dag_id }} failed!\",
        channel=\"#alerts\",
    ),
)
", Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Header 3 ("basic-dag-with-traditional-operators", ["unnumbered", "unlisted"], []) [Str "Basic DAG with Traditional Operators"], CodeBlock ("", ["python"], []) "from datetime import datetime, timedelta
from airflow.sdk import DAG
from airflow.providers.standard.operators.bash import BashOperator

default_args = {
    \"owner\": \"data-team\",
    \"retries\": 2,
    \"retry_delay\": timedelta(minutes=5),
    \"email_on_failure\": True,
    \"email\": [\"team@example.com\"],
}

with DAG(
    dag_id=\"etl_pipeline\",
    default_args=default_args,
    description=\"Daily ETL pipeline\",
    schedule=\"@daily\",
    start_date=datetime(2025, 1, 1),
    catchup=False,
    tags=[\"etl\", \"production\"],
) as dag:

    extract = BashOperator(
        task_id=\"extract\",
        bash_command=\"python /scripts/extract.py --date {{ ds }}\",
    )

    transform = BashOperator(
        task_id=\"transform\",
        bash_command=\"python /scripts/transform.py --date {{ ds }}\",
    )

    load = BashOperator(
        task_id=\"load\",
        bash_command=\"python /scripts/load.py --date {{ ds }}\",
    )

    validate = BashOperator(
        task_id=\"validate\",
        bash_command=\"python /scripts/validate.py --date {{ ds }}\",
    )

    extract >> transform >> load >> validate
", Header 3 ("taskflow-api-with-automatic-xcom", ["unnumbered", "unlisted"], []) [Str "TaskFlow API with Automatic XCom"], CodeBlock ("", ["python"], []) "from datetime import datetime
from airflow.sdk import dag, task


@dag(
    dag_id=\"ml_training_pipeline\",
    schedule=\"@weekly\",
    start_date=datetime(2025, 1, 1),
    catchup=False,
    tags=[\"ml\", \"training\"],
)
def ml_training_pipeline():

    @task
    def fetch_training_data():
        import pandas as pd
        df = pd.read_sql(\"SELECT * FROM features WHERE is_labeled = true\", conn)
        path = \"/tmp/training_data.parquet\"
        df.to_parquet(path)
        return path

    @task
    def preprocess(data_path: str):
        import pandas as pd
        df = pd.read_parquet(data_path)
        # Preprocessing logic
        processed_path = \"/tmp/processed_data.parquet\"
        df.to_parquet(processed_path)
        return processed_path

    @task
    def train_model(data_path: str):
        import pandas as pd
        df = pd.read_parquet(data_path)
        # Model training logic
        model_path = \"/tmp/model.pkl\"
        return {\"model_path\": model_path, \"accuracy\": 0.95}

    @task
    def evaluate_model(training_result: dict):
        accuracy = training_result[\"accuracy\"]
        if accuracy < 0.90:
            raise ValueError(f\"Model accuracy {accuracy} below threshold\")
        return training_result[\"model_path\"]

    @task
    def deploy_model(model_path: str):
        # Deployment logic
        print(f\"Deploying model from {model_path}\")

    # Automatic dependency resolution through function calls
    raw_data = fetch_training_data()
    processed_data = preprocess(raw_data)
    result = train_model(processed_data)
    validated_model = evaluate_model(result)
    deploy_model(validated_model)


ml_training_pipeline()
", Header 3 ("branching-and-conditional-execution", ["unnumbered", "unlisted"], []) [Str "Branching and Conditional Execution"], CodeBlock ("", ["python"], []) "from datetime import datetime
from airflow.sdk import DAG, task
from airflow.providers.standard.operators.bash import BashOperator


with DAG(
    dag_id=\"conditional_pipeline\",
    schedule=\"@daily\",
    start_date=datetime(2025, 1, 1),
    catchup=False,
) as dag:

    @task.branch
    def choose_branch(**context):
        day = context[\"logical_date\"].weekday()
        if day < 5:
            return \"weekday_processing\"
        return \"weekend_processing\"

    weekday = BashOperator(
        task_id=\"weekday_processing\",
        bash_command=\"echo 'Running weekday batch'\",
    )

    weekend = BashOperator(
        task_id=\"weekend_processing\",
        bash_command=\"echo 'Running weekend summary'\",
    )

    choose_branch() >> [weekday, weekend]
", Header 3 ("dynamic-dag-generation", ["unnumbered", "unlisted"], []) [Str "Dynamic DAG Generation"], CodeBlock ("", ["python"], []) "from datetime import datetime
from airflow.sdk import DAG
from airflow.providers.standard.operators.bash import BashOperator

CUSTOMERS = [\"acme\", \"globex\", \"initech\", \"umbrella\"]

for customer in CUSTOMERS:
    dag_id = f\"etl_{customer}\"

    with DAG(
        dag_id=dag_id,
        schedule=\"@daily\",
        start_date=datetime(2025, 1, 1),
        catchup=False,
        tags=[\"customer-etl\", customer],
    ) as dag:

        extract = BashOperator(
            task_id=\"extract\",
            bash_command=f\"python /scripts/extract.py --customer {customer} --date {{{{ ds }}}}\",
        )

        load = BashOperator(
            task_id=\"load\",
            bash_command=f\"python /scripts/load.py --customer {customer} --date {{{{ ds }}}}\",
        )

        extract >> load

    globals()[dag_id] = dag
", Header 3 ("sensor-with-deferrable-operator", ["unnumbered", "unlisted"], []) [Str "Sensor with Deferrable Operator"], CodeBlock ("", ["python"], []) "from datetime import datetime, timedelta
from airflow.sdk import DAG
from airflow.providers.standard.sensors.filesystem import FileSensor
from airflow.providers.standard.operators.bash import BashOperator

with DAG(
    dag_id=\"file_triggered_pipeline\",
    schedule=None,
    start_date=datetime(2025, 1, 1),
    catchup=False,
) as dag:

    wait_for_file = FileSensor(
        task_id=\"wait_for_data\",
        filepath=\"/data/incoming/daily_feed.csv\",
        poke_interval=60,
        timeout=3600,
        mode=\"reschedule\",
    )

    process = BashOperator(
        task_id=\"process_file\",
        bash_command=\"python /scripts/process.py /data/incoming/daily_feed.csv\",
    )

    wait_for_file >> process
", Header 2 ("limitations-and-considerations", ["unnumbered", "unlisted"], []) [Str "Limitations and Considerations"], Para [Strong [Str "Not for Streaming Workloads"], Str " -- Airflow is designed for finite, batch-oriented workflows. It is not a replacement for stream processing systems like Apache Kafka, Apache Flink, or Apache Spark Streaming. Continuously running pipelines with no clear end point are not a good fit."], Para [Strong [Str "Not Event-Driven by Default"], Str " -- While dataset/asset-based scheduling adds some event-driven capabilities, Airflow's core model is schedule-based. True real-time event processing requires complementary systems."], Para [Strong [Str "DAG Parsing Overhead"], Str " -- The scheduler must periodically parse all DAG files (Python modules), which becomes a bottleneck with hundreds or thousands of DAGs. Complex DAG files with heavy imports or database calls at parse time degrade scheduler performance."], Para [Strong [Str "XCom Size Limits"], Str " -- XCom is designed for small data (metadata, file paths, status flags). Passing large datasets through XCom strains the metadata database. Use external storage (S3, GCS) and pass references instead."], Para [Strong [Str "Metadata Database as Single Point of Failure"], Str " -- All components depend on the metadata database. Database outages halt the entire Airflow deployment. High-availability database configurations are essential for production."], Para [Strong [Str "Task Granularity Trade-offs"], Str " -- Very fine-grained tasks (thousands per DAG run) incur scheduling overhead. Very coarse-grained tasks reduce observability and retry granularity. Finding the right balance requires experimentation."], Para [Strong [Str "Complexity of Production Deployment"], Str " -- Running Airflow reliably in production requires managing multiple components (scheduler, webserver, workers, database, optional triggerer and DAG processor), configuring monitoring, and handling upgrades carefully."], Para [Strong [Str "Migration Between Major Versions"], Str " -- Upgrades between major versions (2.x to 3.x) involve metadata database migrations, configuration changes, and potential DAG code modifications. The removal of deprecated features in Airflow 3.0 (such as SequentialExecutor) requires review and testing before upgrading."], Header 2 ("changelog-highlights", ["unnumbered", "unlisted"], []) [Str "Changelog Highlights"], Para [Strong [Str "Airflow 3.0 (April 2025)"], Str " -- The most significant release in the project's history. Introduced the Task SDK with a stable DAG authoring interface under ", Code ("", [], []) "airflow.sdk", Str ". Added a service-oriented Task Execution API (AIP-72) enabling task execution in remote environments. Launched the Edge Executor (AIP-69) for distributed and edge-compute workflows. Delivered a fully modernized React-based UI (AIP-38). Made backfills scheduler-managed (AIP-78). Refactored datasets into \"Assets\" with alias support (AIP-74, AIP-75). Removed SequentialExecutor and legacy plugin capabilities."], Para [Strong [Str "Airflow 2.10 (2024)"], Str " -- Added OpenTelemetry traces for scheduler, triggerer, executor, and processor. Introduced multiple executor support allowing individual tasks to specify different executors within a single DAG. Added a TaskFlow decorator for simple task skipping. Made the SMTP provider pre-installed."], Para [Strong [Str "Airflow 2.0 (December 2020)"], Str " -- Major rewrite introducing the TaskFlow API with ", Code ("", [], []) "@task", Str " decorator, automatic XCom management, and dependency resolution. Added a full REST API, scheduler high availability, smart sensors, and a redesigned UI."], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Link ("", [], []) [Str "Apache Airflow Documentation"] ("https://airflow.apache.org/docs/apache-airflow/stable/", "")]], [Plain [Link ("", [], []) [Str "Apache Airflow GitHub Repository"] ("https://github.com/apache/airflow", "")]], [Plain [Link ("", [], []) [Str "Airflow 3.0.0 Release Notes"] ("https://airflow.apache.org/docs/apache-airflow/3.0.0/release_notes.html", "")]], [Plain [Link ("", [], []) [Str "Airflow 2.10.5 Release Notes"] ("https://airflow.apache.org/docs/apache-airflow/2.10.5/release_notes.html", "")]], [Plain [Link ("", [], []) [Str "Apache Airflow Announcements"] ("https://airflow.apache.org/announcements/", "")]], [Plain [Link ("", [], []) [Str "Airflow REST API Reference"] ("https://airflow.apache.org/docs/apache-airflow/stable/stable-rest-api-ref.html", "")]]]]