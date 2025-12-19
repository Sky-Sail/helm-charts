# yopass

[![Trivy Security Scan - yopass](https://github.com/Sky-Sail/helm-charts/actions/workflows/trivy-scan.yml/badge.svg?branch=main&label=yopass)](https://github.com/Sky-Sail/helm-charts/actions/workflows/trivy-scan.yml)

A Helm chart for deploying Yopass, a secure sharing service.

## Chart Details

**Chart Version:** 1.1.0
**App Version:** 12.4.0

## Description

Yopass is a secure sharing service that allows you to share secrets and sensitive information securely.

**Original Project:** [https://github.com/jhaals/yopass](https://github.com/jhaals/yopass)

## Installation

### Prerequisites

- Kubernetes 1.19+
- Helm 3.0+

### Install the chart

```bash
helm install my-yopass ./
```

### Install with custom values

```bash
helm install my-yopass ./ -f my-values.yaml
```

## Configuration

### Dependencies

This chart supports optional dependencies:

- **memcached** (enabled by default) - Used as the backend storage
- **valkey** (disabled by default) - Alternative backend storage option

You can configure these in `values.yaml`:

```yaml
memcached:
  enabled: true

valkey:
  enabled: false
  auth:
    enabled: false
```

### Key Configuration Options

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of replicas | `1` |
| `image.repository` | Container image repository | `jhaals/yopass` |
| `image.tag` | Container image tag | `""` (uses appVersion) |
| `service.type` | Kubernetes service type | `ClusterIP` |
| `service.port` | Service port | `1337` |
| `ingress.enabled` | Enable ingress | `false` |
| `autoscaling.enabled` | Enable horizontal pod autoscaling | `false` |

See `values.yaml` for all available configuration options.

## Ingress

To enable ingress, set `ingress.enabled` to `true` and configure the ingress settings:

```yaml
ingress:
  enabled: true
  className: nginx
  hosts:
    - host: yopass.example.com
      paths:
        - path: /
          pathType: Prefix
```

## Upgrading

```bash
helm upgrade my-yopass ./
```

## Uninstallation

```bash
helm uninstall my-yopass
```

## License

See the application's license for details.
