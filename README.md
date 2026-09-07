# Proxmox Homelab Infrastructure Platform

A production-style homelab infrastructure project built to demonstrate practical **Infrastructure, DevOps, SysAdmin, Automation, Monitoring, Security, Backup, and CI/CD** skills.

The platform runs on a self-hosted Proxmox environment and uses Infrastructure as Code, Configuration Management, containerized services, monitoring, automated backups, and GitHub Actions-based CI/CD.

---

## Project Overview

This project was built as a practical infrastructure platform rather than a collection of isolated technologies.

The goal is to demonstrate how infrastructure and applications can be:

- provisioned automatically
- configured consistently
- deployed reproducibly
- monitored
- secured
- backed up
- restored
- and recovered from deployment failures

The environment consists of a Proxmox host running dedicated virtual machines for management and application workloads.

---

## Architecture

```text
                         GitHub
                           │
                           │ Push
                           ▼
                  GitHub Actions
                           │
                    ┌──────┴──────┐
                    │             │
                   Test          Build
                    │             │
                    │        Docker Image
                    │             │
                    │             ▼
                    │            GHCR
                    │             │
                    └──────┬──────┘
                           │
                           │ Deploy
                           ▼
                 Self-hosted Runner
                 management-01
                    192.168.1.110
                           │
                           │ SSH
                           ▼
                application-01
                192.168.1.111
                           │
             ┌─────────────┼─────────────┐
             │             │             │
          Traefik      homelab-app   PostgreSQL
             │             │
             │             │
             ▼             │
        app.homelab       │
                           │
                           ▼
                    Prometheus
                           │
                           ▼
                        Grafana
                           │
                           ▼
                       Telegram
```

Infrastructure provisioning and configuration:

```text
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
    ├── Common configuration
    ├── Docker
    ├── Application
    ├── Monitoring
    ├── Security
    └── Maintenance
```

---

## Infrastructure

### Proxmox

The virtualization layer is provided by Proxmox VE.

Current virtual machines:

| VM | IP Address | Role |
|---|---|---|
| management-01 | 192.168.1.110 | Ansible + GitHub Actions self-hosted runner |
| application-01 | 192.168.1.111 | Application platform |

The infrastructure is designed so that management and application workloads are separated.

---

## Infrastructure as Code

### Terraform

Terraform is used to provision the virtual machines on Proxmox.

The Terraform configuration manages:

- VM creation
- CPU allocation
- memory allocation
- disk configuration
- network configuration
- VM metadata

This makes the VM infrastructure reproducible instead of relying entirely on manual Proxmox configuration.

---

## Configuration Management

### Ansible

Ansible is used to configure the operating systems and application platform.

Roles include:

- `common`
- `docker`
- `application`
- `monitoring`
- `security`
- `maintenance`

The configuration includes:

- package installation
- Docker installation
- application deployment configuration
- monitoring stack
- UFW firewall
- SSH hardening
- automatic security updates
- maintenance configuration

Sensitive configuration is protected using **Ansible Vault**.

---

## Application Platform

The application platform runs on `application-01`.

```text
Traefik
   │
   ▼
homelab-app
   │
   ▼
PostgreSQL
```

### Application

The application is containerized using Docker.

The application image is published to GitHub Container Registry:

```text
ghcr.io/etherian3/homelab-app
```

Production deployments use immutable Git SHA tags.

Example:

```text
ghcr.io/etherian3/homelab-app:sha-2d6ce1f
```

This provides traceability between a running production container and its source code commit.

### PostgreSQL

PostgreSQL provides persistent application storage.

Database data is stored using a Docker named volume.

---

## Reverse Proxy

Traefik provides the HTTP entrypoint for the application.

```text
Client
  │
  │ HTTP
  ▼
Traefik :80
  │
  │ Host: app.homelab
  ▼
homelab-app :8000
```

The application port is not directly published to the LAN.

Only the reverse proxy exposes the application externally.

---

## Monitoring

The infrastructure uses:

- Prometheus
- Node Exporter
- Grafana
- Telegram notifications

Monitoring covers:

- CPU utilization
- memory utilization
- disk usage
- network activity
- host availability
- application infrastructure health

A CPU alert was intentionally triggered using `stress-ng` and successfully delivered a Telegram notification.

The alert was subsequently resolved after the load stopped.

---

## Security

The application VM uses UFW with a default-deny inbound policy.

Allowed services include:

```text
22/tcp     SSH
80/tcp     Traefik
3000/tcp   Grafana
```

SSH hardening includes:

```text
PermitRootLogin no
PasswordAuthentication no
KbdInteractiveAuthentication no
PubkeyAuthentication yes
```

The application container itself does not expose port `8000` directly to the LAN.

Secrets such as database credentials are stored outside Git.

---

## Backup & Restore

PostgreSQL backups are performed using `pg_dump`.

The backup process:

1. Creates a PostgreSQL dump
2. Compresses the dump
3. Verifies the archive
4. Copies the backup to `management-01`
5. Verifies the remote archive
6. Applies retention policies

Automated backups are scheduled using a systemd timer.

A real PostgreSQL restore test was also performed using a temporary PostgreSQL environment to verify that the backup could actually be restored.

The project documents the recovery process and its current limitations.

---

## Disaster Recovery

The current environment is intentionally documented as a single-host homelab architecture.

There is currently:

- one Proxmox host
- one application VM
- no HA cluster
- no automatic VM failover

Terraform and Ansible provide infrastructure recovery capability, while PostgreSQL backups provide application data recovery.

The project documents:

- recovery scenarios
- backup strategy
- restore procedures
- RPO/RTO concepts
- current limitations
- future improvements

The objective is to distinguish between **implemented recovery capabilities** and future plans rather than claiming HA that does not exist.

---

# CI/CD

The application uses GitHub Actions for automated CI/CD.

Pipeline:

```text
Git Push
   │
   ▼
Test
   │
   ▼
Build Docker Image
   │
   ▼
Push to GHCR
   │
   ▼
Self-hosted Runner
   │
   ▼
Deploy immutable SHA
   │
   ▼
Health Check
   │
   ├── Success ───────────────► Production
   │
   └── Failure
          │
          ▼
   Automatic Rollback
          │
          ▼
   Previous SHA
          │
          ▼
      Health Check
          │
          ▼
      Production
```

## Self-hosted Runner

The application VM is located on a private LAN.

GitHub-hosted runners cannot directly access the private application VM.

Therefore a GitHub Actions self-hosted runner is installed on:

```text
management-01
192.168.1.110
```

The runner connects to `application-01` through SSH.

---

## Immutable Deployments

Production does not rely solely on the `latest` tag.

Instead, deployments use Git commit SHA tags:

```text
sha-xxxxxxxx
```

This allows a specific application version to be identified and redeployed.

---

## Automatic Rollback

Before deployment, the currently running production image is captured.

If the new deployment fails or the health check fails, the pipeline automatically deploys the previous production image.

The rollback does not rebuild the previous application version.

---

## Rollback Test

Automatic rollback was tested intentionally using an invalid image tag:

```text
sha-deadbee
```

The deployment failed while attempting to pull the non-existent image.

The pipeline then automatically restored the previous production image.

Production was subsequently verified:

```text
Application container: healthy
Application status: running
```

The application endpoint returned:

```json
{
  "application": "homelab-app",
  "status": "running"
}
```

This demonstrates an actual tested failure-recovery workflow rather than only a documented rollback mechanism.

---

## Repository Structure

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
├── applications/
│   └── homelab-app/
│
├── monitoring/
│   └── README.md
│
└── ci-cd/
    └── README.md
```

---

## Technology Stack

| Area | Technology |
|---|---|
| Virtualization | Proxmox VE |
| Infrastructure as Code | Terraform |
| Configuration Management | Ansible |
| OS | Ubuntu |
| Container Runtime | Docker |
| Container Orchestration | Docker Compose |
| Reverse Proxy | Traefik |
| Application | FastAPI / Uvicorn |
| Database | PostgreSQL |
| Container Registry | GitHub Container Registry |
| CI/CD | GitHub Actions |
| Monitoring | Prometheus |
| Metrics | Node Exporter |
| Visualization | Grafana |
| Alerting | Telegram |
| Firewall | UFW |
| Secrets | Ansible Vault |
| Backup | PostgreSQL pg_dump |
| Scheduling | systemd timer |

---

## What This Project Demonstrates

This project demonstrates practical experience with:

- Linux system administration
- Proxmox virtualization
- Infrastructure as Code
- Configuration management
- Docker
- reverse proxy configuration
- PostgreSQL administration
- monitoring and alerting
- firewall configuration
- SSH hardening
- secrets management
- backup and restore
- disaster recovery planning
- Git
- GitHub Actions
- container registries
- immutable deployments
- deployment health checks
- automatic rollback
- failure testing

The emphasis is on **implemented and tested infrastructure**, rather than simply listing technologies as skills.

---

## Future Improvements

Potential future improvements include:

- internal DNS instead of `/etc/hosts`
- HTTPS with automated certificate management
- centralized logging
- additional application health checks
- automated infrastructure backup
- Proxmox VM backup
- more granular monitoring alerts
- infrastructure recovery automation
- staging environment
- pull-request based deployment promotion

---

## Documentation

Detailed documentation is available under [`docs/`](docs/).

Important documents:

- Architecture
- Proxmox infrastructure
- Network configuration
- VM inventory
- Terraform automation
- Ansible automation
- Application platform
- Reverse proxy
- Database
- Monitoring
- Alerting
- Backup & Restore
- Disaster Recovery
- CI/CD

---

## Status

**Project status: Operational**

Core infrastructure, automation, monitoring, backup/restore, security hardening, CI/CD, and automatic rollback have been implemented and tested.
