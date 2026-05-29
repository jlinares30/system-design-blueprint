# ◇ Medallion & Enterprise Data Architecture

This document outlines the end-to-end enterprise system design, detailing user authentication routing, Kubernetes microservices, external system interfaces, and the Medallion Data Lakehouse pipeline.

---

## ▪ End-to-End System Blueprint

The enterprise system is divided into three primary layers: the Application Client & API Edge, the Containerized Business Services Tier, and the Analytic Medallion Pipeline.

```mermaid
graph TD
    %% Clients
    Users((Users)) --> WebApp[Web Application]
    Users --> MobileApp[Mobile Application]

    %% Edge
    WebApp --> IAM[IAM System]
    MobileApp --> IAM
    
    WebApp --> APIGW[API Gateway]
    MobileApp --> MGW[Mobile Gateway]
    
    APIGW -->|Validate JWT| IAM
    MGW -->|Validate JWT| IAM
    APIGW --> Redis[(Edge Redis Cache)]
    MGW --> Redis

    %% Application Core
    subgraph Kubernetes K8s Cluster
        APIGW --> BC_IAM[BC: IAM]
        MGW --> BC_Ship[BC: Shipment Management]
        
        BC_IAM --> DB_IAM[(IAM DB)]
        BC_Ship --> DB_Ship[(Shipment DB)]
        BC_Doc[BC: Document Management] --> DB_Doc[(Doc DB)]
        BC_Contract[BC: Contract Management] --> DB_Contract[(Contract DB)]
        BC_X[BC: Billing / Others] --> DB_X[(Billing DB)]
    end

    %% External Systems
    DB_Ship --> ExtSystems[External Backend Systems]
    DB_Doc --> ExtSystems
    DB_Contract --> ExtSystems
    
    subgraph External Systems & Datastores
        ExtSystems --> SUNAT[SUNAT / Customs]
        ExtSystems --> ERP[ERP System]
        ExtSystems --> CRM[CRM System]
        ExtSystems --> XLSX[Legacy File Dumps / XLSX]
    end

    %% Data Pipeline
    SUNAT --> Bronze[(Bronze Data Lakehouse)]
    ERP --> Bronze
    CRM --> Bronze
    XLSX --> Bronze

    %% Medallion Pipeline
    subgraph Medallion Data Pipeline
        Bronze -->|Cleansing & Enrichment| Silver[(Silver Data Lakehouse)]
        Silver -->|Aggregation & Star Schema| Gold[(Golden Data Lake)]
    end

    %% Serving Layer
    Gold --> BI_Tools[Serving / BI Tools]
    subgraph Consumption Tier
        BI_Tools --> PowerBI[PowerBI]
        BI_Tools --> Looker[Looker]
        BI_Tools --> QuickSight[AWS QuickSight]
    end
```

---

## ▪ Layer 1: Client & API Edge

1. **Authentication & Authorization (IAM):** The Identity & Access Management system handles credentials, role-based access control (RBAC), permission mappings, and client application registration.
2. **API & Mobile Gateways:** All incoming user requests pass through dedicated [API Gateways](file:///d:/u/system-design-blueprint/GLOSSARY.md#api-gateway). Gateways perform token verification ([JWT](file:///d:/u/system-design-blueprint/GLOSSARY.md#jwt-json-web-token) validation) against the IAM system before routing traffic to backend services.
3. **Edge Caching:** A [Redis](file:///d:/u/system-design-blueprint/GLOSSARY.md#redis) cache cluster is placed at the gateway level to cache session metadata, route maps, and transient configurations to reduce query overheads.

<details>
<summary><b>🔍 Design Notes & Clarifications</b></summary>

*   **JWT Verification Offloading:** Gateways check token signatures cryptographically using cached public keys. They do not request verification from the IAM database on every call, avoiding a performance bottleneck.
*   **Edge Cache Strategy:** Redis is configured with an active expiration TTL (Time-To-Live). Cache invalidations are pushed via Pub/Sub from the IAM service when user privileges change.
*   *Add your custom notes, validations, or configurations here...*
</details>

---

## ▪ Layer 2: Business Core (Kubernetes Cluster)

Services are packaged into independent containers and managed inside a [Kubernetes](file:///d:/u/system-design-blueprint/GLOSSARY.md#kubernetes-k8s) cluster. The architecture adheres to [Domain-Driven Design](file:///d:/u/system-design-blueprint/GLOSSARY.md#domain-driven-design-ddd) principles:
*   **[Bounded Contexts](file:///d:/u/system-design-blueprint/GLOSSARY.md#bounded-context) (BCs):** Business units are isolated into dedicated contexts, such as:
    *   *IAM Context:* Manages account lifecycles and scopes.
    *   *Shipment Management Context (Gestión Embarques):* Manages shipping logistics.
    *   *Document Management Context (Gestión Documental):* Manages document ingestion and retrieval.
    *   *Contract Management Context (Gestión Contratos):* Manages client-vendor legal contracts.
*   **[Database-per-Service](file:///d:/u/system-design-blueprint/GLOSSARY.md#database-per-service):** Each Bounded Context owns its schema/database, preventing database-level coupling.

<details>
<summary><b>🔍 Design Notes & Clarifications</b></summary>

*   **Database Isolation Boundaries:** Direct SQL connections between different microservice containers are forbidden. All data exchange is performed via public APIs (REST/gRPC) or events.
*   **Transactional Autonomy:** If an action spans multiple contexts, they achieve eventual consistency using event streaming rather than distributed two-phase commit transactions.
*   *Add your custom notes, namespace configurations, or microservice interaction details here...*
</details>

---

## ▪ Layer 3: External & Operational Data Sinks

Transactional databases feed data or coordinate state changes with external systems and core business backends:
*   **SUNAT / Customs:** Government validation structures.
*   **[ERP](file:///d:/u/system-design-blueprint/GLOSSARY.md#erp-enterprise-resource-planning) & [CRM](file:///d:/u/system-design-blueprint/GLOSSARY.md#crm-customer-relationship-management) Systems:** Legacy databases and CRM interfaces recording company resource allocations and customer history.
*   **Flat Files (XLSX):** Batch exports from offline environments.

<details>
<summary><b>🔍 Design Notes & Clarifications</b></summary>

*   **Legacy Sync Decoupling:** Integrating with legacy ERPs is handled via background message queues. Gateway request threads do not wait synchronously for slow ERP API responses.
*   **File Drop Directory:** The file-based XLSX sink uses a scheduled cron job that picks up files and converts them into JSON streams before ingestion into the data lake.
*   *Add your custom notes on legacy system endpoints or data mapping models here...*
</details>

---

## ▪ Layer 4: Analytics Medallion Architecture

Data from transactional systems and external resources is ingested into a [Lakehouse](file:///d:/u/system-design-blueprint/GLOSSARY.md#lakehouse) pipeline divided into three distinct validation stages of the [Medallion Architecture](file:///d:/u/system-design-blueprint/GLOSSARY.md#medallion-architecture):

### 1. Bronze Layer (Raw Ingestion)
*   **Purpose:** Lands raw, unmodified source data directly from transaction logs, APIs, and file exports.
*   **Design:** Append-only schema preservation. Contains history of all operations, including updates and deletes as raw delta records. No validation checks are run.

### 2. Silver Layer (Cleaned & Enriched)
*   **Purpose:** Standardizes raw records to create an enterprise-wide view of domain concepts.
*   **Design:** Processes raw Bronze data through cleaning steps:
    *   *Data Cleansing:* Deduplicates rows, handles missing/null values, and corrects formatting errors.
    *   *Enrichment:* Normalizes schema definitions and casts data types.

### 3. Gold Layer (Business Ready)
*   **Purpose:** Powers reporting, metrics computation, and business analytics.
*   **Design:** Aggregates Silver datasets into specialized schemas (e.g., [Star Schema](file:///d:/u/system-design-blueprint/GLOSSARY.md#star-schema) with Facts and Dimensions) optimized for querying speed. 

<details>
<summary><b>🔍 Design Notes & Clarifications</b></summary>

*   **Storage and Partitions:** Medallion data tables use the [Delta / Parquet Format](file:///d:/u/system-design-blueprint/GLOSSARY.md#delta-parquet-format) partitioned by transaction date to optimize scanning performance.
*   **Raw Traceability Rule:** The Bronze layer serves as the ultimate source of truth. If a transformation rule in the Silver or Gold layer is found to contain a bug, the tables can be rebuilt entirely by replaying the raw Bronze history.
*   *Add your custom notes on ETL tooling (Spark, dbt), execution schedules, or data catalogs here...*
</details>

---

## ▪ Layer 5: Consumption Tier (Serving Layer)

Business-ready structures in the Gold Data Lake are exposed directly to Business Intelligence (BI) applications to generate reports and drive operational decisions:
*   **PowerBI**
*   **Looker**
*   **AWS QuickSight**

<details>
<summary><b>🔍 Design Notes & Clarifications</b></summary>

*   **Query Access Policies:** Access to the Gold layer is governed by column-level encryption to hide sensitive customer data from general reporting roles.
*   **DirectQuery vs Import Thresholds:** Report developers should prefer data import mode for operational dashboards, and [DirectQuery](file:///d:/u/system-design-blueprint/GLOSSARY.md#directquery) only for real-time tracking dashboards that query large historical tables.
*   *Add your custom notes on BI user roles, gateway connections, or dashboard requirements here...*
</details>
