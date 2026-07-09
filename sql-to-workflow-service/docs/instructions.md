# SQL-to-Workflow Service

## Overview

The SQL-to-Workflow Service is a standalone backend microservice developed for Apache Texera that translates SQL queries into Texera workflow representations.

The service uses Apache Calcite for SQL parsing, validation, logical planning, and optimization. Once a logical plan is generated, it is translated into Texera workflow operators and links, producing a workflow that can be executed within Texera.

This document serves as a guide to the current implementation and is intended to help reviewers quickly understand the project structure and major components.

---

# Architecture

<img width="664" height="321" alt="Architecture diagram low level" src="https://github.com/user-attachments/assets/bc3ac117-b6eb-4aa0-94f8-6f7dace37ed8" />


The architecture diagram illustrates the overall SQL-to-Workflow translation pipeline and the interaction between the major components of the service.

---

# Project Structure

```text
sql-to-workflow-service/
│
├── optimizer/
│   ├── FilterJoinPushDownRule.java
│   └── LogicalPlanRewriter.java
│
├── resource/
│   ├── SqlResource.java
│   ├── WorkflowSqlResource.java
│   └── DatasetPreviewResource.java
│
├── service/
│   └── DatasetService.java
│
├── model/
│   ├── SqlRequest.java
│   ├── TexeraWorkflow.java
│   ├── TexeraOperator.java
│   ├── TexeraLink.java
│   ├── Binding.java
│   └── DatasetBinding.java
│
├── utils/
│   └── Casing.java
│
├── CalciteApplication.java
├── CalciteConfiguration.java
├── SqlServiceConfiguration.java
├── TexeraConverter.java
├── AliasExpander.java
├── AggregateIntent.java
├── SchemaOnlyTable.java
├── CsvDynamicTable.java
└── CorsFilter.java
```

---

# Component Overview

## Core Application

| Component | Description |
|-----------|-------------|
| **CalciteApplication** | Entry point of the SQL-to-Workflow service. Initializes the Dropwizard application and registers the REST resources. |
| **SqlServiceConfiguration** | Stores the service configuration used during application startup. |
| **CalciteConfiguration** | Configures Apache Calcite for SQL parsing, validation, and planning. |
| **CorsFilter** | Enables Cross-Origin Resource Sharing (CORS) for frontend communication. |

---

## REST Resources

| Component | Description |
|-----------|-------------|
| **SqlResource** | Handles SQL parsing and validation requests. |
| **WorkflowSqlResource** | Main REST endpoint responsible for translating SQL queries into Texera workflows. |
| **DatasetPreviewResource** | Provides dataset preview and metadata APIs used during SQL analysis. |

---

## Service Layer

| Component | Description |
|-----------|-------------|
| **DatasetService** | Retrieves dataset metadata and schema information required during SQL parsing and workflow generation. |

---

## Translation Components

| Component | Description |
|-----------|-------------|
| **TexeraConverter** | Core translation engine that converts Apache Calcite logical plans into Texera workflow operators and links. |
| **AliasExpander** | Resolves SQL aliases before workflow generation. |
| **AggregateIntent** | Represents aggregate operations detected during SQL analysis. |

---

## Optimizer

| Component | Description |
|-----------|-------------|
| **FilterJoinPushDownRule** | Applies filter pushdown optimization to logical query plans. |
| **LogicalPlanRewriter** | Rewrites logical plans into forms suitable for workflow generation. |

---

## Workflow Models

| Component | Description |
|-----------|-------------|
| **SqlRequest** | Represents incoming SQL requests. |
| **TexeraWorkflow** | Represents the generated workflow. |
| **TexeraOperator** | Represents an individual workflow operator. |
| **TexeraLink** | Represents connections between workflow operators. |
| **Binding** | Stores parameter bindings used during workflow generation. |
| **DatasetBinding** | Maps datasets referenced in SQL queries to workflow operators. |

---

## Schema Support

| Component | Description |
|-----------|-------------|
| **SchemaOnlyTable** | Exposes dataset schema information to Apache Calcite without loading dataset contents. |
| **CsvDynamicTable** | Represents CSV datasets as Calcite tables for parsing and planning. |

---

## Utility Classes

| Component | Description |
|-----------|-------------|
| **Casing** | Handles SQL identifier casing and quoted/unquoted identifier conventions. |

---
---

# Related Integration (Access Control Service)

In addition to the standalone SQL-to-Workflow service, a supporting resource has been implemented within the **Access Control Service** to persist and retrieve SQL queries associated with Texera workflows.

### `SqlWorkflowResource.scala`

**Location**

```text
access-control-service/
└── src/main/scala/org/apache/texera/service/resource/
    └── SqlWorkflowResource.scala
```

**Purpose**

`SqlWorkflowResource` provides REST APIs for storing and retrieving SQL queries associated with a workflow. It acts as the persistence layer between the frontend and the PostgreSQL database by maintaining the SQL text and parameter bindings for each workflow.

**Endpoints**

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/sql/workflow/{wid}` | `POST` | Stores or updates the SQL query and its parameter bindings for the specified workflow ID. |
| `/sql/workflow/{wid}` | `GET` | Retrieves the stored SQL query and parameter bindings for the specified workflow ID. |

**Responsibilities**

- Persists SQL queries for individual workflows.
- Stores parameter bindings as PostgreSQL `JSONB`.
- Supports updating existing workflow SQL using an upsert operation.
- Retrieves previously saved SQL and bindings for workflow editing or regeneration.
- Returns an empty SQL body when no stored workflow exists for the requested workflow ID.

This resource complements the SQL-to-Workflow service by providing persistent storage for SQL definitions, enabling workflows to be saved and reloaded without regenerating SQL from scratch.

---

# Request Flow

The SQL-to-Workflow translation process follows these steps:

1. A SQL query is received through `WorkflowSqlResource`.
2. Apache Calcite parses and validates the query.
3. A logical relational plan is generated.
4. The optimizer rewrites and optimizes the logical plan.
5. `AliasExpander` resolves aliases where necessary.
6. `TexeraConverter` converts the optimized logical plan into Texera workflow operators and links.
7. A `TexeraWorkflow` object is constructed.
8. The generated workflow is serialized into JSON and returned through the REST API.

---

# Current Functionality

The current implementation supports:

- SQL parsing and validation using Apache Calcite
- Logical query plan generation
- Custom logical plan optimization
- Dataset metadata retrieval
- Dataset preview APIs
- SQL alias resolution
- Translation of logical plans into Texera workflow representations
- Workflow operator generation
- Workflow link generation
- JSON serialization of generated workflows
- REST APIs for SQL processing and workflow generation

---

# Demo

> https://www.youtube.com/watch?v=IBk0Eis8Uqo

The demo video provides an end-to-end walkthrough of the SQL-to-Workflow translation process and showcases the generated workflow output.

---

# Next Steps

The core SQL-to-Workflow service has been implemented, including SQL parsing, logical plan generation, optimization, and workflow generation.

The remaining work primarily involves integrating the service with Texera's existing execution pipeline.

### Pending Integration

- Update the **CSVScan Operator** to support the new hybrid type inference implementation introduced by the SQL-to-Workflow service.
- Align the operator with the newly introduced code structure and supporting files to ensure compatibility with the generated workflow.
- Verify end-to-end execution after integrating the updated CSVScan operator with the generated workflows.

Apart from the CSVScan operator integration, the majority of the SQL-to-Workflow backend implementation is complete.
