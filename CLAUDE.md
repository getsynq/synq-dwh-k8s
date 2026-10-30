# CLAUDE.md

This file gives instructions to Claude Code (claude.ai/code) for this repository.

## Repository overview

This repository deploys the Coalesce Quality data warehouse agent on Kubernetes. It uses Kustomize with a base and overlays. The repository is public. Do not put customer names or real credentials in it.

## Layout

- `base/`: The resources that all overlays share. These are the deployment, the ConfigMap generator and the Secret generator.
- `overlays/example/`: The example overlay. It uses the `example-` name prefix and the `synq` namespace.
- `overlays/synq-staging/`: The staging overlay. It uses the `dev-` name prefix and the `synq-dwh-dev` namespace.
- `synq-dwh-example.yaml`: The example overlay, built into one file by `./build.sh`.
- `keel.yaml`: The Keel deployment, built from `keel-values.yaml` and `keel-template.yaml` by `./keel.sh`.

## Configuration files

- `base/deployment.yaml`: The agent deployment. It mounts the agent configuration at `/opt/synq-dwh/`.
- `base/agent.yaml`: The default agent configuration.
- `base/agent.env`: It declares `QUALITY_CLIENT_ID` and `QUALITY_CLIENT_SECRET` with empty values. The overlay sets the real values.
- `overlays/*/agent.yaml`: The agent configuration of the overlay. It replaces `base/agent.yaml` as a whole. Kustomize does not merge the two files.
- `overlays/*/agent.env`: The secrets of the overlay. Kustomize merges them with `base/agent.env` key by key.

The `example` overlay sets `disableNameSuffixHash: true`. Thus a change to the ConfigMap or the Secret does not restart the pod.

## Commands

```bash
# Deploy with Kustomize
kubectl apply -k overlays/example
kubectl apply -k overlays/synq-staging

# Build synq-dwh-example.yaml, then deploy it
./build.sh
kubectl apply -f synq-dwh-example.yaml

# Test the configuration locally. Do not use this in production.
docker-compose up

# Verify the deployment
kubectl -n synq get deployments
kubectl -n synq get pods
kubectl -n synq logs -l deployment=synq-dwh

# Remove the deployment
kubectl delete -k overlays/example
```

## Image versions

- The image is `europe-docker.pkg.dev/synq-cicd-public/synq-public/synq-dwh`.
- The `upgrade-version.yaml` workflow opens a PR when the registry has a new stable version.
- `scripts/check-and-update-version.sh` changes `base/deployment.yaml` when you run it. It has no dry run and no `--help`.
- If you change the image version, run `./build.sh` to update `synq-dwh-example.yaml`.
- The release notes are the `synq-dwh-v*` releases in the private `getsynq/cloud` repository. Read them before you change the image version. Do not link them from this repository.
- The connection fields are defined in `yamlconfig/types.go` in `getsynq/dwhsupport`. The published schema at https://schemas.synq.io/synq-dwh/v1/config.schema.json can be older than the code.
