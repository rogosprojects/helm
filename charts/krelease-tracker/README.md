# Release Tracker Helm Chart

![Helm Version](https://img.shields.io/badge/helm-v3-blue)

This Helm chart deploys Release Tracker, a comprehensive Kubernetes application that tracks container image deployments across namespaces, providing a modern web interface to monitor and visualize release history with real-time updates.

## Features

- **🔍 Kubernetes Integration**: Monitors Pods, Deployments, StatefulSets, DaemonSets, and ReplicaSets across specified namespaces
- **💾 Data Storage**: SQLite database with automatic deduplication and retention (10 most recent releases per component)
- **🌐 Web Interface**:
  - Modern dashboard with hierarchical table view and full-text search
  - Timeline page with release history visualization and change indicators
  - Badge viewer for generating release status badges
  - Base path support for ingress controllers and path-based routing
- **🔗 REST API**: Comprehensive endpoints for triggering collection, retrieving releases, and accessing history
- **🏷️ Release Badges**: SVG badges for README files with API key authentication
- **🔐 Security**: Optional API key authentication for all endpoints
- **🏗️ Master/Slave Architecture**: Support for multi-client, multi-environment deployments
- **📊 Image SHA Tracking**: Tracks both image tags and SHA256 digests for accurate change detection
- **⚡ Real-time Updates**: Live collection and refresh capabilities
- **🛡️ RBAC Ready**: Complete Kubernetes manifests with proper RBAC configuration

## Quick Start

### Installation

Add the Helm repository and install the chart:

```bash
helm repo add rogosprojects https://rogosprojects.github.io/helm
helm repo update

# Simple installation with default values
helm install krelease-tracker rogosprojects/krelease-tracker

# Installation with custom values file
helm install krelease-tracker rogosprojects/krelease-tracker \
  --namespace observability --create-namespace \
  --values values.yaml
```

### Using with the chart source code:

```bash
# Clone the repository
git clone https://github.com/rogosprojects/helm.git
cd krelease-tracker/helm-chart

# Install directly from the chart directory
helm install krelease-tracker . --namespace observability --create-namespace
```

## Configuration

The following table lists the main configurable parameters of the chart and their default values.

### Global Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of replicas | `1` |
| `image.repository` | Container image repository | `krelease-tracker` |
| `image.tag` | Container image tag | `latest` |
| `image.pullPolicy` | Container image pull policy | `IfNotPresent` |
| `image.pullSecrets` | List of image pull secrets | `[]` |
| `nameOverride` | Override the name of the chart | `""` |
| `fullnameOverride` | Override the full name of the chart | `""` |
| `serviceAccount.create` | Create a service account | `true` |
| `serviceAccount.name` | The name of the service account | `""` |
| `serviceAccount.annotations` | Annotations for the service account | `{}` |
| `rbac.create` | Create RBAC resources | `true` |
| `podAnnotations` | Annotations for the pods | `{}` |
| `podSecurityContext` | Security context for the pods | See `values.yaml` |
| `securityContext` | Security context for the containers | See `values.yaml` |
| `service.type` | Service type | `ClusterIP` |
| `service.port` | Service port | `80` |
| `ingress.enabled` | Enable ingress | `false` |
| `ingress.className` | Ingress class name | `""` |
| `ingress.annotations` | Ingress annotations | `{}` |
| `ingress.hosts` | Ingress hosts configuration | See `values.yaml` |
| `ingress.tls` | Ingress TLS configuration | `[]` |
| `resources` | CPU/Memory resource requests/limits | See `values.yaml` |
| `nodeSelector` | Node selector | `{}` |
| `tolerations` | Tolerations | `[]` |
| `affinity` | Affinity | `{}` |

### Application Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `env.PORT` | Application port | `"8080"` |
| `env.DATABASE_PATH` | Path to SQLite database file | `"/data/releases.db"` |
| `env.NAMESPACES` | Comma-separated namespaces to monitor | `"default"` |
| `env.COLLECTION_INTERVAL` | Minutes between automatic collections | `"60"` |
| `env.IN_CLUSTER` | Use in-cluster Kubernetes configuration | `"true"` |
| `env.API_KEYS` | Comma-separated API keys for authentication | `""` |
| `env.BASE_PATH` | Base path for serving the application | `""` |
| `env.MODE` | Application mode (master or slave) | `"slave"` |
| `env.MASTER_URL` | Master URL for sync (slave mode only) | `""` |
| `env.MASTER_API_KEY` | Master API key for sync (slave mode only) | `""` |
| `env.SYNC_INTERVAL` | Minutes between syncs (slave mode only) | `"5"` |
| `env.CLIENT_NAME` | Client name for multi-client deployments | `"unknown"` |
| `env.ENV_NAME` | Environment name used in badges | `"unknown"` |
| `env.PROXY_URL` | HTTP/HTTPS proxy URL for sync requests | `""` |
| `env.TLS_INSECURE` | Skip TLS certificate verification | `"false"` |
| `extraEnv` | Additional environment variables | `[]` |

### Persistence Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `persistence.enabled` | Enable persistent storage | `false` |
| `persistence.size` | Size of the persistent volume | `1Gi` |
| `persistence.storageClassName` | Storage class name | `""` |
| `persistence.accessModes` | Access modes for the persistent volume | `["ReadWriteOnce"]` |

## Example: Basic Deployment with Persistence

To deploy Release Tracker with persistent storage and API authentication:

```yaml
# values.yaml
persistence:
  enabled: true
  size: 2Gi
  storageClassName: "fast-ssd"

env:
  NAMESPACES: "production,staging,development"
  COLLECTION_INTERVAL: "30"
  API_KEYS: "your-secure-api-key-here"
  ENV_NAME: "production"
  CLIENT_NAME: "my-company"

ingress:
  enabled: true
  className: "nginx"
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
  hosts:
    - host: releases.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: releases-tls
      hosts:
        - releases.example.com

resources:
  requests:
    memory: 256Mi
    cpu: 200m
  limits:
    memory: 1Gi
    cpu: 1000m
```

## Example: Master/Slave Architecture

### Master Instance Configuration

```yaml
# master-values.yaml
env:
  MODE: "master"
  API_KEYS: "master-api-key,slave-sync-key"
  ENV_NAME: "master"
  CLIENT_NAME: "central"

persistence:
  enabled: true
  size: 5Gi

ingress:
  enabled: true
  hosts:
    - host: releases-master.example.com
```

### Slave Instance Configuration

```yaml
# slave-values.yaml
env:
  MODE: "slave"
  MASTER_URL: "https://releases-master.example.com"
  MASTER_API_KEY: "slave-sync-key"
  CLIENT_NAME: "production-cluster"
  ENV_NAME: "production"
  SYNC_INTERVAL: "5"
  NAMESPACES: "production,monitoring"
```

## Badge Usage

Release Tracker provides SVG badges for displaying current release versions in README files:

```markdown
![Release Badge](https://releases.example.com/badges/your-api-key/production/unknown/Deployment/my-app/web)
```

**Badge URL Format:**
```
/badges/{api-key}/{client}/{env}/{workload-kind}/{workload-name}/{container}
```

**Badge States:**
- 🟢 **Green**: Successfully deployed with version
- 🔴 **Red**: Query error, invalid request, or authentication failure
- ⚪ **Gray**: No deployment found
- 🟡 **Yellow**: Multiple deployments found in different namespaces

## Security Considerations

For production deployments, we recommend:

1. **Enable Authentication**: Set strong API keys via `env.API_KEYS`
   ```yaml
   env:
     API_KEYS: "your-secure-32-char-api-key-here,backup-key-for-rotation"
   ```

2. **Namespace Restriction**: Limit monitoring to specific namespaces
   ```yaml
   env:
     NAMESPACES: "production,staging"
   ```

3. **RBAC Configuration**: The chart creates minimal RBAC permissions by default
   - ServiceAccount with read-only access to pods, deployments, statefulsets, daemonsets, and replicasets
   - Cluster-wide permissions only for the specified namespaces

4. **Network Security**:
   - Enable TLS via Ingress with proper certificates
   - Use network policies to restrict access
   - Consider using a dedicated namespace for the application

5. **Resource Limits**: Set appropriate resource limits to prevent resource exhaustion
   ```yaml
   resources:
     limits:
       memory: 512Mi
       cpu: 500m
   ```

## License

This chart is licensed under the MIT License. See the LICENSE file for details.

