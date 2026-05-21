## Development Workflow

### Overview

The development workflow defines how services move from **development → testing → production** within the Services Node environment.

This workflow is designed to:

- Reduce risk when deploying changes
- Ensure consistency between environments
- Provide rollback capability
- Maintain service uptime

---

## Environments

### 1. Development (Local)

#### Location

- Developer machines

#### Purpose

- Initial development and feature implementation
- Rapid iteration without impacting shared systems

#### Characteristics

- Local Docker environments (recommended)
- May use mock or local databases
- Not required to match production exactly

---

### 2. Testing / Staging (Rocky Linux VM)

#### Location

- Rocky Linux Test Environment VM

#### Purpose

- Validate deployments before production
- Integration testing (API + database + routing)
- Configuration validation

#### Characteristics

- Mirrors production as closely as possible
- Uses same:
    - Reverse proxy (Traefik)
    - Container definitions
    - Network structure

---

### 3. Production (Docker VM)

#### Location

- Docker VM (Ubuntu Server)

#### Purpose

- Live service environment

#### Characteristics

- Stable, controlled deployments only
- Connected to:
    - Pi-hole (DNS)
    - PostgreSQL VM
- Exposed via Traefik

---

## Deployment Flow

### Step 1 — Development

- Developer builds or updates a service (e.g. FastAPI app)
- Service is containerised using Docker
- Tested locally

---

### Step 2 — Push to Repository

- Code is pushed to version control (e.g. Git)
- Version is tagged (recommended)

---

### Step 3 — Deploy to Testing

- Updated container is deployed to **Rocky Linux Test VM**
- Validate:
    - Application behaviour
    - Database connectivity
    - Routing via Traefik
    - DNS resolution via Pi-hole

---

### Step 4 — Validation

- Perform:
    - Functional testing
    - Basic performance checks
    - Log review

If issues are found:

- Fix in development
- Repeat cycle

---

### Step 5 — Production Deployment

- Deploy container to **Docker VM**
- Update via:
    - Portainer
- Traefik automatically routes traffic to new service

---

### Step 6 — Post-Deployment Checks

- Verify:
    - Service availability
    - Logs (errors, warnings)
    - Database interactions

---

## Versioning Strategy

### Recommended Approach

- Use **semantic versioning**:
    - `v1.0.0`, `v1.1.0`, etc.

### Benefits

- Clear rollback points
- Traceable deployments
- Easier debugging

---

## Rollback Strategy

### Container-Based Rollback

- Redeploy previous container version

### VM-Level Rollback (if needed)

- Use hypervisor snapshots (testing environment primarily)

### Database Considerations

- Ensure regular backups before deployments
- Avoid destructive schema changes without migration planning

---

## Configuration Management

### Environment Variables

- Store configuration outside containers
- Use `.env` files or Docker secrets

### Differences by Environment

|Setting|Dev|Test|Prod|
|---|---|---|---|
|Debug Mode|On|Off|Off|
|Database|Local|PostgreSQL VM|PostgreSQL VM|
|Domain Names|Localhost|Internal DNS|Internal DNS|

---

## Networking in Workflow

- **Pi-hole**
    - Provides consistent DNS across environments
- **Traefik**
    - Ensures routing behaves identically in test and production