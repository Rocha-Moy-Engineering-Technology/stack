# n8n

> Workflow automation combining visual building with custom code and 400+ integrations

| Field | Value |
|-------|-------|
| Group | Workflow Orchestration |
| Type | API/SDK/UI |
| Open Source | Yes |
| GitHub | [https://github.com/n8n-io/n8n](https://github.com/n8n-io/n8n) |
| Stars | 187836 |
| Documentation | [Official Docs](https://docs.n8n.io/) |

## Overview

n8n (pronounced "nodemation") is a fair-code licensed workflow automation platform that connects applications and automates business processes through a visual node-based editor. It bridges the gap between no-code simplicity and full programmatic control by allowing users to build workflows visually while embedding custom JavaScript or Python at any point. The platform provides over 400 built-in integrations, native AI capabilities through LangChain integration, and flexible deployment options including a managed cloud service and self-hosted installations.

n8n targets technical teams that need both the speed of drag-and-drop workflow building and the flexibility to write code when standard nodes are insufficient. Workflows are represented as directed graphs of interconnected nodes, where data flows from trigger nodes through processing and action nodes. Every workflow execution is tracked, enabling debugging, retry logic, and auditing. The platform supports Role-Based Access Control (RBAC), source control integration, multi-environment promotion, and enterprise Single Sign-On (SSO) via Security Assertion Markup Language (SAML) and OpenID Connect (OIDC).

## Core Concepts

- **Workflows** -- The top-level automation unit. A workflow is a collection of interconnected nodes that defines a sequence of operations. Workflows can be activated for production execution, run manually for testing, or triggered by external events. Each workflow maintains its own settings for timeouts, error handling, and execution history.

- **Nodes** -- Individual processing units within a workflow. Nodes fall into four categories:
  - **Core nodes** -- Built-in utilities for data manipulation: Code (JavaScript/Python), Filter, Merge, Split Out, Set, IF, Switch, Remove Duplicates, and more.
  - **Trigger nodes** -- Entry points that start workflow execution: Schedule Trigger (cron), Webhook, Email Trigger, Manual Trigger, and event-based triggers from third-party services.
  - **Action nodes** -- Connectors to external services (400+ integrations): Slack, GitHub, Google Sheets, Salesforce, Amazon Web Services (AWS), HubSpot, Airtable, PostgreSQL, and many others.
  - **Cluster nodes** -- Specialized nodes for AI and data processing, including LangChain-based AI Agent, Chain, and Tool nodes.

- **Connections** -- Links between nodes that define data flow. A node can have multiple output branches (for conditional routing) and multiple inputs (for data merging). Connections carry structured JSON data between nodes.

- **Credentials** -- Securely stored authentication details for third-party services. Credentials are encrypted at rest, managed centrally, and can be shared across workflows and team members. n8n supports OAuth2, API keys, HTTP header auth, and service-specific authentication methods.

- **Expressions** -- Dynamic references to data within node parameters. Expressions use the syntax `{{ }}` and can access data from previous nodes, environment variables, and built-in functions. JMESPath is available for advanced JSON querying.

- **Executions** -- Individual runs of a workflow. Execution modes include manual (triggered by the user in the editor), partial (running a subset of nodes for debugging), and production (triggered automatically by events or schedules). Execution history is retained for inspection and retry.

- **Items** -- The fundamental data unit passed between nodes. Each item is a JSON object optionally accompanied by binary data (files, images). Nodes process items individually or in batches depending on configuration.

## Architecture

n8n is built as a Node.js application with a TypeScript codebase. The architecture consists of several key components:

- **Workflow Engine** -- The core execution runtime that interprets workflow definitions, manages node execution order, handles data passing between nodes, and tracks execution state. The engine supports both sequential and parallel node execution based on the workflow graph topology.

- **Node Framework** -- A pluggable system where each integration is an independent node package. Nodes implement a standard interface with `execute()` methods, parameter definitions, and credential requirements. Custom community nodes can be installed via npm.

- **Editor UI** -- A Vue.js-based single-page application providing the visual workflow builder. The editor communicates with the backend via a REST API and WebSocket connections for real-time execution feedback.

- **Database Layer** -- Stores workflow definitions, execution history, credentials (encrypted), user data, and settings. Supports SQLite (development/small deployments) and PostgreSQL (production).

- **Webhook Server** -- A dedicated HTTP server that listens for incoming webhook requests and routes them to the appropriate workflow triggers.

- **Task Runners** -- Isolated execution environments for Code nodes. When `N8N_RUNNERS_ENABLED=true`, JavaScript and Python code runs in sandboxed processes rather than the main n8n thread, improving security and stability.

- **Queue Mode** -- For horizontal scaling, n8n supports a queue-based architecture using Redis (or compatible) as a message broker. A main instance handles webhooks and scheduling while worker instances pull executions from the queue. This separates concerns and allows independent scaling of execution capacity.

- **Encryption** -- Credentials are encrypted using an encryption key (`N8N_ENCRYPTION_KEY`). This key must be consistent across all instances in a multi-node deployment and must be preserved during migrations.

## Key Features and Functionality

### Visual Workflow Builder

The drag-and-drop editor allows building complex automations without writing code. Nodes are placed on a canvas and connected with wires. Each node has a configuration panel for setting parameters, mapping data, and defining behavior. The editor provides real-time execution previews showing data at each node.

### Code Node (JavaScript and Python)

When built-in nodes are insufficient, the Code node allows writing custom JavaScript or Python directly within the workflow:

```javascript
// JavaScript: Transform input items
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
```

```python
# Python: Access data using underscore-prefixed globals
first_names = _jmespath(_json.body.people, "[*].first")
return {"firstNames": first_names}
```

### 400+ Integrations

Pre-built connectors for Slack, GitHub, Google Workspace, Salesforce, AWS services, HubSpot, Airtable, PostgreSQL, MySQL, MongoDB, Notion, Jira, Stripe, Twilio, SendGrid, and hundreds more. Each integration node exposes the service's operations as configurable parameters.

### Error Handling and Retry

Workflows support error-handling branches where failed nodes route to dedicated error-handling paths. Automatic retry policies can be configured per workflow. Failed executions are logged with full context for debugging.

### Sub-Workflows

Workflows can call other workflows as sub-routines, enabling modular design and reuse. The Execute Workflow node passes data to a child workflow and receives the output, supporting both synchronous and asynchronous invocation patterns.

### AI and LangChain Integration

Cluster nodes provide native AI capabilities through LangChain. Build AI agents that can use tools, process documents with Retrieval-Augmented Generation (RAG), chain Large Language Model (LLM) calls, and integrate vector stores -- all within the visual workflow builder.

### Enterprise Features

- **Source control** -- Push/pull workflow definitions to Git repositories
- **Environments** -- Promote workflows across development, staging, and production
- **External secrets** -- Integrate with HashiCorp Vault, AWS Secrets Manager, or Azure Key Vault
- **Custom roles** -- Fine-grained RBAC beyond the built-in Owner/Admin/Member roles
- **SAML/OIDC SSO** -- Enterprise identity provider integration
- **Log streaming** -- Forward execution logs to external monitoring platforms
- **Audit logging** -- Track user actions for compliance

## Use Cases

- **DevOps Automation** -- Monitor GitHub repositories, trigger CI/CD pipelines, post deployment notifications to Slack, and synchronize issue trackers across platforms.

- **Data Pipeline Orchestration** -- Extract data from APIs, transform it with Code nodes, load into databases or data warehouses, and schedule recurring synchronization jobs.

- **Customer Relationship Management (CRM) Sync** -- Bidirectionally synchronize contacts and deals between HubSpot, Salesforce, and internal databases with conflict resolution logic.

- **AI-Powered Document Processing** -- Ingest documents via webhooks, extract content, process with LLM-based summarization or classification, and route results to downstream systems.

- **Incident Response** -- Listen for alerts from monitoring tools (PagerDuty, Datadog), enrich with context from infrastructure APIs, create tickets in Jira, and notify on-call engineers via Slack or email.

- **E-commerce Order Management** -- Receive orders via webhook, validate inventory, process payments through Stripe, update Shopify, send confirmation emails, and trigger fulfillment workflows.

- **Content Publishing Pipelines** -- Aggregate content from multiple sources, transform formats, publish to Content Management Systems (CMS), social media platforms, and email marketing tools on a schedule.

## API Reference Summary

n8n exposes a public REST API for programmatic management of workflows, executions, credentials, and users. Authentication uses API keys passed via the `X-N8N-API-KEY` header.

### Workflow Endpoints

```bash
# Create a workflow
curl -X POST "https://your-instance.app.n8n.cloud/api/v1/workflows" \
  -H "X-N8N-API-KEY: your-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Email Notification Workflow",
    "nodes": [
      {
        "id": "start-node",
        "name": "Start",
        "type": "n8n-nodes-base.start",
        "typeVersion": 1,
        "position": [250, 300]
      },
      {
        "id": "http-node",
        "name": "HTTP Request",
        "type": "n8n-nodes-base.httpRequest",
        "typeVersion": 4,
        "position": [450, 300],
        "parameters": {
          "method": "GET",
          "url": "https://api.example.com/data"
        }
      }
    ],
    "connections": {
      "Start": {
        "main": [[{"node": "HTTP Request", "type": "main", "index": 0}]]
      }
    },
    "settings": {
      "saveExecutionProgress": true,
      "saveManualExecutions": true,
      "executionTimeout": 3600
    }
  }'
```

### Key API Resources

- `GET /api/v1/workflows` -- List all workflows
- `POST /api/v1/workflows` -- Create a new workflow
- `GET /api/v1/workflows/{id}` -- Retrieve a specific workflow
- `PUT /api/v1/workflows/{id}` -- Update a workflow
- `DELETE /api/v1/workflows/{id}` -- Delete a workflow
- `POST /api/v1/workflows/{id}/activate` -- Activate a workflow
- `POST /api/v1/workflows/{id}/deactivate` -- Deactivate a workflow
- `GET /api/v1/executions` -- List workflow executions
- `GET /api/v1/executions/{id}` -- Retrieve execution details
- `DELETE /api/v1/executions/{id}` -- Delete an execution
- `GET /api/v1/credentials` -- List stored credentials
- `POST /api/v1/credentials` -- Create credentials
- `GET /api/v1/users` -- List users (admin only)

### Health and Monitoring Endpoints

- `GET /healthz` -- Health check (always enabled on main instance)
- `GET /healthz/readiness` -- Readiness probe
- `GET /metrics` -- Prometheus-format metrics (requires `N8N_METRICS=true`)

## Configuration and Customization

### Key Environment Variables

```bash
# General
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
```

### Custom Nodes

Install community nodes from npm to extend n8n's integration library:

```bash
# Inside the n8n data directory
cd /home/node/.n8n
npm install n8n-nodes-custom-package
```

Community nodes appear alongside built-in nodes in the editor after restarting n8n.

### External Secrets

Enterprise instances can reference secrets from external vaults instead of storing credentials directly:

- HashiCorp Vault
- AWS Secrets Manager
- Azure Key Vault

## Integration Patterns

### Webhook-Driven Automation

Receive events from external systems and process them in real time:

1. Add a **Webhook** trigger node to create an HTTP endpoint
2. Configure the expected method (POST, GET) and authentication
3. Process incoming payloads through transformation and action nodes
4. Respond synchronously if needed using the **Respond to Webhook** node

### Scheduled Batch Processing

Run workflows on a recurring schedule:

1. Add a **Schedule Trigger** node with a cron expression
2. Fetch data from external APIs or databases
3. Transform, filter, and aggregate data
4. Load results into destination systems

### Event-Driven Chaining

Chain multiple workflows for modular processing:

1. Parent workflow receives an event and performs initial processing
2. **Execute Workflow** node calls child workflows with relevant data
3. Child workflows return results to the parent for aggregation
4. Error-handling branches capture and route failures independently

### Polling Pattern

For services without webhook support:

1. **Schedule Trigger** fires at regular intervals
2. Fetch latest records from the external service
3. Compare with previously processed records (using a database or n8n's static data)
4. Process only new or changed records

### AI Agent Pattern

Build autonomous AI agents within workflows:

1. **Webhook** or **Chat Trigger** receives user input
2. **AI Agent** cluster node processes the input with an LLM
3. Agent uses **Tool** nodes to call APIs, query databases, or search documents
4. **Memory** nodes maintain conversation context across interactions
5. Response is sent back through the trigger node

## Examples

### Data Transformation with Code Node

Process API data, count categories, and format a Slack notification:

```javascript
const submissions = $input.all();

// Count categories
let ideaCount = 0;
let featureCount = 0;
let bugCount = 0;

submissions.forEach((submission) => {
  switch (submission.json.property_type[0]) {
    case "Idea":
      ideaCount++;
      break;
    case "Feature":
      featureCount++;
      break;
    case "Bug":
      bugCount++;
      break;
  }
});

// Sort by votes and take top 5
const topSubmissions = submissions
  .sort((a, b) => b.json.property_votes - a.json.property_votes)
  .slice(0, 5);

let topSubmissionText = "";
topSubmissions.forEach((submission) => {
  topSubmissionText += `<${submission.json.url}|${submission.json.name}> - ${submission.json.property_votes} votes\n`;
});

const slackMessage =
  `*Weekly Submission Summary*\n\n` +
  `Ideas: ${ideaCount} | Features: ${featureCount} | Bugs: ${bugCount}\n\n` +
  `*Top 5 Submissions:*\n${topSubmissionText}`;

return [{ json: { slackMessage } }];
```

### JMESPath Expressions for Data Extraction

Use JMESPath within n8n expressions to query nested JSON structures:

```javascript
// In an expression field: extract all first names from a people array
{{ $jmespath($json.body.people, "[*].first") }}

// In a Code node: same operation with variable assignment
let firstNames = $jmespath($json.body.people, "[*].first");
return { firstNames };
```

### Creating a Workflow via the REST API

```bash
curl -X POST "https://n8n.example.com/api/v1/workflows" \
  -H "X-N8N-API-KEY: n8n_api_abc123" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Daily Report Generator",
    "nodes": [
      {
        "id": "schedule-1",
        "name": "Daily Schedule",
        "type": "n8n-nodes-base.scheduleTrigger",
        "typeVersion": 1,
        "position": [250, 300],
        "parameters": {
          "rule": {
            "interval": [{ "field": "cronExpression", "expression": "0 9 * * *" }]
          }
        }
      },
      {
        "id": "http-1",
        "name": "Fetch Metrics",
        "type": "n8n-nodes-base.httpRequest",
        "typeVersion": 4,
        "position": [450, 300],
        "parameters": {
          "method": "GET",
          "url": "https://api.internal.example.com/metrics/daily"
        }
      },
      {
        "id": "code-1",
        "name": "Format Report",
        "type": "n8n-nodes-base.code",
        "typeVersion": 2,
        "position": [650, 300],
        "parameters": {
          "jsCode": "const metrics = $input.first().json;\nreturn [{ json: { report: `Daily active users: ${metrics.dau}` } }];"
        }
      }
    ],
    "connections": {
      "Daily Schedule": {
        "main": [[{ "node": "Fetch Metrics", "type": "main", "index": 0 }]]
      },
      "Fetch Metrics": {
        "main": [[{ "node": "Format Report", "type": "main", "index": 0 }]]
      }
    },
    "settings": {
      "saveExecutionProgress": true,
      "executionTimeout": 300
    }
  }'
```

### PostgreSQL Configuration with SSL

```bash
export DB_TYPE=postgresdb
export DB_POSTGRESDB_DATABASE=n8n
export DB_POSTGRESDB_HOST=db.example.com
export DB_POSTGRESDB_PORT=5432
export DB_POSTGRESDB_USER=n8n
export DB_POSTGRESDB_PASSWORD=secure-password
export DB_POSTGRESDB_SCHEMA=n8n
export DB_POSTGRESDB_SSL_CA=$(pwd)/ca.crt
export DB_POSTGRESDB_SSL_REJECT_UNAUTHORIZED=false

n8n start
```

### Google Cloud Run Deployment

```bash
gcloud run deploy n8n \
  --image=n8nio/n8n:latest \
  --command="/bin/sh" \
  --args="-c,sleep 5;n8n start" \
  --region=$REGION \
  --allow-unauthenticated \
  --port=5678 \
  --memory=2Gi \
  --no-cpu-throttling \
  --set-env-vars="N8N_PORT=5678,N8N_PROTOCOL=https,DB_TYPE=postgresdb,DB_POSTGRESDB_DATABASE=n8n,DB_POSTGRESDB_USER=n8n-user,DB_POSTGRESDB_HOST=/cloudsql/$PROJECT_ID:$REGION:n8n-db,DB_POSTGRESDB_PORT=5432,GENERIC_TIMEZONE=UTC,QUEUE_HEALTH_CHECK_ACTIVE=true" \
  --set-secrets="DB_POSTGRESDB_PASSWORD=n8n-db-password:latest,N8N_ENCRYPTION_KEY=n8n-encryption-key:latest" \
  --add-cloudsql-instances=$PROJECT_ID:$REGION:n8n-db \
  --service-account=n8n-sa@$PROJECT_ID.iam.gserviceaccount.com
```

## Limitations and Considerations

- **Execution Memory** -- Each workflow execution loads all item data into memory. Workflows processing large datasets (millions of rows) can exhaust available memory. Use pagination, batching, and streaming patterns to handle large volumes.

- **SQLite Limitations** -- The default SQLite database does not support concurrent writes well and is not suitable for production deployments with multiple users or high execution volumes. PostgreSQL is recommended for production.

- **Queue Mode Complexity** -- Horizontal scaling with queue mode requires Redis infrastructure and careful configuration. Worker instances need access to the same encryption key, database, and filesystem (for binary data) as the main instance.

- **Code Node Sandboxing** -- While task runners provide isolation, the Code node's sandboxing has limitations. Untrusted code should not be executed without additional security measures. Network access from Code nodes is not restricted by default.

- **Fair-Code License** -- n8n uses the Sustainable Use License for its source code, which allows self-hosting and modification but includes restrictions on offering n8n as a managed service to third parties. This is not a traditional open-source license (e.g., MIT, Apache 2.0).

- **Credential Migration** -- Moving between instances requires the same `N8N_ENCRYPTION_KEY`. Losing this key means all stored credentials become unrecoverable.

- **Webhook URL Stability** -- Self-hosted webhook URLs depend on the instance's hostname. Changing domains or infrastructure requires updating all external systems that send webhooks to n8n.

- **Node Version Compatibility** -- Some community nodes may lag behind n8n core version updates, causing compatibility issues after upgrades. Pin n8n versions in production and test upgrades in a staging environment.

- **Execution History Storage** -- All execution data is stored in the database by default. High-volume workflows can rapidly grow database size. Configure `EXECUTIONS_DATA_MAX_AGE` and `EXECUTIONS_DATA_PRUNE` to manage retention.

## Changelog Highlights

- **1.0 (2024)** -- General availability release with stabilized REST API, RBAC, and enterprise features.
- **AI Nodes** -- Introduction of LangChain-based AI Agent, Chain, Memory, and Tool cluster nodes for building AI-powered workflows.
- **Task Runners** -- Sandboxed execution environment for Code nodes (`N8N_RUNNERS_ENABLED`), improving security by isolating user code from the main n8n process.
- **Python Support** -- Code node expanded from JavaScript-only to include Python as a supported language.
- **Source Control** -- Git-based version control integration for pushing and pulling workflow definitions across environments.
- **External Secrets** -- Native integration with HashiCorp Vault, AWS Secrets Manager, and Azure Key Vault for credential management.
- **Queue Mode** -- Redis-backed execution queue for horizontal scaling with dedicated worker instances.
- **Community Nodes** -- npm-based ecosystem for installing third-party node packages directly from the n8n editor.

## Citations

- [n8n Official Documentation](https://docs.n8n.io/)
- [n8n GitHub Repository](https://github.com/n8n-io/n8n)
- [n8n Hosting and Installation Guide](https://docs.n8n.io/hosting/)
- [n8n Code Node Reference](https://docs.n8n.io/code/)
- [n8n REST API Documentation](https://docs.n8n.io/api/)
- [n8n Integrations Library](https://docs.n8n.io/integrations/)
- [n8n AI and LangChain Documentation](https://docs.n8n.io/advanced-ai/)
- [n8n Configuration Reference](https://docs.n8n.io/hosting/configuration/)

## Discovery Signals

> Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search.

### Keywords

- n8n
- nodemation
- visual workflow builder
- low-code
- fair-code
- Sustainable Use License
- nodes
- triggers
- actions
- core nodes
- cluster nodes
- Code node
- JavaScript
- Python
- 400+ integrations
- credentials
- expressions
- JMESPath
- AI Agent node
- LangChain integration
- queue mode
- Redis
- task runners
- webhook trigger
- schedule trigger
- self-hosted
- Vue.js editor
- RBAC
- SAML/OIDC SSO

### Verb-Noun Tasks

- Drag-and-drop nodes to build automation flows
- Trigger workflows via webhook, schedule, or manual run
- Write custom JavaScript or Python in Code nodes
- Map data between nodes with `{{ }}` expressions and JMESPath
- Integrate with 400+ services (Slack, GitHub, Sheets, Salesforce, etc.)
- Build AI agents with LangChain cluster nodes
- Scale horizontally with Redis-backed queue mode
- Sync workflows to Git for version control
- Promote workflows across dev/staging/prod environments
- Manage secrets via external vaults (Vault, AWS, Azure)
- Enforce RBAC and enterprise SSO
- Expose Prometheus metrics for monitoring

### User Intent Phrases

- I want a visual workflow builder that lets me drop into code when I need to.
- How do I automate Slack notifications when a GitHub issue is opened?
- I need to bridge no-code and JavaScript/Python in the same flow.
- How do I run AI agents inside a visual builder?
- I want to self-host an n8n instance on Kubernetes or Cloud Run.
- How do I scale n8n horizontally with worker instances?
- I need a Zapier alternative I can host myself.
- How do I version-control my workflows in Git?
- I want LangChain agents integrated into business automation flows.
- How do I expose webhook endpoints for third-party services?

### Problem Statements

- Pure code orchestrators are overkill for simple business automations.
- Pure no-code tools hit walls when transformations get complex.
- Zapier/Make charge per execution and we want flat-rate self-hosting.
- AI agents are siloed from the rest of business workflows.
- Manual data entry between SaaS tools wastes hours.
- Webhook bridges and CRM syncs need quick iteration with debugging.

### When to Pick This

- Pick this when your team is technical but wants visual workflow building with code escape hatches.
- Pick this over Activepieces when you want a larger ecosystem (400+ integrations) and embedded JS/Python Code nodes with LangChain cluster nodes.
- Pick this over Node-RED when business automation and SaaS integrations — not IoT protocols — are the primary use case.
- Pick this over Airflow/Prefect/Temporal when you want a visual+code hybrid, not code-first orchestration.
- Pick this over fully no-code tools (Zapier, Make) when you need self-hosting and the Code node escape hatch.
- Pick this when fair-code self-hosting is acceptable (note: Sustainable Use License, not OSI-approved).

### Related Terms and Aliases

- n8n.io
- n8n-nodes-base
- N8N_RUNNERS_ENABLED
- N8N_ENCRYPTION_KEY
- EXECUTIONS_MODE=queue
- Bull queue (Redis)
- workflow automation
- iPaaS alternative
- Zapier alternative
- Make alternative
- LangChain cluster node
- AI Agent / Chain / Memory / Tool nodes
- Execute Workflow node
- community nodes

