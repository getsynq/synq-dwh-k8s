# Coalesce Quality data warehouse agent on Kubernetes

This repository deploys the Coalesce Quality data warehouse agent on Kubernetes. The agent connects to your data warehouses. It reads schemas and metrics and runs monitors for Coalesce Quality.

You can deploy the agent in two ways:

1. With Kustomize, from an overlay in `overlays/`. We recommend this method.
2. With `kubectl apply` and the generated `synq-dwh-example.yaml` file.

## Quick start

```bash
# With Kustomize
kubectl apply -k overlays/example

# OR with the generated file
kubectl apply -f synq-dwh-example.yaml
```

The `example` overlay connects to a test PostgreSQL database and uses example credentials. Change both before you use it.

## Prerequisites

- Access to a Kubernetes cluster.
- `kubectl` v1.14 or later, which includes Kustomize.
- API credentials from Coalesce Quality. Create them under Settings → API in your workspace.

The agent image is public, so you do not need registry credentials.

## Repository layout

```
synq-dwh-k8s/
├── base/                  # Resources that all overlays share
│   ├── deployment.yaml    # The agent deployment
│   ├── kustomization.yaml # Generates the ConfigMap and the Secret
│   ├── agent.yaml         # Default agent configuration
│   └── agent.env          # Default environment variables
├── overlays/
│   └── example/           # Example overlay, copy it for your own environment
└── synq-dwh-example.yaml  # The example overlay, built into one file
```

## Deploy with Kustomize

1. Copy `overlays/example` to a new directory, for example `overlays/production`.
2. In the new `agent.env`, set `QUALITY_CLIENT_ID` and `QUALITY_CLIENT_SECRET`.
3. In the new `agent.env`, add the passwords and keys that your connections use.
4. In the new `agent.yaml`, define your connections. See [Agent configuration](#agent-configuration).
5. Deploy the overlay:

   ```bash
   kubectl apply -k overlays/production
   ```

## Deploy with the generated file

`synq-dwh-example.yaml` is the `example` overlay, built into one file. Use it if you do not want to use Kustomize.

1. Edit the ConfigMap and the Secret in `synq-dwh-example.yaml`.
2. Deploy the file:

   ```bash
   kubectl apply -f synq-dwh-example.yaml
   ```

To build the file again from the overlay, run `./build.sh`.

## Verify the deployment

```bash
kubectl -n synq get deployments
kubectl -n synq get pods
kubectl -n synq logs -l deployment=synq-dwh
```

## Agent configuration

The agent reads its configuration from `agent.yaml`. The schema is documented here:

- Schema documentation: https://schemas.synq.io/synq-dwh/v1/config.html
- JSON Schema: https://schemas.synq.io/synq-dwh/v1/config.schema.json

For autocompletion and validation in your editor, add this line at the top of `agent.yaml`:

```yaml
# yaml-language-server: $schema=https://schemas.synq.io/synq-dwh/v1/config.schema.json
```

An overlay's `agent.yaml` replaces `base/agent.yaml` as a whole. Kustomize does not merge the two files. The agent reads `QUALITY_CLIENT_ID` and `QUALITY_CLIENT_SECRET` from the environment, so an overlay `agent.yaml` does not need a `synq` section.

The example overlay sets `disableNameSuffixHash: true`, so a change to `agent.yaml` or `agent.env` does not restart the pod. After a change, restart the deployment:

```bash
kubectl -n synq rollout restart deployment/example-synq-dwh-deployment
```

Use your own name prefix instead of `example-`.

### Supported connections

The agent connects to PostgreSQL, MySQL, BigQuery, ClickHouse, Snowflake, Redshift, Databricks, Trino, SQL Server, Oracle, Amazon Athena, Microsoft Fabric and IBM Db2 (Linux, UNIX and Windows). The schema documentation lists the fields of each connection type.

### BigQuery without a key

A BigQuery connection can use `impersonate_service_accounts` instead of a service account key. The agent starts with the Application Default Credentials of the pod, for example a GKE workload identity. Then it impersonates the listed service accounts in order. Each service account must grant `roles/iam.serviceAccountTokenCreator` to the identity before it. If a connection has no credentials, the agent uses the Application Default Credentials directly. This needs v0.8.0 or later.

```yaml
connections:
  "bigquery":
    name: "BigQuery"
    bigquery:
      project_id: my-project
      region: EU
      impersonate_service_accounts:
        - reader@my-project.iam.gserviceaccount.com
```

### SQL Server with Kerberos

If an `mssql` connection sets `integrated_auth: true`, the agent signs in as the user that runs it. On Linux, the agent uses the Kerberos ticket from `kinit`. This needs v0.8.0 or later.

## Environment variables

The Secret gets its values from two files:

- `base/agent.env` declares `QUALITY_CLIENT_ID` and `QUALITY_CLIENT_SECRET` with empty values.
- The overlay's `agent.env` sets the real values. Kustomize merges it with the base file key by key.

The agent still reads the older names `SYNQ_CLIENT_ID` and `SYNQ_CLIENT_SECRET`, so you do not have to change an existing deployment. For a new deployment, use the `QUALITY_` names.

## Resources

`base/deployment.yaml` does not set resource requests or limits. To set them, add a patch to your overlay. `overlays/synq-staging/patch-version.yaml` is an example of a deployment patch.

## Automatic image updates

The deployment has annotations for [Keel](https://keel.sh/docs/). Keel checks the registry every 5 minutes and installs new minor and patch versions:

```yaml
annotations:
  keel.sh/policy: minor
  keel.sh/trigger: poll
  keel.sh/pollSchedule: "@every 5m"
```

To install Keel, run `kubectl apply -f keel.yaml`. The basic auth of Keel uses the user `admin` and the password `password`. Change them in `keel-values.yaml` and run `./keel.sh` to build `keel.yaml` again.

If you use GitOps, you can use [FluxCD image automation](https://fluxcd.io/flux/guides/image-update/) or [Argo CD Image Updater](https://argocd-image-updater.readthedocs.io/) instead. These tools write the new image version to your Git repository.

## Troubleshooting

1. Check the status of the pod:

   ```bash
   kubectl -n synq describe pod <pod-name>
   ```

2. Read the logs:

   ```bash
   kubectl -n synq logs <pod-name>
   ```

3. Check the configuration that the pod uses:

   ```bash
   kubectl -n synq describe configmap
   ```

If the problem continues, contact Coalesce Quality support: https://docs.synq.io/support/support

## Remove the deployment

```bash
# With Kustomize
kubectl delete -k overlays/example

# OR with the generated file
kubectl delete -f synq-dwh-example.yaml
```
