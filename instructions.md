# SQL-to-Workflow Service

## Overview

The SQL-to-Workflow Service is a standalone backend microservice developed for Apache Texera that translates SQL queries into Texera workflow representations.

The service uses Apache Calcite for SQL parsing, validation, logical planning, and optimization. Once a logical plan is generated, it is translated into Texera workflow operators and links, producing a workflow that can be executed within Texera.

This document serves as a guide to the current implementation and is intended to help reviewers quickly understand the project structure and major components.

---

# Architecture

> **Insert Architecture Diagram Here**

```md
![SQL-to-Workflow Architecture](docs/architecture-diagram.png)
```

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

> **Insert Demo Video Link Here**

Example:

```text
https://youtu.be/<your-demo-video>
```

The demo video provides an end-to-end walkthrough of the SQL-to-Workflow translation process and showcases the generated workflow output.

---

# Repository Notes

The architecture diagram provides a visual overview of the translation pipeline, while the demo video demonstrates the service in action. This document is intended as a quick reference for navigating the implementation and understanding the responsibilities of the major components within the SQL-to-Workflow service.