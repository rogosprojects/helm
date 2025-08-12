# KPod Monitor Helm Chart

This Helm chart deploys the Kubernetes Pod Monitor application, a modern dashboard for monitoring Kubernetes applications and their pods with real-time updates.

## Prerequisites

- Helm 3.0+
- Metrics Server installed in the cluster (optional, for resource metrics)

## Installing the Chart

To install the chart with the release name `kpods-monitor`:

```bash
helm repo add rogosprojects https://rogosprojects.github.io/helm
helm repo update
# Simple installation with default values
helm install kpods-monitor rogosprojects/kpods-monitor
# Installation with custom values file
helm install kpods-monitor rogosprojects/kpods-monitor  --namespace observability --create-namespace --values values.yaml
```

## Configuration

The following table lists the configurable parameters of the Kubernetes Pod Monitor chart and their default values.

### Global Parameters

| Parameter                 | Description                                     | Default                        |
|---------------------------|-------------------------------------------------|--------------------------------|
| `replicaCount`            | Number of replicas                              | `1`                            |
| `image.repository`        | Image repository                                | `ghcr.io/rogosprojects/kpods-monitor` |
| `image.tag`               | Image tag                                       | `""` (defaults to chart appVersion) |
| `image.pullPolicy`        | Image pull policy                               | `IfNotPresent`                 |
| `imagePullSecrets`        | Image pull secrets                              | `[]`                           |
| `nameOverride`            | Override the name of the chart                  | `""`                           |
| `fullnameOverride`        | Override the full name of the chart             | `""`                           |
| `namespace`               | Namespace for the application                   | `""`                           |
| `serviceAccount.create`   | Create a service account                        | `true`                         |
| `serviceAccount.name`     | The name of the service account                 | `""`                           |
| `podAnnotations`          | Annotations for the pods                        | `{}`                           |
| `podSecurityContext`      | Security context for the pods                   | `{}`                           |
| `securityContext`         | Security context for the containers             | `{}`                           |
| `service.type`            | Service type                                    | `ClusterIP`                    |
| `service.port`            | Service port                                    | `80`                           |
| `service.targetPort`      | Service target port                             | `8080`                         |
| `ingress.enabled`         | Enable ingress                                  | `false`                        |
| `resources`               | CPU/Memory resource requests/limits             | See `values.yaml`              |
| `nodeSelector`            | Node selector                                   | `{}`                           |
| `tolerations`             | Tolerations                                     | `[]`                           |
| `affinity`                | Affinity                                        | `{}`                           |

### Application Configuration

| Parameter                                  | Description                                     | Default                        |
|--------------------------------------------|-------------------------------------------------|--------------------------------|
| `config.general.name`                      | Dashboard name                                  | `"Kubernetes Pod Monitor"`     |
| `config.general.port`                      | Application port                                | `8080`                         |
| `config.general.debug`                     | Enable debug logging                            | `false`                        |
| `config.general.basePath`                  | Base path for serving the application           | `""`                           |
| `config.general.auth.enabled`              | Enable authentication                           | `false`                        |
| `config.general.auth.type`                 | Authentication type (basic, token, none)        | `"none"`                       |
| `config.general.auth.apiKey`               | API key for token authentication                | `""`                           |
| `config.general.auth.username`             | Username for basic authentication               | `"admin"`                      |
| `config.general.auth.password`             | Password for basic authentication               | `"change_this_password"`       |
| `config.cluster.inCluster`                 | Use in-cluster configuration                    | `true`                         |
| `config.cluster.kubeConfigPath`            | Path to kubeconfig file                         | `""`                           |
| `config.applications`                      | Applications to monitor                         | See `values.yaml`              |

## Example: Monitoring Custom Applications

To monitor your own applications, modify the `config.applications` section in your `values.yaml`:

```yaml
config:
  applications:
    - name: "My Application"
      description: "My custom application"
      order: 10
      selector:
        my-namespace:
          deployments:
            - my-deployment
            - my-api
          statefulsets:
            - my-database
```

## Security Considerations

By default, the chart creates a service account with permissions to list and get pods, deployments, statefulsets, and daemonsets across all namespaces. If you need more restricted permissions, modify the RBAC rules in `templates/rbac.yaml`.

For production deployments, consider:

1. Enabling authentication by setting `config.general.auth.enabled` to `true`
2. Using a secure password or API key
3. Enabling TLS via Ingress
4. Setting appropriate resource limits

## Troubleshooting

If the dashboard is not showing any applications:

1. Check that the service account has the necessary permissions
2. Verify that your application selectors match existing resources
3. Check the pod logs for any errors
4. Check the application base path