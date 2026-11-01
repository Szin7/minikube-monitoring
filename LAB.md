# Minikube monitoring lab

This is the monitoring setup I ran on a local Minikube cluster using the Prometheus community Helm chart. The chart provides the Kubernetes workloads; this document records the commands, settings, and checks used in the lab.

The exercise followed [Sakshi9111's Day 12 monitoring instructions](https://github.com/Sakshi9111/broadway-labs/tree/main/day12). The commands below are organized for running the same setup independently.

## 1. Check the prerequisites

Docker, Minikube, kubectl and Helm are required. The lab calls for at least 4 CPUs and 6 GB RAM for Minikube.

```bash
minikube version
kubectl version --client
helm version
```

## 2. Start Minikube

```bash
minikube start --driver=docker --cpus=4 --memory=6144
minikube status
```

If an existing Minikube cluster is running with workloads you need to keep, check its status before changing its resources.

## 3. Install the monitoring stack

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
kubectl create namespace monitoring
```

For a fresh installation, choose a local Grafana password rather than storing it in Git:

```bash
read -rsp 'Grafana admin password: ' GRAFANA_ADMIN_PASSWORD; echo
helm install monitoring prometheus-community/kube-prometheus-stack \
  -n monitoring \
  --set grafana.adminPassword="$GRAFANA_ADMIN_PASSWORD" \
  --set prometheus.prometheusSpec.serviceMonitorSelectorNilUsesHelmValues=false
unset GRAFANA_ADMIN_PASSWORD
```

The original training exercise used `admin123` as its lab password. Replace that on any environment other than a disposable local lab.

The installed chart includes Prometheus, Grafana, Alertmanager, Prometheus Operator, node-exporter and kube-state-metrics.

## 4. Verify the installation

```bash
kubectl get pods -n monitoring -w
```

Stop watching with Ctrl+C when the pods become Ready, then inspect the services:

```bash
kubectl get svc -n monitoring
helm list -n monitoring
```

## 5. Apply the Minikube-specific monitoring settings

The original lab disables four scrape integrations that can show as unavailable in this local Minikube setup:

```bash
helm upgrade monitoring prometheus-community/kube-prometheus-stack -n monitoring \
  --reuse-values \
  --set kubeEtcd.enabled=false \
  --set kubeControllerManager.enabled=false \
  --set kubeScheduler.enabled=false \
  --set kubeProxy.enabled=false
```

This does not stop those Kubernetes control-plane components. It only changes their monitoring configuration.

## 6. Open the monitoring interfaces

Run each command in its own terminal, leaving it running while the corresponding interface is in use.

**Grafana — http://localhost:3000**

```bash
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80
```

Sign in with username `admin` and your configured password.

**Prometheus — http://localhost:9090**

```bash
kubectl port-forward -n monitoring svc/monitoring-kube-prometheus-prometheus 9090:9090
```

Inspect *Status → Targets* to see the scrape results.

**Alertmanager — http://localhost:9093**

```bash
kubectl port-forward -n monitoring svc/monitoring-kube-prometheus-alertmanager 9093:9093
```

## 7. Explore Grafana dashboards

I used these dashboards during the lab:

| Dashboard | Source |
| --- | --- |
| Kubernetes / Compute Resources / Namespace (Pods) | Bundled with the chart |
| Node Exporter / Nodes | Bundled with the chart |
| Node Exporter Full | Grafana dashboard ID `1860` |
| Kubernetes / Views / Pods | Grafana dashboard ID `15760` |

To import: **Dashboards → New → Import**, enter the ID, select the Prometheus data source, then import.

The dashboards expose CPU, memory, disk and network metrics from the local cluster. Screenshots captured from this lab are in the repository's `screenshots/` directory.

## 8. Optional Grafana ingress from the original exercise

If using the ingress option, save the following as `grafana-ingress.yaml` and apply it to the cluster. This option was provided by the source lab; it was **not needed for my port-forward workflow**.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grafana
  namespace: monitoring
spec:
  ingressClassName: nginx
  rules:
    - host: grafana.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: monitoring-grafana
                port:
                  number: 80
```

It requires the Minikube ingress addon, a matching hosts-file entry and the ingress-controller forwarding described in the source guide.

## 9. Save the active Helm settings

The deployed Helm release holds the actual values used during installation:

```bash
helm get values monitoring -n monitoring -o yaml
```

Inspect and **redact any secrets**, particularly the Grafana password, before publishing an exported file. Do not commit an unreviewed Helm values export.

## 10. Keep the environment

I kept the monitoring stack installed after the exercise. Cleanup commands from the source lab are intentionally not run here because they would remove the monitoring installation.

## Reference

Original lab: https://github.com/Sakshi9111/broadway-labs/tree/main/day12
