# Changelog

All notable changes to this homelab infrastructure project are documented here.

## [1.0.0] - 2026-09-07

### Infrastructure

- Built the homelab platform on Proxmox.
- Provisioned `management-01` and `application-01` virtual machines.
- Defined VM infrastructure using Terraform.
- Documented network configuration and VM inventory.

### Configuration Management

- Implemented Ansible configuration management.
- Added reusable roles for common system configuration.
- Automated Docker installation and application server configuration.
- Added Ansible Vault for sensitive configuration.
- Implemented UFW firewall configuration.
- Hardened SSH access by disabling root login and password authentication.
- Configured unattended security updates.

### Application Platform

- Deployed the `homelab-app` application using Docker Compose.
- Added PostgreSQL as the application database.
- Added Docker health checks for application availability.
- Published application images to GitHub Container Registry.

### Reverse Proxy

- Deployed Traefik as the reverse proxy.
- Routed `app.homelab` to the application container.
- Removed the insecure Traefik dashboard exposure from the production configuration.

### Monitoring and Alerting

- Deployed Prometheus for metrics collection.
- Deployed Grafana for monitoring and visualization.
- Deployed Node Exporter for host metrics.
- Added CPU utilization alerting.
- Integrated Telegram notifications.
- Performed a real CPU alert test using `stress-ng`.

### Backup and Recovery

- Implemented PostgreSQL backups using `pg_dump`.
- Added gzip compression and backup verification.
- Added remote backup storage on `management-01`.
- Added backup retention policies.
- Automated backups using a systemd service and timer.
- Performed a real PostgreSQL restore test using a temporary PostgreSQL environment.
- Documented disaster recovery procedures and recovery limitations.

### CI/CD

- Implemented GitHub Actions CI/CD.
- Added automated application testing.
- Added Docker image builds and publishing to GHCR.
- Implemented a self-hosted GitHub Actions runner on `management-01`.
- Enabled deployment into the private homelab network without exposing the application server directly to the public internet.
- Implemented immutable Docker image deployments using Git commit SHA tags.
- Added deployment health verification.
- Added manual rollback using previously published immutable images.
- Implemented automatic rollback when deployment verification fails.
- Performed a real automatic rollback test using an intentionally invalid image tag.
- Verified that the previous healthy production image was automatically restored after the failed deployment.

### Security

- Implemented UFW firewall rules.
- Restricted inbound traffic to required services.
- Disabled SSH password authentication.
- Disabled SSH root login.
- Protected sensitive Ansible configuration using Ansible Vault.
- Kept application environment secrets outside version control.

## Future

The following capabilities are intentionally left as future improvements:

- High availability across multiple Proxmox hosts.
- Multiple production application instances.
- Dedicated staging environment.
- Automated database migration pipeline.
- Proxmox VM backup automation.
- Centralized log aggregation.
- HTTPS with automated certificate management.
- Infrastructure monitoring for the CI/CD runner.
- Deployment concurrency controls.
- GitHub environment protection and approval workflows.
