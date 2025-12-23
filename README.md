# Helm Charts

[![Artifact Hub](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/sky-sail)](https://artifacthub.io/packages/search?repo=sky-sail)

This repository contains Helm charts for deploying applications to Kubernetes.

## Available Charts

Each chart in this repository has its own README with detailed documentation. Browse the chart directories to find specific information about each chart.

## Security Best Practices

All charts in this repository follow security best practices by default:

- **Non-root user**: Containers run as non-root users by default
- **Read-only filesystem**: Filesystems are mounted as read-only where possible
- **Pod Security Context**: Full pod security context is configured with appropriate security settings
- **Resource limits**: CPU and memory limits are defined to prevent resource exhaustion
- **Network policies**: Network policies can be enabled for additional network isolation

These security settings are enabled by default but can be customized in each chart's `values.yaml` file if needed for specific use cases.

## Usage

### Add this repository

```bash
helm repo add skysail https://sky-sail.github.io/helm-charts/
helm repo update
```

**Note:** The repository URL will be available after enabling GitHub Pages and running the publish workflow.

### Artifact Hub

This repository is published to [Artifact Hub](https://artifacthub.io) under the `skysail` organization. When you add the repository to Artifact Hub, **each chart automatically gets its own separate page** for easy discovery and installation. See [DEPLOYMENT.md](DEPLOYMENT.md) for detailed publishing instructions.

### Install a chart

```bash
helm install my-release skysail/<chart-name>
```

For example, to install yopass:

```bash
helm install my-yopass skysail/yopass
```

### Upgrade a chart

```bash
helm upgrade my-release skysail/<chart-name>
```

## Chart Signing and Verification

All charts in this repository are cryptographically signed using GPG to ensure authenticity and integrity.

### Verifying Signed Charts

#### 1. Import the Public GPG Key

```bash
# Download and import the public key
curl https://sky-sail.github.io/helm-charts/pubring.gpg | gpg --import

# Verify the key was imported
gpg --list-keys "Kacper Karczmarzyk"
```

#### 2. Verify a Chart Before Installation

```bash
# Pull and verify a chart
helm pull skysail/yopass --verify --keyring ~/.gnupg/pubring.gpg

# Or install with automatic verification
helm install my-yopass skysail/yopass --verify --keyring ~/.gnupg/pubring.gpg
```

#### 3. Verify the Repository Index

The repository index is also signed:

```bash
# Download and verify the index signature
curl -O https://sky-sail.github.io/helm-charts/index.yaml
curl -O https://sky-sail.github.io/helm-charts/index.yaml.asc
gpg --verify index.yaml.asc index.yaml
```

### Trusting the Key (Optional)

To mark the key as trusted and avoid warnings:

```bash
# Get the key fingerprint
gpg --fingerprint "Kacper Karczmarzyk"

# Trust the key (replace with actual fingerprint)
echo "F81881FAF41751368A6AB6BF9B50CF408ED73162:6:" | gpg --import-ownertrust
```

### GPG Key Information

- **Key ID:** `9B50CF408ED73162`
- **Fingerprint:** `F81881FAF41751368A6AB6BF9B50CF408ED73162`
- **Owner:** Kacper Karczmarzyk <kacper@karczmarzyk.me>
- **Public Key:** Available at [https://sky-sail.github.io/helm-charts/pubring.gpg](https://sky-sail.github.io/helm-charts/pubring.gpg)

For more detailed verification instructions, see the [Helm Chart Provenance documentation](https://helm.sh/docs/topics/provenance/).

## Development

### Linting

This repository uses GitHub Actions to automatically lint YAML files and Helm charts on pull requests and pushes.

### Testing

To test a chart locally:

```bash
helm lint <chart-name>/
helm template <chart-name>/ | kubectl apply --dry-run=client -f -
```

## Contributing

1. Make your changes to the chart
2. Update the chart version in `Chart.yaml`
3. Test your changes locally
4. Submit a pull request
