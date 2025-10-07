# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This repository contains Kubernetes deployment configurations for the SYNQ DWH service. It uses Kustomize for managing environment-specific configurations with a base/overlays pattern.

## Architecture

- **Base Configuration** (`base/`): Contains core Kubernetes resources (deployment, configMap, secrets) that are shared across all environments
- **Overlays** (`overlays/`): Environment-specific configurations that customize the base:
  - `example/`: Example configuration for testing/development
  - `synq-staging/`: Staging environment (dev- prefix, custom namespace `synq-dwh-dev`)
- **Kustomize Pattern**: Uses `configMapGenerator` and `secretGenerator` to create ConfigMaps/Secrets from `agent.yaml` and `agent.env` files
- **Automatic Updates**: The deployment has keel.sh annotations configured for automatic minor version updates via polling every 5 minutes

## Key Configuration Files

- `base/deployment.yaml`: Main deployment definition with volume mounts for agent config
- `base/agent.env`: Base environment variables (secrets like POSTGRES_PASSWORD, SYNQ_CLIENT_ID, SYNQ_CLIENT_SECRET)
- `overlays/*/agent.yaml`: Agent configuration file mounted at `/opt/synq-dwh/`
- `overlays/*/agent.env`: Environment-specific secrets that override/extend base secrets

## Common Commands

### Build and Deploy

```bash

# Deploy using Kustomize (recommended)
kubectl apply -k overlays/example
kubectl apply -k overlays/synq-staging

# Deploy using generated YAML
# Build kustomization and output to synq-dwh-example.yaml
./build.sh
kubectl apply -f synq-dwh-example.yaml
```

### Local Testing

```bash
# Test configuration locally with docker-compose (NOT for production)
docker-compose up
```

**Note**: `docker-compose.yml` is explicitly marked for development/testing only to validate configuration changes.

### Verification

```bash
# Check deployment status
kubectl get deployments -n synq

# Check pods
kubectl get pods -n synq

# View logs
kubectl logs -l deployment=synq-dwh -n synq
```

### Cleanup

```bash
# Remove deployment
kubectl delete -k overlays/example
# OR
kubectl delete -f synq-dwh-example.yaml
```

## Important Notes

- Each overlay uses `namePrefix` to distinguish resources (e.g., `example-`, `dev-`)
- The `example` overlay uses `disableNameSuffixHash: true` to keep ConfigMap/Secret names stable
- Container image is pulled from `europe-docker.pkg.dev/synq-cicd-public/synq-public/synq-dwh`
- Secrets are stored in `agent.env` files and should never be committed with real credentials in example overlays
