# ⎈ 01 - Introducción y Arquitectura de Kubernetes

Kubernetes (comúnmente llamado **K8s**) es una plataforma de código abierto diseñada para automatizar el despliegue, escalado y administración de aplicaciones en contenedores.

---

## 💡 ¿Por qué existe Kubernetes? (La Analogía del Puerto)

Imagina un puerto marítimo gigantesco:
* **Docker** son los **contenedores metálicos de carga**. Te permiten empaquetar una aplicación con todas sus dependencias para que funcione igual en cualquier lugar.
* **Kubernetes** es la **grúa automatizada, el puerto completo y la torre de control**. Se asegura de mover los contenedores, reemplazar los que se rompen, descargar la carga cuando hay mucho trabajo y controlar el tráfico marítimo.

### Evolución de la Infraestructura

```mermaid
flowchart LR
    subgraph Servidor_Fisico["1. Servidor Físico (Bare Metal)"]
        direction TB
        HW1[Hardware / CPU / RAM] --> OS1[Sistema Operativo Único] --> AppA[Aplicación A] & AppB[Aplicación B]
    end

    subgraph Maquinas_Virtuales["2. Máquinas Virtuales (VMs)"]
        direction TB
        HW2[Hardware] --> Hypervisor[Hipervisor]
        Hypervisor --> VM1[VM 1 + SO Guest + App A]
        Hypervisor --> VM2[VM 2 + SO Guest + App B]
    end

    subgraph Contenedores["3. Contenedores (Docker)"]
        direction TB
        HW3[Hardware] --> OS3[SO Host] --> Docker[Motor Docker]
        Docker --> C1[Contenedor App A] & C2[Contenedor App B]
    end

    subgraph Kubernetes_Orquestacion["4. Orquestación (Kubernetes)"]
        direction TB
        K8s[Kubernetes Cluster] --> NodeA[Nodo 1] & NodeB[Nodo 2] & NodeC[Nodo 3]
        NodeA --> P1[Pod A]
        NodeB --> P2[Pod B]
        NodeC --> P3[Pod C]
    end
```

---

## 🏛️ Arquitectura del Cluster: El Capataz y los Trabajadores

Un cluster de Kubernetes está dividido en dos partes principales:
1. **Control Plane (El Plano de Control / La Torre de Control):** Toma las decisiones inteligentes.
2. **Worker Nodes (Los Nodos de Trabajo):** Máquinas (físicas o virtuales) que realmente ejecutan tus aplicaciones.

```mermaid
graph TB
    subgraph Control_Plane["🧠 Control Plane (La Torre de Control)"]
        APIServer["🔌 kube-apiserver<br/>(La Puerta de Entrada)"]
        etcd[("📚 etcd<br/>(La Base de Datos del Estado)")]
        Scheduler["📅 kube-scheduler<br/>(El Asignador de Nodos)"]
        Controller["⚙️ kube-controller-manager<br/>(El Vigilante del Estado)"]

        APIServer <--> etcd
        APIServer <--> Scheduler
        APIServer <--> Controller
    end

    subgraph Worker_Node_1["🖥️ Worker Node 1"]
        Kubelet1["👷 kubelet"]
        Proxy1["🚦 kube-proxy"]
        CRI1["📦 Container Runtime"]
        Pod1["📦 Pod 1 (App)"]
        Pod2["📦 Pod 2 (App)"]

        Kubelet1 --> CRI1 --> Pod1 & Pod2
    end

    subgraph Worker_Node_2["🖥️ Worker Node 2"]
        Kubelet2["👷 kubelet"]
        Proxy2["🚦 kube-proxy"]
        CRI2["📦 Container Runtime"]
        Pod3["📦 Pod 3 (App)"]

        Kubelet2 --> CRI2 --> Pod3
    end

    APIServer <-->|Instrucciones| Kubelet1
    APIServer <-->|Instrucciones| Kubelet2
    APIServer <-->|Reglas de Red| Proxy1
    APIServer <-->|Reglas de Red| Proxy2
```

---

## 🧩 Componentes Explicados Sencillamente

### 1. El Control Plane (Cerebro)

* **`kube-apiserver` (El Recepcionista):** Es el punto central de comunicación. Todo pasa por aquí, ya sea un comando escrito por un programador o una orden interna.
* **`etcd` (El Libro de Recuerdos):** Base de datos clave-valor ultrasegura donde Kubernetes guarda exactamente la foto del estado de todo el sistema (cuántos Pods existen, en qué servidores están, etc.).
* **`kube-scheduler` (El Asignador):** Revisa qué servidores (nodos) tienen suficiente CPU y memoria libre para ubicar las nuevas aplicaciones que queremos desplegar.
* **`kube-controller-manager` (El Vigilante):** Se encarga de comparar constantemente la realidad con lo deseado. Si le pediste 3 réplicas de una app y una se rompe, este componente detecta el fallo y ordena crear una nueva.

### 2. Los Worker Nodes (Los Servidores de Trabajo)

* **`kubelet` (El Capataz del Nodo):** Es un agente que corre en cada servidor de trabajo. Recibe instrucciones de la torre de control y asegura que los contenedores estén sanos y corriendo.
* **`kube-proxy` (El Guardia de Tráfico):** Gestiona la red dentro del nodo, configurando reglas IP para que los usuarios u otros servicios puedan conectarse a las aplicaciones.
* **`Container Runtime` (El Motor de Ejecución):** El software encargando de descargar y correr las imágenes de contenedor (usualmente `containerd`).

---

## 🔄 Ciclo de Vida Visual: ¿Qué pasa al ejecutar un comando?

Mira paso a paso lo que sucede cuando ejecutas el comando `kubectl apply -f mi-app.yaml`:

```mermaid
sequenceDiagram
    autonumber
    actor Dev as 👩‍💻 Desarrollador
    participant API as 🔌 kube-apiserver
    participant DB as 📚 etcd
    participant Sched as 📅 kube-scheduler
    participant Kubelet as 👷 kubelet (Worker Node)
    participant CRI as 📦 Container Runtime

    Dev->>API: 1. `kubectl apply -f mi-app.yaml`
    API->>DB: 2. Guarda la intención ("Crear 1 Pod de mi-app")
    API-->>Dev: 3. Confirmación: "Pod creado (Pendiente)"

    loop Revisión constante
        Sched->>API: 4. Detecta Pod sin asignar a ningún nodo
    end

    Sched->>Sched: 5. Calcula qué nodo tiene espacio libre (Node A)
    Sched->>API: 6. Asigna el Pod al Node A
    API->>DB: 7. Actualiza estado en etcd

    loop Monitoreo del Nodo
        Kubelet->>API: 8. Detecta que tiene asignado un nuevo Pod
    end

    Kubelet->>CRI: 9. Ordena: "Descarga la imagen y corre el contenedor"
    CRI-->>Kubelet: 10. Contenedor iniciado exitosamente
    Kubelet->>API: 11. Informa estado: "Pod en estado Running 🟢"
```

---

## 🔑 Términos Clave de esta Sección

* **[Control Plane](file:///d:/Jorge/system-design-blueprint/GLOSSARY.md#control-plane-plano-de-control):** El cerebro administrativo del cluster.
* **Worker Node:** Servidor donde corren físicamente las aplicaciones.
* **`kubectl`:** Herramienta de línea de comandos utilizada por los desarrolladores para enviarle instrucciones a Kubernetes.
