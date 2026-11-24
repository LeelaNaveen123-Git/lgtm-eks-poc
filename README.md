# LGTM Observability Stack on Amazon EKS (Metrics, Logs, Traces + S3/Glacier)

Full-stack observability platform (**Loki, Grafana, Tempo, Prometheus + Grafana Alloy**) deployed on **Amazon EKS (`observability-prod-eks-01`)** in `ap-south-1`.

## Architecture & AWS Console Configuration
- **Grafana Alloy:** Collects Kubernetes pod logs and OTLP traces (`4317`/`4318`).
- **Grafana Loki:** Stores TSDB v13 index tables and log chunks in `prod-loki-logs-2026-ap-south-1` via **EKS Pod Identity** (`LokiPodIdentityRole`, configured via AWS Console).
- **S3 + Glacier Lifecycle (AWS Console):** Rule `loki-10d-standard-30d-glacier` keeps logs in **S3 Standard for 10 days**, transitions them to **Amazon S3 Glacier Flexible Retrieval for 30 days**, and expires them at **Day 40**.
- **Grafana Dashboards:** Pre-wired **Prometheus** (Metrics), **Loki** (Logs - Dashboard `13639`), and **Tempo** (Traces) with bi-directional `traceID` correlation.

## Quick Deploy
```bash
kubectl apply -f storageclass-gp3.yaml
kubectl create namespace monitoring
helm repo add grafana https://github.io
helm repo add prometheus-community https://github.io
helm repo update

helm upgrade --install loki grafana/loki -n monitoring -f loki-values.yaml
helm upgrade --install tempo grafana/tempo -n monitoring
helm upgrade --install alloy grafana/alloy -n monitoring -f alloy-values.yaml
helm upgrade --install kube-prometheus prometheus-community/kube-prometheus-stack -n monitoring -f kube-prometheus-values.yaml
```
