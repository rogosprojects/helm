# KLogs Viewer Helm Chart

[![Helm Version](https://img.shields.io/badge/helm-v3-blue)](https://helm.sh)

This Helm chart deploys KLogs Viewer, a lightweight web application that allows users to view and download Kubernetes pod logs directly through their browser.

## Features

- **📊 Pod Status Visualization**: See pod status at a glance with color-coded indicators
- **🔍 Label-based Filtering**: Browse pods organized by their labels
- **🔄 Multi-namespace Support**: Monitor all namespaces or limit to specific ones
- **🔒 Secure Access**: Optional token-based authentication
- **🌐 Ingress Support**: Easy integration with your cluster's ingress controller
- **🛡️ RBAC Configuration**: Proper RBAC setup with customizable permissions

## Quick Start


### Installation

Add the Helm repository and install the chart:

```bash
helm repo add rogosprojects https://raw.githubusercontent.com/rogosprojects/helm/master
helm repo update
# Simple installation with default values
helm install klogs-viewer rogosprojects/klogs-viewer
# Installation with custom values file
helm install klogs-viewer rogosprojects/klogs-viewer  --namespace observability --create-namespace --values values.yaml
```

### Using with the chart source code:

```bash
# Clone the repository
git clone https://github.com/rogosprojects/klogs-viewer.git
cd klogs-viewer/helm

# Install directly from the chart directory
helm install klogs-viewer . --namespace observability --create-namespace
```

## Configuration

The following table lists the main configurable parameters of the chart and their default values.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `image.repository` | Container image repository | `rogosprojects/klogs-viewer` |
| `image.tag` | Container image tag | `latest` |
| `image.pullPolicy` | Container image pull policy | `IfNotPresent` |
| `image.pullSecrets` | List of image pull secrets | `[]` |
| `watchAllNamespaces` | Watch all namespaces | `true` |
| `podLabels` | Comma-separated list of pod label selectors | `app=*` |
| `token` | Authentication token (leave empty to disable authentication) | `""` |
| `ingress.className` | Ingress class name | `nginx` |
| `ALLOWED_ORIGINS` | Comma-separated list of allowed WebSocket origins | `""` |

### WebSocket Origin Restrictions

- **For single-domain deployments**: Leave `ALLOWED_ORIGINS` empty to enforce the same-origin policy.
- **For multi-domain deployments**: Set `ALLOWED_ORIGINS` to a comma-separated list of allowed domains.
- **For testing/development only**: Set `ALLOWED_ORIGINS=*` (not recommended for production).

## Security Considerations

For production deployments, we recommend:

1. **Enable Authentication**: Set a strong `token` value
2. **Namespace Restriction**: If not monitoring all namespaces, make

## Troubleshooting

### Common Issues

1. **No pods displayed**
   - Verify the `podLabels` configuration matches your pod labels
   - Check RBAC permissions: the service account needs list/get access to pods

3. **Authentication errors**
   - Verify the token is correctly set in both the secret and when accessing the UI


## License

This chart is licensed under the MIT License. See the LICENSE file for details.
