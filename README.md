# MONITORING_WITH_PROMATHUS_GRAFANA_LOKI_HELM
MONITORING_INSTALLATION
# Kubernetes Monitoring on AWS EC2
## Prometheus, Grafana & Loki – Step-by-Step Installation Guide

This README provides a **complete, production-ready, step-by-step guide** to install **Prometheus, Grafana, and Loki** for **Kubernetes cluster and application monitoring** on an **AWS EC2-based Kubernetes cluster** using **Helm**.

---

## 1. Prerequisites

### Infrastructure
- AWS EC2 instances running Kubernetes (kubeadm / EKS / k3s)
- Minimum:
  - 1 Control Plane
  - 2 Worker Nodes

### Network / Security Group
Allow inbound ports:
- `3000` – Grafana
- `9090` – Prometheus (optional)

### Tools Installed
```bash
kubectl version --client
helm version
```

Install Helm if missing:
```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

---

## 2. Create Monitoring Namespace

```bash
kubectl create namespace monitoring
```

---

## 3. Install Prometheus & Grafana (kube-prometheus-stack)

This stack installs:
- Prometheus
- Grafana
- Alertmanager
- Node Exporter
- kube-state-metrics

### Add Helm Repository
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

### Install Stack
```bash
helm install monitoring prometheus-community/kube-prometheus-stack   --namespace monitoring
```

### Verify Installation
```bash
kubectl get pods -n monitoring
```

Expected Pods:
- prometheus-*
- grafana-*
- alertmanager-*
- node-exporter-*

---

## 4. Access Grafana

### Get Admin Password
```bash
kubectl get secret monitoring-grafana   -n monitoring   -o jsonpath="{.data.admin-password}" | base64 --decode
```

### Port Forward
```bash
kubectl port-forward svc/monitoring-grafana 3000:80 -n monitoring
```

Access Grafana:
```
http://<EC2_PUBLIC_IP>:3000
```

Login:
- Username: `admin`
- Password: retrieved above

---

## 5. Install Loki (Log Monitoring)

### Add Grafana Helm Repo
```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

### Install Loki + Promtail
```bash
helm install loki grafana/loki-stack   --namespace monitoring   --set promtail.enabled=true
```

### Verify Pods
```bash
kubectl get pods -n monitoring | grep loki
```

---

## 6. Configure Loki in Grafana

Usually auto-configured.

Manual configuration (if required):
- Grafana → Settings → Data Sources → Add Data Source
- Type: **Loki**
- URL:
```
http://loki:3100
```
- Save & Test

---

## 7. Kubernetes Cluster Monitoring

Preloaded Dashboards:
- Kubernetes / Nodes
- Kubernetes / Pods
- Kubernetes / Workloads
- Cluster CPU, Memory, Disk, Network

These dashboards come by default with kube-prometheus-stack.

---

## 8. Application Monitoring (Metrics)

Your application must expose `/metrics`.

Example Deployment Annotation:
```yaml
annotations:
  prometheus.io/scrape: "true"
  prometheus.io/path: "/metrics"
  prometheus.io/port: "8080"
```

Prometheus automatically discovers and scrapes metrics.

---

## 9. Application Log Monitoring (Loki)

Grafana → Explore → Loki

Example LogQL Query:
```logql
{namespace="default", pod=~"myapp.*"}
```

Filter logs by:
- namespace
- pod
- container
- labels

---

## 10. Production Best Practices

### Persistent Storage
Use EBS-backed PVCs for:
- Prometheus
- Grafana
- Loki

### Retention Policy
```yaml
prometheus:
  prometheusSpec:
    retention: 15d
```

### Security
- Enable RBAC
- Secure Grafana using ALB / Ingress
- Use IAM Roles for Service Accounts (EKS)

### Alerts
Configure Alertmanager for:
- Node Down
- High CPU / Memory
- Pod Restart Count
- Disk Usage

---

## 11. Architecture Overview

- Prometheus scrapes metrics from Kubernetes and applications
- Grafana visualizes metrics and logs
- Loki stores logs efficiently using labels
- Promtail collects logs from each node
- Alertmanager sends alerts to Email / Slack / PagerDuty

---

## 12. Interview-Ready Summary

- Metrics: Prometheus
- Visualization: Grafana
- Logs: Loki
- Log Agent: Promtail
- Alerting: Alertmanager

---

## 13. Useful Commands

```bash
kubectl get all -n monitoring
helm list -n monitoring
kubectl describe pod <pod-name> -n monitoring
```

---

## 14. References

- https://prometheus.io
- https://grafana.com
- https://kubernetes.io

---

### Author
DevOps / Cloud Monitoring – AWS EC2 Kubernetes
