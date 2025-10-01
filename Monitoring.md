## Monitoring: Prometheus & Grafana on RKE2 (Helm) :

</p>
<p align="center">
<img src="https://github.com/ablaamim/Kubernetes-RKE2/blob/main/imgs/grafana-prometheus.webp" width="400">
</p>


📊 kube-prometheus-stack

The kube-prometheus-stack Helm chart provides a full Kubernetes monitoring setup in one package:

Prometheus for metrics storage

Grafana with preloaded dashboards

Alertmanager for notifications

Exporters (node, kube-state, etc.)

🏗️ Production-ready?

Yes ✅ It’s widely used in production clusters because it’s:

📦 Well-maintained & based on Prometheus Operator best practices

🔒 Supports HA, TLS, RBAC

💾 Persistent storage & resource tuning are required for production use

> install and expose Prometheus, Grafana, and Alertmanager on an RKE2 cluster using Helm. It uses the bundled kube-prometheus-stack (Prometheus Operator) and NGINX Ingress with sslip.io hostnames.

Prereqs :

* kubectl connected to the cluster

* helm v3

* Ingress controller : nginx in this cluster

* Public node IP in this example : 157.230.131.228

Steps :

### Add Helm repo & update

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

### Create namespace

```bash
kubectl create namespace monitoring
```

### Install kube-prometheus-stack

```bash
helm upgrade -i kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  -n monitoring
```

> the stack includes Prometheus, Grafana, Alertmanager, kube-state-metrics, node-exporter, etc.
> grafana is pre-wired with a Prometheus datasource and common Kubernetes dashboards.

#### Check status:

```bash
kubectl get pods -n monitoring

OUTPUT :

NAME                                                      READY   STATUS    RESTARTS   AGE
alertmanager-kube-prometheus-stack-alertmanager-0         2/2     Running   0          140m
kube-prometheus-stack-grafana-5d859d9748-sdk57            3/3     Running   0          140m
kube-prometheus-stack-kube-state-metrics-98df7549-lkz5t   1/1     Running   0          150m
kube-prometheus-stack-operator-878674d6d-xm2rg            1/1     Running   0          150m
kube-prometheus-stack-prometheus-node-exporter-k5nwt      1/1     Running   0          150m
kube-prometheus-stack-prometheus-node-exporter-pbw4x      1/1     Running   0          150m
kube-prometheus-stack-prometheus-node-exporter-qf9fb      1/1     Running   0          150m
kube-prometheus-stack-prometheus-node-exporter-z8457      1/1     Running   0          150m
kube-prometheus-stack-prometheus-node-exporter-zzgs6      1/1     Running   0          150m
prometheus-kube-prometheus-stack-prometheus-0             2/2     Running   0          140m
```

### Expose UIs via NGINX Ingress

> Use sslip.io to map hostnames to the node IP: *.157.230.131.228.sslip.io

```bash
helm upgrade -n monitoring kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  --reuse-values \
  --set grafana.service.type=ClusterIP \
  --set grafana.ingress.enabled=true \
  --set grafana.ingress.ingressClassName=nginx \
  --set grafana.ingress.hosts[0]=grafana.157.230.131.228.sslip.io \
  \
  --set prometheus.ingress.enabled=true \
  --set prometheus.ingress.ingressClassName=nginx \
  --set prometheus.ingress.hosts[0]=prometheus.157.230.131.228.sslip.io \
  \
  --set alertmanager.ingress.enabled=true \
  --set alertmanager.ingress.ingressClassName=nginx \
  --set alertmanager.ingress.hosts[0]=alertmanager.157.230.131.228.sslip.io
```

### Verify

``bash
root@ubuntu-s-4vcpu-8gb-240gb-intel-sfo2-01:~# kubectl get ingress -n monitoring
NAME                                 CLASS   HOSTS                                   ADDRESS                                                                     PORTS   AGE
kube-prometheus-stack-alertmanager   nginx   alertmanager.157.230.131.228.sslip.io   138.68.229.21,138.68.29.45,143.110.133.40,157.230.131.228,157.230.143.116   80      142m
kube-prometheus-stack-grafana        nginx   grafana.157.230.131.228.sslip.io        138.68.229.21,138.68.29.45,143.110.133.40,157.230.131.228,157.230.143.116   80      142m
kube-prometheus-stack-prometheus     nginx   prometheus.157.230.131.228.sslip.io     138.68.229.21,138.68.29.45,143.110.133.40,157.230.131.228,157.230.143.116   80      142m

### Verify Longhorn StorageClass

```bash
kubectl get storageclass
```

### Enable persistence for Grafana

```bash
helm upgrade -n monitoring kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  --reuse-values \
  --set grafana.persistence.enabled=true \
  --set grafana.persistence.storageClassName=longhorn \
  --set grafana.persistence.size=10Gi
```

> This creates a PVC for Grafana’s data (dashboards, config)

### Enable persistence for Prometheus

```bash
helm upgrade -n monitoring kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  --reuse-values \
  --set prometheus.prometheusSpec.storageSpec.volumeClaimTemplate.spec.storageClassName=longhorn \
  --set prometheus.prometheusSpec.storageSpec.volumeClaimTemplate.spec.resources.requests.storage=10Gi
```

### Persistence for Alertmanager

```bash
helm upgrade -n monitoring kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  --reuse-values \
  --set alertmanager.alertmanagerSpec.storage.volumeClaimTemplate.spec.storageClassName=longhorn \
  --set alertmanager.alertmanagerSpec.storage.volumeClaimTemplate.spec.resources.requests.storage=5Gi
```

### Verify

```bash
kubectl get pvc -n monitoring
```