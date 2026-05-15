# Airbyte

> Open-source data integration platform with 600+ connectors for ETL and ELT pipelines

| Field | Value |
|-------|-------|
| Name | Airbyte |
| Group | Data Pipelines |
| Type | API/SDK/Infra |
| Open Source | yes |
| GitHub | [airbytehq/airbyte](https://github.com/airbytehq/airbyte) |
| Stars | 21256 |
| Docs | [docs.airbyte.com](https://docs.airbyte.com/) |

## Overview

Airbyte is an open-source data integration, activation, and agentic data platform that consolidates data from hundreds of sources into data warehouses, lakes, and databases, then distributes that data to operational tools like CRMs and marketing platforms. It provides over 600 pre-built connectors for Extract-Load-Transform (ELT) and Extract-Transform-Load (ETL) pipelines [1].

The platform is available in multiple deployment models [3]:

- **Self-Managed Core**: Free and open-source version for local or self-hosted infrastructure deployment
- **Self-Managed Enterprise**: Highly available solution for organizations prioritizing data sovereignty
- **Airbyte Cloud (Standard/Plus/Pro)**: Fully managed cloud offering with 30-day free trial
- **Enterprise Flex**: Hybrid solution combining managed convenience with separate data planes

Airbyte can be interacted with through a no-code UI, REST API, Python and Java SDKs, Terraform provider, or PyAirbyte (a standalone Python library for data movement without running a server) [3].

## Core Concepts

### Sources, Destinations, and Connectors

A **source** is an API, file, database, or data warehouse from which data is ingested. A **destination** is a data warehouse, lake, database, or analytics tool where data is loaded. A **connector** is the Airbyte component that pulls from sources or pushes to destinations, packaged as Docker images [2].

### Connections

A **connection** is an automated data pipeline that replicates data from a configured source to a configured destination. Connections define sync schedules (scheduled intervals, CRON expressions, or manual triggering), sync modes, stream selection, namespace configuration, and schema change handling [2][8].

### Streams, Records, and Fields

A **stream** is a group of related records — called tables, files, or blobs depending on the destination. A **record** is a single data entry, and a **field** is an attribute of a record (analogous to a database column) [2].

### Sync Modes

Sync modes govern how Airbyte reads from sources and writes to destinations. Five combinations are available [4]:

- **Full Refresh | Overwrite**: Reads entire source, replaces destination data
- **Full Refresh | Append**: Reads entire source, appends to destination
- **Full Refresh | Overwrite + Deduped**: Full read with deduplication on primary key
- **Incremental | Append**: Reads only new/changed records, appends to destination
- **Incremental | Append + Deduped**: Reads changes, appends and deduplicates on primary key

Incremental modes use either cursor-based extraction or Change Data Capture (CDC) for supported databases [4].

### Typing and Deduping

Airbyte's Destinations V2 framework provides one-to-one mapping from streams to destination tables. Raw data is stored in an `airbyte_internal` schema, then typed and deduplicated into final tables with system columns: `_airbyte_raw_id` (unique ID), `_airbyte_extracted_at` (timestamp), and `_airbyte_meta` (error/change tracking) [6].

### Change Data Capture (CDC)

For supported databases (PostgreSQL, MySQL, MSSQL, MongoDB, Oracle DB, SAP HANA, IBM Db2), Airbyte reads database transaction logs to capture all INSERT, UPDATE, and DELETE operations. The initial sync takes a full snapshot; subsequent syncs read from the last log position. CDC metadata columns (`_ab_cdc_lsn`, `_ab_cdc_updated_at`, `_ab_cdc_deleted_at`) track change details [7].

## Architecture

Airbyte consists of a platform layer and a connector layer [5]:

```
┌─────────────────────────────────────────────────┐
│                 Platform Layer                   │
│  ┌──────────┐ ┌──────────┐ ┌──────────────────┐│
│  │  Config   │ │  Web UI  │ │   Temporal       ││
│  │  API      │ │          │ │  (Scheduling)    ││
│  │  Server   │ │          │ │                  ││
│  └─────┬────┘ └──────────┘ └────────┬─────────┘│
│        │                            │           │
│  ┌─────v────┐ ┌──────────┐ ┌───────v──────┐   │
│  │ Database  │ │   Cron   │ │   Worker     │   │
│  │ (Config + │ │ (Cleanup │ │ (Task Queue  │   │
│  │  History) │ │  + Defs) │ │  Consumer)   │   │
│  └──────────┘ └──────────┘ └───────┬───────┘   │
│                                     │           │
│  ┌──────────────┐  ┌───────────────v─────────┐ │
│  │  Bootloader   │  │  Workload API + Launcher│ │
│  │ (Migrations)  │  │  (K8s Pod Management)   │ │
│  └──────────────┘  └─────────────────────────┘ │
└─────────────────────────────────────────────────┘
                         │
                         v
┌─────────────────────────────────────────────────┐
│              Connector Layer (Docker)             │
│  ┌──────────────┐        ┌───────────────────┐  │
│  │   Source      │ ──→   │   Destination      │  │
│  │  Connector    │ JSON  │   Connector        │  │
│  │  (Docker)     │ msgs  │   (Docker)         │  │
│  └──────────────┘        └───────────────────┘  │
│                                                  │
│  Protocol: spec → check → discover → read/write │
└─────────────────────────────────────────────────┘
```

The **platform layer** provides horizontal services: UI, configuration API, job scheduling (via Temporal), logging, and worker task queue management. The **connector layer** consists of independent Docker-packaged modules that push/pull data via the Airbyte Protocol — a JSON message serialization standard using STDIN/STDOUT with message types: RECORD, STATE, LOG, SPEC, CATALOG, CONNECTION_STATUS, and TRACE [5][11].

Connectors implement standard operations: `spec()` (capabilities), `check(config)` (validate connectivity), `discover(config)` (list available streams), and `read()`/`write()` for data transfer [11].

## Key Features

- **600+ Connectors**: Pre-built sources and destinations for databases, APIs, SaaS platforms, file systems, and warehouses
- **Connector Builder**: No-code web-based tool for building custom API source connectors without local development
- **Low-Code CDK**: Declarative YAML framework for HTTP API sources with optional custom Python components
- **Python CDK**: Full-flexibility connector development with pre-built classes and scaffold generators
- **Change Data Capture**: Log-based incremental replication for PostgreSQL, MySQL, MSSQL, MongoDB, Oracle, SAP HANA, and Db2
- **Incremental Sync**: Cursor-based or CDC-based change detection with state checkpointing for resumable syncs
- **Typing and Deduping**: Automatic type-casting and primary-key deduplication in destination tables
- **Schema Propagation**: Automated detection and handling of source schema changes
- **Field Selection**: Exclude specific fields from synchronization at the stream level
- **Sync Schedules**: Scheduled intervals, CRON expressions, or manual triggering
- **PyAirbyte**: Standalone Python library for data extraction without running an Airbyte server
- **Terraform Provider**: Infrastructure-as-code management of Airbyte resources
- **REST API + SDKs**: Programmatic control via REST API with Python and Java SDKs
- **Resumability**: Checkpoint-based progress tracking with automatic retry for failed syncs
- **Namespace Configuration**: Control where replicated data is written in the destination schema
- **Stream Prefix**: Add naming conventions to destination table identifiers
- **Per-Row Error Handling**: `_airbyte_meta` column tracks typing and size changes per record

## Use Cases

- **Data Warehouse Loading**: Consolidate data from SaaS applications (Salesforce, HubSpot, Stripe), databases, and APIs into Snowflake, BigQuery, Databricks, or Redshift
- **ELT Pipelines**: Extract and load raw data into warehouses for transformation with dbt or SQL
- **Database Replication**: Replicate PostgreSQL, MySQL, or MongoDB databases using CDC for near real-time synchronization
- **Analytics Data Integration**: Aggregate marketing, sales, and product data for business intelligence dashboards
- **Data Lake Ingestion**: Load structured and unstructured data into S3, GCS, or Azure Blob Storage data lakes
- **RAG Data Preparation**: Extract and prepare data from various sources for Retrieval-Augmented Generation pipelines using PyAirbyte
- **Reverse ETL / Data Activation**: Distribute warehouse data back to operational tools like CRMs and marketing platforms
- **Migration**: Move data between databases or from legacy systems to modern cloud infrastructure

## API Reference

### REST API

```bash
# List sources (Cloud)
curl --request GET \
  --url 'https://api.airbyte.com/v1/sources?workspaceIds=<WORKSPACE_ID>' \
  --header 'authorization: Bearer <TOKEN>'

# Base URLs
# Cloud: https://api.airbyte.com/v1/
# Self-managed (local): http://localhost:8000/api/public/v1/
# Self-managed (web): <YOUR_AIRBYTE_URL>/api/public/v1/
```

Access tokens are short-lived and require regular renewal. The API supports creating and managing sources, destinations, connections, and triggering syncs [12].

### PyAirbyte API

```python
import airbyte as ab

# Initialize source
source = ab.get_source(
    "source-faker",
    config={"count": 5_000},
    install_if_missing=True,
)

# Validate connection
source.check()

# Select streams
source.select_all_streams()
# Or select specific streams: source.select_streams(["users", "products"])

# Read data
result = source.read()

# Access stream data
for name, records in result.streams.items():
    print(f"Stream {name}: {len(list(records))} records")
```

### Airbyte Protocol (Connector Interface)

Sources implement four operations [11]:

- `spec()` → Returns connector specification with configuration requirements
- `check(config)` → Validates connectivity and credentials
- `discover(config)` → Returns catalog of available streams and schemas
- `read(config, catalog, state)` → Extracts records with state checkpoints

Destinations implement: `spec()`, `check(config)`, and `write(config, catalog, messages)`.

## Configuration

### Connection Configuration

Connections are configured with [8]:

- **Sync Schedule**: Scheduled intervals, CRON expressions, or manual triggering
- **Sync Mode**: Per-stream selection of Full Refresh or Incremental reading with Overwrite, Append, or Deduped writing
- **Namespace**: Determines where replicated data is written in the destination
- **Stream Prefix**: Optional prefix added to destination table names
- **Cursor Field**: Defines which field tracks new/updated records for incremental syncs
- **Primary Key**: Used for deduplication in Append Deduped and Overwrite Deduped modes
- **Field Selection**: Include or exclude specific fields from sync
- **Schema Change Handling**: Configure automatic or manual approval of source schema changes

### Deployment Configuration (Helm)

Custom `values.yaml` for Kubernetes deployments supports:

- State and logging storage (S3, GCS)
- Secret management
- External database configuration
- Ingress configuration
- Resource limits and scaling [10]

### Connector Configuration

Each connector requires source-specific configuration (credentials, endpoints, database connection strings) defined through the UI, API, or Terraform provider.

## Integration Patterns

### dbt Transformation

Airbyte integrates with dbt for post-load transformation. The connections UI includes a dedicated dbt Transformation tab for configuring transformations that run after data loads [8].

### Terraform Provider

Manage Airbyte resources as infrastructure-as-code using the official Terraform provider for automated, version-controlled pipeline management [3].

### Orchestration Integration

Airbyte syncs can be triggered and monitored through external orchestrators like Apache Airflow, Dagster, or Prefect using the REST API or SDKs.

### PyAirbyte for AI/ML

PyAirbyte enables direct data extraction into Python environments for machine learning pipelines, RAG implementations, and data science workflows without running an Airbyte server [9].

## Examples

### Basic PyAirbyte Data Extraction

```python
import airbyte as ab

# Extract data from GitHub
source = ab.get_source(
    "source-github",
    config={
        "credentials": {"personal_access_token": "ghp_..."},
        "repositories": ["airbytehq/airbyte"],
    },
    install_if_missing=True,
)

source.check()
source.select_streams(["commits", "pull_requests"])
result = source.read()

# Process commits
for record in result["commits"]:
    print(record["sha"], record["message"])
```

### API-Driven Connection Setup

```bash
# Create a source
curl --request POST \
  --url https://api.airbyte.com/v1/sources \
  --header 'authorization: Bearer <TOKEN>' \
  --header 'content-type: application/json' \
  --data '{
    "name": "My Postgres Source",
    "workspaceId": "<WORKSPACE_ID>",
    "configuration": {
      "sourceType": "postgres",
      "host": "db.example.com",
      "port": 5432,
      "database": "production",
      "username": "airbyte_user",
      "password": "secret"
    }
  }'
```

## Limitations

- **Kubernetes complexity**: Production deployments require Kubernetes clusters with Helm, increasing operational overhead for small teams [10]
- **Docker resource usage**: Connectors run as Docker containers, consuming significant resources when running many concurrent syncs
- **CDC constraints**: CDC requires primary keys, only captures table data (not views), and does not capture TRUNCATE or ALTER operations [7]
- **Schema change sensitivity**: CDC syncs may fail if schema changes are not properly managed; Airbyte recommends manual approval of schema changes for CDC sources [7]
- **Connector maturity variance**: While 600+ connectors exist, quality and feature completeness vary between official Airbyte connectors, marketplace connectors, and community-built connectors
- **Java CDK unavailable**: The Java CDK is being revamped and currently does not accept contributions; custom destination development options are limited [13]
- **Eventual consistency**: Syncs run on scheduled intervals rather than streaming continuously; real-time replication is not supported
- **Typing and deduping overhead**: The intermediate raw table approach consumes additional storage and compute in destination warehouses [6]
- **Full refresh on new CDC tables**: Adding new tables to a CDC connection requires a full initial snapshot before incremental tracking begins [7]

## Changelog

- **Destinations V2**: Typing and deduping framework with one-to-one stream-to-table mapping, per-row error handling, and incremental loading to final tables
- **Direct-Load Tables**: Emerging replacement for typing and deduping that eliminates intermediate raw JSON blobs
- **Connector Builder**: No-code web-based tool for building custom API source connectors
- **Low-Code CDK**: Declarative YAML framework for HTTP API sources
- **PyAirbyte**: Standalone Python library for data extraction without server infrastructure
- **CDC Support**: Log-based incremental replication for PostgreSQL, MySQL, MSSQL, MongoDB, Oracle, SAP HANA, and Db2
- **Airbyte Protocol v0.5.2**: Current protocol version with STATE, RECORD, CATALOG, and TRACE message types
- **Terraform Provider**: Infrastructure-as-code management for Airbyte resources
- **AI Agents**: Data exploration capabilities for agentic workflows

## Citations

- [1] Airbyte Documentation Home - https://docs.airbyte.com/
- [2] Core Concepts - https://docs.airbyte.com/using-airbyte/core-concepts/
- [3] Getting Started - https://docs.airbyte.com/using-airbyte/getting-started/
- [4] Sync Modes - https://docs.airbyte.com/using-airbyte/core-concepts/sync-modes/
- [5] Architecture Overview - https://docs.airbyte.com/understanding-airbyte/high-level-view/
- [6] Typing and Deduping - https://docs.airbyte.com/understanding-airbyte/typing-deduping/
- [7] Change Data Capture - https://docs.airbyte.com/understanding-airbyte/cdc/
- [8] Configuring Connections - https://docs.airbyte.com/cloud/managing-airbyte-cloud/configuring-connections/
- [9] PyAirbyte Getting Started - https://docs.airbyte.com/using-airbyte/pyairbyte/getting-started/
- [10] Deploying Airbyte - https://docs.airbyte.com/deploying-airbyte/
- [11] Airbyte Protocol - https://docs.airbyte.com/understanding-airbyte/airbyte-protocol/
- [12] API Documentation - https://docs.airbyte.com/api-documentation/
- [13] Connector Development - https://docs.airbyte.com/connector-development/

## Discovery Signals

> Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search.

### Keywords

ELT, ETL, data integration, data pipelines, connectors, CDC, change data capture, incremental sync, PyAirbyte, data replication, Airbyte Protocol, Airbyte Cloud, Self-Managed Enterprise, Connector Builder, Low-Code CDK, data warehouse, data lake, reverse ETL, sources, destinations, streams, sync modes, typing and deduping, Destinations V2, Temporal scheduling, Terraform provider, schema propagation, namespace, dbt transformation, data activation

### Verb-Noun Tasks

- Replicate PostgreSQL/MySQL/MongoDB databases via CDC to Snowflake or BigQuery
- Load Salesforce, HubSpot, and Stripe data into a data warehouse
- Configure incremental Append + Deduped sync with primary-key deduplication
- Build a custom HTTP API source connector with the Low-Code CDK
- Trigger Airbyte syncs from Apache Airflow, Dagster, or Prefect
- Extract data into Python with PyAirbyte for RAG preparation
- Deploy Airbyte on Kubernetes with Helm and external Postgres
- Manage sources, destinations, and connections via the REST API
- Provision Airbyte resources as infrastructure-as-code with Terraform
- Schedule syncs with CRON expressions or manual triggering
- Run post-load dbt transformations on synced data
- Handle source schema changes with automatic propagation

### User Intent Phrases

- How do I sync a PostgreSQL database to BigQuery using change data capture?
- What is the fastest way to consolidate SaaS data into Snowflake?
- How can I build a custom Airbyte connector for a REST API without writing Python?
- How do I run Airbyte locally for prototyping a data pipeline?
- What sync mode should I use to incrementally load new records and deduplicate?
- How do I use PyAirbyte to pull GitHub data into a Jupyter notebook for ML?
- How do I deploy Airbyte to a Kubernetes cluster in production?
- How can I trigger an Airbyte connection from Airflow on a schedule?
- How do I manage Airbyte sources and destinations with Terraform?
- What are the differences between Self-Managed Core, Enterprise, and Airbyte Cloud?

### Problem Statements

- Hundreds of SaaS tools each ship data in different shapes, with no unified way to consolidate them
- Hand-rolling database replication with CDC requires deep expertise in transaction logs
- Schema drift in source systems silently breaks downstream pipelines
- Adding a new SaaS source to an existing warehouse takes weeks of bespoke engineering
- Real-time replication is not always required, but resumable, checkpointed batch syncs are
- Custom connector code is hard to maintain across upgrades and dependency changes
- Pipeline configurations live in disparate places and are hard to version-control

### When to Pick This

- Pick this when you need to move structured data between systems (databases, APIs, SaaS, warehouses) rather than parse documents — Unstructured is the right choice for PDFs, DOCX, and unstructured files
- Pick this when you want 600+ pre-built connectors instead of writing custom extractors
- Pick this when you need log-based CDC from PostgreSQL/MySQL/MongoDB into a warehouse
- Pick this when you want infrastructure-as-code pipeline management via Terraform
- Pick this when you need to combine ELT into a warehouse with reverse-ETL data activation back to operational tools
- Pick this when batch/scheduled replication is acceptable and continuous streaming is not required
- Pick this when you want a no-code UI plus REST API plus SDKs in the same platform

### Related Terms and Aliases

- ELT platform, ETL platform, data integration platform
- Open-source Fivetran alternative
- Connector Development Kit (CDK), Low-Code CDK, Python CDK
- PyAirbyte (embedded Python library)
- Change Data Capture (CDC), log-based replication
- Destinations V2, Direct-Load Tables
- Reverse ETL, data activation
- Airbyte Protocol (JSON-RPC over STDIN/STDOUT)
- Honchopkin / Honcho — n/a; uses Temporal for scheduling
