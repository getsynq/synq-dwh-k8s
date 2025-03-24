# SYNQ DWH Kubernetes Deployment Guide

This guide provides step-by-step instructions for deploying the SYNQ DWH service on Kubernetes. You can choose between two deployment methods:

1. **Recommended**: Using Kustomize (for better environment management and configuration)
2. Direct deployment using kubectl

## Quick Start

If you're already familiar with Kubernetes and want to deploy quickly:

```bash
# Using Kustomize (Recommended)
kubectl apply -k k8s/overlays/example

# OR using direct deployment
kubectl apply -f k8s/synq-dwh-example.yaml
```

## Prerequisites

Before you begin, ensure you have:

- ✅ Kubernetes cluster access configured
- ✅ `kubectl` CLI tool installed
- ✅ `kustomize` installed (if using Kustomize approach)
- ✅ Access to the container registry where the images are stored

## Project Structure

```
synq-dwh-k8s/
├── base/                  # Base Kubernetes configurations
│   ├── deployment.yaml    # Main deployment configuration
│   ├── kustomization.yaml # Base kustomization file
│   ├── agent.yaml        # Agent configuration
│   └── agent.env         # Environment variables, store secrets here
└── overlays/             # Environment-specific overlays
    └── example/          # Example environment configuration
```

## Detailed Deployment Instructions

### 1. Using Kustomize (Recommended)

Kustomize provides better environment management and configuration customization:

1. **Review Base Configuration**

   - Navigate to `k8s/base/`
   - Review and modify configurations as needed
   - Pay special attention to resource limits and environment variables

2. **Environment Setup**

   - Choose an existing environment overlay from `k8s/overlays/`
   - Or create a new one by copying the `example` directory

3. **Deploy**

   ```bash
   # Using kubectl with built-in kustomize
   kubectl apply -k k8s/overlays/example

   # OR using standalone kustomize (for older kubectl versions)
   kustomize build k8s/overlays/example | kubectl apply -f -
   ```

### 2. Direct Deployment Using kubectl

For simpler deployments without environment-specific configurations:

```bash
kubectl apply -f k8s/synq-dwh-example.yaml
```

⚠️ **Note**: When using direct deployment, you'll need to manually redeploy when configuration changes as checksums won't update automatically.

## Post-Deployment Verification

After deployment, verify everything is working correctly:

```bash
# Check deployment status
kubectl get deployments

# Check pods status
kubectl get pods

# View logs
kubectl logs -l app=synq-dwh
```

## Configuration Guide

### Environment Variables

Environment variables are managed in two places:

- Base configuration: `k8s/base/.env`
- Environment-specific variables: Located in respective overlay directories

### Resource Configuration

Resource limits and requests can be adjusted in:

- Base deployment: `k8s/base/deployment.yaml`
- Environment-specific: In respective overlay's patch files

## Troubleshooting Guide

If you encounter issues, follow these steps:

1. **Check Pod Status**

   ```bash
   kubectl describe pod <pod-name>
   ```

2. **View Logs**

   ```bash
   kubectl logs <pod-name>
   ```

3. **Verify Configurations**
   ```bash
   kubectl get configmap
   kubectl describe configmap
   ```

## Cleanup Instructions

To remove the deployment:

```bash
# Using Kustomize
kustomize build k8s/overlays/example | kubectl delete -f -

# OR direct deployment
kubectl delete -f k8s/synq-dwh-example.yaml
```

## Need Help?

If you encounter any issues or need assistance:

1. Check the troubleshooting section above
2. Review the logs using `kubectl logs`
3. Contact the SYNQ team for support
