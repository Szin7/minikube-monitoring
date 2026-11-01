# Minikube Monitoring with Prometheus and Grafana

A Kubernetes monitoring setup built on Minikube using Prometheus, Grafana, and Helm.

This project covers setting up a monitoring stack, collecting metrics from Kubernetes workloads, and visualizing resource usage through Grafana dashboards.

## Tech Stack

- **Kubernetes (Minikube)** — Local Kubernetes cluster
- **Helm** — Monitoring stack installation and configuration
- **Prometheus** — Metrics collection and storage
- **Grafana** — Monitoring dashboards and visualization
- **Alertmanager** — Alert handling and management
- **Node Exporter** — Node-level resource metrics
- **kube-state-metrics** — Kubernetes workload metrics

## Implementation

### 1. Monitoring Stack Setup

Deployed `kube-prometheus-stack` using Helm in a dedicated `monitoring` namespace.

The installation includes Prometheus, Grafana, Alertmanager, Prometheus Operator, Node Exporter, and kube-state-metrics.

### 2. Kubernetes Monitoring

Configured and explored monitoring for:

- Node CPU and memory utilization
- Kubernetes Pod resource consumption
- Network traffic and disk activity
- Container and workload metrics
- Kubernetes monitoring targets

### 3. Grafana Dashboards

Used the built-in Kubernetes dashboards and imported additional dashboards for more detailed monitoring.

| Dashboard | ID | Purpose |
|---|---|---|
| Node Exporter Full | 1860 | CPU, memory, disk, and network monitoring |
| Kubernetes / Views / Pods | 15760 | Pod-level resource monitoring |
| Kubernetes / Compute Resources / Namespace (Pods) | Built-in | Namespace and Pod metrics |

### 4. Monitoring Verification

Verified the monitoring setup using Kubernetes commands, Prometheus targets, and Grafana dashboards.

Checked Pod readiness, Kubernetes Services, and collected metrics to confirm the monitoring components were functioning.

## Accessing the Monitoring Tools

The monitoring interfaces can be accessed locally using Kubernetes port forwarding.

**Grafana**

```bash
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80
```

Open `http://localhost:3000`

**Prometheus**

```bash
kubectl port-forward -n monitoring svc/monitoring-kube-prometheus-prometheus 9090:9090
```

Open `http://localhost:9090`

**Alertmanager**

```bash
kubectl port-forward -n monitoring svc/monitoring-kube-prometheus-alertmanager 9093:9093
```

Open `http://localhost:9093`

## Screenshots

Screenshots of the Kubernetes monitoring dashboards, Node Exporter metrics, Prometheus targets, and monitoring components will be included here.

## What I Learned

This lab helped me understand how monitoring works in a Kubernetes environment, particularly how Prometheus collects metrics from different targets and how Grafana can be used to analyze them.

I also gained practical experience with Helm installations, Kubernetes namespaces and services, monitoring dashboards, and inspecting the health of cluster components.

## Reference

This setup was based on the [Day 12 monitoring lab from Sakshi9111/broadway-labs](https://github.com/Sakshi9111/broadway-labs/tree/main/day12), with the installation, verification, and dashboard testing performed in my local Minikube environment.
