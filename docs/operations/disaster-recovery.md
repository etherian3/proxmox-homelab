# Disaster Recovery

## Overview

Disaster Recovery (DR) adalah proses untuk memulihkan infrastructure, applications, dan data setelah terjadi kegagalan atau kehilangan resource.

Pada homelab ini, Disaster Recovery dirancang sebagai bagian dari operational reliability, bukan hanya sebagai backup database.

Current architecture masih menggunakan single Proxmox host dan single application VM sehingga belum memiliki high availability atau automatic failover.

Tujuan DR pada project ini adalah memastikan bahwa infrastructure dapat dipulihkan secara terstruktur dan bahwa backup yang dibuat benar-benar dapat digunakan untuk recovery.

---

## Current Architecture

```text
Proxmox
│
├── management-01
│   └── Ansible Control Node
│
└── application-01
    ├── Traefik
    ├── homelab-app
    ├── PostgreSQL
    ├── Prometheus
    ├── Grafana
    └── Node Exporter
```

Infrastructure provisioning menggunakan Terraform, sedangkan server configuration dan service deployment menggunakan Ansible.

Database menggunakan PostgreSQL dengan persistent Docker volume.

---

## Recovery Objectives

Two important DR concepts used in this project are:

### RPO — Recovery Point Objective

RPO menentukan seberapa banyak data yang masih dapat hilang setelah terjadi incident.

Contoh:

```text
RPO = 24 hours
```

berarti kehilangan data hingga 24 jam masih dapat diterima.

Current PostgreSQL backup process masih manual sehingga belum memiliki RPO otomatis yang terjamin.

### RTO — Recovery Time Objective

RTO menentukan target waktu untuk mengembalikan service agar kembali tersedia.

Contoh:

```text
RTO = 2 hours
```

berarti service ditargetkan kembali operational dalam waktu maksimal dua jam.

Current project belum menetapkan RTO production-grade secara formal.

---

## Current Recovery Capabilities

| Capability | Status |
|---|---|
| Infrastructure as Code | Implemented |
| VM provisioning with Terraform | Implemented |
| Server configuration with Ansible | Implemented |
| Application deployment with Docker Compose | Implemented |
| PostgreSQL logical backup | Implemented |
| PostgreSQL restore test | Passed |
| Backup data verification | Passed |
| Automated backup | Planned |
| Remote backup storage | Planned |
| Proxmox VM backup | Planned |
| Automated recovery | Planned |
| Full infrastructure recovery test | Planned |
| High availability | Not implemented |

---

## PostgreSQL Recovery

PostgreSQL currently uses logical backups generated with `pg_dump`.

The backup workflow is:

```text
PostgreSQL
    │
    ▼
pg_dump
    │
    ▼
SQL Backup
    │
    ▼
Backup Storage
```

A restore test was performed using an isolated PostgreSQL 16 container.

The workflow was:

```text
Production PostgreSQL
        │
        ▼
     pg_dump
        │
        ▼
SQL Backup
        │
        ▼
Temporary PostgreSQL
        │
        ▼
Restore
        │
        ▼
Data Verification
```

The restore successfully recreated the test table and restored all test records.

The production PostgreSQL container was not replaced or stopped during the restore test.

The temporary restore environment was removed after verification.

---

## Infrastructure Recovery

Infrastructure recovery is supported by Infrastructure as Code.

Terraform manages VM provisioning while Ansible manages server configuration.

The intended recovery flow is:

```text
Proxmox Available
       │
       ▼
Terraform
       │
       ▼
VM Provisioning
       │
       ▼
Ansible
       │
       ▼
Docker / Services
       │
       ▼
Application Recovery
```

This reduces dependency on manually configuring every component after a disaster.

However, a complete end-to-end infrastructure recovery test has not yet been performed.

---

## Failure Scenarios

The DR design considers several failure scenarios.

### Scenario 1 — Application Container Failure

Example:

```text
homelab-app container stops
```

Recovery:

```text
Docker restart policy
        │
        ▼
Container restarted
        │
        ▼
Application becomes available
```

This is primarily handled through Docker's container restart policy.

---

### Scenario 2 — PostgreSQL Container Failure

The PostgreSQL container uses a persistent Docker volume.

Recovery consists of:

1. Restore or recreate the PostgreSQL container.
2. Reattach the persistent data volume if the data is intact.
3. Restore from logical backup if required.
4. Verify database connectivity.
5. Verify application functionality.

---

### Scenario 3 — Application VM Failure

If `application-01` becomes unavailable, the intended recovery process is:

```text
Provision replacement VM
        │
        ▼
Terraform
        │
        ▼
Configure operating system
        │
        ▼
Ansible
        │
        ▼
Deploy services
        │
        ▼
Restore required data
        │
        ▼
Validate application
```

This scenario has not yet been tested end-to-end.

---

### Scenario 4 — Proxmox Host Failure

A complete Proxmox host failure is a higher-impact scenario.

Current architecture has a single Proxmox host, therefore there is currently no automatic failover.

Recovery would require:

1. Recover or replace the Proxmox host.
2. Restore VM infrastructure.
3. Reapply Terraform configuration where appropriate.
4. Reconfigure servers using Ansible.
5. Restore application data from available backups.
6. Validate application and monitoring services.

This remains a planned DR exercise.

---

## Recovery Dependencies

Recovery depends on several layers:

```text
Physical Hardware
       │
       ▼
Proxmox
       │
       ▼
Virtual Machines
       │
       ▼
Operating System
       │
       ▼
Docker
       │
       ▼
Application Services
       │
       ▼
Persistent Data
```

The project attempts to make each configuration layer reproducible.

Terraform handles infrastructure.

Ansible handles server configuration.

Docker Compose handles application services.

PostgreSQL backups protect database data.

---

## Current DR Gaps

The current environment still has several limitations:

- Single Proxmox host
- Single application VM
- No VM-level backup strategy
- No remote backup destination
- No automated PostgreSQL backup schedule
- No backup retention policy
- No automated backup verification
- No automated restore process
- No full infrastructure recovery test
- No high availability
- No secondary application host

These limitations are intentionally documented rather than hidden.

---

## DR Roadmap

The next improvements are:

### Phase 1 — Automated Database Backup

Implement scheduled PostgreSQL backups.

```text
Cron / systemd timer
        │
        ▼
pg_dump
        │
        ▼
Timestamped Backup
```

### Phase 2 — Backup Retention

Implement a retention policy so old backups are automatically removed.

Example target:

```text
Daily backups
7-day retention
```

### Phase 3 — Remote Backup

Store backups outside the application VM.

Potential architecture:

```text
application-01
      │
      ▼
PostgreSQL Backup
      │
      ▼
Remote Backup Storage
```

This protects against application VM storage failure.

### Phase 4 — Proxmox VM Backup

Introduce VM-level backups to protect the complete application environment.

### Phase 5 — Full Recovery Exercise

Perform an end-to-end recovery test:

```text
Failure
  ↓
Infrastructure Recovery
  ↓
VM Recovery
  ↓
Ansible Configuration
  ↓
Application Deployment
  ↓
Database Restore
  ↓
Application Validation
```

The result of the exercise should be documented with:

- Recovery duration
- Failed steps
- Manual intervention required
- Data loss
- Improvements identified

---

## Operational Principle

Backups are not considered reliable simply because backup files exist.

A backup is considered useful only when it can be successfully restored and verified.

For this reason, this project includes an explicit restore test rather than treating `pg_dump` completion as proof of recoverability.

---

## Target Architecture

The long-term DR architecture is:

```text
                    ┌──────────────────┐
                    │     Proxmox      │
                    └────────┬─────────┘
                             │
                     ┌───────▼────────┐
                     │ application-01 │
                     └───────┬────────┘
                             │
                    ┌────────▼────────┐
                    │   PostgreSQL    │
                    └────────┬────────┘
                             │
                         pg_dump
                             │
                    ┌────────▼────────┐
                    │ Remote Backup   │
                    │    Storage      │
                    └─────────────────┘

Terraform ──────────────► Infrastructure
Ansible ────────────────► Configuration
Docker ─────────────────► Services
Backup ────────────────► Data Recovery
```

The architecture will evolve toward automated backup, remote storage, infrastructure recovery, and repeatable disaster recovery testing.
