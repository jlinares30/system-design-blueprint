# 📦 02 - Objetos Básicos: Pods, Deployments y Services

Para trabajar con Kubernetes no manipulamos contenedores directamente. En su lugar, utilizamos **Objetos de Kubernetes** definidos mediante archivos YAML.

---

## 1. El Pod: La Unidad Mínima de Kubernetes

Un **Pod** es la envoltura o cápsula más pequeña que puedes crear en Kubernetes. Generalmente contiene un solo contenedor (tu aplicación), pero a veces puede llevar contenedores secundarios auxiliares llamados **Sidecars** (por ejemplo, para enviar logs o métricas).

### Anatomía Interna de un Pod

```mermaid
graph TD
    subgraph Pod["📦 Pod (IP compartida: 10.244.0.15)"]
        direction TB
        subgraph Red_y_Storage["🌐 Recursos Compartidos"]
            IP["Dirección IP Única"]
            Vol["Volumen de Disco Compartido"]
        end

        subgraph Contenedores
            C1["🚀 Contenedor Principal<br/>(Mi Aplicación Web - Port 8080)"]
            C2["🛠️ Contenedor Sidecar<br/>(Colector de Logs - Envoy/Fluentd)"]
        end

        IP <--> C1 & C2
        Vol <--> C1 & C2
    end
```

> 💡 **Regla de oro:** Todos los contenedores dentro de un mismo Pod comparten la misma red (se leen por `localhost`) y pueden compartir el mismo almacenamiento.

---

## 2. Deployments: Garantizando Autorrecuperación y Escala

Si creas un Pod individual y el servidor físico falla, ese Pod muere para siempre. Por eso, en producción nunca creamos Pods sueltos; utilizamos un **Deployment**.

Un **Deployment** se encarga de:
1. Mantener encendida la cantidad exacta de réplicas que pidas.
2. Reemplazar Pods destruidos automáticamente.
3. Actualizar la versión de tu código **sin interrumpir el servicio (Zero-Downtime)**.

### Estrategia de Actualización Sin Caídas (Rolling Update)

```mermaid
stateDiagram-v2
    [*] --> Version_1: Estado Inicial (3 Pods v1.0)
    
    state Version_1 {
        Pod_V1_A --> Pod_V1_B
        Pod_V1_B --> Pod_V1_C
    }

    Version_1 --> Despliegue_Progresivo: Inicia `kubectl set image v2.0`

    state Despliegue_Progresivo {
        [*] --> Crea_Nuevos: Nace Pod 1 (v2.0)
        Crea_Nuevos --> Elimina_Viejos: Verifica salud de Pod 1 (v2.0) -> Elimina Pod A (v1.0)
        Elimina_Viejos --> Proceso_Repetido: Nace Pod 2 (v2.0) -> Elimina Pod B (v1.0)
    }

    Despliegue_Progresivo --> Version_2: Finalizado sin caídas 🚀

    state Version_2 {
        Pod_V2_A --> Pod_V2_B
        Pod_V2_B --> Pod_V2_C
    }
```

---

## 3. Services (Servicios): Redes y Descubrimiento

Los Pods son **efímeros**: nacen, mueren y al renacer cambian de IP. ¿Cómo hacen otras aplicaciones o usuarios para conectarse a ellos si sus direcciones IP cambian todo el tiempo?

La respuesta es un **Service (Servicio)**. Un Servicio funciona como un **punto de contacto fijo y balanceador de carga interno**.

### Tipos de Servicios en Kubernetes

```mermaid
graph TB
    subgraph Red_Externa["🌐 Internet / Clientes Externos"]
        User["👩‍💻 Usuario en Navegador"]
    end

    subgraph Cluster_K8s["☸️ Cluster de Kubernetes"]
        subgraph Services["Tipos de Servicio"]
            NodePort["🚪 NodePort<br/>(Expone un puerto alto en los servidores<br/>ej. 30080)"]
            LoadBalancer["⚖️ LoadBalancer<br/>(Crea un Balanceador de carga<br/>en AWS / GCP / Azure)"]
            ClusterIP["🔒 ClusterIP<br/>(IP Interna solo visible dentro del cluster)"]
        end

        subgraph Mis_Pods["Pods de mi Aplicación"]
            P1["📦 Pod 1"]
            P2["📦 Pod 2"]
            P3["📦 Pod 3"]
        end
    end

    User -->|Tráfico Público| LoadBalancer
    User -->|Acceso por IP:Puerto| NodePort
    LoadBalancer --> ClusterIP
    NodePort --> ClusterIP
    ClusterIP -->|Balancea tráfico| P1 & P2 & P3
```

* **`ClusterIP` (Por defecto):** Otorga una IP fija **solo accesible desde dentro del cluster**. Ideal para bases de datos o servicios internos.
* **`NodePort`:** Abre un puerto específico (entre 30000 y 32767) directamente en todas las máquinas de tu cluster.
* **`LoadBalancer`:** Se conecta con tu proveedor de nube (AWS ALB, GCP Load Balancer) para otorgarte una IP pública oficial.

---

## 4. ConfigMaps y Secrets: Desacoplando Configuraciones

Nunca debes hardcodear contraseñas, URLs de bases de datos o claves API dentro de la imagen de tu contenedor.

```mermaid
flowchart LR
    subgraph Almacenamiento_Config["⚙️ Configuraciones en Kubernetes"]
        CM["📄 ConfigMap<br/>(Datos públicos: PUERTO=8080, MODO=prod)"]
        SEC["🔐 Secret<br/>(Datos sensibles codificados en Base64: DB_PASSWORD)"]
    end

    subgraph Aplicacion["📦 Pod de Aplicación"]
        App["🚀 Tu Aplicación"]
    end

    CM -->|Inyectado como variables de entorno o archivos| App
    SEC -->|Inyectado de forma segura| App
```

* **`ConfigMap`:** Almacena variables de entorno, archivos de configuración `.env` o `.json` en texto plano.
* **`Secret`:** Almacena datos confidenciales (tokens, llaves SSH, contraseñas de bases de datos).

---

## 📄 Ejemplo Práctico de Manifiesto YAML

A continuación se muestra cómo se define un **Deployment** y un **Service** en un único archivo declarativo:

```yaml
# 1. Definición del Servicio
apiVersion: v1
kind: Service
metadata:
  name: mi-servicio-web
spec:
  type: ClusterIP
  ports:
    - port: 80          # Puerto expuesto por el Servicio
      targetPort: 8080  # Puerto donde escucha el Pod internamente
  selector:
    app: mi-web-app     # Busca los Pods con esta etiqueta

---
# 2. Definición del Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mi-deployment-web
spec:
  replicas: 3           # Queremos 3 Pods exactamente iguales
  selector:
    matchLabels:
      app: mi-web-app
  template:
    metadata:
      labels:
        app: mi-web-app
    spec:
      containers:
        - name: contenedor-web
          image: nginx:alpine
          ports:
            - containerPort: 8080
```
