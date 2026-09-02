# Network Architecture

## Overview

The homelab currently operates on a single LAN network using the `192.168.1.0/24` address space.

The Proxmox host provides virtual networking for the homelab virtual machines through a Linux bridge.

The current network is intentionally simple while the infrastructure is being developed.

```text
                    Home LAN
                192.168.1.0/24
                       │
                       │
                ┌──────┴──────┐
                │   Router    │
                │ 192.168.1.1 │
                └──────┬──────┘
                       │
                       │
                 Proxmox Host
                       │
                    vmbr0
                       │
              ┌────────┴────────┐
              │                 │
              ▼                 ▼
       management-01      application-01
       192.168.1.110      192.168.1.111
```

---

## Network Addressing

Current network configuration:

| Component | Address | Purpose |
|---|---|---|
| LAN | `192.168.1.0/24` | Homelab network |
| Gateway | `192.168.1.1` | Default gateway |
| management-01 | `192.168.1.110` | Automation |
| application-01 | `192.168.1.111` | Application & monitoring |

The `/24` subnet provides addresses from `192.168.1.1` through `192.168.1.254`.

---

## Proxmox Virtual Bridge

The virtual machines are connected to the Proxmox Linux bridge:

```text
vmbr0
```

The bridge provides Layer 2 connectivity between the virtual machines and the physical LAN.

Conceptually:

```text
Physical Network Interface
          │
          ▼
        vmbr0
       /     \
      /       \
     ▼         ▼
 management   application
     VM           VM
```

This allows the VMs to communicate with other devices on the LAN while retaining their own IP addresses.

---

## IP Configuration

The VMs use static IP addressing.

### management-01

```text
IP Address : 192.168.1.110/24
Gateway    : 192.168.1.1
```

### application-01

```text
IP Address : 192.168.1.111/24
Gateway    : 192.168.1.1
```

Static addressing is useful for infrastructure services because their endpoints remain predictable.

For example:

```text
192.168.1.110 → management node
192.168.1.111 → application node
```

---

## Network Traffic Flow

The management workflow currently follows:

```text
Operator
   │
   │ SSH
   ▼
management-01
   │
   │ Ansible / SSH
   ▼
application-01
```

Application traffic follows:

```text
Client
   │
   ▼
app.homelab
   │
   ▼
application-01
   │
   ▼
Traefik
   │
   ▼
homelab-app
```

Monitoring traffic remains internal to the application node:

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

## Hostname Resolution

The application currently uses:

```text
app.homelab
```

for local access.

At the current stage of the project, hostname resolution for this internal domain is handled locally on the client through the hosts file.

Conceptually:

```text
app.homelab
     │
     ▼
192.168.1.111
```

This approach is sufficient for the current small environment but is not intended to be the final architecture.

---

## Why Not Internal DNS Yet?

The current network contains only a small number of infrastructure nodes.

Introducing a dedicated DNS service at this stage would add another infrastructure component without providing significant operational benefit.

The current priority is to establish:

- Infrastructure as Code
- Configuration management
- Containerization
- Monitoring
- Alerting
- CI/CD

Internal DNS is therefore maintained as a future improvement.

---

## Security Considerations

The current network is a flat LAN.

There is currently no dedicated VLAN or separate management network.

This means:

```text
Management
Application
Monitoring
Client Devices
```

remain within the same primary LAN environment.

This is acceptable for the current homelab development stage, but it is not equivalent to a production-grade segmented network.

Future security improvements may include:

- Management VLAN
- Application VLAN
- Monitoring VLAN
- Firewall rules between network segments
- Restricted management access
- Dedicated internal DNS
- Service-level network policies

---

## Docker Networking

Network segmentation also exists at the container layer on `application-01`.

The application stack uses separate Docker networks for different communication requirements.

Conceptually:

```text
                    application-01
                           │
                    Docker Engine
                           │
             ┌─────────────┴─────────────┐
             │                           │
             ▼                           ▼
        proxy network             app network
             │                           │
          Traefik                  homelab-app
                                      │
                                      ▼
                                  PostgreSQL
```

Traefik requires access to the proxy network so it can route incoming requests to application containers.

The application and PostgreSQL communicate through their internal application network.

This provides basic container-level isolation even though the underlying VM network is currently flat.

---

## Network Design Principles

The network design follows several principles:

### Predictable Addressing

Infrastructure nodes use fixed IP addresses so automation and service discovery remain predictable.

### Separation of Responsibilities

The network design separates:

- Management traffic
- Application traffic
- Monitoring traffic
- Container-to-container communication

where practical within the current architecture.

### Avoid Unnecessary Complexity

The homelab is intentionally kept simple until a new infrastructure component solves a real operational problem.

### Designed for Expansion

The current `192.168.1.0/24` network can later be extended with VLANs and additional network segments.

---

## Current Limitations

The current network has several known limitations:

- Single LAN segment
- No VLAN segmentation
- No dedicated management network
- No dedicated internal DNS
- Client hostname resolution currently relies on local hosts configuration
- No documented firewall policy between infrastructure components

These are known limitations rather than hidden assumptions.

---

## Future Network Architecture

A possible future design is:

```text
                         Router / Firewall
                                │
                    ┌───────────┼───────────┐
                    │           │           │
                    ▼           ▼           ▼
               Management   Application   Monitoring
                 VLAN          VLAN          VLAN
                    │           │           │
                    ▼           ▼           ▼
               Management    Workloads    Observability
                   VM            VMs           VM
```

The migration to network segmentation will be considered when the number of workloads and infrastructure components justifies the additional complexity.

---

## Related Documentation

- [Architecture](../architecture/architecture.md)
- [Proxmox Infrastructure](proxmox.md)
- [VM Inventory](vm-inventory.md)
- [Terraform](../automation/terraform.md)
- [Ansible](../automation/ansible.md)
