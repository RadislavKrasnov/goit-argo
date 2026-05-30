# goit-argo GitOps Repository

This repository contains Kubernetes and ArgoCD manifests for the MLOps infrastructure deployed to the Kubernetes cluster.

The repository is used as a GitOps source of truth. ArgoCD watches the `namespaces/*` directories and applies the manifests from Git to the cluster automatically.

## Purpose

The goal of this repository is to describe the Kubernetes resources and ArgoCD Applications required for the MLOps stack.

It currently deploys:

* `application` namespace
* MLflow tracking server
* PostgreSQL database for MLflow backend storage
* MinIO object storage for MLflow artifacts
* `monitoring` namespace
* Prometheus Pushgateway for custom metric pushing

## Repository structure

```text
goit-argo/
├── namespaces
│   ├── application
│   │   ├── minio.yaml
│   │   ├── mlflow.yaml
│   │   ├── ns.yaml
│   │   └── postgres.yaml
│   └── monitoring
│       ├── ns.yaml
│       └── pushgateway.yaml
├── .gitignore
└── README.md
```

## Directory description

### `namespaces/application`

This folder contains manifests for the main MLOps application namespace.

Files:

* `ns.yaml`
  Creates the `application` namespace.

* `postgres.yaml`
  Creates an ArgoCD Application that deploys PostgreSQL from the Bitnami Helm chart.
  PostgreSQL is used as the MLflow backend store.

* `minio.yaml`
  Creates an ArgoCD Application that deploys MinIO from the Bitnami Helm chart.
  MinIO is used as S3-compatible artifact storage for MLflow.

* `mlflow.yaml`
  Creates an ArgoCD Application that deploys MLflow from a Helm chart.
  MLflow uses PostgreSQL for metadata and MinIO for artifacts.

### `namespaces/monitoring`

This folder contains manifests for monitoring-related tools.

Files:

* `ns.yaml`
  Creates the `monitoring` namespace.

* `pushgateway.yaml`
  Creates an ArgoCD Application that deploys Prometheus Pushgateway.
  Pushgateway can be used by jobs, scripts, or experiments to push custom metrics into the monitoring stack.

## How GitOps works here

1. Kubernetes infrastructure is created with Terraform.
2. Terraform installs ArgoCD into the cluster.
3. Terraform creates an ArgoCD ApplicationSet.
4. The ApplicationSet watches the `namespaces/*` folders in this repository.
5. ArgoCD applies namespace manifests.
6. ArgoCD creates child Applications for MLflow, PostgreSQL, MinIO, and Pushgateway.
7. Each child Application deploys its component from the configured Helm chart.
8. When files in this repository are changed and pushed to GitHub, ArgoCD syncs the cluster state with the Git state.

## Deployed components

### MLflow

MLflow is deployed into the `application` namespace.

It is configured with:

* PostgreSQL backend store
* MinIO artifact store
* ClusterIP service on port `5000`
* Automated ArgoCD sync
* Self-healing enabled
* Pruning enabled

### PostgreSQL

PostgreSQL is deployed into the `application` namespace.

It is used by MLflow to store experiment metadata, runs, parameters, metrics, and model tracking information.

### MinIO

MinIO is deployed into the `application` namespace.

It provides S3-compatible object storage for MLflow artifacts.

The default bucket is:

```text
mlflow-artifacts
```

### Prometheus Pushgateway

Prometheus Pushgateway is deployed into the `monitoring` namespace.

It exposes port `9091` and is intended for pushing custom metrics from short-lived jobs or scripts.

## Useful checks

Check ArgoCD Applications:

```bash
kubectl get applications -n argocd
```

Check application namespace:

```bash
kubectl get all -n application
```

Check monitoring namespace:

```bash
kubectl get all -n monitoring
```

Check MLflow-related services:

```bash
kubectl get svc -n application
```

Check Pushgateway service:

```bash
kubectl get svc -n monitoring
```

## Open MLflow locally

First check the MLflow service name:

```bash
kubectl get svc -n application
```

Then port-forward MLflow:

```bash
kubectl port-forward -n application svc/mlflow 5000:5000
```

Open in browser:

```text
http://localhost:5000
```

## Open MinIO locally

First check the MinIO service:

```bash
kubectl get svc -n application
```

Port-forward the MinIO console:

```bash
kubectl port-forward -n application svc/minio 9001:9001
```

Open in browser:

```text
http://localhost:9001
```

Default credentials used in the manifest:

```text
Username: minio
Password: minio123
```

## Open Pushgateway locally

Port-forward Pushgateway:

```bash
kubectl port-forward -n monitoring svc/pushgateway-prometheus-pushgateway 9091:9091
```

Open in browser:

```text
http://localhost:9091
```

If the service name is different, check it with:

```bash
kubectl get svc -n monitoring
```

## Sync behavior

The ArgoCD Applications use automated sync with:

* `prune: true`
* `selfHeal: true`

This means:

* resources removed from Git can be removed from the cluster;
* manual changes in the cluster can be reverted by ArgoCD;
* Git remains the source of truth.

## Important note about secrets

This repository currently contains simple development credentials directly in manifests, for example MinIO and PostgreSQL credentials.

This is acceptable only for local learning or demo environments.

For production or shared environments, credentials should be moved to a secure secret management solution, for example:

* Kubernetes Secrets managed outside Git
* Sealed Secrets
* External Secrets Operator
* AWS Secrets Manager
* HashiCorp Vault

## Typical workflow

1. Edit Kubernetes or ArgoCD manifests locally.
2. Commit changes.
3. Push changes to GitHub.
4. ArgoCD detects the change.
5. ArgoCD syncs the cluster.
6. Verify the result with `kubectl` or the ArgoCD UI.

Example:

```bash
git add .
git commit -m "Update MLflow configuration"
git push
```

Then check:

```bash
kubectl get applications -n argocd
kubectl get pods -n application
kubectl get pods -n monitoring
```

## Related repositories

This repository contains only GitOps manifests.

Terraform code that installs ArgoCD, creates the EKS cluster, and configures the ApplicationSet is stored separately in the infrastructure repository or Terraform folder of the main project.
