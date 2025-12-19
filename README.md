# Helm Charts

[![Artifact Hub](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/sky-sail)](https://artifacthub.io/packages/search?repo=sky-sail)

This repository contains Helm charts for deploying applications to Kubernetes.

## Available Charts

Each chart in this repository has its own README with detailed documentation. Browse the chart directories to find specific information about each chart.

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
