# Portfolio Evidence

This document records the implemented and tested capabilities of the
Proxmox Homelab Infrastructure Platform.

The purpose is to provide concrete evidence of practical infrastructure,
DevOps, and system administration work rather than simply listing
technologies.

---

## 1. Infrastructure

### Proxmox Virtualization

Implemented a self-hosted Proxmox environment with separate virtual
machines for management and application workloads.

| VM | IP | Purpose |
|---|---|---|
| management-01 | 192.168.1.110 | Ansible control node and GitHub Actions self-hosted runner |
| application-01 | 192.168.1.111 | Application platform |

### Evidence

- Proxmox virtual machines created and operational
- Separate management and application workloads
- Static network configuration
- VM inventory documented

---

## 2. Infrastructure as Code

### Terraform

Terraform is used to provision Proxmox virtual machines.

Implemented configuration includes:

- VM creation
- CPU allocation
- memory allocation
- disk configuration
- network configuration
- VM metadata

### Evidence

Terraform configuration is maintained in:

`infrastructure/terraform-vm-templates/`

The resulting VMs are actively used by the homelab platform.

---

## 3. Configuration Management

### Ansible

Ansible is used to configure the Ubuntu servers consistently.

Implemented roles:

- common
- docker
- application
- monitoring
- security
- maintenance

### Security Automation

The security role configures:

- UFW
- default-deny inbound policy
- SSH access
- SSH hardening

SSH configuration includes:

- root login disabled
- password authentication disabled
- keyboard-interactive authentication disabled
- public key authentication enabled

### Maintenance Automation

The maintenance role configures:

- unattended-upgrades
- automatic package list updates
- security updates
- unused dependency removal
- reboot requirement reporting

### Secrets

Sensitive Ansible configuration is protected using Ansible Vault.

---

## 4. Application Platform

### Docker

The application platform uses Docker Compose.

Services include:

- homelab-app
- PostgreSQL
- Traefik
- Prometheus
- Grafana
- Node Exporter

### Application

The application is packaged as a Docker image and published to GitHub
Container Registry.

Production images use immutable Git SHA tags.

Example:

`ghcr.io/etherian3/homelab-app:sha-2d6ce1f`

### Health Check

The application container has a Docker health check.

A deployment is only considered successful when the container reaches:

`healthy`

---

## 5. Reverse Proxy

### Traefik

Traefik provides the HTTP entrypoint for the application.

Traffic flow:

Client

↓

Traefik :80

↓

homelab-app :8000

The application container does not directly expose port 8000 to the LAN.

The application is accessed through:

`app.homelab`

---

## 6. Monitoring

The monitoring stack consists of:

- Prometheus
- Node Exporter
- Grafana

Prometheus collects host metrics from Node Exporter.

Grafana visualizes the collected metrics.

Monitoring includes:

- CPU
- memory
- disk
- network
- availability

---

## 7. Alerting

A CPU utilization alert was configured in Grafana.

The alert triggers when CPU usage exceeds the configured threshold.

### Test

A controlled CPU load was generated using `stress-ng`.

The alert successfully triggered and delivered a Telegram notification.

After the CPU load stopped, the alert returned to the resolved state.

This confirms that the alerting pipeline was tested end-to-end.

---

## 8. Backup

PostgreSQL backups are implemented using `pg_dump`.

The backup process:

1. Creates a database dump
2. Compresses the dump
3. Verifies the gzip archive
4. Copies the backup to management-01
5. Verifies the remote archive
6. Applies retention policies

The backup process is automated using a systemd timer.

---

## 9. Restore Testing

A real PostgreSQL restore test was performed.

A test dataset was created in PostgreSQL and backed up.

The backup was restored into a temporary PostgreSQL environment.

The restored data was verified successfully.

The temporary restore environment was then removed.

This validates that the backup process produces usable recovery data.

---

## 10. Disaster Recovery

The environment documents a realistic recovery model.

Current architecture:

- single Proxmox host
- single application VM
- no HA cluster
- no automatic VM failover

Terraform provides infrastructure provisioning capability.

Ansible provides server configuration capability.

PostgreSQL backups provide application data recovery.

The recovery documentation explicitly distinguishes implemented
capabilities from future improvements.

---

## 11. CI/CD

GitHub Actions provides the application CI/CD pipeline.

Pipeline:

Git Push

↓

Test

↓

Build Docker Image

↓

Push Image to GHCR

↓

Self-hosted Runner

↓

Deploy immutable SHA image

↓

Health Check

↓

Production

---

## 12. Self-hosted GitHub Actions Runner

GitHub-hosted runners cannot directly access the private LAN.

A self-hosted GitHub Actions runner is therefore installed on:

`management-01`

The runner connects to `application-01` through SSH.

This allows GitHub Actions to deploy workloads inside the private homelab network.

---

## 13. Immutable Deployment

Production deployments use Git SHA image tags.

Example:

`sha-2d6ce1f`

This provides:

- deployment traceability
- reproducibility
- deterministic rollback
- easier incident investigation

The running production container can be mapped directly to a source
code commit.

---

## 14. Automatic Rollback

The CI/CD pipeline captures the currently running production image
before deployment.

If deployment or health verification fails, the pipeline automatically
deploys the previous production image.

Rollback therefore does not require rebuilding the previous version.

---

## 15. Automatic Rollback Test

Automatic rollback was intentionally tested.

The pipeline was instructed to deploy an invalid image tag:

`sha-deadbee`

The image does not exist in GitHub Container Registry.

Expected failure:

Deployment fails while attempting to pull the new image.

Recovery:

The pipeline automatically deploys the previously running immutable
SHA image.

### Final Verification

After rollback:

- the application container was running
- Docker health status was `healthy`
- the application endpoint returned a successful response

Application response:

```json
{
  "application": "homelab-app",
  "status": "running"
}
```
This confirms that automatic rollback was not only implemented but
actually tested against a controlled deployment failure.
16. Security
Implemented security controls include:
- UFW firewall
- default-deny inbound traffic
- SSH key-based authentication
- root SSH login disabled
- password SSH authentication disabled
- application port not directly exposed
- secrets excluded from Git
- Ansible Vault for encrypted configuration
17. Operational Verification
The platform has been tested through multiple operational scenarios:
- application deployment
- application health verification
- monitoring
- CPU alert triggering
- PostgreSQL backup
- PostgreSQL restore
- deployment rollback
- production recovery after failed deployment
The goal of these tests is to demonstrate operational behavior,
not only successful initial installation.
18. Skills Demonstrated
Infrastructure
- Proxmox
- Linux
- networking
- virtualization
Automation
- Terraform
- Ansible
- Bash
- Git
Containers
- Docker
- Docker Compose
- GitHub Container Registry
Application Infrastructure
- FastAPI
- Uvicorn
- PostgreSQL
- Traefik
Observability
- Prometheus
- Node Exporter
- Grafana
- Telegram alerting
Security
- UFW
- SSH hardening
- Ansible Vault
Operations
- backup
- restore
- retention
- disaster recovery
- failure testing
CI/CD
- GitHub Actions
- self-hosted runner
- immutable deployments
- health checks
- automatic rollback
