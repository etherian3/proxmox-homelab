# Proxmox Homelab

A production-style homelab built to demonstrate practical **DevOps, Infrastructure, Linux Administration, Networking, Containerization, Monitoring, CI/CD, Backup, and Disaster Recovery** skills.

The environment is built on Proxmox and managed using Infrastructure as Code and configuration automation.

---

## Architecture

```text
                              GitHub
                                │
                                │ git push
                                ▼
                       ┌──────────────────┐
                       │ GitHub Actions   │
                       │                  │
                       │ Test             │
                       │ Docker Build     │
                       │ Push to GHCR     │
                       └────────┬─────────┘
                                │
                                ▼
                    Self-hosted Runner
                    management-01
                    192.168.1.110
                                │
                                │ SSH
                                ▼
                    ┌─────────────────────┐
                    │ application-01      │
                    │ 192.168.1.111       │
                    │                     │
                    │ ┌─────────────────┐ │
                    │ │ Traefik         │ │
                    │ └────────┬────────┘ │
                    │          │          │
                    │ ┌────────▼────────┐ │
                    │ │ homelab-app     │ │
                    │ │ Docker          │ │
                    │ └────────┬────────┘ │
                    │          │          │
                    │ ┌────────▼────────┐ │
                    │ │ PostgreSQL 16   │ │
                    │ └─────────────────┘ │
                    │                     │
                    │ Prometheus          │
                    │ Grafana             │
                    │ Node Exporter       │
                    └─────────────────────┘
                                │
                                ▼
                         Telegram Alerts
```

---

## Goals

This project is designed to demonstrate how a small production-like infrastructure can be designed, automated, monitored, deployed, and recovered.

The focus is not simply on installing technologies, but on implementing operational workflows around them.

### Key engineering goals

- Infrastructure as Code
- Automated server configuration
- Containerized applications
- Reverse proxy and service networking
- Observability and alerting
- Automated CI/CD
- Immutable container deployments
- Deployment health verification
- Application rollback
- Database backup and restore testing
- Disaster recovery planning
- Reproducible infrastructure

---

## Technology Stack

| Area | Technology |
|---|---|
| Hypervisor | Proxmox VE |
| Operating System | Ubuntu |
| Infrastructure as Code | Terraform |
| Configuration Management | Ansible |
| Container Runtime | Docker |
| Orchestration | Docker Compose |
| Reverse Proxy | Traefik |
| Application | Python / FastAPI |
| Database | PostgreSQL 16 |
| Metrics | Prometheus |
| Visualization | Grafana |
| Host Metrics | Node Exporter |
| CI/CD | GitHub Actions |
| Container Registry | GitHub Container Registry |
| Automation Runner | GitHub Actions Self-hosted Runner |
| Notifications | Telegram |
| Version Control | Git / GitHub |

---

# Infrastructure

## Proxmox

The homelab runs on a Proxmox VE host.

Virtual machines are provisioned as infrastructure rather than manually configured machines.

Current virtual machines:

| VM | IP | Role |
|---|---|---|
| management-01 | 192.168.1.110 | Ansible control node / CI runner |
| application-01 | 192.168.1.111 | Application / database / monitoring |

See:

- [`docs/infrastructure/proxmox.md`](docs/infrastructure/proxmox.md)
- [`docs/infrastructure/network.md`](docs/infrastructure/network.md)
- [`docs/infrastructure/vm-inventory.md`](docs/infrastructure/vm-inventory.md)

---

# Infrastructure as Code

## Terraform

Terraform is used to provision the homelab virtual machines.

The goal is to make VM provisioning reproducible instead of relying on manual creation through the Proxmox GUI.

```text
Terraform
   │
   ▼
Proxmox
   │
   ├── management-01
   └── application-01
```

Documentation:

[`docs/automation/terraform.md`](docs/automation/terraform.md)

---

# Configuration Management

## Ansible

Ansible configures the operating systems and services after the VMs are provisioned.

The Ansible control node runs on `management-01`.

The automation repository contains:

- inventory
- SSH configuration
- Docker installation
- application configuration
- monitoring configuration
- database configuration
- Traefik configuration

Documentation:

[`docs/automation/ansible.md`](docs/automation/ansible.md)

---

# Application Platform

The application runs using Docker Compose.

```text
Traefik
   │
   ▼
homelab-app
   │
   ▼
PostgreSQL
```

The application is exposed through:

```text
app.homelab
```

Traefik handles HTTP routing while Docker networks isolate application and proxy traffic.

Documentation:

- [`docs/services/application.md`](docs/services/application.md)
- [`docs/services/reverse-proxy.md`](docs/services/reverse-proxy.md)
- [`docs/services/database.md`](docs/services/database.md)

---

# Observability

The infrastructure includes a monitoring stack consisting of:

```text
Node Exporter
      │
      ▼
 Prometheus
      │
      ▼
  Grafana
```

Metrics include:

- CPU utilization
- Memory utilization
- Disk utilization
- Network traffic
- Host availability
- Application infrastructure health

Grafana dashboards are used for visualization.

Prometheus alerting is configured for high CPU utilization, with Telegram notifications.

Documentation:

- [`docs/services/monitoring.md`](docs/services/monitoring.md)
- [`docs/operations/alerting.md`](docs/operations/alerting.md)

---

# CI/CD

The application uses GitHub Actions for automated deployment.

```text
Git push
   │
   ▼
Test
   │
   ▼
Docker Build
   │
   ▼
Push to GHCR
   │
   ▼
Self-hosted Runner
   │
   ▼
application-01
   │
   ▼
Health Check
```

The production deployment uses immutable Docker image tags based on Git commit SHA.

Example:

```text
ghcr.io/etherian3/homelab-app:sha-fe60eb3
```

This makes it possible to identify exactly which source revision is running in production.

---

## Immutable Deployments

Instead of deploying only:

```text
latest
```

the deployment system uses:

```text
sha-<commit>
```

Example:

```text
sha-fe60eb3
```

This provides:

- deployment traceability
- reproducibility
- safer releases
- easier rollback
- no dependency on a mutable production tag

---

## Health Verification

Deployments are not considered successful merely because the Docker container starts.

The deployment script waits for the application Docker health check:

```text
healthy
```

Only after the health check succeeds is the deployment considered successful.

---

## Rollback

Previous immutable images can be deployed without rebuilding.

Example:

```bash
./scripts/rollback.sh sha-92e9e15
```

A real rollback test has been performed successfully.

The application was rolled back from:

```text
sha-fe60eb3
```

to:

```text
sha-92e9e15
```

and returned to a healthy state.

The application was subsequently restored to:

```text
sha-fe60eb3
```

Documentation:

[`ci-cd/README.md`](ci-cd/README.md)

---

# Backup & Restore

PostgreSQL backups are performed using `pg_dump`.

A real restore test has been performed against a temporary PostgreSQL instance.

The restore successfully recovered test data and verified the restored records.

The project treats backup verification as an operational requirement rather than assuming that a successful backup command automatically means recoverability.

Documentation:

[`docs/operations/backup-restore.md`](docs/operations/backup-restore.md)

---

# Disaster Recovery

The project documents recovery procedures and current infrastructure limitations.

Current architecture intentionally uses a single Proxmox host and a single application VM, so it does not provide high availability.

Recovery planning covers:

- infrastructure recreation
- VM provisioning
- configuration management
- database restoration
- application deployment
- recovery objectives
- current limitations
- future improvements

Documentation:

[`docs/operations/disaster-recovery.md`](docs/operations/disaster-recovery.md)

---

# Project Structure

```text
proxmox-homelab/
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
├── applications/
│   └── homelab-app/
│
├── monitoring/
│
├── ci-cd/
│   └── README.md
│
├── README.md
└── CHANGELOG.md
```

---

# Engineering Practices Demonstrated

This project demonstrates practical experience with:

### Infrastructure

- Proxmox virtualization
- Linux server administration
- VM networking
- Infrastructure as Code
- Terraform

### Automation

- Ansible
- SSH automation
- Idempotent configuration
- Secrets management with Ansible Vault

### Containers

- Docker
- Docker Compose
- Container networking
- Persistent volumes
- Health checks
- GHCR

### Networking

- Private LAN networking
- Reverse proxy
- HTTP routing
- Docker networks
- Service isolation

### Observability

- Prometheus
- Grafana
- Node Exporter
- Alerting
- Telegram notifications

### CI/CD

- GitHub Actions
- Self-hosted runners
- Automated testing
- Docker image builds
- Container registry publishing
- Immutable deployments
- Deployment verification
- Rollback

### Operations

- PostgreSQL backup
- Restore testing
- Disaster recovery planning
- Recovery procedures
- Operational documentation

---

# Lessons Learned

The project intentionally documents problems encountered during implementation rather than presenting only the final state.

Examples include:

- GitHub-hosted runners cannot directly access private homelab IP addresses.
- A self-hosted runner is useful when deployment targets exist inside a private network.
- `latest` is convenient but less suitable for traceable production deployments.
- Immutable image tags make rollback significantly simpler.
- Container startup does not necessarily mean an application is healthy.
- Backups should be tested through an actual restore.
- Disaster recovery planning must reflect the real infrastructure rather than an idealized architecture.

---

# Future Improvements

Planned improvements include:

- Automated PostgreSQL backup scheduling
- Remote backup storage
- Proxmox VM backup automation
- Staging environment
- Automated database migrations
- Automatic rollback on failed deployment
- Deployment history
- CI/CD concurrency controls
- Manual deployment workflow
- GitHub environment protection
- Additional infrastructure monitoring
- Higher availability architecture

---

# Documentation

Detailed documentation:

- [Architecture](docs/architecture/architecture.md)
- [Proxmox](docs/infrastructure/proxmox.md)
- [Network](docs/infrastructure/network.md)
- [VM Inventory](docs/infrastructure/vm-inventory.md)
- [Terraform](docs/automation/terraform.md)
- [Ansible](docs/automation/ansible.md)
- [Application](docs/services/application.md)
- [Reverse Proxy](docs/services/reverse-proxy.md)
- [Database](docs/services/database.md)
- [Monitoring](docs/services/monitoring.md)
- [Alerting](docs/operations/alerting.md)
- [Backup & Restore](docs/operations/backup-restore.md)
- [Disaster Recovery](docs/operations/disaster-recovery.md)
- [CI/CD](ci-cd/README.md)

---

# Project Status

The core infrastructure and application platform are operational.

Current production capabilities:

- Terraform VM provisioning
- Ansible configuration management
- Dockerized application
- PostgreSQL
- Traefik reverse proxy
- Prometheus monitoring
- Grafana dashboards
- Telegram alerting
- GitHub Actions CI/CD
- GHCR image publishing
- Immutable deployments
- Deployment health verification
- Tested rollback
- PostgreSQL backup and restore testing
- Disaster recovery documentation
