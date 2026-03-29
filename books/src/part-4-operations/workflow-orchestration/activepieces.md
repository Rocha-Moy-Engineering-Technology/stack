# Activepieces

> Open-source no-code business automation with 635+ integrations

| Field | Value |
|-------|-------|
| Group | Workflow Orchestration & Automation |
| Type | API/UI |
| Open Source | Yes |
| GitHub | [https://github.com/activepieces/activepieces](https://github.com/activepieces/activepieces) |
| Stars | 20928 |
| Documentation | [Official Docs](https://www.activepieces.com/docs/) |

## Overview

Activepieces is an open-source, no-code automation platform designed to be extensible through a community-driven integration ecosystem. It enables users to build automation workflows called Flows, which consist of Triggers that initiate execution and Actions that perform tasks. The platform ships with over 635 integrations (called Pieces), approximately 60% of which are contributed by the community.

Built in TypeScript and distributed as a self-hosted application, Activepieces targets both technical and non-technical users who need to automate business processes without writing code. It offers a visual flow builder through its web UI alongside a comprehensive REST API for programmatic control. The platform supports full network isolation for security-sensitive deployments and provides an Embedding SDK that allows SaaS vendors to white-label the automation builder within their own products.

Activepieces differentiates itself from other workflow automation tools through its open piece ecosystem (npm packages in TypeScript), native AI integration capabilities, and enterprise features including multi-tenancy, Role-Based Access Control (RBAC), Single Sign-On (SSO), audit logging, and Git-based deployment workflows with project releases for staging and production environments.

## Core Concepts

**Flows** are the primary automation unit in Activepieces. Each flow defines a complete automation workflow consisting of exactly one trigger and one or more actions. Flows can be enabled or disabled, versioned, and organized into folders within a project. Data flows through the steps sequentially, with each action able to reference outputs from any preceding step.

**Triggers** initiate flow execution. Activepieces supports two trigger types: polling triggers that periodically check an external endpoint for new data, and webhook triggers that listen for incoming HTTP requests on a dedicated URL. Triggers define when and how a flow starts running.

**Actions** perform discrete operations within a flow. Each action receives data from previous steps, executes its logic (such as calling an external API, transforming data, or sending a notification), and produces output that subsequent steps can consume. Actions can include branching logic, loops, and delays.

**Pieces** are reusable integration packages that provide triggers and actions for specific applications or services. Each piece is an npm package written in TypeScript that defines authentication requirements, available triggers, and available actions. Pieces can be official (maintained by the Activepieces team), community-contributed (published via npm), or private (shared within an organization).

**Connections** store authentication credentials for external services. When a piece requires access to a third-party API, a connection holds the necessary tokens, API keys, or OAuth2 credentials. Connections are scoped to a project and can be managed through the UI or REST API.

**Projects** provide organizational isolation within an Activepieces instance. Each project contains its own flows, connections, and members. Projects enable multi-tenancy where different teams or customers operate independently within the same platform deployment.

**Flow Runs** represent individual executions of a flow. Each run tracks the trigger event, the execution status of every step, input and output data at each step, duration, and any errors encountered. Flow runs provide the audit trail for automation execution.

## Architecture

Activepieces follows a monolithic application architecture packaged as a single Docker image that contains the frontend UI, backend API server, and flow execution engine. The architecture separates concerns across three external dependencies:

1. **Database (PostgreSQL)** -- Stores all persistent data: flows, connections, projects, users, flow run history, and piece metadata. In development mode, PGLite provides an embedded alternative that requires no external database process.

2. **Queue (Redis)** -- Manages flow execution scheduling, trigger polling, and worker coordination. In development mode, an in-memory queue substitutes for Redis, limiting the deployment to a single instance.

3. **Frontend (Angular/React UI)** -- The visual flow builder and administration interface served as a single-page application from the same container.

The execution model processes flows as follows:

1. A trigger fires (webhook received or polling detects new data)
2. The flow execution is enqueued in the queue system
3. A worker picks up the execution and runs each action step sequentially
4. Each step's input is resolved from previous step outputs using expression references
5. Step results are persisted to the database as part of the flow run record
6. On completion or failure, the flow run status is updated

The Embedding SDK operates through JSON Web Token (JWT) authentication. The host application generates a JWT that identifies the customer, and the SDK uses this token to render the Activepieces builder within an iframe, providing white-label automation capabilities.

Git Sync enables version-controlled flow management. Flows can be exported to a Git repository and promoted across environments (staging to production) through project releases, supporting controlled deployment workflows.

## Key Features and Functionality

**Visual Flow Builder** provides a drag-and-drop interface for assembling automation workflows. Users select triggers and actions from the piece catalog, configure each step through forms, and map data between steps using expression references. The builder supports branching, loops, and conditional logic without code.

**635+ Integrations** span productivity tools, CRMs, databases, messaging platforms, cloud services, and AI providers. The piece catalog covers applications such as Google Workspace, Slack, Salesforce, PostgreSQL, OpenAI, and hundreds more. Each integration is a self-contained npm package with defined authentication and operations.

**Native AI Pieces and Agents** embed AI capabilities directly into automation workflows. Users can incorporate Large Language Model (LLM) calls, text generation, classification, and other AI operations as standard flow actions without external tooling or custom code.

**Human-in-the-Loop Approvals** allow flows to pause and wait for manual approval before continuing execution. Combined with configurable time delays, this enables workflows where sensitive operations require human oversight before proceeding.

**Embedding SDK** enables SaaS vendors to embed the Activepieces flow builder within their own applications. The SDK handles user provisioning through JWT tokens, supports piece visibility customization, predefined connections, and navigation control. This provides a white-label automation experience for end users.

**REST API** exposes programmatic control over all platform resources: projects, users, connections, flows, flow runs, and pieces. The API supports automation of platform management, integration with CI/CD pipelines, and building custom administrative interfaces.

**Git Sync and Project Releases** enable version-controlled flow management. Flows are exported to a Git repository, reviewed through standard pull request workflows, and promoted between environments using project releases. This supports staging-to-production deployment patterns.

**Audit Logging** tracks platform events across flows, connections, folders, and user activities. Audit logs provide compliance visibility and troubleshooting capabilities for enterprise deployments.

**Multi-Tenancy and RBAC** allow a single Activepieces instance to serve multiple isolated projects with distinct user roles and permissions. SSO integration supports enterprise identity providers. Custom branding enables platform operators to tailor the UI appearance.

**Hot Reloading for Piece Development** accelerates the creation of custom integrations. Developers working on new pieces see changes reflected immediately without restarting the platform, reducing the feedback loop during integration development.

## Use Cases

- **Business process automation**: Connecting CRM, email, project management, and communication tools into automated workflows that eliminate manual data entry and handoffs
- **SaaS product integration layer**: Embedding the flow builder into a SaaS product to provide customers with native automation capabilities without building a workflow engine from scratch
- **AI-powered workflows**: Incorporating LLM calls into business processes for content generation, data classification, summarization, or intelligent routing
- **Approval workflows**: Building multi-step processes that require human review at critical decision points before proceeding with automated actions
- **Data synchronization**: Keeping data consistent across multiple systems by triggering sync flows when records are created, updated, or deleted in any connected application
- **DevOps automation**: Automating incident response, deployment notifications, and infrastructure monitoring alerts through webhook-triggered flows
- **Customer onboarding**: Orchestrating multi-step onboarding sequences that span email, CRM updates, access provisioning, and notification delivery

## API Reference Summary

All endpoints use the base path `/v1/` and require authentication via API key or JWT token.

**Projects**:

- `POST /v1/projects` -- Create a new project
- `POST /v1/projects/{id}` -- Update a project
- `GET /v1/projects` -- List all projects
- `DELETE /v1/projects/{id}` -- Delete a project

**Users**:

- `POST /v1/users/{id}` -- Update a user
- `GET /v1/users` -- List users
- `DELETE /v1/users/{id}` -- Delete a user

**Connections**:

- `POST /v1/app-connections` -- Upsert a connection (matched by app name)
- `GET /v1/app-connections` -- List connections
- `DELETE /v1/app-connections/{id}` -- Delete a connection

**Flows**:

- `POST /v1/flows` -- Create a flow
- `POST /v1/flows/{id}` -- Apply an operation to a flow (publish, update)
- `GET /v1/flows/{id}` -- Get a flow by ID
- `GET /v1/flows` -- List flows
- `DELETE /v1/flows/{id}` -- Delete a flow

**Flow Runs**:

- `GET /v1/flow-runs/{id}` -- Get a flow run by ID
- `GET /v1/flow-runs` -- List flow runs

**Pieces**:

- `POST /v1/pieces` -- Install a piece to the platform
- `GET /v1/sample-data` -- Get sample data for a piece

**Git Sync**:

- `POST /v1/git-repos` -- Configure (upsert) a Git repository for a project

Additional API resources include User Invitations, Project Members, Project Releases, Global Connections, Folders, Templates, and Queue Metrics.

**Example: Creating a flow via the API**:

```bash
curl -X POST https://your-instance.com/v1/flows \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "displayName": "New Lead Notification",
    "folderId": null
  }'
```

**Example: Listing flow runs**:

```bash
curl -X GET "https://your-instance.com/v1/flow-runs?limit=10&status=SUCCEEDED" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

## Configuration and Customization

Activepieces is configured primarily through environment variables. Key configuration categories include:

**Database Configuration**:

- `AP_DB_TYPE` -- Database type (`POSTGRES` for production, `PGLITE` for development)
- `AP_POSTGRES_DATABASE` -- PostgreSQL database name
- `AP_POSTGRES_HOST` -- PostgreSQL host address
- `AP_POSTGRES_PORT` -- PostgreSQL port
- `AP_POSTGRES_USERNAME` -- PostgreSQL username
- `AP_POSTGRES_PASSWORD` -- PostgreSQL password

**Queue Configuration**:

- `AP_REDIS_TYPE` -- Queue type (`REDIS` for production, `MEMORY` for development)
- `AP_REDIS_HOST` -- Redis host address
- `AP_REDIS_PORT` -- Redis port

**Application Configuration**:

- `AP_FRONTEND_URL` -- Publicly accessible URL of the Activepieces instance (required for webhooks)
- `AP_ENCRYPTION_KEY` -- Key used to encrypt connection credentials at rest
- `AP_JWT_SECRET` -- Secret used to sign JSON Web Tokens for authentication

**Telemetry**:

- `AP_TELEMETRY_ENABLED` -- Enable or disable anonymous usage telemetry

For Docker Compose deployments, these variables are set in the `.env` file generated by `tools/deploy.sh` or manually configured from `.env.example`.

**Piece Management**: Administrators can control which pieces are available to users through the admin console. Pieces can be installed from the public registry, published privately, or developed locally with hot reloading enabled during development.

## Integration Patterns

**Webhook-Driven Automation**: External systems send HTTP POST requests to Activepieces webhook URLs, triggering flows that process the incoming data. This pattern suits event-driven architectures where third-party services need to notify Activepieces of state changes.

```text
External Service --> POST webhook URL --> Activepieces Flow --> Actions
```

**Polling-Based Integration**: For services that do not support webhooks, polling triggers periodically check an external endpoint for new data. When new records are detected, the flow executes with the new data as input.

**Embedded Automation (White-Label)**: SaaS applications embed the Activepieces builder using the Embedding SDK. The host application generates a JWT, initializes the SDK, and renders the builder in an iframe:

```html
<script src="https://cdn.activepieces.com/sdk/embed.js"></script>
<script>
  activepieces.configure({
    prefix: "/automation",
    instanceUrl: "https://your-activepieces-instance.com",
    jwtToken: "GENERATED_JWT_TOKEN"
  });
  activepieces.connect();
</script>
```

**API-Driven Flow Management**: CI/CD pipelines and administrative scripts use the REST API to create, update, enable, and disable flows programmatically. This supports infrastructure-as-code approaches where flow definitions are managed alongside application code.

**Git Sync for Environment Promotion**: Flows are developed and tested in a staging project, exported to a Git repository, reviewed via pull requests, and promoted to production through project releases. This pattern provides change control and rollback capabilities.

```text
Staging Project --> Git Export --> Pull Request --> Merge --> Production Release
```

## Examples

**Building a custom piece** (TypeScript npm package):

A piece defines its metadata, authentication, triggers, and actions. The following illustrates the structure of a minimal piece definition:

```typescript
import { createPiece, PieceAuth } from '@activepieces/pieces-framework';
import { sendMessage } from './lib/actions/send-message';
import { newMessage } from './lib/triggers/new-message';

export const myIntegration = createPiece({
  displayName: 'My Integration',
  auth: PieceAuth.SecretText({
    displayName: 'API Key',
    required: true,
  }),
  triggers: [newMessage],
  actions: [sendMessage],
  logoUrl: 'https://example.com/logo.png',
});
```

**Defining an action within a piece**:

```typescript
import { createAction, Property } from '@activepieces/pieces-framework';

export const sendMessage = createAction({
  name: 'send_message',
  displayName: 'Send Message',
  description: 'Send a message to a channel',
  props: {
    channel: Property.ShortText({
      displayName: 'Channel',
      required: true,
    }),
    message: Property.LongText({
      displayName: 'Message',
      required: true,
    }),
  },
  async run(context) {
    const { channel, message } = context.propsValue;
    const apiKey = context.auth;

    const response = await fetch('https://api.example.com/messages', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${apiKey}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ channel, message }),
    });

    return response.json();
  },
});
```

**Defining a webhook trigger**:

```typescript
import { createTrigger, TriggerStrategy } from '@activepieces/pieces-framework';

export const newMessage = createTrigger({
  name: 'new_message',
  displayName: 'New Message',
  description: 'Triggers when a new message is received',
  type: TriggerStrategy.WEBHOOK,
  props: {},
  async onEnable(context) {
    // Register webhook with external service
    await registerWebhook(context.webhookUrl, context.auth);
  },
  async onDisable(context) {
    // Deregister webhook
    await deregisterWebhook(context.auth);
  },
  async run(context) {
    return [context.payload.body];
  },
});
```

**Setting up piece development with hot reloading**:

```bash
git clone https://github.com/activepieces/activepieces.git
cd activepieces
npm install
# Start in development mode with hot reloading
npx nx serve-dev activepieces
```

## Limitations and Considerations

- **Single-container development mode limitations**: The PGLite and in-memory queue configuration supports only one instance per machine and is unsuitable for production workloads. Production deployments require external PostgreSQL and Redis services.
- **Embedding SDK availability**: The white-label embedding feature requires a paid edition subscription. Self-hosted open-source deployments do not include embedding capabilities without a commercial license.
- **Sequential execution model**: Flow actions execute sequentially. There is no built-in parallel execution of branches within a single flow, which can limit throughput for workflows with independent parallel tasks.
- **Piece ecosystem maturity**: While the catalog exceeds 635 integrations, individual piece quality and feature completeness varies. Community-contributed pieces may lack comprehensive error handling or cover only a subset of an application's API surface.
- **No-code constraints**: Complex data transformations, conditional logic, and error handling can become unwieldy in the visual builder for advanced use cases. Users with complex requirements may need to develop custom pieces in TypeScript.
- **Webhook URL requirement**: Webhook-based triggers require a publicly accessible URL. Local development setups need tunneling tools like ngrok, adding friction to the development workflow.
- **Resource consumption**: The all-in-one container bundles the frontend, backend, and worker processes. In high-throughput scenarios, separating workers from the API server requires Docker Compose or Kubernetes deployment.

## Changelog Highlights

Activepieces is under active development with frequent releases. The platform has evolved from a basic flow automation tool into a comprehensive platform with significant milestones including: the introduction of the Embedding SDK for white-label SaaS integration, native AI pieces and agent capabilities, Git Sync for version-controlled flow management, project releases for environment promotion workflows, audit logging for enterprise compliance, and the expansion of the piece ecosystem to over 635 integrations with MCP (Model Context Protocol) server support for AI agents. The platform was originally created in December 2022 and is written in TypeScript under a permissive open-source license.

## Citations

- [1] [Activepieces Documentation](https://www.activepieces.com/docs/)
- [2] [Activepieces GitHub Repository](https://github.com/activepieces/activepieces)
- [3] [Activepieces Docker Installation](https://www.activepieces.com/docs/install/options/docker)
- [4] [Activepieces Docker Compose Installation](https://www.activepieces.com/docs/install/options/docker-compose)
- [5] [Activepieces Embedding SDK](https://www.activepieces.com/docs/embedding/overview)
- [6] [Activepieces Piece Development](https://www.activepieces.com/docs/developers/building-pieces/overview)
- [7] [Activepieces REST API Reference](https://www.activepieces.com/docs/developers/api-reference)
