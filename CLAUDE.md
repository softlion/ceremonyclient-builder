# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains a Helm chart and automated build system for deploying Quilibrium blockchain nodes in Kubernetes. It does NOT contain the Quilibrium node source code itself - instead, it downloads official signed binaries from https://releases.quilibrium.com and packages them into Docker images published to `vapolia/quilibrium` on Docker Hub.

**Key Concept**: This is a build/deployment wrapper around the upstream Quilibrium ceremonyclient project (https://github.com/QuilibriumNetwork/ceremonyclient). The actual node runs in containers, and this chart manages the Kubernetes deployment configuration.

## Architecture

### Build Pipeline (.github/workflows/build-and-deploy.yml)

The GitHub Actions workflow has three jobs that run sequentially:

1. **check-version-exists**: Checks if a new Quilibrium release exists
   - Downloads release manifest from https://releases.quilibrium.com/release
   - Extracts version (e.g., "2.0.3.4" becomes "v2.0.3-p4")
   - Checks if git tag or Docker tag already exists
   - Sets `build_required` output to control downstream jobs

2. **build**: Builds and publishes Docker image (only if new version detected)
   - Clones upstream ceremonyclient repository with version verification:
     - First attempts to checkout tag `vX.X.X-pX` (e.g., v2.0.4-p5 for version 2.0.4.5)
     - If tag doesn't exist, searches for merged PR with exact title `vX.X.X.X` (e.g., v2.0.4.5)
     - If neither exists, fails with error message showing missing version
   - Downloads official signed binaries for node and qclient (linux-amd64)
   - Modifies the upstream Dockerfile.release:
     - Changes base image from Alpine to Debian Bookworm Slim
     - Skips Go compilation, uses pre-downloaded binaries instead
   - Uses `task build` from upstream Taskfile to build the image
   - Tags and pushes to Docker Hub as `vapolia/quilibrium:latest`, `vapolia/quilibrium:{VERSION}`, and `vapolia/quilibrium:{TRANSFORMED_VERSION}`

3. **update**: Automatically upgrades the Helm release in production
   - Only runs after successful build
   - Deploys to namespace `vapolia` with existing PVC `quilibrium`

### Helm Chart (helm/quilibrium/)

The chart deploys a single-replica Quilibrium node with:

**Deployment** (templates/deployment.yaml):
- Container runs node binary from Docker image
- Environment variables configure listening addresses via `DEFAULT_LISTEN_GRPC_MULTIADDR`, `DEFAULT_LISTEN_REST_MULTIADDR`, `DEFAULT_STATS_MULTIADDR`
- Mounts PVC at `/root/.config` for persistent node data (keys, store, config.yml)
- Supports nodeSelector, affinity, and tolerations for advanced scheduling

**Services**:
- **ClusterIP** (templates/service_clusterip.yaml): Internal service for gRPC (8337) and REST (8338) APIs
- **NodePort** (templates/service_nodeport.yaml): Exposes P2P ports (8336, 8340) and dynamically opens worker ports (50000-5000N, 60000-6000N based on `ports.workers` value)

**Storage** (templates/pvc.yaml):
- Creates PVC if `persistence.existingClaim` not provided
- PVC stores node identity (`key.yml`), config (`config.yml`), and blockchain data (`store/` folder)

**Port Configuration** (values.yaml):
- P2P: 8336 (main), 8340 (master)
- gRPC: 8337 (internal API)
- REST: 8338 (internal API)
- Worker ports: 50000-5000N and 60000-6000N (N = number of workers)

## Common Commands

### Helm Deployment

Install or upgrade with existing PVC:
```bash
helm upgrade quilibrium helm/quilibrium --install \
  --set persistence.existingClaim=yourPVCName
```

Install with auto-created PVC:
```bash
helm upgrade quilibrium helm/quilibrium --install \
  --set persistence.size=100Gi
```

### Node Operations

Check node status and balance (wait 5 minutes after start):
```bash
kubectl exec quilibrium-xxxxxxxxxx-xxxxx -- node -node-info
```

View logs:
```bash
kubectl logs -f quilibrium-xxxxxxxxxx-xxxxx
```

Clean store folder (frees disk space):
```bash
kubectl scale deployment quilibrium --replicas=0
# Manually delete files in the PVC's store/ directory
kubectl scale deployment quilibrium --replicas=1
```

Recover from failed pods:
```bash
kubectl delete pods --field-selector=status.phase!=Running -A
```

### GitHub Actions

Manually trigger build check (does not auto-run on schedule):
```bash
# Use GitHub UI: Actions -> "Check and Build Docker Image" -> Run workflow
```

## Important Configuration Notes

**values.yaml**:
- `send_stats`: Telemetry endpoint (default: "/dns/stats.quilibrium.com/tcp/443")
- `ports.workers`: Number of worker processes (affects port allocation)
- `persistence.existingClaim`: REQUIRED in production - must reference pre-created PVC

**Node Configuration** (in PVC root):
- `config.yml`: Set `maxFrames: 1001` (not `-1`) to prevent 2TB disk usage
- `key.yml`: Node identity - MUST be backed up immediately after creation
- `store/`: Can be deleted to reclaim space (requires node restart)

**Host Requirements** (Debian):
Network buffer sizes must be increased:
```bash
sudo nano /etc/sysctl.d/99-quilibrium.conf
net.core.rmem_max=7500000
net.core.wmem_max=7500000

sudo sysctl --system
```

## Development Workflow

When modifying the Helm chart:
1. Make changes to templates or values.yaml
2. Test locally: `helm template helm/quilibrium` to verify template rendering
3. Test deployment: `helm upgrade quilibrium helm/quilibrium --install --dry-run --debug`
4. Deploy: `helm upgrade quilibrium helm/quilibrium --install`

When modifying the Docker build:
1. Changes to .github/workflows/build-and-deploy.yml affect the automated pipeline
2. The workflow modifies the upstream Dockerfile.release - any sed commands must account for changes in upstream versions
3. Test builds locally by following the workflow steps manually
4. Note: The build tags the git repo BEFORE building to prevent re-runs on failure

## External Dependencies

- Upstream source: https://github.com/QuilibriumNetwork/ceremonyclient
- Release binaries: https://releases.quilibrium.com/
- Docker images: https://hub.docker.com/r/vapolia/quilibrium/tags
- Documentation: https://docs.quilibrium.com/docs/run-node/advanced-configuration/
