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

---

## ▪ Conceptos Avanzados de Datos y Patrones de Integración

En la arquitectura empresarial moderna, la transición de los datos transaccionales a los analíticos requiere patrones específicos para garantizar el rendimiento, la frescura de los datos y evitar el impacto negativo en los sistemas de producción:

### 1. [ODS (Operational Data Store)](file:///d:/u/system-design-blueprint/GLOSSARY.md#ods-operational-data-store)
El **ODS** actúa como una base de datos central provisional que consolida datos operativos en tiempo real de múltiples fuentes (como ERP, CRM y bases de datos transaccionales). 
*   **Propósito:** Ofrecer consultas rápidas de estado y reportes operativos diarios sin sobrecargar las bases de datos de producción (OLTP).
*   **Características:** Almacena datos con un nivel bajo de transformación o agregación, usualmente enfocándose en el estado actual o datos recientes (últimos 30 a 90 días).

```mermaid
flowchart TD
    subgraph Fuentes_OLTP [Bases de Datos Transaccionales - OLTP]
        DB1[(App DB)]
        DB2[(CRM System)]
        DB3[(ERP System)]
    end

    subgraph Integracion [Capa de Integración en Tiempo Real]
        CDC[CDC / Replicación Activa]
    end

    subgraph Capa_ODS [Operational Data Store - ODS]
        ODS_DB[(ODS Database)]
        ODS_DB -->|Tiempo Real / Datos Recientes| Rep_Ops[Reportes Operacionales / Call Center]
        ODS_DB -->|Consultas Rápidas de Estado| Ops_Dash[Dashboards Operacionales del Día]
    end

    subgraph Capa_Analitica [Capa Analítica de Largo Plazo]
        ETL[ETL / ELT Pipeline]
        Lakehouse[(Data Lakehouse / Medallion)]
    end

    %% Flujos de datos
    DB1 -->|CDC / Eventos| CDC
    DB2 -->|CDC / Eventos| CDC
    DB3 -->|CDC / Eventos| CDC

    CDC -->|Ingestión Continua| ODS_DB
    ODS_DB -->|Carga de Historial Incremental| ETL
    ETL -->|Refinamiento y Agregación| Lakehouse
```


### 2. [CDC (Change Data Capture)](file:///d:/u/system-design-blueprint/GLOSSARY.md#cdc-change-data-capture)
El patrón **CDC** es un conjunto de tecnologías que identifica y captura cambios (inserciones, actualizaciones y eliminaciones) realizados en una base de datos origen, publicando estos eventos en tiempo real a consumidores descendentes.
*   **Implementación común:** Lectura de los registros de transacciones (transaction logs) del motor de base de datos (por ejemplo, usando herramientas como Debezium y Kafka Connect).
*   **Beneficios:** Evita realizar consultas periódicas costosas (`SELECT * FROM table WHERE updated_at > ...`) que causan bloqueos e impacto de rendimiento en producción.

### 3. [Database Lookups](file:///d:/u/system-design-blueprint/GLOSSARY.md#database-lookup)
Un **Lookup** en base de datos es una operación de consulta de referencia que busca valores específicos en tablas secundarias utilizando un identificador (por ejemplo, buscar el nombre del cliente basado en `client_id` al procesar un evento de envío).
*   **Estrategias de optimización:** Dado que los lookups frecuentes en pipelines de datos pueden convertirse en un cuello de botella, se optimizan mediante índices secundarios, almacenamiento en memoria (como Redis) o uniones locales (joins en caché).

### 4. [Triggers](file:///d:/u/system-design-blueprint/GLOSSARY.md#trigger)
Un **Trigger** (disparador) es un bloque de código procedimental que reside y se ejecuta automáticamente dentro del motor de base de datos en respuesta a eventos de manipulación de datos (`INSERT`, `UPDATE`, `DELETE`).
*   **Consideraciones en diseño:** Aunque son útiles para mantener la integridad de datos a nivel local o auditar cambios simples, el uso excesivo de triggers dificulta la depuración del sistema, introduce latencia oculta en las transacciones y limita la escalabilidad horizontal. En sistemas distribuidos, suele preferirse el patrón CDC para reaccionar a cambios de forma asíncrona.

---

## ▪ Comparativas de Arquitectura de Datos

### Cuadro Comparativo: Datos Operacionales (Transaccionales) vs. Datos Analíticos

| Característica / Criterio | Datos Operacionales (OLTP / Transaccional) | Datos Analíticos (OLAP / Analítico) |
| :--- | :--- | :--- |
| **Propósito Principal** | Ejecutar operaciones diarias y registrar transacciones de negocio. | Soportar la toma de decisiones, análisis de tendencias y BI. |
| **Operaciones Comunes** | Escrituras, actualizaciones y lecturas rápidas de registros individuales (CRUD). | Consultas complejas de lectura y agregación masiva de datos. |
| **Diseño / Estructura** | Altamente normalizado (3NF) para eliminar redundancia. | Desnormalizado (Esquemas de Estrella, Copo de Nieve, Columnar). |
| **Dimensión Temporal** | Estado actual y en tiempo real (instantánea del momento). | Histórico acumulativo (años de evolución del negocio). |
| **Concurrencia** | Miles de transacciones simultáneas por segundo (TPS). | Consultas concurrentes moderadas pero de alto costo de cómputo. |
| **Garantías de Datos** | Estricto cumplimiento de propiedades [ACID](file:///d:/u/system-design-blueprint/GLOSSARY.md#acid). | Consistencia eventual y procesamiento por lotes ([BASE](file:///d:/u/system-design-blueprint/GLOSSARY.md#base)). |
| **Volumen de Datos** | Relativamente bajo o mediano (datos de producción activos). | Altamente masivo (terabytes a petabytes). |

### Cuadro Comparativo: ODS (Operational Data Store) vs. Arquitectura Medallón

| Característica / Criterio | Operational Data Store (ODS) | Arquitectura Medallón (Lakehouse) |
| :--- | :--- | :--- |
| **Enfoque Principal** | Integración y visualización en tiempo real de operaciones diarias activas. | Limpieza, refinamiento estructural y almacenamiento analítico a largo plazo. |
| **Estructura de Datos** | Estructuras operacionales normalizadas o semiesféricas. | Dividida en capas progresivas: Bronce (Crudo), Plata (Limpio) y Oro (Agregado). |
| **Histórico** | Limitado (usualmente almacena datos recientes del negocio, ej. 30 días). | Completo e ilimitado (persiste la historia total de transacciones). |
| **Almacenamiento Físico** | Motores relacionales tradicionales (OLTP como PostgreSQL, SQL Server). | Formatos de archivos distribuidos optimizados ([Delta / Parquet](file:///d:/u/system-design-blueprint/GLOSSARY.md#delta-parquet-format)). |
| **Ingestión típica** | Replicación activa de base de datos o CDC en tiempo real. | Procesamiento batch programado o pipelines de streaming incremental. |
| **Consumidores** | Sistemas operacionales de soporte, Call Centers, tableros de control diarios. | Herramientas de BI ([PowerBI](file:///d:/u/system-design-blueprint/03-data-tier-architecture/medallion-architecture.md#powerbi), [Looker](file:///d:/u/system-design-blueprint/03-data-tier-architecture/medallion-architecture.md#looker)) y modelos de Machine Learning. |

---

> [!IMPORTANT]
> Why Medallion Architecture?
> Why is important to use all this process to analyze data?
> 
> 1. **Data quality:** By processing data through multiple layers, we can ensure that the data is clean and accurate.
> 2. **Data governance:** By using a medallion architecture, we can ensure that the data is properly governed.
> 3. **Data lineage:** By using a medallion architecture, we can track the lineage of the data as it flows through the system.
> 4. **Data security:** By using a medallion architecture, we can ensure that the data is properly secured.
> 5. **Data can be use for differents purposes:** Data can be used for different purposes, such as **analytics**, **machine learning**, and **business intelligence**.
> 6. **Data can be used for real-time processing:** Data can be processed in real-time as it flows through the system.

---

## ▪ Casos de Estudio y Aplicaciones Reales

Para comprender cómo se aplica la arquitectura Medallón en escenarios empresariales complejos con restricciones de negocio, rendimiento y regulatorias, consulta los siguientes casos de uso:

*   **[Caso Práctico: Banco Finanzas Perú (Riesgo Crediticio)](file:///d:/Jorge/system-design-blueprint/03-data-tier-architecture/cases/case-medallion-architecture.md)**
    *   *Objetivo:* Reducir el tiempo de evaluación crediticia de 72 horas a menos de 15 minutos.
    *   *Desafíos:* Ingestión de datos legados (AS/400), streaming en tiempo real (App Móvil), rate-limiting estricto (API SBS/INFOCORP) y cumplimiento regulatorio (retención de 5 años).
