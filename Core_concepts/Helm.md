# Helm - The Kubernetes Package Manager

## What is Helm?

Helm is the package manager for Kubernetes. It allows you to define, install, and upgrade complex Kubernetes applications. Helm uses charts, which are packages of pre-configured Kubernetes resources.

## Key Concepts

- **Chart**: A Helm package containing all the resource definitions necessary to run an application in Kubernetes
- **Repository**: A place where charts are stored and shared
- **Release**: An instance of a chart running in a Kubernetes cluster
- **Values**: Configuration that can be supplied to customize a chart during installation

## Installing Helm

### On macOS:
```bash
brew install helm
```

### On Linux:
```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### On Windows (using Chocolatey):
```bash
choco install kubernetes-helm
```

## Basic Helm Commands

### Add a Repository
```bash
helm repo add [NAME] [URL]
```

Example:
```bash
helm repo add stable https://charts.helm.sh/stable
```

### Update Repositories
```bash
helm repo update
```

### Search for Charts
```bash
helm search repo [KEYWORD]
```

Example:
```bash
helm search repo nginx
```

### Install a Chart
```bash
helm install [RELEASE_NAME] [CHART_NAME]
```

Example:
```bash
helm install my-nginx bitnami/nginx
```

### List Installed Releases
```bash
helm list
```

### Uninstall a Release
```bash
helm uninstall [RELEASE_NAME]
```

### Upgrade a Release
```bash
helm upgrade [RELEASE_NAME] [CHART_NAME]
```

## Customizing Chart Installation

### Using Values File
```bash
helm install -f values.yaml [RELEASE_NAME] [CHART_NAME]
```

### Setting Values on Command Line
```bash
helm install --set key1=value1,key2=value2 [RELEASE_NAME] [CHART_NAME]
```

## Creating Your Own Charts

### Create a New Chart
```bash
helm create [CHART_NAME]
```

### Chart Directory Structure
```
mychart/
  Chart.yaml          # Chart metadata
  values.yaml         # Default configuration values
  charts/             # Directory for chart dependencies
  templates/          # Directory for template files
  templates/NOTES.txt # Usage notes
```

### Validating a Chart
```bash
helm lint [CHART_PATH]
```

### Package a Chart
```bash
helm package [CHART_PATH]
```

## CKAD Exam Tips for Helm

1. **Focus on Usage**: For CKAD, focus primarily on using existing charts rather than creating your own.
2. **Know the Basic Commands**: Be comfortable with `helm install`, `helm list`, `helm uninstall`, and `helm upgrade`.
3. **Values Customization**: Understand how to override default values using either the `--set` flag or a values file.
4. **Resources Created**: Be able to inspect what Kubernetes resources were created by a Helm release.
5. **Troubleshooting**: Know how to check the status of a Helm release and debug common issues.

## Common Use Cases

1. **Installing Application Stacks**: Installing comprehensive applications like monitoring stacks, databases, or web applications with a single command.
2. **Standardizing Deployments**: Ensuring consistent deployments across different environments.
3. **Release Management**: Managing application lifecycle with easy upgrades and rollbacks.

## Relevant Documentation

- [Official Helm Documentation](https://helm.sh/docs/)
- [Helm Hub](https://artifacthub.io/) - Find and publish Helm charts
- [Helm Best Practices](https://helm.sh/docs/chart_best_practices/)