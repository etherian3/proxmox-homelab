# VM Inventory

## Overview

The homelab infrastructure currently consists of two Ubuntu virtual machines running on Proxmox VE.

The VMs are separated by responsibility:

- `management-01` is used for infrastructure automation.
- `application-01` hosts the application platform and monitoring stack.

This separation provides a basic management-plane and workload-plane architecture.

---

## Virtual Machine Inventory

| VM | VMID | IP Address | vCPU | RAM | Disk | Role |
|---|---:|---|---:|---:|---:|---|
| `management-01` | `100` | `192.168.1.110/24` | 2 | 2 GB | 24 GB | Ansible Control Node |
| `application-01` | — | `192.168.1.111/24` | — | — | — | Application & Monitoring |

> Resource information for `application-01` will be documented after the final VM configuration is recorded.

---

## management-01

### Purpose

`management-01` acts as the management and automation node for the homelab.

Its primary responsibility is to provide a centralized location for infrastructure configuration and Ansible automation.

### Configuration

| Property | Value |
|---|---|
| Hostname | `management-01` |
| VMID | `100` |
| IP Address | `192.168.1.110/24` |
| Gateway | `192.168.1.1` |
| vCPU | 2 |
| RAM | 2 GB |
| Disk | 24 GB |
| OS | Ubuntu |
| Primary Role | Ansible Control Node |

### Installed Tooling

The VM contains the Ansible automation environment used to manage the homelab.

The project uses a Python virtual environment to isolate the Ansible installation from the system Python environment.

Current Ansible version:

```text
Ansible Core 2.17.14
```

The Ansible environment is used to manage both the management and application nodes.

---

## application-01

### Purpose

`application-01` is the primary workload server.

It hosts the containerized application platform together with its supporting services.

### Network Configuration

| Property | Value |
|---|---|
| Hostname | `application-01` |
| IP Address | `192.168.1.111/24` |
| Role | Application & Monitoring Server |

### Services

The following services currently run on this VM:

```text
application-01
│
├── Traefik
│
├── homelab-app
│
├── PostgreSQL
│
├── Prometheus
│
├── Grafana
│
└── Node Exporter
```

### Application

The custom application runs inside Docker and listens internally on port `8000`.

Traffic is exposed through Traefik rather than relying on direct application access.

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
homelab-app:8000
```

### Database

PostgreSQL runs as a separate Docker container.

The application communicates with PostgreSQL through an internal Docker network.

### Monitoring

The host is monitored using Node Exporter.

Metrics are collected by Prometheus and visualized through Grafana.

---

## Network Layout

```text
                     LAN
                      │
              ┌───────┴────────┐
              │                │
              ▼                ▼
       management-01     application-01
       192.168.1.110     192.168.1.111
              │                │
              │                ├── Traefik
              │                ├── App
              │                ├── PostgreSQL
              │                ├── Prometheus
              │                ├── Grafana
              │                └── Node Exporter
              │
              └── Ansible
```

---

## Management Flow

The intended operational flow is:

```text
Developer / Operator
        │
        ▼
Git Repository
        │
        ▼
management-01
        │
        ▼
Ansible
        │
        ▼
application-01
```

Infrastructure provisioning is handled separately through Terraform:

```text
Terraform
    │
    ▼
Proxmox VE
    │
    ▼
Virtual Machines
```

This creates a separation between:

- Infrastructure provisioning
- Server configuration
- Application runtime

---

## Design Decisions

### Separate Management Node

A dedicated management node was introduced instead of running automation directly from the application server.

Benefits:

- Separates management responsibilities from workloads
- Provides a centralized automation environment
- Better represents a production-style architecture
- Makes future expansion easier

### Separate Application Node

Application workloads and monitoring services are isolated from the management node.

This allows the infrastructure to evolve toward additional workload nodes without changing the management architecture.

---

## Future Expansion

The current two-node architecture is intentionally small.

Future infrastructure may introduce:

```text
                 Proxmox VE
                     │
       ┌─────────────┼─────────────┐
       │             │             │
       ▼             ▼             ▼
 management-01  application-01  monitoring-01
       │             │             │
     Ansible       Docker        Monitoring
```

Additional nodes will only be introduced when they provide a clear infrastructure or learning benefit.

---

## Related Documentation

- [Architecture](../architecture/architecture.md)
- [Terraform](../automation/terraform.md)
- [Ansible](../automation/ansible.md)
- [Application](../services/application.md)
- [Monitoring](../services/monitoring.md)
