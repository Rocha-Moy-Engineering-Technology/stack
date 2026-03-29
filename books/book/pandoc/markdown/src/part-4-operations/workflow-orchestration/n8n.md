[Header 1 ("n8n", [], []) [Str "n8n"], BlockQuote [Para [Str "Workflow automation combining visual building with custom code and 400+ integrations"]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.5)), (AlignDefault, (ColWidth 0.5))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Workflow Orchestration & Automation"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "API/SDK/UI"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Yes"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "https://github.com/n8n-io/n8n"] ("https://github.com/n8n-io/n8n", "")]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "175811"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Documentation"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "Official Docs"] ("https://docs.n8n.io/", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "n8n (pronounced \"nodemation\") is a fair-code licensed workflow automation platform that connects applications and automates business processes through a visual node-based editor. It bridges the gap between no-code simplicity and full programmatic control by allowing users to build workflows visually while embedding custom JavaScript or Python at any point. The platform provides over 400 built-in integrations, native AI capabilities through LangChain integration, and flexible deployment options including a managed cloud service and self-hosted installations."], Para [Str "n8n targets technical teams that need both the speed of drag-and-drop workflow building and the flexibility to write code when standard nodes are insufficient. Workflows are represented as directed graphs of interconnected nodes, where data flows from trigger nodes through processing and action nodes. Every workflow execution is tracked, enabling debugging, retry logic, and auditing. The platform supports Role-Based Access Control (RBAC), source control integration, multi-environment promotion, and enterprise Single Sign-On (SSO) via Security Assertion Markup Language (SAML) and OpenID Connect (OIDC)."], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], BulletList [[Para [Strong [Str "Workflows"], Str " -- The top-level automation unit. A workflow is a collection of interconnected nodes that defines a sequence of operations. Workflows can be activated for production execution, run manually for testing, or triggered by external events. Each workflow maintains its own settings for timeouts, error handling, and execution history."]], [Para [Strong [Str "Nodes"], Str " -- Individual processing units within a workflow. Nodes fall into four categories:"], BulletList [[Plain [Strong [Str "Core nodes"], Str " -- Built-in utilities for data manipulation: Code (JavaScript/Python), Filter, Merge, Split Out, Set, IF, Switch, Remove Duplicates, and more."]], [Plain [Strong [Str "Trigger nodes"], Str " -- Entry points that start workflow execution: Schedule Trigger (cron), Webhook, Email Trigger, Manual Trigger, and event-based triggers from third-party services."]], [Plain [Strong [Str "Action nodes"], Str " -- Connectors to external services (400+ integrations): Slack, GitHub, Google Sheets, Salesforce, Amazon Web Services (AWS), HubSpot, Airtable, PostgreSQL, and many others."]], [Plain [Strong [Str "Cluster nodes"], Str " -- Specialized nodes for AI and data processing, including LangChain-based AI Agent, Chain, and Tool nodes."]]]], [Para [Strong [Str "Connections"], Str " -- Links between nodes that define data flow. A node can have multiple output branches (for conditional routing) and multiple inputs (for data merging). Connections carry structured JSON data between nodes."]], [Para [Strong [Str "Credentials"], Str " -- Securely stored authentication details for third-party services. Credentials are encrypted at rest, managed centrally, and can be shared across workflows and team members. n8n supports OAuth2, API keys, HTTP header auth, and service-specific authentication methods."]], [Para [Strong [Str "Expressions"], Str " -- Dynamic references to data within node parameters. Expressions use the syntax ", Code ("", [], []) "{{ }}", Str " and can access data from previous nodes, environment variables, and built-in functions. JMESPath is available for advanced JSON querying."]], [Para [Strong [Str "Executions"], Str " -- Individual runs of a workflow. Execution modes include manual (triggered by the user in the editor), partial (running a subset of nodes for debugging), and production (triggered automatically by events or schedules). Execution history is retained for inspection and retry."]], [Para [Strong [Str "Items"], Str " -- The fundamental data unit passed between nodes. Each item is a JSON object optionally accompanied by binary data (files, images). Nodes process items individually or in batches depending on configuration."]]], Header 2 ("installation-and-setup", ["unnumbered", "unlisted"], []) [Str "Installation and Setup"], Header 3 ("npm", ["unnumbered", "unlisted"], []) [Str "npm"], Para [Str "Install n8n globally via npm for local development:"], CodeBlock ("", ["bash"], []) "npm install n8n -g
n8n start
", Para [Str "n8n starts on port 5678 by default. Open ", Code ("", [], []) "http://localhost:5678", Str " to access the workflow editor."], Header 3 ("docker-sqlite", ["unnumbered", "unlisted"], []) [Str "Docker (SQLite)"], Para [Str "Run n8n with Docker using the default SQLite database:"], CodeBlock ("", ["bash"], []) "docker volume create n8n_data

docker run -it --rm \\
  --name n8n \\
  -p 5678:5678 \\
  -e GENERIC_TIMEZONE=\"UTC\" \\
  -e TZ=\"UTC\" \\
  -e N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true \\
  -e N8N_RUNNERS_ENABLED=true \\
  -v n8n_data:/home/node/.n8n \\
  docker.n8n.io/n8nio/n8n
", Header 3 ("docker-postgresql", ["unnumbered", "unlisted"], []) [Str "Docker (PostgreSQL)"], Para [Str "For production workloads, use PostgreSQL as the backing database:"], CodeBlock ("", ["bash"], []) "docker volume create n8n_data

docker run -it --rm \\
  --name n8n \\
  -p 5678:5678 \\
  -e GENERIC_TIMEZONE=\"UTC\" \\
  -e TZ=\"UTC\" \\
  -e N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true \\
  -e N8N_RUNNERS_ENABLED=true \\
  -e DB_TYPE=postgresdb \\
  -e DB_POSTGRESDB_DATABASE=n8n \\
  -e DB_POSTGRESDB_HOST=postgres-host \\
  -e DB_POSTGRESDB_PORT=5432 \\
  -e DB_POSTGRESDB_USER=n8n \\
  -e DB_POSTGRESDB_PASSWORD=secret \\
  -v n8n_data:/home/node/.n8n \\
  docker.n8n.io/n8nio/n8n
", Header 3 ("docker-compose-production", ["unnumbered", "unlisted"], []) [Str "Docker Compose (Production)"], Para [Str "A production-ready Docker Compose configuration with PostgreSQL:"], CodeBlock ("", ["yaml"], []) "version: '3.8'

services:
  n8n:
    image: docker.n8n.io/n8nio/n8n
    restart: always
    ports:
      - \"5678:5678\"
    environment:
      - GENERIC_TIMEZONE=UTC
      - TZ=UTC
      - N8N_HOST=n8n.example.com
      - N8N_PORT=5678
      - N8N_PROTOCOL=https
      - WEBHOOK_URL=https://n8n.example.com/
      - N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true
      - N8N_RUNNERS_ENABLED=true
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=n8n
      - DB_POSTGRESDB_USER=n8n
      - DB_POSTGRESDB_PASSWORD=${POSTGRES_PASSWORD}
      - N8N_ENCRYPTION_KEY=${N8N_ENCRYPTION_KEY}
    volumes:
      - n8n_data:/home/node/.n8n
    depends_on:
      - postgres

  postgres:
    image: postgres:15
    restart: always
    environment:
      - POSTGRES_USER=n8n
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=n8n
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  n8n_data:
  postgres_data:
", Header 3 ("n8n-cloud", ["unnumbered", "unlisted"], []) [Str "n8n Cloud"], Para [Str "For managed hosting, sign up at ", Link ("", [], []) [Str "n8n.io"] ("https://n8n.io", ""), Str ". n8n Cloud handles infrastructure, upgrades, and SSL certificates. It provides a dashboard for managing instances, versions, and user access."], Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Para [Str "n8n is built as a Node.js application with a TypeScript codebase. The architecture consists of several key components:"], BulletList [[Para [Strong [Str "Workflow Engine"], Str " -- The core execution runtime that interprets workflow definitions, manages node execution order, handles data passing between nodes, and tracks execution state. The engine supports both sequential and parallel node execution based on the workflow graph topology."]], [Para [Strong [Str "Node Framework"], Str " -- A pluggable system where each integration is an independent node package. Nodes implement a standard interface with ", Code ("", [], []) "execute()", Str " methods, parameter definitions, and credential requirements. Custom community nodes can be installed via npm."]], [Para [Strong [Str "Editor UI"], Str " -- A Vue.js-based single-page application providing the visual workflow builder. The editor communicates with the backend via a REST API and WebSocket connections for real-time execution feedback."]], [Para [Strong [Str "Database Layer"], Str " -- Stores workflow definitions, execution history, credentials (encrypted), user data, and settings. Supports SQLite (development/small deployments) and PostgreSQL (production)."]], [Para [Strong [Str "Webhook Server"], Str " -- A dedicated HTTP server that listens for incoming webhook requests and routes them to the appropriate workflow triggers."]], [Para [Strong [Str "Task Runners"], Str " -- Isolated execution environments for Code nodes. When ", Code ("", [], []) "N8N_RUNNERS_ENABLED=true", Str ", JavaScript and Python code runs in sandboxed processes rather than the main n8n thread, improving security and stability."]], [Para [Strong [Str "Queue Mode"], Str " -- For horizontal scaling, n8n supports a queue-based architecture using Redis (or compatible) as a message broker. A main instance handles webhooks and scheduling while worker instances pull executions from the queue. This separates concerns and allows independent scaling of execution capacity."]], [Para [Strong [Str "Encryption"], Str " -- Credentials are encrypted using an encryption key (", Code ("", [], []) "N8N_ENCRYPTION_KEY", Str "). This key must be consistent across all instances in a multi-node deployment and must be preserved during migrations."]]], Header 2 ("key-features-and-functionality", ["unnumbered", "unlisted"], []) [Str "Key Features and Functionality"], Header 3 ("visual-workflow-builder", ["unnumbered", "unlisted"], []) [Str "Visual Workflow Builder"], Para [Str "The drag-and-drop editor allows building complex automations without writing code. Nodes are placed on a canvas and connected with wires. Each node has a configuration panel for setting parameters, mapping data, and defining behavior. The editor provides real-time execution previews showing data at each node."], Header 3 ("code-node-javascript-and-python", ["unnumbered", "unlisted"], []) [Str "Code Node (JavaScript and Python)"], Para [Str "When built-in nodes are insufficient, the Code node allows writing custom JavaScript or Python directly within the workflow:"], CodeBlock ("", ["javascript"], []) "// JavaScript: Transform input items
const items = $input.all();
const newItems = items.map((item) => {
  const firstName = item.json.personal_info.first_name;
  const jobTitle = item.json.work_info.job_title;
  return {
    json: {
      firstName,
      jobTitle,
    },
  };
});
return newItems;
", CodeBlock ("", ["python"], []) "# Python: Access data using underscore-prefixed globals
first_names = _jmespath(_json.body.people, \"[*].first\")
return {\"firstNames\": first_names}
", Header 3 ("400-integrations", ["unnumbered", "unlisted"], []) [Str "400+ Integrations"], Para [Str "Pre-built connectors for Slack, GitHub, Google Workspace, Salesforce, AWS services, HubSpot, Airtable, PostgreSQL, MySQL, MongoDB, Notion, Jira, Stripe, Twilio, SendGrid, and hundreds more. Each integration node exposes the service's operations as configurable parameters."], Header 3 ("error-handling-and-retry", ["unnumbered", "unlisted"], []) [Str "Error Handling and Retry"], Para [Str "Workflows support error-handling branches where failed nodes route to dedicated error-handling paths. Automatic retry policies can be configured per workflow. Failed executions are logged with full context for debugging."], Header 3 ("sub-workflows", ["unnumbered", "unlisted"], []) [Str "Sub-Workflows"], Para [Str "Workflows can call other workflows as sub-routines, enabling modular design and reuse. The Execute Workflow node passes data to a child workflow and receives the output, supporting both synchronous and asynchronous invocation patterns."], Header 3 ("ai-and-langchain-integration", ["unnumbered", "unlisted"], []) [Str "AI and LangChain Integration"], Para [Str "Cluster nodes provide native AI capabilities through LangChain. Build AI agents that can use tools, process documents with Retrieval-Augmented Generation (RAG), chain Large Language Model (LLM) calls, and integrate vector stores -- all within the visual workflow builder."], Header 3 ("enterprise-features", ["unnumbered", "unlisted"], []) [Str "Enterprise Features"], BulletList [[Plain [Strong [Str "Source control"], Str " -- Push/pull workflow definitions to Git repositories"]], [Plain [Strong [Str "Environments"], Str " -- Promote workflows across development, staging, and production"]], [Plain [Strong [Str "External secrets"], Str " -- Integrate with HashiCorp Vault, AWS Secrets Manager, or Azure Key Vault"]], [Plain [Strong [Str "Custom roles"], Str " -- Fine-grained RBAC beyond the built-in Owner/Admin/Member roles"]], [Plain [Strong [Str "SAML/OIDC SSO"], Str " -- Enterprise identity provider integration"]], [Plain [Strong [Str "Log streaming"], Str " -- Forward execution logs to external monitoring platforms"]], [Plain [Strong [Str "Audit logging"], Str " -- Track user actions for compliance"]]], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], BulletList [[Para [Strong [Str "DevOps Automation"], Str " -- Monitor GitHub repositories, trigger CI/CD pipelines, post deployment notifications to Slack, and synchronize issue trackers across platforms."]], [Para [Strong [Str "Data Pipeline Orchestration"], Str " -- Extract data from APIs, transform it with Code nodes, load into databases or data warehouses, and schedule recurring synchronization jobs."]], [Para [Strong [Str "Customer Relationship Management (CRM) Sync"], Str " -- Bidirectionally synchronize contacts and deals between HubSpot, Salesforce, and internal databases with conflict resolution logic."]], [Para [Strong [Str "AI-Powered Document Processing"], Str " -- Ingest documents via webhooks, extract content, process with LLM-based summarization or classification, and route results to downstream systems."]], [Para [Strong [Str "Incident Response"], Str " -- Listen for alerts from monitoring tools (PagerDuty, Datadog), enrich with context from infrastructure APIs, create tickets in Jira, and notify on-call engineers via Slack or email."]], [Para [Strong [Str "E-commerce Order Management"], Str " -- Receive orders via webhook, validate inventory, process payments through Stripe, update Shopify, send confirmation emails, and trigger fulfillment workflows."]], [Para [Strong [Str "Content Publishing Pipelines"], Str " -- Aggregate content from multiple sources, transform formats, publish to Content Management Systems (CMS), social media platforms, and email marketing tools on a schedule."]]], Header 2 ("api-reference-summary", ["unnumbered", "unlisted"], []) [Str "API Reference Summary"], Para [Str "n8n exposes a public REST API for programmatic management of workflows, executions, credentials, and users. Authentication uses API keys passed via the ", Code ("", [], []) "X-N8N-API-KEY", Str " header."], Header 3 ("workflow-endpoints", ["unnumbered", "unlisted"], []) [Str "Workflow Endpoints"], CodeBlock ("", ["bash"], []) "# Create a workflow
curl -X POST \"https://your-instance.app.n8n.cloud/api/v1/workflows\" \\
  -H \"X-N8N-API-KEY: your-api-key\" \\
  -H \"Content-Type: application/json\" \\
  -d '{
    \"name\": \"Email Notification Workflow\",
    \"nodes\": [
      {
        \"id\": \"start-node\",
        \"name\": \"Start\",
        \"type\": \"n8n-nodes-base.start\",
        \"typeVersion\": 1,
        \"position\": [250, 300]
      },
      {
        \"id\": \"http-node\",
        \"name\": \"HTTP Request\",
        \"type\": \"n8n-nodes-base.httpRequest\",
        \"typeVersion\": 4,
        \"position\": [450, 300],
        \"parameters\": {
          \"method\": \"GET\",
          \"url\": \"https://api.example.com/data\"
        }
      }
    ],
    \"connections\": {
      \"Start\": {
        \"main\": [[{\"node\": \"HTTP Request\", \"type\": \"main\", \"index\": 0}]]
      }
    },
    \"settings\": {
      \"saveExecutionProgress\": true,
      \"saveManualExecutions\": true,
      \"executionTimeout\": 3600
    }
  }'
", Header 3 ("key-api-resources", ["unnumbered", "unlisted"], []) [Str "Key API Resources"], BulletList [[Plain [Code ("", [], []) "GET /api/v1/workflows", Str " -- List all workflows"]], [Plain [Code ("", [], []) "POST /api/v1/workflows", Str " -- Create a new workflow"]], [Plain [Code ("", [], []) "GET /api/v1/workflows/{id}", Str " -- Retrieve a specific workflow"]], [Plain [Code ("", [], []) "PUT /api/v1/workflows/{id}", Str " -- Update a workflow"]], [Plain [Code ("", [], []) "DELETE /api/v1/workflows/{id}", Str " -- Delete a workflow"]], [Plain [Code ("", [], []) "POST /api/v1/workflows/{id}/activate", Str " -- Activate a workflow"]], [Plain [Code ("", [], []) "POST /api/v1/workflows/{id}/deactivate", Str " -- Deactivate a workflow"]], [Plain [Code ("", [], []) "GET /api/v1/executions", Str " -- List workflow executions"]], [Plain [Code ("", [], []) "GET /api/v1/executions/{id}", Str " -- Retrieve execution details"]], [Plain [Code ("", [], []) "DELETE /api/v1/executions/{id}", Str " -- Delete an execution"]], [Plain [Code ("", [], []) "GET /api/v1/credentials", Str " -- List stored credentials"]], [Plain [Code ("", [], []) "POST /api/v1/credentials", Str " -- Create credentials"]], [Plain [Code ("", [], []) "GET /api/v1/users", Str " -- List users (admin only)"]]], Header 3 ("health-and-monitoring-endpoints", ["unnumbered", "unlisted"], []) [Str "Health and Monitoring Endpoints"], BulletList [[Plain [Code ("", [], []) "GET /healthz", Str " -- Health check (always enabled on main instance)"]], [Plain [Code ("", [], []) "GET /healthz/readiness", Str " -- Readiness probe"]], [Plain [Code ("", [], []) "GET /metrics", Str " -- Prometheus-format metrics (requires ", Code ("", [], []) "N8N_METRICS=true", Str ")"]]], Header 2 ("configuration-and-customization", ["unnumbered", "unlisted"], []) [Str "Configuration and Customization"], Header 3 ("key-environment-variables", ["unnumbered", "unlisted"], []) [Str "Key Environment Variables"], CodeBlock ("", ["bash"], []) "# General
N8N_HOST=n8n.example.com          # Hostname for the instance
N8N_PORT=5678                      # HTTP port (default: 5678)
N8N_PROTOCOL=https                 # Protocol: http or https
WEBHOOK_URL=https://n8n.example.com/  # External webhook URL
N8N_ENCRYPTION_KEY=your-secret-key    # Credential encryption key

# Database
DB_TYPE=postgresdb                 # Database type: sqlite or postgresdb
DB_POSTGRESDB_HOST=localhost       # PostgreSQL host
DB_POSTGRESDB_PORT=5432            # PostgreSQL port
DB_POSTGRESDB_DATABASE=n8n         # Database name
DB_POSTGRESDB_USER=n8n             # Database user
DB_POSTGRESDB_PASSWORD=secret      # Database password
DB_POSTGRESDB_SCHEMA=public        # Schema name

# PostgreSQL SSL (optional)
DB_POSTGRESDB_SSL_CA=/path/to/ca.crt
DB_POSTGRESDB_SSL_REJECT_UNAUTHORIZED=false

# Timezone
GENERIC_TIMEZONE=UTC               # Workflow timezone
TZ=UTC                             # System timezone

# Execution
N8N_RUNNERS_ENABLED=true           # Enable sandboxed task runners
EXECUTIONS_TIMEOUT=3600            # Default execution timeout (seconds)
EXECUTIONS_DATA_MAX_AGE=168        # Execution history retention (hours)

# Queue Mode (for scaling)
EXECUTIONS_MODE=queue              # Enable queue mode
QUEUE_BULL_REDIS_HOST=redis-host   # Redis host for queue
QUEUE_BULL_REDIS_PORT=6379         # Redis port
QUEUE_HEALTH_CHECK_ACTIVE=true     # Health check on workers

# Monitoring
N8N_METRICS=true                   # Enable /metrics endpoint
N8N_ENDPOINT_HEALTH=health         # Custom health endpoint path

# Security
N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true  # File permission checks
", Header 3 ("custom-nodes", ["unnumbered", "unlisted"], []) [Str "Custom Nodes"], Para [Str "Install community nodes from npm to extend n8n's integration library:"], CodeBlock ("", ["bash"], []) "# Inside the n8n data directory
cd /home/node/.n8n
npm install n8n-nodes-custom-package
", Para [Str "Community nodes appear alongside built-in nodes in the editor after restarting n8n."], Header 3 ("external-secrets", ["unnumbered", "unlisted"], []) [Str "External Secrets"], Para [Str "Enterprise instances can reference secrets from external vaults instead of storing credentials directly:"], BulletList [[Plain [Str "HashiCorp Vault"]], [Plain [Str "AWS Secrets Manager"]], [Plain [Str "Azure Key Vault"]]], Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Header 3 ("webhook-driven-automation", ["unnumbered", "unlisted"], []) [Str "Webhook-Driven Automation"], Para [Str "Receive events from external systems and process them in real time:"], OrderedList (1, DefaultStyle, DefaultDelim) [[Plain [Str "Add a ", Strong [Str "Webhook"], Str " trigger node to create an HTTP endpoint"]], [Plain [Str "Configure the expected method (POST, GET) and authentication"]], [Plain [Str "Process incoming payloads through transformation and action nodes"]], [Plain [Str "Respond synchronously if needed using the ", Strong [Str "Respond to Webhook"], Str " node"]]], Header 3 ("scheduled-batch-processing", ["unnumbered", "unlisted"], []) [Str "Scheduled Batch Processing"], Para [Str "Run workflows on a recurring schedule:"], OrderedList (1, DefaultStyle, DefaultDelim) [[Plain [Str "Add a ", Strong [Str "Schedule Trigger"], Str " node with a cron expression"]], [Plain [Str "Fetch data from external APIs or databases"]], [Plain [Str "Transform, filter, and aggregate data"]], [Plain [Str "Load results into destination systems"]]], Header 3 ("event-driven-chaining", ["unnumbered", "unlisted"], []) [Str "Event-Driven Chaining"], Para [Str "Chain multiple workflows for modular processing:"], OrderedList (1, DefaultStyle, DefaultDelim) [[Plain [Str "Parent workflow receives an event and performs initial processing"]], [Plain [Strong [Str "Execute Workflow"], Str " node calls child workflows with relevant data"]], [Plain [Str "Child workflows return results to the parent for aggregation"]], [Plain [Str "Error-handling branches capture and route failures independently"]]], Header 3 ("polling-pattern", ["unnumbered", "unlisted"], []) [Str "Polling Pattern"], Para [Str "For services without webhook support:"], OrderedList (1, DefaultStyle, DefaultDelim) [[Plain [Strong [Str "Schedule Trigger"], Str " fires at regular intervals"]], [Plain [Str "Fetch latest records from the external service"]], [Plain [Str "Compare with previously processed records (using a database or n8n's static data)"]], [Plain [Str "Process only new or changed records"]]], Header 3 ("ai-agent-pattern", ["unnumbered", "unlisted"], []) [Str "AI Agent Pattern"], Para [Str "Build autonomous AI agents within workflows:"], OrderedList (1, DefaultStyle, DefaultDelim) [[Plain [Strong [Str "Webhook"], Str " or ", Strong [Str "Chat Trigger"], Str " receives user input"]], [Plain [Strong [Str "AI Agent"], Str " cluster node processes the input with an LLM"]], [Plain [Str "Agent uses ", Strong [Str "Tool"], Str " nodes to call APIs, query databases, or search documents"]], [Plain [Strong [Str "Memory"], Str " nodes maintain conversation context across interactions"]], [Plain [Str "Response is sent back through the trigger node"]]], Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Header 3 ("data-transformation-with-code-node", ["unnumbered", "unlisted"], []) [Str "Data Transformation with Code Node"], Para [Str "Process API data, count categories, and format a Slack notification:"], CodeBlock ("", ["javascript"], []) "const submissions = $input.all();

// Count categories
let ideaCount = 0;
let featureCount = 0;
let bugCount = 0;

submissions.forEach((submission) => {
  switch (submission.json.property_type[0]) {
    case \"Idea\":
      ideaCount++;
      break;
    case \"Feature\":
      featureCount++;
      break;
    case \"Bug\":
      bugCount++;
      break;
  }
});

// Sort by votes and take top 5
const topSubmissions = submissions
  .sort((a, b) => b.json.property_votes - a.json.property_votes)
  .slice(0, 5);

let topSubmissionText = \"\";
topSubmissions.forEach((submission) => {
  topSubmissionText += `<${submission.json.url}|${submission.json.name}> - ${submission.json.property_votes} votes\\n`;
});

const slackMessage =
  `*Weekly Submission Summary*\\n\\n` +
  `Ideas: ${ideaCount} | Features: ${featureCount} | Bugs: ${bugCount}\\n\\n` +
  `*Top 5 Submissions:*\\n${topSubmissionText}`;

return [{ json: { slackMessage } }];
", Header 3 ("jmespath-expressions-for-data-extraction", ["unnumbered", "unlisted"], []) [Str "JMESPath Expressions for Data Extraction"], Para [Str "Use JMESPath within n8n expressions to query nested JSON structures:"], CodeBlock ("", ["javascript"], []) "// In an expression field: extract all first names from a people array
{{ $jmespath($json.body.people, \"[*].first\") }}

// In a Code node: same operation with variable assignment
let firstNames = $jmespath($json.body.people, \"[*].first\");
return { firstNames };
", Header 3 ("creating-a-workflow-via-the-rest-api", ["unnumbered", "unlisted"], []) [Str "Creating a Workflow via the REST API"], CodeBlock ("", ["bash"], []) "curl -X POST \"https://n8n.example.com/api/v1/workflows\" \\
  -H \"X-N8N-API-KEY: n8n_api_abc123\" \\
  -H \"Content-Type: application/json\" \\
  -d '{
    \"name\": \"Daily Report Generator\",
    \"nodes\": [
      {
        \"id\": \"schedule-1\",
        \"name\": \"Daily Schedule\",
        \"type\": \"n8n-nodes-base.scheduleTrigger\",
        \"typeVersion\": 1,
        \"position\": [250, 300],
        \"parameters\": {
          \"rule\": {
            \"interval\": [{ \"field\": \"cronExpression\", \"expression\": \"0 9 * * *\" }]
          }
        }
      },
      {
        \"id\": \"http-1\",
        \"name\": \"Fetch Metrics\",
        \"type\": \"n8n-nodes-base.httpRequest\",
        \"typeVersion\": 4,
        \"position\": [450, 300],
        \"parameters\": {
          \"method\": \"GET\",
          \"url\": \"https://api.internal.example.com/metrics/daily\"
        }
      },
      {
        \"id\": \"code-1\",
        \"name\": \"Format Report\",
        \"type\": \"n8n-nodes-base.code\",
        \"typeVersion\": 2,
        \"position\": [650, 300],
        \"parameters\": {
          \"jsCode\": \"const metrics = $input.first().json;\\nreturn [{ json: { report: `Daily active users: ${metrics.dau}` } }];\"
        }
      }
    ],
    \"connections\": {
      \"Daily Schedule\": {
        \"main\": [[{ \"node\": \"Fetch Metrics\", \"type\": \"main\", \"index\": 0 }]]
      },
      \"Fetch Metrics\": {
        \"main\": [[{ \"node\": \"Format Report\", \"type\": \"main\", \"index\": 0 }]]
      }
    },
    \"settings\": {
      \"saveExecutionProgress\": true,
      \"executionTimeout\": 300
    }
  }'
", Header 3 ("postgresql-configuration-with-ssl", ["unnumbered", "unlisted"], []) [Str "PostgreSQL Configuration with SSL"], CodeBlock ("", ["bash"], []) "export DB_TYPE=postgresdb
export DB_POSTGRESDB_DATABASE=n8n
export DB_POSTGRESDB_HOST=db.example.com
export DB_POSTGRESDB_PORT=5432
export DB_POSTGRESDB_USER=n8n
export DB_POSTGRESDB_PASSWORD=secure-password
export DB_POSTGRESDB_SCHEMA=n8n
export DB_POSTGRESDB_SSL_CA=$(pwd)/ca.crt
export DB_POSTGRESDB_SSL_REJECT_UNAUTHORIZED=false

n8n start
", Header 3 ("google-cloud-run-deployment", ["unnumbered", "unlisted"], []) [Str "Google Cloud Run Deployment"], CodeBlock ("", ["bash"], []) "gcloud run deploy n8n \\
  --image=n8nio/n8n:latest \\
  --command=\"/bin/sh\" \\
  --args=\"-c,sleep 5;n8n start\" \\
  --region=$REGION \\
  --allow-unauthenticated \\
  --port=5678 \\
  --memory=2Gi \\
  --no-cpu-throttling \\
  --set-env-vars=\"N8N_PORT=5678,N8N_PROTOCOL=https,DB_TYPE=postgresdb,DB_POSTGRESDB_DATABASE=n8n,DB_POSTGRESDB_USER=n8n-user,DB_POSTGRESDB_HOST=/cloudsql/$PROJECT_ID:$REGION:n8n-db,DB_POSTGRESDB_PORT=5432,GENERIC_TIMEZONE=UTC,QUEUE_HEALTH_CHECK_ACTIVE=true\" \\
  --set-secrets=\"DB_POSTGRESDB_PASSWORD=n8n-db-password:latest,N8N_ENCRYPTION_KEY=n8n-encryption-key:latest\" \\
  --add-cloudsql-instances=$PROJECT_ID:$REGION:n8n-db \\
  --service-account=n8n-sa@$PROJECT_ID.iam.gserviceaccount.com
", Header 2 ("limitations-and-considerations", ["unnumbered", "unlisted"], []) [Str "Limitations and Considerations"], BulletList [[Para [Strong [Str "Execution Memory"], Str " -- Each workflow execution loads all item data into memory. Workflows processing large datasets (millions of rows) can exhaust available memory. Use pagination, batching, and streaming patterns to handle large volumes."]], [Para [Strong [Str "SQLite Limitations"], Str " -- The default SQLite database does not support concurrent writes well and is not suitable for production deployments with multiple users or high execution volumes. PostgreSQL is recommended for production."]], [Para [Strong [Str "Queue Mode Complexity"], Str " -- Horizontal scaling with queue mode requires Redis infrastructure and careful configuration. Worker instances need access to the same encryption key, database, and filesystem (for binary data) as the main instance."]], [Para [Strong [Str "Code Node Sandboxing"], Str " -- While task runners provide isolation, the Code node's sandboxing has limitations. Untrusted code should not be executed without additional security measures. Network access from Code nodes is not restricted by default."]], [Para [Strong [Str "Fair-Code License"], Str " -- n8n uses the Sustainable Use License for its source code, which allows self-hosting and modification but includes restrictions on offering n8n as a managed service to third parties. This is not a traditional open-source license (e.g., MIT, Apache 2.0)."]], [Para [Strong [Str "Credential Migration"], Str " -- Moving between instances requires the same ", Code ("", [], []) "N8N_ENCRYPTION_KEY", Str ". Losing this key means all stored credentials become unrecoverable."]], [Para [Strong [Str "Webhook URL Stability"], Str " -- Self-hosted webhook URLs depend on the instance's hostname. Changing domains or infrastructure requires updating all external systems that send webhooks to n8n."]], [Para [Strong [Str "Node Version Compatibility"], Str " -- Some community nodes may lag behind n8n core version updates, causing compatibility issues after upgrades. Pin n8n versions in production and test upgrades in a staging environment."]], [Para [Strong [Str "Execution History Storage"], Str " -- All execution data is stored in the database by default. High-volume workflows can rapidly grow database size. Configure ", Code ("", [], []) "EXECUTIONS_DATA_MAX_AGE", Str " and ", Code ("", [], []) "EXECUTIONS_DATA_PRUNE", Str " to manage retention."]]], Header 2 ("changelog-highlights", ["unnumbered", "unlisted"], []) [Str "Changelog Highlights"], BulletList [[Plain [Strong [Str "1.0 (2024)"], Str " -- General availability release with stabilized REST API, RBAC, and enterprise features."]], [Plain [Strong [Str "AI Nodes"], Str " -- Introduction of LangChain-based AI Agent, Chain, Memory, and Tool cluster nodes for building AI-powered workflows."]], [Plain [Strong [Str "Task Runners"], Str " -- Sandboxed execution environment for Code nodes (", Code ("", [], []) "N8N_RUNNERS_ENABLED", Str "), improving security by isolating user code from the main n8n process."]], [Plain [Strong [Str "Python Support"], Str " -- Code node expanded from JavaScript-only to include Python as a supported language."]], [Plain [Strong [Str "Source Control"], Str " -- Git-based version control integration for pushing and pulling workflow definitions across environments."]], [Plain [Strong [Str "External Secrets"], Str " -- Native integration with HashiCorp Vault, AWS Secrets Manager, and Azure Key Vault for credential management."]], [Plain [Strong [Str "Queue Mode"], Str " -- Redis-backed execution queue for horizontal scaling with dedicated worker instances."]], [Plain [Strong [Str "Community Nodes"], Str " -- npm-based ecosystem for installing third-party node packages directly from the n8n editor."]]], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Link ("", [], []) [Str "n8n Official Documentation"] ("https://docs.n8n.io/", "")]], [Plain [Link ("", [], []) [Str "n8n GitHub Repository"] ("https://github.com/n8n-io/n8n", "")]], [Plain [Link ("", [], []) [Str "n8n Hosting and Installation Guide"] ("https://docs.n8n.io/hosting/", "")]], [Plain [Link ("", [], []) [Str "n8n Code Node Reference"] ("https://docs.n8n.io/code/", "")]], [Plain [Link ("", [], []) [Str "n8n REST API Documentation"] ("https://docs.n8n.io/api/", "")]], [Plain [Link ("", [], []) [Str "n8n Integrations Library"] ("https://docs.n8n.io/integrations/", "")]], [Plain [Link ("", [], []) [Str "n8n AI and LangChain Documentation"] ("https://docs.n8n.io/advanced-ai/", "")]], [Plain [Link ("", [], []) [Str "n8n Configuration Reference"] ("https://docs.n8n.io/hosting/configuration/", "")]]]]