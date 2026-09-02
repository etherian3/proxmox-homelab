# Application Platform

## Overview

The homelab includes a custom containerized application deployed on `application-01`.

The application is packaged as a Docker image and runs as part of a Docker Compose-based application stack.

The application source code is maintained independently in:

`applications/homelab-app`

This repository is included in the main `proxmox-homelab` repository as a Git submodule.

---

## Application Architecture

The current application architecture is:

```text
Client
  │
  ▼
app.homelab
  │
  ▼
Traefik
  │
  ▼
homelab-app
  │
  ▼
PostgreSQL
```

The application itself is not directly exposed as the primary user-facing endpoint.

Traefik provides the entry point and routes requests to the application container.

---

## Application Host

The application runs on:

```text
Hostname    : application-01
IP Address  : 192.168.1.111
Platform    : Ubuntu
Runtime     : Docker
```

The application container listens on:

```text
8000/tcp
```

The application stack is managed through Docker Compose.

---

## Application Components

The current application environment consists of:

```text
application-01
│
├── Traefik
│
├── homelab-app
│
└── PostgreSQL
```

Each component has a specific responsibility.

| Component | Responsibility |
|---|---|
| Traefik | Reverse proxy / routing |
| homelab-app | Application |
| PostgreSQL | Persistent relational database |

---

## Application Container

The custom application runs inside a Docker container.

The application server is started using Uvicorn.

Conceptually:

```text
Docker
  │
  ▼
homelab-app
  │
  ▼
Uvicorn
  │
  ▼
Application
  │
  ▼
:8000
```

The application source repository contains the application code, Dockerfile, Python dependencies, and Docker Compose configuration.

---

## Docker Compose

Docker Compose is used to define and operate the application stack.

The Compose configuration describes the services and their relationships.

Conceptually:

```text
Docker Compose
│
├── Traefik
├── homelab-app
└── PostgreSQL
```

This provides a reproducible definition of the application runtime environment.

---

## Network Architecture

The application stack uses Docker networks to control communication between services.

The architecture separates the reverse-proxy connectivity from the internal application/database communication.

```text
                    Docker Host
                         │
                  ┌──────┴──────┐
                  │             │
                  ▼             ▼
             proxy network   app network
                  │             │
                  ▼             ├── homelab-app
               Traefik          │
                                └── PostgreSQL
```

Traefik connects to the proxy network so it can route incoming requests to the application.

The application communicates with PostgreSQL through the internal application network.

---

## Request Flow

A request to the application follows this path:

```text
User
 │
 │ HTTP
 ▼
app.homelab
 │
 ▼
Traefik
 │
 │ Docker network
 ▼
homelab-app:8000
 │
 │ Database queries
 ▼
PostgreSQL
```

This architecture separates the public-facing routing layer from the application and database services.

---

## Application Repository

The application is maintained independently from the infrastructure repository.

```text
proxmox-homelab
      │
      ▼
applications/
      │
      ▼
homelab-app
```

This separation allows the application lifecycle to evolve independently from the infrastructure lifecycle.

The main infrastructure repository documents how the application is deployed and operated.

The application repository contains the implementation itself.

---

## Deployment

The current deployment process is configuration-driven.

Ansible prepares the application environment and manages the deployment configuration on `application-01`.

The general workflow is:

```text
Git
 │
 ▼
homelab-app
 │
 ▼
Ansible
 │
 ▼
application-01
 │
 ▼
Docker Compose
 │
 ├── Traefik
 ├── homelab-app
 └── PostgreSQL
```

The current deployment is automated through Ansible configuration.

Full CI/CD automation is planned as a future improvement.

---

## Configuration and Secrets

Application configuration may contain environment-specific values such as database credentials.

Sensitive values are not intended to be committed as plaintext into Git.

The infrastructure deployment uses Ansible Vault for protected configuration.

This allows the infrastructure repository to remain version-controlled without exposing sensitive credentials.

---

## Application Availability

The application is accessed locally through:

```text
app.homelab
```

The hostname resolves to the application VM:

```text
app.homelab
      │
      ▼
192.168.1.111
```

Traefik then routes the request to the application container.

---

## Operational Responsibilities

The application platform introduces several operational responsibilities:

### Application

Responsible for:

- Application logic
- HTTP handling
- Runtime dependencies

### Docker

Responsible for:

- Container lifecycle
- Application runtime isolation
- Container networking

### Traefik

Responsible for:

- Request routing
- Reverse proxying
- Service exposure

### PostgreSQL

Responsible for:

- Persistent application data
- Relational database operations

### Ansible

Responsible for:

- Server configuration
- Deployment configuration
- Service orchestration

---

## Monitoring

The application host is monitored independently from the application itself.

Host metrics are collected using Node Exporter and Prometheus.

```text
application-01
      │
      ▼
Node Exporter
      │
      ▼
Prometheus
      │
      ▼
Grafana
```

This allows infrastructure health to be observed while the application is running.

Application-specific metrics and health monitoring can be expanded later.

---

## Current Status

Implemented:

- [x] Custom application
- [x] Docker containerization
- [x] Docker Compose deployment
- [x] Uvicorn application server
- [x] PostgreSQL
- [x] Docker networking
- [x] Traefik integration
- [x] Local hostname access
- [x] Ansible deployment
- [x] Git repository
- [x] Application repository as Git submodule

---

## Future Improvements

Planned improvements include:

- [ ] Automated application tests
- [ ] CI pipeline
- [ ] Automated Docker image build
- [ ] Container image registry
- [ ] Automated deployment
- [ ] Deployment rollback
- [ ] Application health monitoring
- [ ] Application-specific Prometheus metrics
- [ ] Better secret management
- [ ] Production-style deployment strategy

---

## Portfolio Value

The application platform demonstrates practical experience beyond infrastructure provisioning.

It combines:

- Application development
- Docker containerization
- Docker Compose
- Reverse proxy
- Database integration
- Infrastructure automation
- Configuration management
- Monitoring

This creates a complete path from infrastructure to a running workload.

---

## Related Documentation

- [Architecture](../architecture/architecture.md)
- [VM Inventory](../infrastructure/vm-inventory.md)
- [Network Architecture](../infrastructure/network.md)
- [Ansible](../automation/ansible.md)
- [Traefik](reverse-proxy.md)
- [Database](database.md)
- [Monitoring](monitoring.md)
- [Alerting](../operations/alerting.md)
