# 🌐 VPS Infrastructure: Advanced Self-Hosted Ecosystem

Este repositorio documenta la arquitectura de mi infraestructura privada alojada en un VPS Linux. He diseñado un ecosistema de nube personal basado en **Docker**, centrado en la **seguridad Zero-Trust**, la **observabilidad en tiempo real** y la **automatización CI/CD**.

---

## 🏗️ Arquitectura del Sistema (Diagrama de Red)

```mermaid
graph TD
    subgraph Internet
        User((Usuario))
        WG_Client[Wireguard VPN Client]
    end

    subgraph Security_Layer [Capa de Acceso y Seguridad]
        Traefik[Traefik Reverse Proxy]
        Authelia[Authelia SSO + 2FA]
        Wireguard[Wireguard VPN Server]
    end

    subgraph Core_Services [Servicios Core]
        Gitea[Gitea Git Service]
        Runner[Gitea Runner]
        Portainer[Portainer UI]
        Adminer[Adminer DB Manager]
    end

    subgraph Monitoring_Stack [Observabilidad LGP]
        Prometheus[Prometheus Metrics]
        Grafana[Grafana Dashboards]
        Loki[Loki Logs]
        cAdvisor[cAdvisor Containers]
        NodeExp[Node Exporter Host]
        Promtail[Promtail Log Collector]
    end

    subgraph Data_Layer [Persistencia]
        Postgres[(PostgreSQL Cluster)]
        Redis[(Redis Cache)]
    end

    User -->|HTTPS/443| Traefik
    WG_Client -->|Encrypted Tunnel| Wireguard
    Traefik -->|Auth Guard| Authelia
    
    Authelia -.-> Gitea
    Authelia -.-> Portainer
    Authelia -.-> Grafana
    
    Prometheus --> cAdvisor
    Prometheus --> NodeExp
    Loki --> Promtail
    Grafana --> Prometheus
    Grafana --> Loki

    Gitea --> Runner
    Runner -->|Deploy| Core_Services

    🛠️ Stack Tecnológico Detallado
```

## 1. Networking & Edge
Traefik v3: Cerebro de enrutamiento con resolución automática de certificados TLS vía Let's Encrypt.

Wireguard: VPN de alto rendimiento para acceso cifrado a la red interna del VPS, evitando exponer herramientas críticas a internet.

## 2. Seguridad & Acceso (SSO)
Authelia: Implementación de Single Sign-On (SSO) con autenticación de dos factores (2FA via Microsoft Authenticator). Protege el acceso a Gitea, Portainer, Grafana y Adminer.

## 3. Observabilidad (LGP Stack)
Prometheus: Recolección de métricas de series temporales de todo el stack.

Grafana: Visualización avanzada de métricas y logs a través de dashboards personalizados.

Loki + Promtail: Agregación y centralización de logs de todos los contenedores Docker.

cAdvisor & Node Exporter: Monitoreo de hardware (CPU, RAM, Disco) y estadísticas de red tanto del Host como de cada contenedor individual.

## 4. Base de Datos & Persistencia
PostgreSQL: Motor de base de datos relacional principal para aplicaciones de producción.

Redis: Almacenamiento en caché en memoria para alto rendimiento.

Adminer: Interfaz web ligera para la gestión de bases de datos, protegida tras el portal de seguridad SSO.

## 5. Ciclo de Vida & Gestión (GitOps)
Gitea & Gitea Runner: Servidor de Git privado y ejecución nativa de pipelines CI/CD.

Portainer: Gestión visual de contenedores, imágenes, redes y volúmenes de Docker.
---
## 🚀 Flujo CI/CD Automático
He implementado un pipeline de despliegue automatizado que garantiza un flujo de trabajo profesional:

Push: El código se sube a la instancia privada de Gitea.

Build: Gitea Runner dispara la construcción de la imagen Docker (Multi-stage build para optimización de peso).

Registry: La imagen construida se almacena de forma segura en un Registry Privado.

Deploy: El Runner actualiza el stack en el VPS mediante Docker Compose, aplicando cambios sin tiempo de inactividad.

Mirror: Sincronización automática (Mirroring) de ramas productivas hacia GitHub para visibilidad pública.

Alerting: Notificaciones instantáneas del estado del despliegue enviadas a Telegram.

---
## 🛡️ Fortalecimiento de Seguridad
Aislamiento de Redes: Uso de redes Docker separadas (web, internal, monitoring) para que los servicios solo tengan visibilidad de sus dependencias directas.

Zero Exposure Policy: Las herramientas de monitoreo y gestión no son accesibles desde el internet público; requieren túnel VPN o autenticación SSO forzada.

Principio de Privilegio Mínimo: Configuración de contenedores para ejecutarse con usuarios no-root siempre que la compatibilidad lo permita.

---
*Documentado por [Victor Ponce](https://victorponce.dev) — Entusiasta de la Infraestructura y el Self-Hosting.*

---