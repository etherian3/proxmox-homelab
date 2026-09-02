# Proxmox Homelab Infrastructure

> A production-like self-hosted infrastructure environment built on Proxmox VE to practice and demonstrate Infrastructure as Code, configuration management, containerization, monitoring, alerting, and DevOps automation.

## Overview

This project is a personal homelab infrastructure built on **Proxmox VE**.

The goal is not only to run applications, but to design and operate an infrastructure environment that follows practices commonly used in modern IT infrastructure and DevOps environments.

The project covers the infrastructure lifecycle from provisioning virtual machines to configuring servers, deploying applications, monitoring infrastructure, and handling operational alerts.

### Project Goals

- Practice Infrastructure as Code
- Automate server configuration
- Deploy containerized applications
- Implement reverse proxy infrastructure
- Deploy centralized monitoring
- Implement infrastructure alerting
- Build operational and troubleshooting experience
- Document infrastructure decisions and procedures
- Build a reproducible and maintainable homelab environment

---

## Architecture

```text
                         Proxmox VE
                             │
              ┌──────────────┴──────────────┐
              │                             │
              ▼                             ▼
       management-01                 application-01
       192.168.1.110                 192.168.1.111
              │                             │
              │ Ansible                     │
              │                             │
              └──────────────┐              │
                             │              │
                             ▼              ▼
                         Configuration   Docker
                                         │
                              ┌──────────┼──────────┐
                              │          │          │
                              ▼          ▼          ▼
                           Traefik     App      PostgreSQL
                              │
                              ▼
                        app.homelab


                    Monitoring Layer
                           │
                    ┌──────┴──────┐
                    ▼             ▼
              Node Exporter   Application VM
                    │
                    ▼
                Prometheus
                    │
                    ▼
                 Grafana
                    │
                    ▼
              Grafana Alerting
                    │
                    ▼
                 Telegram
```

Detailed architecture documentation:

- [`docs/architecture/architecture.md`](docs/architecture/architecture.md)

---

## Infrastructure

The infrastructure runs on **Proxmox VE** using Ubuntu virtual machines.

| Host | IP Address | Purpose |
|---|---|---|
| `management-01` | `192.168.1.110` | Automation / Ansible Control Node |
| `application-01` | `192.168.1.111` | Application and Monitoring |

### Infrastructure as Code

**Terraform** is used to provision the virtual machine infrastructure on Proxmox.

Current provisioning includes:

- VM creation
- VM resources
- Network configuration
- Static IP configuration
- Proxmox VM lifecycle management

Terraform implementation:

[`infrastructure/terraform-vm-templates`](infrastructure/terraform-vm-templates)

---

## Configuration Management

**Ansible** is used to configure and manage the Ubuntu servers after provisioning.

Current automation includes:

- Common server configuration
- Docker installation
- Application infrastructure
- Monitoring infrastructure
- Service configuration

Ansible implementation:

[`infrastructure/homelab-ansible`](infrastructure/homelab-ansible)

---

## Application Platform

The application platform runs on Docker.

```text
Docker Host
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

The custom application is deployed as a container and served through Traefik.

Application source:

[`applications/homelab-app`](applications/homelab-app)

---

## Reverse Proxy

**Traefik** provides the reverse proxy layer for containerized services.

Application traffic follows:

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

This allows the application to be accessed through a hostname instead of exposing the application container directly as the primary entry point.

---

## Database

The application uses **PostgreSQL** as its database backend.

```text
homelab-app
      │
      ▼
PostgreSQL
```

The database runs as a container and communicates with the application through an internal Docker network.

---

## Monitoring

The infrastructure is monitored using:

- **Prometheus**
- **Grafana**
- **Node Exporter**

Monitoring flow:

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

Current monitoring includes:

- CPU utilization
- Memory utilization
- Disk utilization
- Network traffic
- Host availability

A Grafana dashboard has been created to visualize the host metrics.

---

## Alerting

Grafana Alerting is configured to detect high CPU utilization.

Current alert:

```text
CPU utilization > 80%
for 5 minutes
```

Notification flow:

```text
CPU Load
   │
   ▼
Node Exporter
   │
   ▼
Prometheus
   │
   ▼
Grafana Alerting
   │
   ▼
Telegram
```

The alert lifecycle has been tested using `stress-ng`.

Test result:

```text
Normal
  │
  ▼
CPU stress generated
  │
  ▼
CPU > threshold
  │
  ▼
Alert: Firing
  │
  ▼
Telegram notification
  │
  ▼
Stress stopped
  │
  ▼
Alert: Normal
```

This verifies both alert triggering and recovery behavior.

---

## Repository Structure

The project separates the main portfolio documentation from the implementation repositories.

```text
proxmox-homelab/
│
├── README.md
├── CHANGELOG.md
│
├── docs/
│   ├── architecture/
│   ├── infrastructure/
│   ├── automation/
│   ├── services/
│   └── operations/
│
├── infrastructure/
│   ├── terraform-vm-templates/
│   └── homelab-ansible/
│
└── applications/
    └── homelab-app/
```

The implementation repositories are maintained as Git submodules, allowing each project to retain its own Git history and lifecycle.

---

## Technology Stack

| Category | Technology |
|---|---|
| Virtualization | Proxmox VE |
| Operating System | Ubuntu |
| Infrastructure as Code | Terraform |
| Configuration Management | Ansible |
| Containerization | Docker |
| Container Orchestration | Docker Compose |
| Reverse Proxy | Traefik |
| Application | Python / FastAPI |
| Database | PostgreSQL |
| Metrics | Prometheus |
| Visualization | Grafana |
| Host Metrics | Node Exporter |
| Alerting | Grafana Alerting |
| Notifications | Telegram |
| Version Control | Git / GitHub |

---

## Current Status

### Completed

- [x] Proxmox infrastructure
- [x] Ubuntu virtual machines
- [x] Terraform VM provisioning
- [x] Ansible control node
- [x] Automated server configuration
- [x] Docker infrastructure
- [x] Containerized application
- [x] PostgreSQL
- [x] Traefik reverse proxy
- [x] Prometheus
- [x] Grafana
- [x] Node Exporter
- [x] Infrastructure monitoring dashboard
- [x] CPU alerting
- [x] Telegram notifications
- [x] Alert firing test
- [x] Alert recovery test
- [x] Git-based repository structure
- [x] Infrastructure documentation

### Roadmap

- [ ] GitHub Actions CI/CD
- [ ] Automated application deployment
- [ ] Automated testing
- [ ] Improved secrets management
- [ ] Internal DNS
- [ ] Backup and restore strategy
- [ ] Disaster recovery procedures
- [ ] Infrastructure hardening
- [ ] Additional monitoring and alert rules
- [ ] Infrastructure change automation

---

## Documentation

### Architecture

- [Architecture Overview](docs/architecture/architecture.md)

### Infrastructure

- Proxmox Infrastructure
- Network Architecture
- VM Inventory

### Automation

- Terraform
- Ansible

### Services

- Application
- Traefik
- PostgreSQL
- Monitoring

### Operations

- Alerting
- Backup & Restore
- Disaster Recovery

Documentation will be expanded as each infrastructure component is implemented and validated.

---

## Purpose

This homelab is primarily a learning and portfolio project.

Rather than only listing technologies on a CV, the project provides practical evidence of working with infrastructure, automation, application deployment, monitoring, troubleshooting, and operational workflows.

The infrastructure is continuously improved as new DevOps and infrastructure practices are introduced.

---

## Author

**Etherian3**

GitHub:

[`github.com/etherian3`](https://github.com/etherian3)
