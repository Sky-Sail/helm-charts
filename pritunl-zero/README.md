# pritunl-zero

[![Kubesec Security Scan](https://github.com/Sky-Sail/helm-charts/actions/workflows/kubesec-scan.yml/badge.svg)](https://github.com/Sky-Sail/helm-charts/actions/workflows/kubesec-scan.yml)

> 🎯 **Secure by default** | 🛡️ **Production ready** | ⚡ **Kubernetes best practices**

A Helm chart for deploying Pritunl Zero, a Zero Trust Network Access (ZTNA) solution.

## Description 📝

Pritunl Zero is a Zero Trust Network Access (ZTNA) solution that provides secure remote access to your infrastructure without traditional VPNs. It uses modern authentication methods and provides granular access control.

**Original Project:** [https://github.com/pritunl/pritunl-zero](https://github.com/pritunl/pritunl-zero)

## Security Status 🔒

This chart follows security best practices and is scanned regularly:

- ✅ **Kubesec Scan**: Automated security scanning on every push and PR
- ✅ **Non-root user**: Containers run as non-root by default
- ✅ **Read-only filesystem**: Immutable root filesystem with automatic `/tmp` emptyDir mount
- ✅ **Pod Security Context**: Full security context configuration
- ✅ **Resource limits**: CPU and memory limits defined

Check the [Kubesec scan results](https://github.com/Sky-Sail/helm-charts/actions/workflows/kubesec-scan.yml) for detailed security analysis!

## Installation

### Prerequisites

- ☸️ Kubernetes 1.19+
- 🎩 Helm 3.0+
- 🗄️ MongoDB instance (required for Pritunl Zero)
- 💾 Storage class for PVC (if persistence is enabled)

### Install the chart

```bash
helm install my-pritunl-zero ./
```

### Install with custom values

```bash
helm install my-pritunl-zero ./ -f my-values.yaml
```

### Test values files

This chart includes test values files for different scenarios:

- **`values-minimal.yaml`**: Minimal configuration with no ingress or gateway enabled. Tests standard deployment with security best practices.
- **`values-full.yaml`**: Full configuration with all resources enabled (ingress, gateway). Tests all features and optional configurations.

You can use these for testing:

```bash
# Test minimal configuration
helm template test-release ./ -f values-minimal.yaml

# Test full configuration
helm template test-release ./ -f values-full.yaml
```

## Configuration

### Required Environment Variables

Pritunl Zero requires two mandatory environment variables:

1. **`MONGO_URI`**: MongoDB connection string
   - Format: `mongodb://[username:password@]host[:port][/database]`
   - Example: `mongodb://mongo.example.com:27017/pritunl-zero`

2. **`NODE_ID`**: Unique identifier for each node
   - Must be unique for each instance
   - Example: `node-1`, `node-2`, etc.

Configure these in `values.yaml`:

**Option 1: Direct configuration (simple, but password in plain text):**

```yaml
env:
  MONGO_URI: "mongodb://mongo.example.com:27017/pritunl-zero"
  NODE_ID: "node-1"
```

**Option 2: Using an existing Kubernetes secret (recommended for production):**

First, create a secret with your MongoDB connection string:

```bash
kubectl create secret generic mongo-secret \
  --from-literal=mongo-uri='mongodb://user:password@mongo.example.com:27017/pritunl-zero'
```

Then configure the chart to use it:

```yaml
mongoSecret:
  enabled: true
  name: mongo-secret
  key: mongo-uri

env:
  NODE_ID: "node-1"
```

### Key Configuration Options

| Parameter | Description | Default |
|-----------|-------------|---------|
| `image.repository` | Container image repository | `docker.io/pritunl/pritunl-zero` |
| `image.tag` | Container image tag | `""` (uses appVersion) |
| `service.type` | Kubernetes service type | `ClusterIP` |
| `service.port` | Service port | `80` |
| `persistence.enabled` | Enable PVC for /etc config storage | `true` |
| `persistence.size` | Size of PVC | `1Gi` |
| `persistence.storageClass` | Storage class for PVC | `""` (default) |
| `persistence.accessModes` | Access modes for PVC | `["ReadWriteOnce"]` |
| `env.MONGO_URI` | MongoDB connection URI (if not using secret) | `""` |
| `env.NODE_ID` | Unique node identifier | `""` (required) |
| `mongoSecret.enabled` | Use existing secret for MONGO_URI | `false` |
| `mongoSecret.name` | Name of the existing secret | `""` |
| `mongoSecret.key` | Key in secret containing MONGO_URI | `"mongo-uri"` |
| `ingress.enabled` | Enable ingress | `false` |
| `gateway.enabled` | Enable Gateway API (HTTPRoute) | `false` |

See `values.yaml` for all available configuration options.

## Persistence

This chart uses a **StatefulSet** and creates a PersistentVolumeClaim (PVC) to store Pritunl Zero configuration files at `/etc`. This is required because Pritunl Zero needs to write its configuration file (`/etc/pritunl-zero.json`) and the container runs with a read-only root filesystem for security.

**Default configuration:**

```yaml
persistence:
  enabled: true
  size: 1Gi
  storageClass: ""  # Uses default storage class
  accessModes:
    - ReadWriteOnce
```

**Disable persistence (not recommended):**

```yaml
persistence:
  enabled: false
```

**Note:** If persistence is disabled, the `/etc` mount will not be created and Pritunl Zero may fail to start if it needs to write configuration files.

## Ingress

To enable ingress, set `ingress.enabled` to `true` and configure the ingress settings.

### Using Domain List (Recommended)

You can use a simple domain list to automatically create hosts entries:

```yaml
ingress:
  enabled: true
  className: nginx
  domains:
    - pritunl-zero.example.com
    - zero.example.com
  tls:
    - secretName: pritunl-zero-tls
      hosts:
        - pritunl-zero.example.com
        - zero.example.com
```

### Using Manual Hosts Configuration

Alternatively, you can manually configure hosts:

```yaml
ingress:
  enabled: true
  className: nginx
  hosts:
    - host: pritunl-zero.example.com
  tls:
    - secretName: pritunl-zero-tls
      hosts:
        - pritunl-zero.example.com
```

**Note:** If `domains` is specified, it takes precedence over `hosts` configuration. All ingress routes use the root path `/` only.

### Node Management Ingress

This chart supports a separate ingress for node management/admin interface. This allows you to expose the management interface on a different domain than the main application.

```yaml
ingress:
  enabled: true
  domains:
    - pritunl-zero.example.com
  nodeManagement:
    enabled: true
    className: nginx
    domains:
      - manage.pritunl-zero.example.com
      - admin.pritunl-zero.example.com
    tls:
      - secretName: node-management-tls
        hosts:
          - manage.pritunl-zero.example.com
          - admin.pritunl-zero.example.com
```

**Configuration options:**

- `ingress.nodeManagement.enabled`: Enable/disable node management ingress (default: `false`)
- `ingress.nodeManagement.className`: Ingress class name (defaults to `ingress.className` if not specified)
- `ingress.nodeManagement.domains`: List of domains for node management
- `ingress.nodeManagement.hosts`: Manual hosts configuration (used if domains is empty)
- `ingress.nodeManagement.annotations`: Custom annotations for node management ingress
- `ingress.nodeManagement.tls`: TLS configuration for node management ingress

## Gateway API (HTTPRoute)

This chart supports the Kubernetes Gateway API standard using HTTPRoute. To enable Gateway support:

### Using Domain List (Recommended)

You can use a simple domain list to automatically create hostnames entries:

```yaml
gateway:
  enabled: true
  gatewayName: my-gateway
  gatewayNamespace: default  # Optional, defaults to release namespace
  domains:
    - pritunl-zero.example.com
    - zero.example.com
  # Optional: filters
  filters:
    - type: RequestHeaderModifier
      requestHeaderModifier:
        set:
          - name: X-Forwarded-Proto
            value: https
```

### Using Manual Hosts Configuration

Alternatively, you can manually configure hosts:

```yaml
gateway:
  enabled: true
  gatewayName: my-gateway
  gatewayNamespace: default  # Optional, defaults to release namespace
  hosts:
    - hostname: pritunl-zero.example.com
      tls:
        certificateRefs:
          - name: pritunl-zero-tls
  # Optional: filters
  filters:
    - type: RequestHeaderModifier
      requestHeaderModifier:
        set:
          - name: X-Forwarded-Proto
            value: https
```

### Using Parent References

You can also use `parentRefs` for more advanced configurations:

```yaml
gateway:
  enabled: true
  parentRefs:
    - name: my-gateway
      namespace: default
  domains:
    - pritunl-zero.example.com
```

**Note:** If `domains` is specified, it takes precedence over `hosts` configuration.

### Node Management HTTPRoute

This chart supports a separate HTTPRoute for node management/admin interface. This allows you to expose the management interface on a different domain than the main application.

```yaml
gateway:
  enabled: true
  gatewayName: my-gateway
  domains:
    - pritunl-zero.example.com
  nodeManagement:
    enabled: true
    domains:
      - manage.pritunl-zero.example.com
      - admin.pritunl-zero.example.com
    # Optional: filters
    filters:
      - type: RequestHeaderModifier
        requestHeaderModifier:
          set:
            - name: X-Forwarded-Proto
              value: https
```

**Configuration options:**

- `gateway.nodeManagement.enabled`: Enable/disable node management HTTPRoute (default: `false`)
- `gateway.nodeManagement.gatewayName`: Gateway name (defaults to `gateway.gatewayName` if not specified)
- `gateway.nodeManagement.gatewayNamespace`: Gateway namespace (defaults to `gateway.gatewayNamespace` or release namespace)
- `gateway.nodeManagement.domains`: List of domains for node management
- `gateway.nodeManagement.hosts`: Manual hosts configuration (used if domains is empty)
- `gateway.nodeManagement.annotations`: Custom annotations for node management HTTPRoute
- `gateway.nodeManagement.filters`: Optional filters for node management HTTPRoute
- `gateway.nodeManagement.parentRefs`: Custom parent references (alternative to gatewayName)

For more information, see the [Gateway API documentation](https://gateway-api.sigs.k8s.io/).

## MongoDB Setup

Pritunl Zero requires a MongoDB instance. You can use:

- External MongoDB instance
- MongoDB deployed in Kubernetes (e.g., using the Bitnami MongoDB chart)
- MongoDB Atlas or other managed MongoDB services

Ensure the MongoDB instance is accessible from your Kubernetes cluster and configure the `MONGO_URI` accordingly.

### Using Kubernetes Secrets for MongoDB Credentials

For production deployments, it's recommended to store MongoDB credentials in a Kubernetes secret rather than in plain text in `values.yaml`.

**Create a secret with MongoDB connection string:**

```bash
kubectl create secret generic pritunl-zero-mongo \
  --from-literal=mongo-uri='mongodb://username:password@mongo.example.com:27017/pritunl-zero' \
  --namespace=your-namespace
```

**Configure the chart to use the secret:**

```yaml
mongoSecret:
  enabled: true
  name: pritunl-zero-mongo
  key: mongo-uri  # Default: "mongo-uri"

env:
  NODE_ID: "node-1"
```

**Note:** The secret must exist in the same namespace where you're deploying the chart. The secret should contain the full MongoDB connection string in the format: `mongodb://[username:password@]host[:port][/database]`

## Upgrading

```bash
helm upgrade my-pritunl-zero ./
```

## Uninstallation

```bash
helm uninstall my-pritunl-zero
```

## License

See the application's license for details.

---

### 🔒 Secured with Kubesec | ⚡ Production Ready

Made with ❤️ for Kubernetes

