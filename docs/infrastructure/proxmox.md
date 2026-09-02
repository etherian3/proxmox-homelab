# Proxmox Infrastructure

## Overview

Proxmox VE is the virtualization platform used as the foundation of this homelab.

It provides the compute and virtualization layer on which the Ubuntu virtual machines are deployed.

The infrastructure is intentionally designed so that higher-level services do not depend directly on the physical host configuration.

```text
Physical Hardware
       │
       ▼
  Proxmox VE
       │
       ├── management-01
       │
       └── application-01
```

---

## Why Proxmox?

Proxmox was selected as the virtualization platform because it provides a practical environment for learning infrastructure administration without requiring multiple physical servers.

It provides:

- Virtual machine management
- Linux container support
- Virtual networking
- Virtual storage
- VM lifecycle management
- Web-based administration
- API-based infrastructure management

The API is particularly useful for this project because Terraform can interact with Proxmox to provision infrastructure programmatically.

---

## Infrastructure Layer

The current architecture separates the infrastructure into several logical layers:

```text
┌───────────────────────────────────────┐
│           Application Layer           │
│   homelab-app / PostgreSQL / Traefik │
├───────────────────────────────────────┤
│          Monitoring Layer             │
│ Prometheus / Grafana / Node Exporter │
├───────────────────────────────────────┤
│       Configuration Layer             │
│              Ansible                  │
├───────────────────────────────────────┤
│       Provisioning Layer              │
│             Terraform                 │
├───────────────────────────────────────┤
│        Virtualization Layer           │
│            Proxmox VE                 │
├───────────────────────────────────────┤
│          Physical Hardware            │
└───────────────────────────────────────┘
```

This separation allows each tool to have a clearly defined responsibility.

---

## Virtual Machines

The current Proxmox environment contains two primary Ubuntu VMs.

| VM | VMID | IP Address | Purpose |
|---|---:|---|---|
| `management-01` | `100` | `192.168.1.110` | Infrastructure Automation |
| `application-01` | — | `192.168.1.111` | Application & Monitoring |

Detailed VM information is maintained in:

- [VM Inventory](vm-inventory.md)

---

## Management Node

`management-01` provides the management and automation layer.

It hosts the Ansible control environment used to configure and manage the homelab servers.

```text
Operator
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

The management node does not host the primary application workload.

This provides separation between infrastructure management and application workloads.

---

## Application Node

`application-01` is the primary workload VM.

It runs Docker and hosts the current application platform:

```text
application-01
│
├── Traefik
├── homelab-app
├── PostgreSQL
├── Prometheus
├── Grafana
└── Node Exporter
```

This VM therefore represents the main workload plane of the current homelab.

---

## Networking

The virtual machines are connected to the Proxmox virtual network and use the homelab LAN.

Current addressing:

```text
Network: 192.168.1.0/24
Gateway: 192.168.1.1

management-01   192.168.1.110
application-01  192.168.1.111
```

The VMs use the Proxmox bridge interface to communicate with the physical LAN.

The detailed network design is documented separately in:

- [Network Architecture](network.md)

---

## VM Provisioning

VM provisioning is automated using Terraform.

The intended lifecycle is:

```text
Terraform Configuration
        │
        ▼
Terraform Plan
        │
        ▼
Terraform Apply
        │
        ▼
Proxmox API
        │
        ▼
Virtual Machine
```

This removes the need to manually create every VM through the Proxmox web interface.

Terraform is responsible for infrastructure provisioning, while Ansible is responsible for post-provisioning configuration.

This creates a clear separation of responsibilities:

| Tool | Responsibility |
|---|---|
| Proxmox | Virtualization |
| Terraform | VM provisioning |
| Ansible | Server configuration |
| Docker | Application runtime |

Terraform implementation:

`infrastructure/terraform-vm-templates`

---

## Configuration Management

After a VM has been provisioned, Ansible is used to configure the operating system and services.

```text
Proxmox
   │
   ▼
Ubuntu VM
   │
   ▼
Ansible
   │
   ├── Common configuration
   ├── Docker
   ├── Application
   └── Monitoring
```

This allows the server configuration to be represented as code rather than relying entirely on manual configuration.

Ansible implementation:

`infrastructure/homelab-ansible`

---

## Infrastructure Lifecycle

The overall lifecycle is:

```text
1. Define Infrastructure
          │
          ▼
2. Terraform Provisioning
          │
          ▼
3. Ubuntu VM Available
          │
          ▼
4. Ansible Configuration
          │
          ▼
5. Docker Runtime
          │
          ▼
6. Application Deployment
          │
          ▼
7. Monitoring & Alerting
```

This lifecycle is the foundation of the project's DevOps workflow.

---

## Current State

The Proxmox infrastructure is operational.

Implemented:

- Proxmox VE installation
- Ubuntu VM deployment
- Static VM networking
- Dedicated management VM
- Dedicated application VM
- Terraform-based VM provisioning
- Ansible-based server configuration
- Docker workload environment
- Monitoring infrastructure

---

## Future Improvements

Potential future improvements to the virtualization layer include:

- Additional workload VMs
- Dedicated monitoring VM
- Infrastructure resource optimization
- VM backup automation
- Snapshot strategy
- Disaster recovery procedures
- Infrastructure hardening
- More advanced network segmentation

These improvements will only be introduced when they provide a meaningful operational or learning benefit.

---

## Related Documentation

- [Architecture](../architecture/architecture.md)
- [VM Inventory](vm-inventory.md)
- [Network Architecture](network.md)
- [Terraform](../automation/terraform.md)
- [Ansible](../automation/ansible.md)
