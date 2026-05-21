# Services Node Plan

---
## *Update 2026 04 01*
Server Current,
Hpe Proliant DL360 Gen10
2x Xeon Gold 6132
128gb DDr4 ecc Ram
2x 800W PSU (1x Redundancy)
Storage
2x 256GB Sata SSD (Raid 1)
2x HDD (Raid 1, Plan to expand or grow later on)
end of update
## Requirements

### Functional Requirements

- Host internal services (APIs, dashboards, management tools)
- Provide internal DNS resolution and network-wide ad blocking
- Support containerised application deployment
- Provide a centralised database service
- Enable OS provisioning and management (PXE boot, imaging)
- Support a dedicated testing/staging environment

### Non-Functional Requirements

- Reliability within a single-node constraint
- Clear service isolation between infrastructure components
- Ease of management and deployment
- Backup and recovery capability
- Scalable design for future expansion

### Constraints

- Single physical services node (no clustering)
- Limited hardware resources (shared across VMs)
- Internal-only availability (no external exposure assumed)

---

## Architecture Overview

The Services Node is designed as a **fully virtualised environment**, with a hypervisor hosting multiple dedicated virtual machines.

### Core Components

- Hypervisor (Proxmox / ESXi / similar)
- Multiple VMs providing isolated services:
    - DNS (Pi-hole)
    - Container platform (Docker host)
    - Database (PostgreSQL)
    - Testing environment (Rocky Linux)
    - OS management (FOG)

### Design Principles

- **Service isolation via VMs**
- **Containerisation for applications**    
- **Centralised ingress via reverse proxy**
- **Internal-only service exposure**

---

## Virtualisation vs Bare Metal

### Selected Approach: Fully Virtualised Infrastructure

The system will run on a hypervisor, with each major service deployed in its own virtual machine.

### VM Layout

|VM Name|Role|
|---|---|
|Pi-hole VM|DNS + ad blocking|
|Docker VM|Hosts containerised services|
|PostgreSQL VM|Database server|
|Rocky Linux VM|Testing / staging environment|
|FOG VM|OS provisioning and imaging|

---

### Rationale

#### Advantages

- Strong isolation between services
- Simplified backup strategy (VM snapshots)
- Easier recovery from failure
- Cleaner resource allocation
- Reduced risk of cross-service interference

#### Trade-offs

- Slight performance overhead vs bare metal
- Increased operational complexity
- More resource planning required

---

### Design Decision

This approach is preferred because:

- Infrastructure services (DNS, DB) are decoupled from application workloads
- Failures are contained to individual VMs
- The environment is easier to reason about and document

---

## Pi-hole

### Role

- Primary internal DNS resolver
- Network-wide ad and tracker blocking
- Local DNS for internal services

### Configuration

- Static IP assignment
- Configured as the primary DNS server for the network
- Local DNS entries for services (e.g. internal domains)

### Dependencies

- Required by all services for name resolution

### Risks

- Single point of failure for DNS

### Mitigation Options

- Secondary Pi-hole instance (future improvement)
- Fallback upstream DNS configuration

---

## Core Services

### Docker VM (Ubuntu Server)

This VM hosts all application-layer services using Docker.

#### Services

- Portainer
    - Container management interface
- Homepage
    - Internal service dashboard
- Traefik (Reverse Proxy)
    - Handles routing between services
    - Central entry point for all HTTP/HTTPS traffic
- FastAPI Application
    - Backend service layer
    - Connects to PostgreSQL

---

### Reverse Proxy (Traefik)

#### Responsibilities

- Route traffic to appropriate services
- Provide a single entry point
- Handle TLS termination (if implemented)

#### Importance

This is a **critical component**, if it fails, services become inaccessible.

---

### Database (PostgreSQL VM)

#### Role

- Central data store for applications

#### Design Considerations

- Not exposed externally
- Accessible only from internal services (e.g. Docker VM)
- Regular backups required

---

### OS Management (FOG VM)

#### Role

- PXE boot services
- OS imaging and deployment
- Device provisioning

#### Scope

- Supports broader network (workstations, render nodes)    
- Operates independently from application services

---

## Testing

### Rocky Linux Test Environment

#### Purpose

- Validate configurations before production deployment
- Test new services and updates
- Provide a safe environment for experimentation

### Design Notes

- Separate VM ensures isolation from production workloads
- Should mirror production configuration where possible

### Recommendations

- Use snapshots before major changes
- Maintain parity with Docker VM where applicable
- Consider network segmentation if needed

---

## Networking Overview

### Logical Layers

- **Infrastructure Layer**
    - Pi-hole
    - PostgreSQL
- **Application Layer**
    - Docker VM services
- **Management Layer**
    - FOG server
- **Testing Layer**
    - Rocky Linux VM

### Key Points

- Internal-only communication between services
- DNS resolution handled centrally by Pi-hole
- Reverse proxy acts as the entry point for applications    

---

## Risks & Considerations

### Single Point of Failure

- Entire system depends on one physical node

### DNS Dependency

- Pi-hole outage impacts all services

### Resource Contention

- Multiple VMs sharing hardware

### Mitigations

- VM resource limits (CPU/RAM allocation)
- Regular backups (VM snapshots + DB dumps)
- Monitoring (future improvement)    

---

## Future Improvements

- Add monitoring stack (Prometheus + Grafana)
- Implement automated backups
- Introduce secondary DNS (failover Pi-hole)
- Consider high availability (long-term)
- Add VLAN/network segmentation

![[Pasted image 20260318170143.png]]
## Related
[[Hosting]]
[[Test Env]]
[[OS Management]]
[[Development Workflow]]