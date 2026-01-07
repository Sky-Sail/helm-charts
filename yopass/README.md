# yopass

[![Kubesec Security Scan](https://github.com/Sky-Sail/helm-charts/actions/workflows/kubesec-scan.yml/badge.svg)](https://github.com/Sky-Sail/helm-charts/actions/workflows/kubesec-scan.yml)

> 🎯 **Secure by default** | 🛡️ **Production ready** | ⚡ **Kubernetes best practices**

A Helm chart for deploying Yopass, a secure sharing service.

## Description 📝

Yopass is a secure sharing service that allows you to share secrets and sensitive information securely.

**Original Project:** [https://github.com/jhaals/yopass](https://github.com/jhaals/yopass)

## Security Status 🔒

This chart follows security best practices and is scanned regularly:

- ✅ **Kubesec Scan**: Automated security scanning on every push and PR
- ✅ **Non-root user**: Containers run as non-root by default
- ✅ **Read-only filesystem**: Immutable root filesystem
- ✅ **Pod Security Context**: Full security context configuration
- ✅ **Resource limits**: CPU and memory limits defined

Check the [Kubesec scan results](https://github.com/Sky-Sail/helm-charts/actions/workflows/kubesec-scan.yml) for detailed security analysis!

## Installation

### Prerequisites

- ☸️ Kubernetes 1.19+
- 🎩 Helm 3.0+

### Install the chart

```bash
helm install my-yopass ./
```

### Install with custom values

```bash
helm install my-yopass ./ -f my-values.yaml
```

### Test values files

This chart includes test values files for different scenarios:

- **`values-minimal.yaml`**: Minimal configuration with no ingress, gateway, or HTTPProxy enabled. Tests standard deployment with security best practices.
- **`values-full.yaml`**: Full configuration with all resources enabled (ingress, gateway, HTTPProxy). Tests all features and optional configurations.

You can use these for testing:

```bash
# Test minimal configuration
helm template test-release ./ -f values-minimal.yaml

# Test full configuration
helm template test-release ./ -f values-full.yaml
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

## Gateway API (HTTPRoute)

This chart supports the Kubernetes Gateway API standard using HTTPRoute. To enable Gateway support:

```yaml
gateway:
  enabled: true
  gatewayName: my-gateway
  gatewayNamespace: default  # Optional, defaults to release namespace
  hosts:
    - hostname: yopass.example.com
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /
```

Alternatively, you can use `parentRefs` for more advanced configurations:

```yaml
gateway:
  enabled: true
  parentRefs:
    - name: my-gateway
      namespace: default
  hosts:
    - hostname: yopass.example.com
```

For more information, see the [Gateway API documentation](https://gateway-api.sigs.k8s.io/).

### HTTP-to-HTTPS Redirect

This chart supports automatic HTTP-to-HTTPS redirect using a separate HTTPRoute resource. When enabled, it creates a redirect HTTPRoute that attaches to the HTTP listener (port 80) and redirects all traffic to HTTPS.

To enable HTTP-to-HTTPS redirect:

```yaml
gateway:
  enabled: true
  gatewayName: my-gateway
  hosts:
    - hostname: yopass.example.com
  httpRedirect:
    enabled: true
    redirect:
      scheme: "https"
      statusCode: 301
      port: 443
```

**How it works:**

1. When `gateway.httpRedirect.enabled: true`, a separate HTTPRoute named `{release-name}-yopass-redirect` is created
2. The redirect HTTPRoute attaches to the HTTP listener (port 80) via `sectionName: "http"`
3. The main HTTPRoute automatically attaches only to the HTTPS listener (port 443) via `sectionName: "https"` to prevent conflicts
4. All HTTP traffic is redirected to HTTPS using the configured status code (default: 301)

**Configuration options:**

- `httpRedirect.enabled`: Enable/disable HTTP-to-HTTPS redirect (default: `false`)
- `httpRedirect.redirect.scheme`: Redirect scheme (default: `"https"`)
- `httpRedirect.redirect.statusCode`: HTTP status code for redirect (default: `301`)
- `httpRedirect.redirect.port`: Target port for redirect (default: `443`)
- `httpRedirect.redirect.hostname`: Optional hostname override for redirect
- `httpRedirect.parentRefs`: Optional custom parent references (defaults to `gatewayName + "http"` section)
- `httpRedirect.hosts`: Optional hosts for redirect HTTPRoute (defaults to `gateway.hosts`)

## Contour HTTPProxy

This chart supports Project Contour's HTTPProxy resource. To enable HTTPProxy support:

```yaml
httpproxy:
  enabled: true
  virtualhost:
    fqdn: yopass.example.com
    tls:
      secretName: yopass-tls
      minimumProtocolVersion: "1.2"
  routes:
    - conditions:
        - prefix: /
      timeoutPolicy:
        response: 60s
        idle: 3600s
      retryPolicy:
        count: 3
        perTryTimeout: 10s
```

For more information, see the [Contour HTTPProxy documentation](https://projectcontour.io/docs/v1.28.0/config/httpproxy/).

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

---

### 🔒 Secured with Kubesec | ⚡ Production Ready

Made with ❤️ for Kubernetes
