# Ansible Configuration Management

## Overview

Ansible is used as the configuration management and server automation layer of the homelab.

After virtual machines are provisioned by Terraform, Ansible configures the operating systems and deploys the required infrastructure services.

The Ansible control node runs on:

```text
management-01
192.168.1.110
```

The primary managed workload node is:

```text
application-01
192.168.1.111
```

The implementation is maintained in:

`infrastructure/homelab-ansible`

---

## Why Ansible?

Terraform is responsible for provisioning infrastructure, but it is not the ideal tool for managing the configuration of an operating system and its services.

Ansible fills that gap.

The workflow is:

```text
Terraform
    │
    ▼
Ubuntu VM
    │
    ▼
Ansible
    │
    ├── Operating System
    ├── Docker
    ├── Application
    └── Monitoring
```

This creates a clear separation between infrastructure provisioning and configuration management.

---

## Ansible Control Node

The Ansible control environment runs on `management-01`.

The project uses a dedicated Python virtual environment:

```text
.venv/
```

The virtual environment isolates the project dependencies from the system Python environment.

Current Ansible version:

```text
Ansible Core 2.17.14
```

The system-level Ansible installation is not used for this project.

---

## Inventory

The Ansible inventory defines the hosts managed by Ansible.

Current inventory:

```text
all
├── management
│   └── management-01
│
└── application
    └── application-01
```

Current addresses:

```text
management-01   192.168.1.110
application-01  192.168.1.111
```

The inventory allows playbooks and roles to target infrastructure based on responsibility rather than hard-coding individual hosts throughout the automation.

---

## Repository Structure

The current Ansible repository follows a role-based structure:

```text
homelab-ansible/
│
├── ansible.cfg
├── site.yml
│
├── inventory/
│   └── hosts.yml
│
├── group_vars/
│   └── application/
│       └── vault.yml
│
├── collections/
│   └── requirements.yml
│
└── roles/
    ├── common/
    ├── docker/
    ├── application/
    └── monitoring/
```

Each role has a focused responsibility.

---

## Roles

### Common

The `common` role contains baseline server configuration shared by managed hosts.

Typical responsibilities include:

- Base system configuration
- Package management
- Common operating system settings
- System preparation

The goal is to keep common configuration reusable across multiple servers.

---

### Docker

The `docker` role prepares the server to run containerized workloads.

Its responsibility is to install and configure the Docker environment required by the application and monitoring stack.

Conceptually:

```text
Ubuntu
  │
  ▼
Ansible
  │
  ▼
Docker
```

---

### Application

The `application` role deploys the application infrastructure.

Its responsibilities include:

- Preparing the application directory
- Deploying application configuration
- Managing Docker Compose configuration
- Providing required application files
- Starting the application stack

The application itself is maintained separately in:

`homelab-app`

Ansible acts as the deployment/configuration mechanism rather than replacing the application repository.

---

### Monitoring

The `monitoring` role deploys the observability stack.

Current components include:

```text
Prometheus
Grafana
Node Exporter
```

The resulting architecture is:

```text
Node Exporter
      │
      ▼
Prometheus
      │
      ▼
Grafana
```

---

## Playbook

The main playbook is:

```text
site.yml
```

The playbook acts as the entry point for applying the desired server configuration.

Conceptually:

```text
site.yml
   │
   ├── common
   │
   ├── docker
   │
   ├── application
   │
   └── monitoring
```

This allows the entire server configuration to be applied through a single automation entry point.

---

## Configuration

The project uses:

```text
ansible.cfg
```

Important configuration includes:

- Inventory location
- Python interpreter selection
- Vault password file
- Host key checking
- Retry file behavior

The configuration ensures that the project uses the intended inventory and Python environment consistently.

---

## Secrets Management

Sensitive configuration is managed using **Ansible Vault**.

The application group variables contain encrypted data:

```text
group_vars/application/vault.yml
```

The file begins with the Ansible Vault format:

```text
$ANSIBLE_VAULT;1.1;AES256
```

Sensitive values are therefore not stored as plaintext in the Git repository.

The local Vault password is stored outside the repository.

```text
~/.ansible-vault-password
```

The password file is protected with restrictive filesystem permissions and is not committed to Git.

---

## Collections

The project declares required Ansible collections separately:

```text
collections/requirements.yml
```

The current external collection requirement includes:

```text
community.docker
```

This allows Docker-related Ansible functionality to be managed explicitly as a project dependency.

---

## Idempotency

An important Ansible principle used by this project is **idempotency**.

Running the automation multiple times should converge the server toward the desired state without unnecessarily changing resources that are already correctly configured.

Conceptually:

```text
First Run
    │
    ▼
Configuration Applied
    │
    ▼
Server Reaches Desired State
    │
    ▼
Second Run
    │
    ▼
No Unnecessary Changes
```

This makes automation safer to repeat and easier to use during infrastructure maintenance.

---

## Configuration Workflow

The current workflow is:

```text
Terraform
    │
    ▼
Provision VM
    │
    ▼
Verify SSH
    │
    ▼
Ansible Inventory
    │
    ▼
ansible all -m ping
    │
    ▼
site.yml
    │
    ▼
Roles
    │
    ├── common
    ├── docker
    ├── application
    └── monitoring
```

This provides a clear progression from infrastructure creation to server configuration.

---

## Validation

Before applying configuration, connectivity can be verified with:

```bash
ansible all -m ping
```

A successful result confirms that the Ansible control node can communicate with the managed hosts.

Configuration changes can then be applied through the main playbook.

The automation should be reviewed and tested before being used for destructive infrastructure changes.

---

## Separation of Repositories

The Ansible repository is maintained independently from the main portfolio repository.

The relationship is:

```text
proxmox-homelab
       │
       ▼
infrastructure/
       │
       ▼
homelab-ansible
```

The main repository documents the architecture and operational workflow, while the Ansible repository contains the actual automation implementation.

---

## Terraform vs Ansible

The two tools complement each other:

| Terraform | Ansible |
|---|---|
| Provision infrastructure | Configure infrastructure |
| Create VM | Configure OS |
| Define CPU/RAM/Disk | Install packages |
| Define network | Configure Docker |
| Manage infrastructure lifecycle | Deploy services |
| Declarative infrastructure | Configuration automation |

The combined workflow is:

```text
Terraform
   │
   │ "What infrastructure should exist?"
   ▼
Proxmox VM
   │
   │ "How should this server be configured?"
   ▼
Ansible
   │
   ▼
Configured Server
```

---

## Current Status

Implemented:

- [x] Ansible control node
- [x] Python virtual environment
- [x] Inventory
- [x] SSH-based host management
- [x] Common role
- [x] Docker role
- [x] Application role
- [x] Monitoring role
- [x] Ansible Vault
- [x] External collection management
- [x] Main deployment playbook
- [x] Automated application configuration
- [x] Automated monitoring configuration

---

## Future Improvements

Planned improvements include:

- [ ] CI validation with `ansible-lint`
- [ ] Automated syntax checking
- [ ] Molecule testing for roles
- [ ] More reusable role variables
- [ ] Environment-specific inventories
- [ ] Improved secret management
- [ ] Automated deployment through GitHub Actions
- [ ] Automated rollback procedures

---

## Portfolio Value

The Ansible implementation demonstrates practical experience with:

- Configuration management
- Server automation
- Infrastructure orchestration
- Role-based automation
- Inventory management
- Secrets management
- Idempotent configuration
- Docker automation
- Monitoring deployment
- Infrastructure reproducibility

This demonstrates the transition from manually configured servers to infrastructure managed as code.

---

## Related Documentation

- [Architecture](../architecture/architecture.md)
- [Proxmox Infrastructure](../infrastructure/proxmox.md)
- [VM Inventory](../infrastructure/vm-inventory.md)
- [Network Architecture](../infrastructure/network.md)
- [Terraform](terraform.md)
- [Application](../services/application.md)
- [Monitoring](../services/monitoring.md)
