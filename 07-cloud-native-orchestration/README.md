# ⎈ 07 - Orquestación Cloud Native con Kubernetes

¡Bienvenido al módulo de **Kubernetes (k8s)**! Esta guía está diseñada para llevarte desde **cero absoluto** hasta comprender la arquitectura, el autoescalado, la resiliencia y el despliegue de aplicaciones en la nube de forma amigable, clara y altamente visual.

---

## 🗺️ Hoja de Ruta de Aprendizaje

Haz clic en cualquiera de los temas para comenzar:

1. **[01. Introducción y Arquitectura de Kubernetes](file:///d:/Jorge/system-design-blueprint/07-cloud-native-orchestration/01-introduccion-y-arquitectura-k8s.md)**
   * ¿Por qué existe Kubernetes? La analogía del puerto marítimo.
   * El Control Plane (Torre de Control) y los Worker Nodes (Nodos de Trabajo).
   * Ciclo de vida visual de un comando (`kubectl apply`).

2. **[02. Objetos Básicos: Pods, Deployments y Services](file:///d:/Jorge/system-design-blueprint/07-cloud-native-orchestration/02-objetos-basicos-pods-deployments-services.md)**
   * Anatomía de un Pod.
   * Deployments y estrategias de actualización sin caídas (Rolling Updates).
   * Redes y exposición de servicios (ClusterIP, NodePort, LoadBalancer).
   * Manejo de variables y contraseñas con ConfigMaps y Secrets.

3. **[03. Almacenamiento Persistente y Redes Avanzadas](file:///d:/Jorge/system-design-blueprint/07-cloud-native-orchestration/03-almacenamiento-redes-avanzadas.md)**
   * Gestión de volúmenes sin perder datos (StorageClass, PV, PVC).
   * Enrutamiento de URLs con Ingress Controller.
   * Introducción visual a Service Mesh (Istio).

4. **[04. Escalabilidad, Resiliencia y Seguridad](file:///d:/Jorge/system-design-blueprint/07-cloud-native-orchestration/04-escalabilidad-autoscaling-seguridad.md)**
   * Autoescalado automático frente a picos de tráfico (HPA, VPA, Karpenter).
   * Health checks y revisiones médicas (Liveness, Readiness, Startup Probes).
   * Control de acceso y roles con RBAC.

5. **[Caso Práctico: Despliegue de una Tienda en Línea](file:///d:/Jorge/system-design-blueprint/07-cloud-native-orchestration/cases/caso-practico-despliegue-microservicios.md)**
   * Ejemplo completo listo para producción (Frontend + API Backend + Base de datos PostgreSQL).
