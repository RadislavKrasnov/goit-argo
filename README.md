# goit-argo GitOps Repository

This repository contains Kubernetes and ArgoCD manifests.

ArgoCD watches the `namespaces/*` folders through an ApplicationSet created by Terraform.

## Structure

```text
goite-argo/
├── namespaces
│   ├── application
│   │   ├── nginx.yaml
│   │   └── ns.yaml
│   └── infra-tools
│       ├── mlflow-application.yaml
│       └── ns.yaml
└── README.md
```

## What is deployed

- `namespaces/application/ns.yaml` creates the `application` namespace.
- `namespaces/application/nginx.yaml` deploys a small demo Nginx Deployment and Service.
- `namespaces/infra-tools/ns.yaml` creates/keeps the `infra-tools` namespace.
- `namespaces/infra-tools/mlflow-application.yaml` creates an ArgoCD Application that deploys MLflow from a Helm chart into the `application` namespace.

## How GitOps works here

1. You push this repository to GitHub.
2. Terraform installs ArgoCD into EKS.
3. Terraform creates an ArgoCD ApplicationSet.
4. ApplicationSet scans `namespaces/*`.
5. ArgoCD applies Kubernetes YAML files and creates the MLflow Application.
6. The MLflow Application deploys MLflow through Helm.

## Useful checks

```bash
kubectl get applications -n infra-tools
kubectl get pods -n infra-tools
kubectl get pods -n application
kubectl get svc -n application
```

## Open MLflow locally

First check the service name and port:

```bash
kubectl get svc -n application
```

Then port-forward the MLflow service. Example:

```bash
kubectl port-forward -n application svc/mlflow 5000:5000
```

Open:

```text
http://localhost:5000
```
