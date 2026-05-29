# ◇ Caso Práctico: Banco Finanzas Perú
## Consolidación de Datos para Detección de Riesgo Crediticio

Este caso práctico detalla la aplicación del patrón de [Arquitectura Medallón](../medallion-architecture.md) en un escenario real del sector financiero peruano, diseñado para reducir los tiempos de evaluación crediticia de **24-72 horas a menos de 15 minutos**.

---

### Contexto del Negocio y Problema
**Banco Finanzas Perú** es una entidad financiera de mediana escala. Sus decisiones de crédito eran lentas debido a que la información requerida estaba distribuida en sistemas legacy fragmentados. El equipo de riesgos cruzaba manualmente reportes en Excel, lo que generaba demoras inaceptables de días para aprobar o rechazar solicitudes de crédito.

#### Fuentes de Datos de Origen
*   **Core Bancario (AS/400):** Cuentas, saldos y movimientos históricos (Legacy / Batch).
*   **CRM Comercial:** Datos de cliente, segmentación e historial (Microservicio REST).
*   **Motor de Préstamos:** Solicitudes, cuotas y estados de morosidad (Microservicio REST).
*   **SBS / INFOCORP:** Score crediticio externo y deudas consolidadas (API Externa).
*   **App Móvil:** Comportamiento digital, logins y transferencias (Event Streaming / Kafka).

---

### Diagrama de Flujo del Dato (Medallón)

```mermaid
flowchart TD
    subgraph Orígenes de Datos
        Core["Core Bancario (AS/400) - Batch CSV (Nocturno)"]
        CRM["CRM Comercial - API REST (Pull / CDC)"]
        Prestamos["Motor de Préstamos - API REST (Pull / CDC)"]
        Kafka["App Móvil - Kafka Event Streaming (Real-time)"]
        SBS["SBS / INFOCORP - API Externa (Limitado 50 req/h)"]
    end

    subgraph Capa Bronze (Raw & Preserved)
        B_Core[(Bronze: raw_core_accounts)]
        B_CRM[(Bronze: raw_crm_clients)]
        B_Prestamos[(Bronze: raw_loans)]
        B_Kafka[(Bronze: raw_app_events)]
        B_SBS[(Bronze: raw_sbs_scores)]
    end

    subgraph Capa Silver (Cleaned & Unified)
        S_Clientes[(Silver: dim_clientes)]
        S_Cuentas[(Silver: fact_saldos_historicos)]
        S_Prestamos[(Silver: fact_prestamos)]
        S_Eventos[(Silver: fact_actividad_app)]
        S_Riesgo[(Silver: dim_score_financiero)]
    end

    subgraph Capa Gold (Business Ready)
        G_Riesgo[(Gold: customer_credit_360)]
    end

    subgraph Consumidores (Serving)
        ML[Modelos ML / Evaluación en Tiempo Real]
        BI[Dashboards Analista de Riesgo - PowerBI]
    end

    %% Ingestión a Bronze
    Core -->|SFTP + Airflow| B_Core
    CRM -->|Debezium CDC / API Pull| B_CRM
    Prestamos -->|Debezium CDC / API Pull| B_Prestamos
    Kafka -->|Spark Streaming / Kafka Connect| B_Kafka
    SBS -->|Airflow Pull bajo demanda / Cacheado| B_SBS

    %% Transformaciones a Silver
    B_Core -->|Deduplicación & Limpieza| S_Cuentas
    B_CRM -->|Estandarización de IDs - DNI/RUC| S_Clientes
    B_Prestamos -->|Conversión de Tipos & Nulos| S_Prestamos
    B_Kafka -->|Filtros de Logins y Transacciones| S_Eventos
    B_SBS -->|Normalización de Rangos de Score| S_Riesgo

    %% Consolidación a Gold
    S_Clientes & S_Cuentas & S_Prestamos & S_Eventos & S_Riesgo -->|Agregación & Reglas de Negocio| G_Riesgo

    %% Consumo
    G_Riesgo --> BI
    G_Riesgo --> ML
```

---

### Solución Técnica y Preguntas de Diseño

#### 1. Ingestión de Datos (Capa Bronze)
El ingreso de información se ha diseñado para evitar la sobrecarga de los sistemas transaccionales y respetar las cuotas del proveedor externo:
*   **Core Bancario (AS/400):** Ingestión nocturna batch vía **Apache Airflow**. Se extrae el dump de CSV desde un servidor SFTP y se vuelca sin modificar a formato Parquet/Delta en la capa Bronze.
*   **CRM y Motor de Préstamos:** Ingestión near real-time mediante **CDC (Change Data Capture)** usando **Debezium**, o cargas incrementales basadas en API REST.
*   **SBS / INFOCORP (API Externa):** 
    > [!IMPORTANT]
    > **Restricción de 50 peticiones/hora:** Para no exceder la cuota, se implementa un mecanismo de caché en la capa Silver. Si un cliente solicita una evaluación, el motor verifica si existe un registro en `dim_score_financiero` con menos de 15 días de antigüedad. Si no existe o caducó, se realiza la llamada a la API externa, se guarda el resultado crudo en Bronze y se refresca Silver.
*   **App Móvil:** Streaming de eventos en tiempo real mediante **Kafka Connect** escribiendo directamente a tablas Delta en Bronze.

#### 2. Transformaciones y Limpieza (Capa Silver)
Se aplica un modelo relacional limpio (Dimensiones y Hechos):
*   **Identidad de Clientes:** Limpieza y estandarización del identificador único del cliente (DNI, CE, RUC).
*   **Estandarización y Normalización:** Casteo de tipos de datos, formateo de fechas e integración de tipos de cambio oficiales para convertir monedas a una divisa base (Soles - PEN).
*   **Reglas de Negocio Iniciales:** Deduplicación de transacciones, cálculo de días de mora históricos, y filtrado de eventos ruidosos de la aplicación móvil.

#### 3. Producto de Datos (Capa Gold)
Se construye una vista unificada desnormalizada llamada **`customer_credit_360`**:
*   Consolida toda la información demográfica, de saldos del Core, comportamiento de pago, score SBS y comportamiento en la app móvil.
*   **Consumo ML:** Expuesto mediante un motor de base de datos de baja latencia o Feature Store para que el modelo predictivo de riesgo evalúe solicitudes automáticas en tiempo real.
*   **Consumo BI:** Tablas de consulta analítica optimizadas para tableros de PowerBI y Looker utilizados por los analistas de riesgos.

#### 4. Orquestación y Cómputo (Airflow + Databricks)
*   **Apache Airflow:** Orquesta la secuencia de tareas, gatilla los jobs de Spark en Databricks y gestiona las dependencias de los pipelines batch e híbridos.
*   **Databricks:** Provee la capacidad de procesamiento distribuido bajo demanda para procesar grandes volúmenes de datos usando PySpark, manteniendo las propiedades ACID gracias a **Delta Lake**.

#### 5. Trazabilidad, Cumplimiento y Costos
*   **Cumplimiento SBS (Retención de 5 años):** Se habilita el linaje automático mediante **Unity Catalog** en Databricks. La retención se maneja en almacenamiento en la nube de bajo costo (ej. AWS S3 o Azure ADLS Gen2), implementando políticas de ciclo de vida para archivar logs históricos a clases de almacenamiento frías (Glacier / Archive) después de un año, optimizando enormemente el costo.
*   **Nube vs On-Premise:** El enfoque híbrido en la nube es altamente recomendable frente a soluciones físicas (On-Premise) debido al ahorro en CapEx y a la flexibilidad de escalar/apagar los clústeres de cómputo Spark de Databricks cuando no existan pipelines activos.
