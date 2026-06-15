# Kubernetes Kustomize Tutorial

A beginner-friendly project demonstrating how to use **Kustomize** in Kubernetes to manage multiple environments (Dev and Prod) using a single base configuration.

## What is Kustomize?

Kustomize is a Kubernetes-native configuration management tool that allows you to customize Kubernetes manifests without modifying the original YAML files.

It helps you:

- Reuse common Kubernetes configurations
- Manage multiple environments (dev, staging, prod)
- Apply patches and transformations
- Avoid duplicating YAML files
- Keep configurations clean and maintainable

Kustomize is built directly into `kubectl`.

## Base Configuration

### deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app
  template:
    metadata:
      labels:
        app: app
    spec:
      containers:
        - name: app
          image: nginx:latest
          ports:
            - containerPort: 80
```

### service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: app
spec:
  selector:
    app: app
  ports:
    - port: 80
      targetPort: 80
```

### base/kustomization.yaml

```yaml
resources:
  - deployment.yaml
  - service.yaml
```

## Development Environment

### overlays/dev/kustomization.yaml

```yaml
resources:
  - ../../base

namespace: dev

replicas:
  - name: app
    count: 1
```

## Production Environment

### overlays/prod/cpu-limit.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  template:
    spec:
      containers:
        - name: app
          resources:
            limits:
              cpu: "2"
```

### overlays/prod/kustomization.yaml

```yaml
resources:
  - ../../base

namespace: prod

replicas:
  - name: app
    count: 5

patches:
  - path: cpu-limit.yaml
```

## Prerequisites

Install:

- Kubernetes Cluster (Minikube, Kind, EKS, AKS, GKE, etc.)
- kubectl

Verify installation:

```bash
kubectl version --client
```

## Build Kustomize Manifests

Generate manifests without applying them:

### Development

```bash
kubectl kustomize overlays/dev
```

### Production

```bash
kubectl kustomize overlays/prod
```

## Apply to Kubernetes Cluster

### Development

```bash
kubectl apply -k overlays/dev
```

### Production

```bash
kubectl apply -k overlays/prod
```

## Verify Resources

Check Deployments:

```bash
kubectl get deployments
```

Check Services:

```bash
kubectl get svc
```

Check Pods:

```bash
kubectl get pods
```

## Useful Kustomize Features

### Namespace

```yaml
namespace: prod
```

Applies namespace to all resources.

### Name Prefix

```yaml
namePrefix: prod-
```

Example:

```text
app → prod-app
```

### Name Suffix

```yaml
nameSuffix: -v1
```

Example:

```text
app → app-v1
```

### Labels

```yaml
labels:
  - pairs:
      environment: prod
```

Adds labels to resources automatically.

### ConfigMap Generator

```yaml
configMapGenerator:
  - name: app-config
    literals:
      - ENV=prod
```

Creates ConfigMaps dynamically.

### Secret Generator

```yaml
secretGenerator:
  - name: app-secret
    literals:
      - PASSWORD=mysecret
```

Creates Kubernetes Secrets dynamically.

## Common Commands

Build manifests:

```bash
kubectl kustomize overlays/dev
```

Apply manifests:

```bash
kubectl apply -k overlays/dev
```

Delete resources:

```bash
kubectl delete -k overlays/dev
```

Preview changes:

```bash
kubectl diff -k overlays/dev
```

## Key Takeaway

Kustomize follows a simple philosophy:

> Write Kubernetes manifests once in a Base and customize them through Overlays for different environments.

This helps reduce duplication, improve maintainability, and simplify Kubernetes deployments.
