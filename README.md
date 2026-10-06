<div align="center">

# ⚡ LandmansCore — Enterprise AI & ERP Operating Platform
### Distributed Polyglot Microservices Architecture with Event-Driven AI Pipelines

[![.NET Version](https://img.shields.io/badge/.NET-10.0-512BD4?logo=dotnet&logoColor=white)](https://dotnet.microsoft.com/)
[![C# Clean Architecture](https://img.shields.io/badge/Backend-C%23_%7C_DDD_%7C_CQRS-239120?logo=csharp&logoColor=white)](#)
[![Python FastAPI](https://img.shields.io/badge/AI_Services-Python_FastAPI-009688?logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![RabbitMQ](https://img.shields.io/badge/Messaging-RabbitMQ_Event--Driven-FF6600?logo=rabbitmq&logoColor=white)](https://www.rabbitmq.com/)
[![PostgreSQL & pgvector](https://img.shields.io/badge/Database-PostgreSQL_%7C_pgvector-336791?logo=postgresql&logoColor=white)](https://github.com/pgvector/pgvector)
[![MinIO](https://img.shields.io/badge/Storage-MinIO_S3_Compatible-C72C48?logo=minio&logoColor=white)](https://min.io/)
[![SignalR](https://img.shields.io/badge/Real--Time-SignalR_WebSockets-512BD4)](#)
[![React](https://img.shields.io/badge/Frontend-React_TypeScript_%7C_Vite-61DAFB?logo=react&logoColor=black)](https://react.dev/)

<p align="center">
  <a href="#-executive-summary">Executive Summary</a> •
  <a href="#-distributed-system-architecture">System Architecture</a> •
  <a href="#-polyglot-microservices-breakdown">Microservices Breakdown</a> •
  <a href="#-event-driven-messaging--job-orchestration">Event-Driven Lifecycle</a> •
  <a href="#-secure-text-to-sql--hardened-ai-services">Hardened AI Services</a>
</p>

</div>

---

## 📌 Executive Summary

Enterprise Enterprise Resource Planning (ERP) and Customer Relationship Management (CRM) environments are traditionally monolithic, rigid, and slow to integrate unstructured cognitive capabilities. Conversely, standalone generative AI prototypes frequently fail in corporate deployments due to lack of strict data modeling, auditability, transactional isolation, and asynchronous workload scaling.

**LandmansCore** is a production-grade, distributed polyglot enterprise operating system. It marries a high-throughput **.NET Clean Architecture core** (Domain-Driven Design, Entity Framework Core, SignalR) with an autonomous **Python AI Services subsystem** (Text-to-SQL, OCR Ingestion, pgvector RAG, MinIO S3 storage). 

The platform bridges real-time web clients with computational AI workers via an **asynchronous RabbitMQ event bus**, ensuring zero-block user experiences, hard transactional boundaries, and multi-tenant ledger security.

---

## 🏛️ Distributed System Architecture

The ecosystem operates across decoupled infrastructure tiers communicating through dual-channel message contracts and real-time WebSocket state synchronizations:

```mermaid
graph TD
    Client[React 19 Dashboard<br/>Vite + TypeScript] <-->|SignalR WebSockets<br/>Live Job Telemetry| WebApi[.NET WebAPI Gateway<br/>Controllers & Real-Time Hubs]
    
    subgraph CoreBackend ["1. Enterprise .NET Core Tier (DDD & CQRS)"]
        WebApi --> AppService[Application & Domain Logic<br/>Tenant / Customer / CRM Entities]
        AppService --> PostgresSQL[(Transactional PostgreSQL<br/>Core Business Ledger)]
        AppService --> RabbitPublisher[RabbitMQ Event Publisher<br/>Topic: ai.job.requested]
    end

    subgraph MessagingTier ["2. Asynchronous Message Broker"]
        RabbitPublisher --> RabbitMQ{RabbitMQ Exchange<br/>Message Broker & Dead Letter Queues}
        RabbitMQ --> PythonConsumer[Python Worker Consumer<br/>ai-services/messaging/consumer.py]
    end

    subgraph AIServices ["3. Python AI & Semantic Worker Subsystem"]
        PythonConsumer --> AIServiceRouter{Task Dispatcher}
        AIServiceRouter --> TextToSQL[Text-to-SQL Engine<br/>Sandboxed Read-Only SQL Execution]
        AIServiceRouter --> OCRProcessor[Multimodal OCR Engine<br/>Document Text Reconstruction]
        AIServiceRouter --> VectorEngine[Vector & RAG Engine<br/>pgvector HNSW Semantic Search]
        
        AIServices -.-> MinIO[(MinIO Object Storage<br/>S3 Secure Document Buckets)]
        AIServices -.-> ReadOnlyDB[(PostgreSQL Database<br/>Enforced Read-Only Role)]
    end

    PythonConsumer -->|Publish: ai.job.completed| RabbitMQ
    RabbitMQ --> NetConsumer[C# Status Consumer<br/>AiJobStatusConsumer.cs]
    NetConsumer --> WebApi
```
---

## 🔬 Polyglot Microservices Breakdown

### 1. Enterprise .NET Core Architecture (`/backend`)
Engineered according to strict **Domain-Driven Design (DDD)** and **Clean Architecture** patterns:
* **`LandmansCore.Domain`:** Isolated domain primitives, domain events, and core enterprise entities (`Tenant`, `Customer`, `Employee`, `CrmContact`, `AiDocument`, `AiJob`).
* **`LandmansCore.Infrastructure`:** Implementation of persistence layers, repository contracts (`AiJobRepository`), RabbitMQ publishers (`RabbitMqMessagePublisher.cs`), and background message listeners (`AiJobStatusConsumer.cs`).
* **`LandmansCore.WebApi`:** RESTful API gateways (`AiDocumentController`, `CrmController`, `ReportingController`) and persistent real-time streaming hubs (`SignalR`).

### 2. Autonomous Python AI Services (`/ai-services`)
A specialized asynchronous Python runtime designed for heavy matrix calculations and LLM orchestration:
* **Asynchronous Queue Worker (`messaging/consumer.py`):** Consumes incoming event frames from RabbitMQ, executes compute-heavy AI tasks in the background, and reports telemetry back to the message broker.
* **Database & Migration Engine:** Independent **Alembic** migration pipeline maintaining dedicated relational models (`models.py`, `session.py`).
* **Object Storage Layer (`test_minio_upload.py`):** Integrates **MinIO** for S3-compatible, on-premise object storage, managing raw document scans and compiled report artifacts.

### 3. Reactive Real-Time Frontend (`/frontend-skeleton`)
A high-performance modern web dashboard built with **React, TypeScript, and Vite**:
* **Live WebSocket Integration (`SignalRService.ts`):** Automatically registers listeners to backend hubs. Long-running OCR and Text-to-SQL jobs stream progress bars and state transitions live into the UI without polling.
* **Modular Operations Layout:** Unified interface spanning AI Reporting (`AiReporting.tsx`), Backoffice operations, CRM tracking, and HR data management.

---

## 🔄 Event-Driven Messaging & Job Lifecycle

To ensure optimal responsiveness, computationally expensive generative AI processes are completely decoupled from the synchronous HTTP request-response cycle:

```text
[1. User Action]  ──> HTTP POST /api/aidocument/process
[2. .NET Gateway] ──> Persists AiJob (Status: Pending) into PostgreSQL
[3. RabbitMQ]     ──> Publishes event to queue: `ai.job.processing`
[4. Python AI]    ──> Consumes message, fetches file from MinIO, executes OCR/LLM
[5. Python AI]    ──> Emits completion frame: `ai.job.completed` (with payload)
[6. .NET Consumer]──> Updates database record & triggers SignalR Hub
[7. React Client] ──> Receives instant WebSocket push: Updates UI state in real time (< 50ms)
```

---

## 🔒 Hardened AI Services & Secure Text-to-SQL

Allowing language models to interact with relational databases introduces severe security hazards (SQL Injection, unauthorized data mutations, cross-tenant leaks). 

LandmansCore implements a **Multi-Layer Defensive Text-to-SQL Architecture**:

### 1. Database-Level Read-Only Sandboxing
The Python Text-to-SQL engine (`services/text_to_sql`) connects to the database via a strictly isolated role provisioned via `create-readonly-role.sql`:
* Granted exclusively `SELECT` privileges.
* Hard revocation of `INSERT`, `UPDATE`, `DELETE`, `DROP`, `ALTER`, or execution permissions on administrative stored procedures.
* Even in the event of an adversarial prompt injection, the LLM is physically incapable of altering database state.

### 2. Schema Context Injection & Sanitization
* The `schema_context.py` pipeline dynamically injects authorized table DDLs and foreign-key topologies into the LLM system prompt.
* SQL queries are validated through an abstract syntax tree (AST) parser prior to execution to enforce query timeouts, pagination limits (`LIMIT 100`), and tenant scoping.

### 3. Integrated pgvector Dense Retrieval
* Document indexers (`services/vector/document_indexer.py`) chunk corporate documentation, store dense embeddings in PostgreSQL using the **pgvector HNSW extension**, and execute semantic hybrid search alongside structured tabular reporting.

---


## 🛠️ Enterprise Tech Stack Matrix

| Domain | Technology | Implementation Role & Production Specifications |
| :--- | :--- | :--- |
| **Enterprise Backend** | **.NET 10.0 (C#)** | Clean Architecture, Domain-Driven Design (DDD), Entity Framework Core |
| **Real-Time WebSockets**| **Microsoft SignalR** | Live bi-directional state synchronization pushing background AI job updates |
| **AI Worker Microservice**| **Python 3.12+ & FastAPI** | Asynchronous task execution, schema context extraction & Text-to-SQL parsing |
| **Message Broker** | **RabbitMQ** | Decoupled event exchange routing `ai.job.requested` and `ai.job.completed` queues |
| **Primary Database** | **PostgreSQL 16+** | Relational transactional ledger with multi-schema multi-tenant isolation |
| **Vector Engine** | **pgvector (HNSW)** | Dense vector indexing for hybrid document retrieval and contextual RAG |
| **Object Storage (S3)** | **MinIO** | On-premise S3-compatible storage for raw invoices, PDFs, and compiled report assets |
| **Frontend Runtime** | **React 19 & TypeScript** | Component-driven operational dashboard compiled via Vite |
| **Migration Engines** | **Alembic & EF Core** | Dual-layer schema evolution managing independent service database states |
| **Container Orchestration**| **Docker Compose** | Multi-container stack (Postgres, RabbitMQ, MinIO, AI Services, .NET, Client) |

---

## 📂 Repository Topology

```text
landmans_core/
├── backend/                              # Enterprise .NET Core Monorepo
│   ├── src/
│   │   ├── LandmansCore.Domain/          # Core Domain Layer (DDD Entities, Aggregates & Enums)
│   │   │   └── Entities/                 # Tenant, Customer, Employee, CrmContact, AiDocument, AiJob
│   │   ├── LandmansCore.Infrastructure/  # Adapters, Persistence & External Services
│   │   │   ├── Messaging/                # RabbitMqMessagePublisher.cs, AiJobStatusConsumer.cs
│   │   │   ├── Repositories/             # AiJobRepository.cs, EF Core DbContexts
│   │   │   └── Security/                 # Claims, Tenant Scoping & Authorization Policies
│   │   └── LandmansCore.WebApi/          # Presentation Gateway & Endpoints
│   │       ├── Controllers/              # AiDocumentController, CrmController, ReportingController
│   │       ├── Hubs/                     # Real-time SignalR Event Hubs
│   │       └── Program.cs                # DI Container, Pipeline Middleware & Hosted Services
│   ├── tests/                            # Unit & Integration Tests (xUnit)
│   └── LandmansCore.sln                  # Visual Studio / .NET Solution File
│
├── ai-services/                          # Autonomous Python AI Microservice
│   ├── alembic/                          # Standalone Database Migrations for AI Models
│   ├── app/
│   │   ├── api/                          # FastAPI Routers (ad_hoc_ai.py, rag_router.py)
│   │   ├── messaging/                    # RabbitMQ Consumer Worker (consumer.py)
│   │   ├── services/
│   │   │   ├── ocr/                      # Layout-aware document OCR processor
│   │   │   ├── text_to_sql/              # AST-sanitized, read-only Text-to-SQL engine
│   │   │   └── vector/                   # Document indexer, embeddings & semantic search
│   │   ├── db/                           # SQLAlchemy models & session factory
│   │   └── core/config.py                # Pydantic Settings environment configuration
│   ├── requirements.txt                  # Python AI dependency lockfile
│   └── test_minio_upload.py              # S3 Object Storage integration test harness
│
├── frontend-skeleton/                    # React 19 Client Dashboard
│   ├── src/
│   │   ├── pages/                        # AiReporting.tsx, CrmPage.tsx, DashboardHome.tsx, HrPage.tsx
│   │   ├── services/                     # SignalRService.ts (WebSocket listener), PdfExportService.ts
│   │   ├── components/                   # Reusable glassmorphic UI widgets & layout frames
│   │   └── App.tsx                       # Client router and live state provider
│   ├── vite.config.ts                    # Vite build configuration
│   └── package.json                      # Node.js dependencies
│
└── infra/                                # Infrastructure as Code (IaC) & Data Seeding
    ├── db/init/
    │   ├── create-readonly-role.sql      # Sandboxed database role for Text-to-SQL security
    │   ├── init-schemas.sql              # Multi-tenant schema isolation setup
    │   ├── pgvector_migration.sql        # Vector extension initialization & HNSW indexes
    │   └── seed.sql                      # Initial enterprise test seed data
    ├── docker/docker-compose.yml         # Containerized multi-service orchestration stack
    └── setup-landmans-core-v2.sh         # Automated host environment bootstrapping script
```

---

## ⚡ Quickstart & Local Deployment

### Prerequisites
* **.NET 10.0 SDK**
* **Python 3.12+**
* **Node.js 18+** & npm
* **Docker & Docker Compose**

---

### 1. Launch Containerized Infrastructure Stack

Start the core infrastructure services (PostgreSQL with pgvector, RabbitMQ Broker, and MinIO S3 Object Storage):

```bash
# Navigate to infrastructure directory
cd infra/docker

# Spin up core infrastructure containers
docker compose up -d postgres rabbitmq minio

# Verify container health
docker compose ps
```

---

### 2. Database Initialization & Security Hardening

Execute the automated bootstrapping script or run initial schema migrations:

```bash
# Apply initial database schema, pgvector extensions, and sandboxed read-only roles
psql -h localhost -U postgres -d landmans_core -f ../db/init/init-schemas.sql
psql -h localhost -U postgres -d landmans_core -f ../db/init/create-readonly-role.sql
psql -h localhost -U postgres -d landmans_core -f ../db/init/pgvector_migration.sql
psql -h localhost -U postgres -d landmans_core -f ../db/init/seed.sql
```

---

### 3. Start Python AI Microservices & RabbitMQ Worker

```bash
cd ../../ai-services

# Create and activate virtual environment
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Run database migrations
alembic upgrade head

# Launch background RabbitMQ worker consumer
python -m app.messaging.consumer &

# Start FastAPI service
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

---

### 4. Launch .NET Core WebAPI Backend

```bash
cd ../backend

# Restore NuGet dependencies
dotnet restore

# Run WebAPI host
dotnet run --project src/LandmansCore.WebApi
```
> The .NET WebAPI is now operational at `http://localhost:5000` (Swagger UI at `/swagger`).

---

### 5. Launch React 19 Frontend Dashboard

```bash
cd ../frontend-skeleton

# Install dependencies
npm install

# Start development server
npm run dev
```
> Access the live management suite at `http://localhost:5173`.

---

## 📡 Messaging Contracts & Event Protocol

### RabbitMQ Event: `ai.job.requested`
Published by `.NET WebAPI` when a resource-heavy AI job is queued:
```json
{
  "jobId": "8f3b2e10-9c1a-4d2b-b5d1-3e5f2a1b4c6e",
  "tenantId": "tenant-acme-corp",
  "jobType": "TextToSqlReporting",
  "payload": {
    "naturalLanguageQuery": "Calculate quarterly revenue grouped by sales region for 2026",
    "targetSchema": "accounting"
  },
  "timestamp": "2026-10-06T20:30:00Z"
}
```

### RabbitMQ Event: `ai.job.completed`
Emitted by `Python Worker Consumer` upon successful execution:
```json
{
  "jobId": "8f3b2e10-9c1a-4d2b-b5d1-3e5f2a1b4c6e",
  "status": "Success",
  "executionTimeMs": 420,
  "result": {
    "generatedSql": "SELECT region, SUM(amount) FROM accounting.invoices WHERE year = 2026 GROUP BY region;",
    "dataRowCount": 4,
    "summary": "Regional aggregation successfully reconciled against ledger."
  }
}
```

### Real-Time SignalR WebSocket Broadcast
Pushed instantly to the client browser via `.NET SignalR Hub`:
```json
// Client event: "ReceiveJobCompleted"
{
  "jobId": "8f3b2e10-9c1a-4d2b-b5d1-3e5f2a1b4c6e",
  "status": "Completed",
  "progressPercentage": 100,
  "telemetry": {
    "ocrConfidence": 0.98,
    "readOnlySqlExecuted": true
  }
}
```

---

## 👨‍💻 Engineering & Systems Architecture

Architected by **Elvijs Landmans** ([landmansIT](https://landmansit.de)).

* **Polyglot Design Mastery:** Choosing the right tool for the job—combining the raw transactional throughput and type safety of **.NET Clean Architecture** with the mathematical computing power of **Python AI ecosystems**.
* **Zero-Block Event Sourcing:** Leveraging **RabbitMQ** to isolate compute-intensive LLM/OCR workloads from synchronous HTTP gateway threads.
* **Defensive Sandboxing by Default:** Treating stochastic AI models as untrusted actors by enforcing hardware-level read-only SQL execution boundaries.

---

## 📄 License

Proprietary Software. All Rights Reserved. Enterprise platform licensed for corporate operations.
