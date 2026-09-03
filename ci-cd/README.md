# CI/CD Pipeline

## Overview

The homelab application uses GitHub Actions to automate testing, container image builds, registry publishing, deployment, health verification, and rollback.

The deployment pipeline uses immutable Docker image tags based on the Git commit SHA.

This ensures that each production deployment can be traced to an exact source-code revision and rolled back without rebuilding the application.

---

## Architecture

```text
Developer
    │
    │ git push
    ▼
GitHub Repository
    │
    ▼
GitHub Actions
    │
    ├── Test
    │     └── Validate Python application
    │
    ├── Build
    │     └── Build Docker image
    │
    ├── Push
    │     └── GitHub Container Registry
    │           ├── sha-<commit>
    │           └── latest
    │
    ▼
Self-hosted GitHub Actions Runner
management-01
    │
    │ SSH
    ▼
application-01
    │
    ▼
Deployment Script
scripts/deploy.sh
    │
    ├── Pull immutable image
    ├── Update container
    ├── Wait for health check
    └── Verify deployment
    │
    ▼
Production
homelab-app
```

---

## Infrastructure

| Component | Role |
|---|---|
| GitHub | Source code and CI/CD orchestration |
| GitHub Actions | CI/CD pipeline |
| GHCR | Docker image registry |
| management-01 | Self-hosted Actions runner |
| application-01 | Production application server |
| Docker Compose | Application orchestration |
| Traefik | Reverse proxy |
| Prometheus | Metrics |
| Grafana | Monitoring |

---

## Why a Self-hosted Runner?

The production application is hosted inside the private homelab network.

The application server uses:

```text
192.168.1.111
```

A GitHub-hosted runner cannot directly reach this private LAN address.

Instead, a self-hosted GitHub Actions runner was deployed on:

```text
management-01
192.168.1.110
```

The runner can communicate with the application server over SSH:

```text
GitHub Actions
      │
      ▼
management-01
      │
      │ SSH
      ▼
application-01
```

This allows the CI/CD pipeline to deploy into the private homelab network without exposing the application server directly to the public internet.

---

## Pipeline Stages

### 1. Test

The pipeline installs the Python dependencies and validates that the application can be imported successfully.

```bash
python -c "from app.main import app; print('Application import successful')"
```

A failed test prevents the build and deployment stages from running.

---

### 2. Build

After successful tests, GitHub Actions builds the Docker image using the application's `Dockerfile`.

The image is published to GitHub Container Registry.

Images are tagged using the commit SHA:

```text
ghcr.io/etherian3/homelab-app:sha-<commit>
```

The default branch also receives the `latest` tag.

---

### 3. Deployment

The deployment job runs on the self-hosted runner.

The runner connects to `application-01` over SSH and executes:

```bash
/opt/homelab-app/scripts/deploy.sh sha-<commit>
```

The deployment script:

1. Receives the immutable image tag.
2. Pulls the corresponding image from GHCR.
3. Updates the application container.
4. Waits for the Docker health check.
5. Fails the deployment if the application becomes unhealthy.
6. Reports the deployed image.

---

## Immutable Image Deployment

The production environment does not depend on the mutable `latest` tag for deployment.

For example:

```text
ghcr.io/etherian3/homelab-app:sha-fe60eb3
```

The running container can be inspected with:

```bash
docker inspect --format='{{.Config.Image}}' homelab-app
```

This allows the exact production version to be identified at any time.

---

## Health Verification

The application container uses a Docker health check.

The deployment script waits for:

```text
healthy
```

before reporting a successful deployment.

This prevents the CI/CD pipeline from considering a deployment successful merely because the Docker container started.

---

## Rollback

Rollback is performed using a previously published immutable image.

Example:

```bash
./scripts/rollback.sh sha-92e9e15
```

The rollback script:

1. Detects the currently running image.
2. Displays the current version.
3. Displays the rollback target.
4. Requests confirmation.
5. Calls `deploy.sh` with the target version.
6. Verifies application health.

Example rollback:

```text
sha-fe60eb3
      │
      │ rollback
      ▼
sha-92e9e15
      │
      ▼
healthy
```

No Docker image rebuild is required.

---

## Rollback Test

A real rollback test was performed in the homelab.

Production was running:

```text
sha-fe60eb3
```

It was rolled back to:

```text
sha-92e9e15
```

The rollback completed successfully and the application returned to a healthy state.

Production was subsequently restored to:

```text
sha-fe60eb3
```

This verifies that previously published immutable images can be used for operational recovery.

---

## Deployment Verification

The deployment can be verified with:

```bash
docker inspect --format='{{.Config.Image}}' homelab-app
```

and:

```bash
docker ps --filter name=homelab-app
```

Expected state:

```text
ghcr.io/etherian3/homelab-app:sha-<commit>
```

and:

```text
Up ... (healthy)
```

---

## Security Considerations

The deployment uses SSH from the self-hosted runner to the application server.

Sensitive credentials are not stored in the Git repository.

The application environment file is kept locally on the server:

```text
.env
```

and is excluded through `.gitignore`.

The GitHub Actions workflow uses GitHub's built-in `GITHUB_TOKEN` for publishing images to GHCR.

The production server does not need to expose SSH or the application directly to the public internet for GitHub-hosted runners.

---

## Current Limitations

This is a homelab environment and intentionally has a relatively small deployment footprint.

Current limitations include:

- Single Proxmox host
- Single production application VM
- Single self-hosted runner
- No high availability
- No automated database migration pipeline
- No automated remote backup storage
- No multi-environment staging/production separation

These limitations are documented rather than hidden because the purpose of the project is to demonstrate realistic infrastructure engineering and operational decision-making.

---

## Future Improvements

Potential improvements include:

- Automated rollback on failed health checks
- Deployment history tracking
- Staging environment
- Database migration automation
- Automated PostgreSQL backups
- Remote backup storage
- Proxmox VM backup automation
- Infrastructure monitoring for the CI/CD runner
- Deployment notifications
- Concurrency controls to prevent overlapping deployments
- Manual `workflow_dispatch` deployments
- GitHub environment protection rules
- Automated release/version management
