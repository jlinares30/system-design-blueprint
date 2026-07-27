# 📈 04 - Escalabilidad, Resiliencia y Seguridad

Una de las mayores fortalezas de Kubernetes es su capacidad para **autoescalar la infraestructura automáticamente** en momentos de alta demanda y garantizar que las aplicaciones con fallos se recuperen de inmediato sin intervención humana.

---

## 1. Autoescalado Automático (Autoscaling)

Kubernetes escala a tres niveles diferentes:

### A. Horizontal Pod Autoscaler (HPA)
Monitorea continuamente métricas como el consumo de CPU, memoria o solicitudes entrantes. Si el consumo supera un umbral (ej. 70% CPU), crea más réplicas del Pod. Cuando el tráfico baja, destruye los Pods sobrantes.

```mermaid
graph TD
    subgraph HPA_Controller["📊 HPA Controller (Vigilante de Métricas)"]
        Metrics["Métricas de CPU > 70%"]
    end

    subgraph Trafico["⚡ Carga de Tráfico"]
        Tr1["🔥 Evento / Pico de Tráfico (Black Friday)"]
    end

    subgraph Estado_Pods["Evolución de Pods de la Aplicación"]
        P1["📦 Pod 1 (CPU 90%)"]
        P2["📦 Pod 2 (CPU 88%)"]
        P3["📦 Pod 3 (Nuevo Pod Creado)"]
        P4["📦 Pod 4 (Nuevo Pod Creado)"]
    end

    Tr1 --> P1 & P2
    P1 & P2 --> Metrics
    Metrics -->|Ordena escalar| HPA_Controller
    HPA_Controller -->|Crea réplicas| P3 & P4
```

### B. Cluster Autoscaler y Karpenter (Escalado de Nodos)
Si el HPA intenta crear nuevos Pods pero **los servidores físicos (nodos) ya no tienen memoria disponible**, el **Cluster Autoscaler** o **Karpenter** le pide automáticamente al proveedor de nube (AWS, GCP, Azure) que encienda un nuevo servidor e instale las dependencias de inmediato.

---

## 2. Health Checks y Revisiones Médicas (Probes)

¿Cómo sabe Kubernetes si tu aplicación está viva, lista para recibir clientes o si se quedó "congelada" en un bucle infinito? Lo hace usando tres tipos de revisiones automáticas (**Probes**):

```mermaid
flowchart TD
    subgraph Probes_K8s["🩺 Health Checks del Kubelet"]
        Startup["1. Startup Probe<br/>¿La aplicación ya terminó de encender y cargar archivos?"]
        Readiness["2. Readiness Probe<br/>¿La app está lista para recibir clientes en este instante?"]
        Liveness["3. Liveness Probe<br/>¿La app sigue funcionando bien o se congeló?"]
    end

    subgraph Acciones["Acciones Automáticas"]
        A_Wait["⏳ Esperar a que inicie la app"]
        A_Traffic["🚦 Si responde SI: Enviar Tráfico HTTP<br/>❌ Si responde NO: Quitar del Servicio temporalmente"]
        A_Restart["💀 Si responde NO: Matar y Reiniciar el Pod de inmediato"]
    end

    Startup -->|En proceso| A_Wait
    Startup -->|Completado| Readiness & Liveness
    Readiness --> A_Traffic
    Liveness --> A_Restart
```

---

## 3. Clases de Calidad de Servicio (QoS) y Límites de Recursos

Para evitar que una sola aplicación consuma toda la memoria del servidor y haga colapsar a las demás, debemos especificar **Resource Requests** (lo mínimo que necesita para encender) y **Resource Limits** (lo máximo que se le permite consumir).

```yaml
resources:
  requests:
    memory: "256Mi"   # Reserva garantizada de memoria
    cpu: "250m"       # 0.25 núcleos de CPU
  limits:
    memory: "512Mi"   # Si la app supera los 512MB, K8s la mata con error OOMKilled (Out Of Memory)
    cpu: "500m"       # Limita el uso máximo de CPU
```

---

## 4. Seguridad y Control de Acceso (RBAC)

**RBAC (Role-Based Access Control)** define **quién** puede realizar **qué acciones** sobre **qué recursos** del cluster.

```mermaid
graph LR
    subgraph Sujeto["👤 Usuario / Servicio"]
        User["👨‍💻 Desarrollador Juan"]
    end

    subgraph Permisos["📜 Reglas de Seguridad (Role)"]
        Role["Role: Dev-Backend<br/>• Leer Pods: SÍ<br/>• Ver Logs: SÍ<br/>• Borrar Nodos: NO"]
    end

    subgraph Enlace["🔗 Asignación (RoleBinding)"]
        RB["RoleBinding: Juan ➔ Dev-Backend"]
    end

    User --> RB --> Role
```

* **Role:** Conjunto de permisos dentro de un espacio de nombres (*Namespace*).
* **ClusterRole:** Conjunto de permisos globales en todo el cluster.
* **RoleBinding / ClusterRoleBinding:** El puente que vincula a un usuario o aplicación con un `Role` o `ClusterRole`.
