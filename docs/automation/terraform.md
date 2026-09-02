# Terraform Infrastructure as Code

## Overview

Terraform is used as the Infrastructure as Code (IaC) layer of the homelab.

Its responsibility is to define and provision the virtual machine infrastructure running on Proxmox VE.

The Terraform project is maintained as an independent repository and included in this portfolio repository as a Git submodule.

Implementation:

`infrastructure/terraform-vm-templates`

---

## Why Terraform?

Without Infrastructure as Code, virtual machines can be created manually through the Proxmox web interface.

While manual provisioning works for a small environment, it makes infrastructure harder to reproduce and maintain.

Terraform changes the workflow from:

```text
Manual Configuration
       │
       ▼
Proxmox Web Interface
       │
       ▼
VM
```

to:

```text
Terraform Configuration
       │
       ▼
terraform plan
       │
       ▼
Review Changes
       │
       ▼
terraform apply
       │
       ▼
Proxmox
       │
       ▼
VM
```

The infrastructure configuration therefore becomes version-controlled and reproducible.

---

## Terraform Responsibility

Terraform is responsible for the infrastructure layer.

Current responsibilities include:

- Defining virtual machines
- Defining VM resources
- Configuring VM networking
- Assigning VM IP configuration
- Managing VM lifecycle
- Communicating with Proxmox through its API/provider

Terraform is **not** responsible for configuring application services inside the operating system.

That responsibility belongs to Ansible.

---

## Infrastructure Separation

The project follows this responsibility model:

| Layer | Tool | Responsibility |
|---|---|---|
| Virtualization | Proxmox | Run virtual machines |
| Provisioning | Terraform | Create/manage VMs |
| Configuration | Ansible | Configure operating systems |
| Runtime | Docker | Run containers |
| Application | homelab-app | Application logic |
| Monitoring | Prometheus/Grafana | Observability |

This separation prevents Terraform from becoming responsible for tasks better handled by configuration management.

---

## Provisioning Workflow

The current workflow is:

```text
Developer
    │
    ▼
Terraform Repository
    │
    ▼
Terraform Configuration
    │
    ▼
terraform plan
    │
    ▼
terraform apply
    │
    ▼
Proxmox API
    │
    ▼
Virtual Machine
    │
    ▼
Ansible
```

Terraform therefore handles the initial infrastructure lifecycle before Ansible takes over server configuration.

---

## Current Virtual Machines

Terraform currently provisions the homelab management infrastructure.

### management-01

```text
Name       : management-01
VMID       : 100
IP Address : 192.168.1.110/24
Gateway    : 192.168.1.1
vCPU       : 2
RAM        : 2 GB
Disk       : 24 GB
OS         : Ubuntu
```

The VM is used as the Ansible control node.

### application-01

```text
Name       : application-01
IP Address : 192.168.1.111/24
OS         : Ubuntu
Role       : Application / Monitoring
```

The VM is used as the workload node for the application and supporting services.

Detailed resource information is maintained in:

- [VM Inventory](../infrastructure/vm-inventory.md)

---

## Network Configuration

Terraform defines the VM network configuration required for the infrastructure.

The current network uses:

```text
Network      : 192.168.1.0/24
Gateway      : 192.168.1.1
Management   : 192.168.1.110
Application  : 192.168.1.111
```

The VMs are connected through the Proxmox virtual bridge:

```text
vmbr0
```

---

## Proxmox Integration

Terraform communicates with Proxmox through the provider/API layer.

Conceptually:

```text
Terraform
    │
    │ API
    ▼
Proxmox
    │
    ▼
VM Lifecycle
```

This allows infrastructure changes to be expressed through Terraform configuration rather than requiring every change to be performed manually in the Proxmox interface.

---

## Terraform State

Terraform maintains state to track the relationship between the configuration and the infrastructure resources.

Conceptually:

```text
Terraform Configuration
        │
        ├──────────────┐
        │              │
        ▼              ▼
 Terraform State    Proxmox
        │              │
        └──────┬───────┘
               │
               ▼
        Desired vs Actual
```

The state allows Terraform to determine which resources already exist and what changes are required.

Terraform state contains infrastructure metadata and must therefore be handled carefully.

Sensitive credentials should never be committed to the repository.

---

## Infrastructure Lifecycle

Terraform supports the following lifecycle:

```text
Create
  │
  ▼
Provision
  │
  ▼
Modify
  │
  ▼
Plan Changes
  │
  ▼
Apply Changes
  │
  ▼
Destroy
```

The desired infrastructure is defined declaratively.

Instead of describing every individual command needed to create a VM, the configuration describes the desired final state.

---

## Terraform and Ansible

Terraform and Ansible intentionally have different responsibilities.

```text
              Infrastructure Lifecycle
                       │
          ┌────────────┴────────────┐
          │                         │
          ▼                         ▼
      Terraform                  Ansible
          │                         │
          ▼                         ▼
   Provision VM              Configure VM
          │                         │
          ▼                         ▼
      Ubuntu VM              Docker / Services
```

### Terraform

Answers:

> "What infrastructure should exist?"

Examples:

- VM
- CPU
- Memory
- Disk
- Network
- IP configuration

### Ansible

Answers:

> "How should the server be configured?"

Examples:

- Packages
- Docker
- Application services
- Monitoring
- Configuration files

This separation makes the infrastructure easier to understand and maintain.

---

## Git Workflow

Terraform configuration is version-controlled using Git.

Changes follow a basic workflow:

```text
Modify Terraform
       │
       ▼
git diff
       │
       ▼
terraform plan
       │
       ▼
Review
       │
       ▼
git commit
       │
       ▼
git push
```

The Terraform repository is independently versioned and referenced from the main portfolio repository as a Git submodule.

---

## Reproducibility

One of the primary goals of using Terraform is reproducibility.

A new environment should be able to follow the same general process:

```text
Clone Repository
       │
       ▼
Configure Provider
       │
       ▼
Initialize Terraform
       │
       ▼
terraform plan
       │
       ▼
terraform apply
       │
       ▼
Infrastructure Created
```

This reduces dependence on undocumented manual steps.

---

## Current Limitations

The current Terraform implementation is intentionally focused on the existing Proxmox environment.

Known future improvements include:

- Remote Terraform state
- Better state locking
- More reusable VM modules
- Variable-driven environment configuration
- Additional VM templates
- Automated validation
- CI-based `terraform plan`
- Automated infrastructure deployment

These improvements will be introduced incrementally.

---

## Portfolio Value

The Terraform implementation demonstrates practical experience with:

- Infrastructure as Code
- Declarative infrastructure
- Proxmox automation
- VM lifecycle management
- Network configuration
- Version-controlled infrastructure
- Separation of provisioning and configuration management
- Reproducible infrastructure workflows

The implementation repository contains the actual Terraform configuration used by the homelab.

---

## Related Documentation

- [Architecture](../architecture/architecture.md)
- [Proxmox Infrastructure](../infrastructure/proxmox.md)
- [Network Architecture](../infrastructure/network.md)
- [VM Inventory](../infrastructure/vm-inventory.md)
- [Ansible](ansible.md)
