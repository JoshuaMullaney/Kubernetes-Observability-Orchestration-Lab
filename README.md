# Kubernetes SRE Project

A hands-on site reliability engineering project demonstrating Kubernetes deployment, health monitoring, and observability using a production-grade toolstack.

Built to demonstrate core SRE competencies: container orchestration, liveness/readiness probes, observability pipelines, and automated health checking.

---

## Project Overview

| Component | Technology |
|---|---|
| Container Orchestration | Kubernetes (Minikube) |
| Application | nginx (containerized web server) |
| Monitoring Stack | Prometheus + Grafana (kube-prometheus-stack) |
| Health Check Script | Python 3 |
| Deployment Method | Declarative YAML (`kubectl apply`) |

---

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                  Kubernetes Cluster                  │
│                                                     │
│   ┌─────────────────┐     ┌─────────────────────┐  │
│   │   default ns    │     │    monitoring ns     │  │
│   │                 │     │                     │  │
│   │  ┌───────────┐  │     │  ┌───────────────┐  │  │
│   │  │  my-app   │◄─┼─────┼──│  Prometheus   │  │  │
│   │  │  (nginx)  │  │     │  │  (scraping)   │  │  │
│   │  └───────────┘  │     │  └───────┬───────┘  │  │
│   │                 │     │          │           │  │
│   └─────────────────┘     │  ┌───────▼───────┐  │  │
│                           │  │    Grafana    │  │  │
│                           │  │  (dashboards) │  │  │
│                           │  └───────────────┘  │  │
│                           └─────────────────────┘  │
└─────────────────────────────────────────────────────┘
          ▲
          │
┌─────────┴──────────┐
│  health_check.py   │
│  (external probe)  │
└────────────────────┘
```

---

## Features

### 1. Kubernetes Deployment with Health Probes (`my-app.yaml`)
- Deploys nginx as a containerized application
- Configured with **liveness probe** — Kubernetes automatically restarts the container if the app becomes unresponsive
- Configured with **readiness probe** — Kubernetes removes the pod from the load balancer if it is not ready to serve traffic
- Resource **requests and limits** defined (CPU + memory) enabling proper scheduling and QoS classification (`Burstable`)

### 2. Observability Stack (Prometheus + Grafana)
- Deployed via Helm using the `kube-prometheus-stack` chart into a dedicated `monitoring` namespace
- Prometheus scrapes real-time metrics from all pods across the cluster
- Grafana dashboards visualize CPU utilisation, memory utilisation, and network usage per namespace and pod
- AlertManager included for future alerting configuration

### 3. Python Health Check Script (`health_check.py`)
- Polls the application endpoint every 15 seconds
- Classifies response into **OK / WARN / CRITICAL** based on response time thresholds
- Prints timestamped status lines to the console
- Prints a rolling summary every 5 checks
- Exits cleanly on Ctrl+C with a final summary report

---

## Getting Started

### Prerequisites
- [Minikube](https://minikube.sigs.k8s.io/docs/start/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Helm](https://helm.sh/docs/intro/install/)
- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- Python 3.8+

### 1. Start the cluster
```bash
minikube start --driver=docker
```

### 2. Deploy the application
```bash
kubectl apply -f my-app.yaml
kubectl expose deployment my-app --port=80 --type=NodePort
kubectl get pods  # wait for Running
```

### 3. Deploy the observability stack
```bash
kubectl create namespace monitoring
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring
kubectl get pods -n monitoring  # wait for all Running
```

### 4. Open Grafana
```bash
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80
```
Navigate to **http://localhost:3000**

Retrieve the admin password:
```bash
kubectl get secret -n monitoring prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 --decode
```

### 5. Run the health check script
```bash
# Get your app URL
minikube service my-app --url

# Update APP_URL in health_check.py, then run:
python health_check.py
```

---

## Key Concepts Demonstrated

- **Declarative configuration** — infrastructure defined as code in YAML, applied via `kubectl apply`
- **Self-healing** — liveness probes trigger automatic pod restarts on failure
- **Traffic management** — readiness probes prevent unhealthy pods from receiving requests
- **Observability** — metrics pipeline from application → Prometheus → Grafana dashboards
- **Automated health checking** — external probe script with OK/WARN/CRITICAL classification
- **Namespace isolation** — application and monitoring workloads separated into dedicated namespaces

---

## Skills Demonstrated

`Kubernetes` `Helm` `Prometheus` `Grafana` `Docker` `Python` `Bash` `Linux` `CI/CD Concepts` `Observability` `Incident Response` `Infrastructure as Code`

---

## Author

**Joshua Mullaney**  
[LinkedIn](https://linkedin.com/in/joshua-mullaney-7422a59a) | [GitHub](https://github.com/JoshuaMullaney)
