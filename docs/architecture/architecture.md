# Homelab Architecture

## Overview

This homelab is a self-hosted infrastructure environment built on Proxmox VE to simulate a production-like IT infrastructure and DevOps workflow.

The project is designed to demonstrate the complete lifecycle of infrastructure and application deployment:

```text
Infrastructure Provisioning
        │
        ▼
    Terraform
        │
        ▼
     Proxmox
        │
        ▼
   Ubuntu VMs
        │
        ▼
      Ansible
        │
        ├──────────────┐
        ▼              ▼
     Docker         Host Configuration
        │
        ▼
    Application
        │
        ├── Traefik
        ├── PostgreSQL
        └── Monitoring
                │
                ▼
        Prometheus + Grafana
                │
                ▼
          Grafana Alerting
                │
                ▼
             Telegram
```

## Infrastructure

The current infrastructure consists of a Proxmox VE host running multiple virtual machines.

### Virtual Machines

| VM | IP Address | Role |
|---|---|---|
| management-01 | 192.168.1.110 | Ansible Control Node |
| application-01 | 192.168.1.111 | Application & Monitoring Server |

### Management VM

`management-01` is responsible for infrastructure automation.

Responsibilities:

- Ansible control node
- Infrastructure configuration
- Remote server management
- Deployment orchestration

### Application VM

`application-01` hosts the application and supporting services.

Responsibilities:

- Docker runtime
- Application
- PostgreSQL
- Traefik
- Prometheus
- Grafana
- Node Exporter

## Automation

### Terraform

Terraform is used for infrastructure provisioning.

Current responsibility:

```text
Terraform
    │
    ▼
Proxmox
    │
    ├── management-01
    └── application-01
```

Terraform configuration is maintained separately in:

`infrastructure/terraform-vm-templates`

This repository is included as a Git submodule.

### Ansible

Ansible is used for server configuration and application infrastructure deployment.

Current responsibilities include:

- Base system configuration
- Docker installation/configuration
- Application deployment
- Monitoring deployment
- Service configuration

Ansible configuration is maintained separately in:

`infrastructure/homelab-ansible`

This repository is included as a Git submodule.

## Application Layer

The application environment runs using Docker Compose.

Current services include:

```text
Traefik
   │
   ▼
homelab-app
   │
   ▼
PostgreSQL
```

The application is exposed through Traefik using:

```text
app.homelab
```

The application source code is maintained separately in:

`applications/homelab-app`

## Reverse Proxy

Traefik acts as the reverse proxy and entry point for containerized services.

Traffic flow:

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

This separates external service access from the internal container ports.

## Database

PostgreSQL is deployed as a container alongside the application.

Architecture:

```text
homelab-app
     │
     │ PostgreSQL connection
     ▼
PostgreSQL
```

The database is isolated from the public application entry point and communicates through the Docker network.

## Monitoring

The monitoring stack consists of:

- Prometheus
- Grafana
- Node Exporter

Architecture:

```text
application-01
      │
      ▼
Node Exporter
      │
      │ metrics
      ▼
Prometheus
      │
      │ PromQL
      ▼
Grafana
```

Node Exporter exposes host-level metrics such as:

- CPU usage
- Memory usage
- Disk usage
- Network traffic
- Host availability

Prometheus collects these metrics and Grafana visualizes them.

## Alerting

Grafana Alerting is used to detect infrastructure conditions.

The current implemented alert monitors high CPU utilization.

Flow:

```text
Node Exporter
      │
      ▼
Prometheus
      │
      ▼
Grafana Alert Rule
      │
      │ CPU > 80%
      ▼
Telegram
```

The alert lifecycle has been tested using `stress-ng`.

Test flow:

```text
CPU load generated
        │
        ▼
CPU utilization increases
        │
        ▼
Prometheus collects metric
        │
        ▼
Grafana alert enters Firing state
        │
        ▼
Telegram notification
        │
        ▼
CPU load stops
        │
        ▼
Alert returns to Normal
```

## Network

The current homelab uses the existing LAN network.

```text
LAN
│
├── Proxmox
│
├── 192.168.1.110
│     └── management-01
│
└── 192.168.1.111
      └── application-01
            ├── Traefik
            ├── Application
            ├── PostgreSQL
            ├── Prometheus
            └── Grafana
```

Internal application access currently uses:

```text
app.homelab
```

with local hostname resolution.

## Repository Architecture

The project intentionally separates implementation repositories from the main portfolio repository.

```text
proxmox-homelab
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

The main repository provides the overall project context and documentation, while implementation repositories remain independently versioned.

## Current Status

### Implemented

- Proxmox VE infrastructure
- Terraform VM provisioning
- Ubuntu virtual machines
- Ansible automation
- Docker
- Docker Compose
- Containerized application
- PostgreSQL
- Traefik reverse proxy
- Prometheus
- Grafana
- Node Exporter
- Grafana monitoring dashboard
- CPU alerting
- Telegram notifications
- Alert firing and recovery testing
- Git-based project organization

### Planned

- CI/CD with GitHub Actions
- Automated application deployment
- Improved secrets management
- Backup and restore procedures
- Disaster recovery documentation
- Internal DNS
- Additional monitoring and alerts
- Infrastructure hardening
- Automated testing
