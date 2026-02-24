# Node-RED

> Flow-based programming tool with browser-based visual editor and 5000+ nodes

| Field | Value |
|-------|-------|
| Group | Workflow Orchestration & Automation |
| Type | UI |
| Open Source | Yes |
| GitHub | [https://github.com/node-red/node-red](https://github.com/node-red/node-red) |
| Stars | 22816 |
| Documentation | [Official Docs](https://nodered.org/docs/) |

## Overview

Node-RED is a low-code programming tool for building event-driven applications using a flow-based programming model. Created by Nick O'Leary and Dave Conway-Jones at IBM's Emerging Technology Services group in early 2013, it was open-sourced in September 2013 and became a founding project of the JS Foundation in October 2016. When the Node.js Foundation and JS Foundation merged in 2019, Node-RED transitioned to the OpenJS Foundation, where it remains today. [1][2]

The platform provides a browser-based visual editor for wiring together hardware devices, APIs, and online services. Flows are stored as JSON and can be version-controlled, shared, and imported across environments. Built on Node.js, Node-RED takes full advantage of its event-driven, non-blocking I/O model, making it well suited for running on low-cost hardware such as Raspberry Pi and BeagleBone boards as well as cloud infrastructure. [1]

Node-RED's ecosystem includes over 5,000 community-contributed nodes published to npm and listed in the Node-RED Flow Library. In 2021, co-creator Nick O'Leary established FlowFuse, Inc. to develop Node-RED into a secure, professional, and scalable platform for enterprise and industrial applications. [2]

## Core Concepts

**Nodes** are the fundamental building blocks of Node-RED. Each node has a well-defined purpose: it receives messages, performs processing, and passes messages to the next node. A node has at most one input port and zero or more output ports. Nodes are triggered by receiving a message from the preceding node or by an external event such as an HTTP request, a timer firing, or a hardware state change. [3]

**Flows** are the visual programs created in the editor workspace. Each flow is represented as a tab in the editor and consists of nodes connected by wires. A single tab may contain multiple disconnected groups of nodes, each forming an independent flow within the same workspace. [3]

**Wires** are the connections between nodes that define how messages travel through a flow. A wire connects an output port of one node to the input port of another, establishing the message routing path. [3]

**Messages** are JavaScript objects passed between nodes. By convention, messages contain a `payload` property holding the primary data, though any property can be set. Messages are referenced as `msg` within the editor and function nodes. A message object can carry arbitrary properties alongside payload, enabling metadata propagation through the flow. [3]

**Context** is a mechanism for storing data that persists between messages without requiring wire connections. Three scopes are available: Node context (visible only to the setting node), Flow context (accessible to all nodes on the same tab), and Global context (accessible across all nodes in the runtime). Context supports both in-memory storage (default) and persistent file-system storage. [3][5]

**Configuration Nodes** (config nodes) are specialized nodes that hold reusable configuration shared across multiple regular nodes. For example, an MQTT Broker config node stores connection details that multiple MQTT input and output nodes reference. Config nodes appear in a dedicated sidebar panel rather than in the workspace. [3]

**Subflows** are collections of nodes packaged as a single reusable node. They reduce visual complexity in large flows and enable component reuse across multiple locations. A subflow template defines the internal flow once, and subflow instances appear as single nodes in the workspace. [3]

**Palette** is the left-side panel in the editor listing all available nodes organized by category. Additional nodes can be installed via the command line (`npm install`) or through the built-in Palette Manager in the editor. [3]

## Installation and Setup

### Local Installation via npm

Node-RED requires a supported version of Node.js. Install globally with npm:

```bash
sudo npm install -g --unsafe-perm node-red
```

On Windows, omit `sudo`:

```bash
npm install -g --unsafe-perm node-red
```

Start Node-RED:

```bash
node-red
```

The editor is available at `http://localhost:1880`. To change the default port:

```bash
node-red --port 3000
```

[4]

### Docker

Run Node-RED in a container with persistent data storage:

```bash
docker run -it -p 1880:1880 -v node_red_data:/data --name mynodered nodered/node-red
```

For background operation, replace `-it` with `-d`. Configure via environment variables:

```bash
docker run -d -p 1880:1880 \
  -v node_red_data:/data \
  -e FLOWS=my_flows.json \
  -e TZ=America/New_York \
  -e NODE_RED_ENABLE_SAFE_MODE=true \
  --name mynodered nodered/node-red
```

Bind-mount a host directory for data persistence:

```bash
docker run -it -p 1880:1880 -v /home/pi/.node-red:/data --name mynodered nodered/node-red
```

[6]

### Raspberry Pi

Use the all-in-one install script designed for Raspberry Pi:

```bash
bash <(curl -sL https://raw.githubusercontent.com/node-red/linux-installers/master/deb/update-nodejs-and-nodered)
```

This script installs Node.js, npm, and Node-RED with appropriate configurations for the Pi hardware. [4]

### Snap

On supported Linux distributions:

```bash
sudo snap install node-red
```

[4]

### Cloud Deployments

Node-RED supports deployment on AWS (Elastic Beanstalk or EC2), Microsoft Azure (Virtual Machines), and FlowFuse (multi-tenant managed platform for enterprise environments). [4]

## Architecture

Node-RED's architecture consists of three main layers:

```
Editor (Browser)       Browser-based visual flow editor
    |
Runtime (Node.js)      Flow execution engine, node lifecycle, message routing
    |
Storage (Pluggable)    Flow persistence, credential storage, library management
```

**Editor Layer**: A single-page web application served by the runtime. The editor communicates with the runtime via the Admin API to deploy flows, manage nodes, and configure the runtime. The editor consists of the header (deploy button, main menu), the palette (left panel with available nodes), the workspace (central canvas for flow editing), and the sidebar (information, help, debug, config nodes, context data panels). [7]

**Runtime Layer**: The Node.js-based execution engine that loads flow definitions, instantiates nodes, manages message passing between nodes, and handles node lifecycle events (start, stop, deploy). The runtime exposes three APIs: the Admin API for remote administration, the Runtime API for embedding Node-RED in existing Node.js applications, and the Storage API as a pluggable persistence layer. [8]

**Storage Layer**: A pluggable system for persisting flow definitions, credentials, and library entries. The default implementation uses the local file system, storing flows as JSON files in the user directory (`~/.node-red/`). Custom storage plugins can redirect persistence to databases or cloud storage. [8]

**Message Routing**: Messages flow through wires between nodes. Each node processes the incoming `msg` object and either returns it (modified or unmodified) to pass downstream, returns `null` to stop the message, or uses `node.send()` for asynchronous dispatch. The runtime clones messages when a node sends to multiple outputs to prevent unintended side effects. [9]

## Key Features and Functionality

**Visual Flow Editor**: The browser-based editor provides drag-and-drop node placement, wire drawing between ports, node configuration dialogs, flow tab organization, grouping, search, and import/export of flows as JSON. The editor includes a built-in debug panel for inspecting messages at any point in a flow. [7]

**Built-in Core Nodes**: Node-RED ships with a set of essential nodes:

- **Inject**: Triggers a flow manually (button click) or automatically at intervals using cron-style scheduling. Payloads can be strings, numbers, timestamps, or other data types
- **Debug**: Displays messages in the Debug sidebar with structured inspection, timestamps, and source identification
- **Function**: Executes custom JavaScript code on messages (detailed below)
- **Change**: Modifies message properties without code using Set, Change (search/replace), Move, and Delete operations. Supports JSONata expressions
- **Switch**: Routes messages to different outputs based on property evaluation rules. Rule types include Value comparison, Sequence matching, JSONata Expression, and Otherwise (default)
- **Template**: Generates text output using Mustache templating syntax with access to message properties
- **HTTP In / HTTP Response**: Creates HTTP endpoints within Node-RED for building REST APIs and webhooks
- **MQTT In / MQTT Out**: Publish and subscribe to MQTT topics for IoT messaging
- **WebSocket**: Bidirectional WebSocket communication
- **TCP / UDP**: Raw network socket communication
- **Serial**: Serial port communication for hardware integration

[10]

**Function Nodes**: Execute custom JavaScript with full access to the message object, context stores, and Node.js modules:

```javascript
// Basic message transformation
msg.payload = msg.payload.toUpperCase();
return msg;
```

```javascript
// Multiple outputs: route by topic
if (msg.topic === "temperature") {
    return [msg, null];
} else {
    return [null, msg];
}
```

```javascript
// Asynchronous operation with node.send
const response = await fetch("https://api.example.com/data");
const data = await response.json();
msg.payload = data;
node.send(msg);
node.done();
```

Function nodes support On Start (initialization) and On Stop (cleanup) lifecycle hooks, and can import external npm modules when `functionExternalModules` is enabled in settings. [9]

**Context Storage**: Share state between nodes without message wiring:

```javascript
// Node-scoped context
context.set("count", (context.get("count") || 0) + 1);

// Flow-scoped context
flow.set("sharedValue", msg.payload);
let value = flow.get("sharedValue");

// Global context
global.set("config", { threshold: 42 });
let config = global.get("config");
```

Context supports multiple simultaneous stores (memory and filesystem) with configurable defaults. [5]

**Environment Variables**: Reference environment variables in node properties using `${ENV_VAR}` syntax. Access in function nodes with `env.get("FOO")`, in JSONata expressions with `$env('ENV_VAR')`, and in template nodes with `{{env.COLOUR}}`. Environment variables can be defined at subflow instance, flow/group, or global level. Built-in variables (since version 2.2) include `NR_NODE_ID`, `NR_NODE_NAME`, `NR_FLOW_ID`, `NR_FLOW_NAME`, and others for runtime introspection. [11]

**Custom Node Development**: Extend Node-RED by creating custom nodes as npm packages. Each node consists of a JavaScript file (runtime logic), an HTML file (editor UI and configuration dialog), and a `package.json` for npm distribution. Published nodes are discoverable in the Node-RED Flow Library. [12]

**Subflow Modules**: Package subflows as installable npm modules, allowing complex reusable components to be distributed and versioned independently of the flows that use them. [12]

## Use Cases

**IoT Data Collection and Processing**: Connect sensors and devices via MQTT, serial, TCP, or WebSocket protocols. Process incoming telemetry data with transformation, filtering, and aggregation nodes before forwarding to databases or dashboards.

**Home Automation**: Integrate with home automation protocols (MQTT, HTTP, WebSocket) to build custom automation rules. Node-RED runs efficiently on Raspberry Pi hardware, making it a popular choice for local home automation controllers.

**REST API Prototyping**: Use HTTP In and HTTP Response nodes to rapidly prototype REST APIs without writing server code. Route requests through processing nodes and return structured JSON responses.

**Industrial Automation**: Monitor and control industrial equipment through serial, Modbus, and OPC-UA protocols. FlowFuse provides enterprise features for multi-tenant, multi-site industrial deployments.

**Data Integration and ETL**: Wire together disparate data sources (databases, APIs, file systems, message queues) to build extraction, transformation, and loading pipelines with visual debugging.

**AI/ML Pipeline Integration**: Connect to LLM APIs, machine learning inference endpoints, and data preprocessing services. Use function nodes for prompt construction and response parsing. Community nodes provide integrations with OpenAI, Hugging Face, and other AI services.

**Alert and Notification Systems**: Monitor data streams and trigger notifications via email, Slack, Telegram, SMS, or push notifications based on configurable threshold rules.

## API Reference Summary

### Admin API

The Admin HTTP API enables remote runtime administration and is used by both the editor and the `node-red-admin` command-line tool: [8]

**Authentication**:
- `GET /auth/login` -- Get the active authentication scheme
- `POST /auth/token` -- Exchange credentials for an access token
- `POST /auth/revoke` -- Revoke an access token

**Flows**:
- `GET /flows` -- Get the active flow configuration
- `POST /flows` -- Set the active flow configuration (deploy)
- `GET /flows/state` -- Get the runtime state of active flows
- `POST /flows/state` -- Set the runtime state (start/stop)
- `POST /flow` -- Add a single flow to the active configuration
- `GET /flow/:id` -- Get an individual flow configuration
- `PUT /flow/:id` -- Update an individual flow configuration
- `DELETE /flow/:id` -- Delete an individual flow

**Nodes**:
- `GET /nodes` -- List all installed node modules
- `POST /nodes` -- Install a new node module
- `GET /nodes/:module` -- Get module information
- `PUT /nodes/:module` -- Enable or disable a module
- `DELETE /nodes/:module` -- Remove a node module

**Settings and Diagnostics**:
- `GET /settings` -- Get runtime settings
- `GET /diagnostics` -- Get runtime diagnostics

### Hooks API

The Hooks API provides insertion points for custom code at key runtime operations, enabling middleware-style processing during flow deployment, message routing, and node lifecycle events. [8]

### Storage API

A pluggable interface for configuring where the runtime stores flow definitions, credentials, and library entries. Custom implementations can replace the default filesystem storage with databases or cloud storage backends. [8]

### Context Store API

A pluggable interface for storing context data outside the default in-memory or filesystem stores. Enables integration with external databases or caches for context persistence. [8]

## Configuration and Customization

Node-RED is configured via `settings.js` located in the user directory (`~/.node-red/` by default). Key configuration options: [13]

### Server Settings

```javascript
module.exports = {
    uiPort: 1880,                    // Editor and API port (default: 1880)
    uiHost: "0.0.0.0",              // Bind address (default: all IPv4 interfaces)
    httpAdminRoot: "/admin",         // Editor URL path (default: /)
    httpNodeRoot: "/api",            // HTTP node endpoint path (default: /)
    httpStatic: "/home/user/public", // Serve static files from directory
    userDir: "/home/user/.node-red", // User data directory
    flowFile: "flows.json",          // Flow file name (default: flows_<hostname>.json)
    nodesDir: "/home/user/.node-red/nodes", // Additional node search directory
    disableEditor: false,            // Set true to disable the editor UI
}
```

### Security Configuration

```javascript
module.exports = {
    // HTTPS with TLS certificates
    https: {
        key: require("fs").readFileSync("privkey.pem"),
        cert: require("fs").readFileSync("cert.pem"),
    },

    // User authentication with bcrypt-hashed passwords
    adminAuth: {
        type: "credentials",
        users: [{
            username: "admin",
            password: "$2b$08$...",  // Generated via: node-red admin hash-pw
            permissions: "*",        // Full access
        }, {
            username: "viewer",
            password: "$2b$08$...",
            permissions: "read",     // Read-only access
        }],
    },

    // HTTP node route authentication
    httpNodeAuth: { user: "user", pass: "$2b$08$..." },
}
```

Passwords are hashed using bcrypt. Generate hashes with `node-red admin hash-pw`. Fine-grained permissions (`flows.read`, `flows.write`) are available since version 0.14. Passport-based OAuth/OpenID strategies (GitHub, Twitter) are supported for external authentication. [14]

### Node Configuration

```javascript
module.exports = {
    functionGlobalContext: {
        os: require("os"),           // Expose modules to function nodes
    },
    functionExternalModules: true,   // Allow npm imports in function nodes
    functionTimeout: 0,              // Function node timeout (0 = no timeout)
    debugMaxLength: 1000,            // Max debug message length (chars)
    mqttReconnectTime: 5000,         // MQTT reconnect delay (ms)
    serialReconnectTime: 5000,       // Serial reconnect delay (ms)
    socketReconnectTime: 10000,      // TCP reconnect delay (ms)
}
```

### Context Storage

```javascript
module.exports = {
    contextStorage: {
        default: "memory",
        memory: { module: "memory" },
        file: {
            module: "localfilesystem",
            // Writes cached values to disk every 30 seconds
        },
    },
}
```

### Logging

Six log levels are available: `fatal`, `error`, `warn`, `info` (default), `debug`, `trace`. Custom logger modules can direct log events to databases or external logging services. [13]

## Integration Patterns

### IoT Protocol Bridge

Use Node-RED as a protocol translation layer between IoT devices and cloud services. MQTT nodes subscribe to device topics, function nodes transform payloads, and HTTP request nodes forward data to REST APIs or cloud platforms:

```
[MQTT In] -> [Function: Transform] -> [HTTP Request: POST to API]
```

### Webhook Processor

Create HTTP endpoints that receive webhook payloads, process them through transformation and routing logic, and trigger downstream actions:

```
[HTTP In: POST /webhook] -> [Switch: Route by event type] -> [Function: Process] -> [HTTP Response]
```

### Database ETL Pipeline

Connect to source databases, transform records, and load them into destination systems. Use inject nodes for scheduling and debug nodes for monitoring:

```
[Inject: Cron schedule] -> [MySQL In: Query] -> [Function: Transform] -> [PostgreSQL: Insert]
```

### AI/ML Service Integration

Call LLM APIs from within flows using HTTP request nodes or community AI nodes. Construct prompts in function nodes and parse structured responses:

```
[HTTP In: User query] -> [Function: Build prompt] -> [HTTP Request: LLM API] -> [Function: Parse response] -> [HTTP Response]
```

### Dashboard and Monitoring

Combine data collection flows with Node-RED Dashboard nodes (community package `node-red-dashboard`) to build real-time monitoring interfaces with charts, gauges, and controls.

## Examples

### HTTP API Endpoint

Create a simple REST endpoint that returns sensor data:

```json
[
    {
        "id": "http-in",
        "type": "http in",
        "url": "/api/sensor",
        "method": "get",
        "wires": [["function"]]
    },
    {
        "id": "function",
        "type": "function",
        "func": "msg.payload = { temperature: 22.5, humidity: 65, timestamp: Date.now() };\nreturn msg;",
        "wires": [["http-response"]]
    },
    {
        "id": "http-response",
        "type": "http response",
        "statusCode": 200,
        "wires": []
    }
]
```

### Function Node with Context

Track message count across invocations using node context:

```javascript
// Function node: Message counter
let count = context.get("messageCount") || 0;
count += 1;
context.set("messageCount", count);

msg.payload = {
    originalPayload: msg.payload,
    messageNumber: count,
    processedAt: new Date().toISOString(),
};
return msg;
```

### MQTT to HTTP Bridge

Forward MQTT messages to a REST API with payload transformation:

```javascript
// Function node: Transform MQTT payload for API
let sensorData = JSON.parse(msg.payload);

msg.url = "https://api.example.com/telemetry";
msg.method = "POST";
msg.headers = { "Content-Type": "application/json" };
msg.payload = {
    deviceId: msg.topic.split("/")[1],
    temperature: sensorData.temp,
    humidity: sensorData.hum,
    timestamp: new Date().toISOString(),
};
return msg;
```

### Switch Node Routing

Route messages based on temperature thresholds (configured in the Switch node):

```javascript
// Function node after Switch: High temperature alert
msg.payload = {
    alert: "HIGH_TEMPERATURE",
    value: msg.payload.temperature,
    message: "Temperature exceeded threshold: " + msg.payload.temperature + "C",
};
return msg;
```

### Environment Variable Configuration

Use environment variables for configurable deployments:

```javascript
// Function node: Use environment variables
let apiKey = env.get("API_KEY");
let endpoint = env.get("API_ENDPOINT") || "https://api.default.com";

msg.url = endpoint + "/data";
msg.headers = {
    "Authorization": "Bearer " + apiKey,
    "Content-Type": "application/json",
};
return msg;
```

## Limitations and Considerations

**Single-Threaded Execution**: Node-RED runs on Node.js, which is single-threaded. CPU-intensive operations in function nodes block the entire runtime, including all other flows. Long-running computations should be offloaded to external services or worker processes.

**Not Designed for High-Throughput**: Node-RED is optimized for event-driven, low-to-moderate throughput workloads. Applications requiring sustained high message rates (tens of thousands per second) may encounter performance bottlenecks in the message routing layer.

**JavaScript Only**: Function nodes are limited to JavaScript. Developers requiring Python, Go, or other language runtimes must invoke external processes or services via HTTP, TCP, or exec nodes.

**Editor Security by Default**: The Node-RED editor is not secured by default and should not be exposed to untrusted networks without configuring `adminAuth` and HTTPS. Unsecured deployments risk unauthorized flow modification and credential exposure. [14]

**Flow Complexity at Scale**: While subflows and multiple tabs help organize large projects, very complex deployments with hundreds of flows can become difficult to navigate, debug, and maintain in the visual editor.

**Limited Built-in Testing**: Node-RED does not include a built-in unit testing framework for flows. The community `node-red-node-test-helper` package provides testing utilities, but flow testing requires additional setup compared to code-based workflow engines.

**Version Control Challenges**: Flows are stored as JSON files. While JSON can be version-controlled, diff and merge operations on flow files are less readable than on source code, making collaborative development with multiple contributors more complex.

**Context Storage Durability**: The default in-memory context store loses all data on restart. The file-system store writes cached values every 30 seconds by default, creating a window for data loss on unexpected shutdown. [5]

## Changelog Highlights

- **Node-RED 4.1** (July 2025): Latest stable release with incremental improvements and bug fixes
- **Node-RED 4.0** (June 2024): Major version release with modernization updates
- **Node-RED 3.1** (September 2023): Added global-level environment variable settings via User Settings, built-in subflow environment variables (`NR_SUBFLOW_NAME`, `NR_SUBFLOW_ID`, `NR_SUBFLOW_PATH`), and Template node environment variable support
- **Node-RED 3.0** (July 2022): Major version with Template node enhancements and platform modernization
- **Node-RED 2.2**: Introduced built-in environment variables (`NR_NODE_ID`, `NR_NODE_NAME`, `NR_FLOW_ID`, `NR_FLOW_NAME`) for runtime introspection
- **Node-RED 2.1**: Added flow-level and group-level environment variable definitions
- **Node-RED 5.0 Roadmap**: Community modernization initiative focused on updating the Node-RED user experience, presented at Node-RED Con [15]

## Citations

- [1] Node-RED Documentation - <https://nodered.org/docs/>
- [2] About Node-RED - <https://nodered.org/about/>
- [3] Core Concepts - <https://nodered.org/docs/user-guide/concepts>
- [4] Getting Started - <https://nodered.org/docs/getting-started/>
- [5] Working with Context - <https://nodered.org/docs/user-guide/context>
- [6] Running under Docker - <https://nodered.org/docs/getting-started/docker>
- [7] Editor Guide - <https://nodered.org/docs/user-guide/editor/>
- [8] API Reference - <https://nodered.org/docs/api/>
- [9] Writing Functions - <https://nodered.org/docs/user-guide/writing-functions>
- [10] Node Guide - <https://nodered.org/docs/user-guide/nodes>
- [11] Environment Variables - <https://nodered.org/docs/user-guide/environment-variables>
- [12] Creating Nodes - <https://nodered.org/docs/creating-nodes/>
- [13] Configuration - <https://nodered.org/docs/user-guide/runtime/configuration>
- [14] Securing Node-RED - <https://nodered.org/docs/user-guide/runtime/securing-node-red>
- [15] Node-RED Blog - <https://nodered.org/blog/>
