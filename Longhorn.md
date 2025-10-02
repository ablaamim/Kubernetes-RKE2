# Longhorn

</p>
<p align="center">
<img src="https://github.com/ablaamim/Kubernetes-RKE2/blob/main/imgs/longhorn.webp" width="400">
</p>


>  Longhorn provides cloud native and highly available persistent block storage for Kubernetes, with backups and disaster recovery.

### Let's add the Helm Repository for Longhorn!

```bash
helm repo add longhorn https://charts.longhorn.io
helm repo update
```

### Now let's install Longhorn with the following commands:

```bash
# server(s): rke2-cp-01
# Create the Longhorn Namespace and Install Longhorn
kubectl create namespace longhorn-system

helm upgrade -i longhorn longhorn/longhorn --namespace longhorn-system --set ingress.enabled=true --set ingress.host=longhorn.165.22.133.131.sslip.io

# Wait for the deployment and rollout
sleep 30

# Verify the status of Longhorn
kubectl get pods --namespace longhorn-system
```

### Exploring Rancher Longhorn

> Once all the pods show as Running in the longhorn-system namespace, you can access Rancher Longhorn! Just like the Rancher Manager, we are utilizing sslip.io, so there is no additional configuration required to access Longhorn. Let's head over to the domain name.


### Add longhorn monitoring service

```bash
helm upgrade -n longhorn-system longhorn longhorn/longhorn \
  --reuse-values \
  --set metrics.serviceMonitor.enabled=true
```

### add label

```bash
kubectl patch servicemonitor longhorn-prometheus-servicemonitor \
  -n longhorn-system \
  --type merge \
  -p '{"metadata":{"labels":{"release":"kube-prometheus-stack"}}}'

```