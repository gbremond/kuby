# 🎲 kuby

A simple experiment with Kubernetes, ArgoCD, and the Grafana stack.

---
## 📋 Prerequisites

Before you start, ensure you have:

- ✅ a Kubernetes cluster
- ✅ `kubectl` CLI installed
- ✅ `argocd` CLI installed

---

## 📦 Instructions

### 1️⃣ Install Argo CD

Run the following commands to set up Argo CD:

```bash
kubectl create namespace argocd

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

argocd admin initial-password -n argocd
```

Create application referencing your repository.

### 2️⃣ Access All Services

Use the following commands to port-forward services for local access:

```bash
kubectl -n argocd port-forward svc/argocd-server 8081:443 &
kubectl -n monitoring port-forward svc/dashboard-kong-proxy 8443:443 &
kubectl -n monitoring port-forward svc/grafana 3000:80 &
kubectl -n monitoring port-forward svc/loki 3100 &
kubectl -n apps port-forward svc/nginx-hello-world 8082:80 &
```

### 🔐 Secrets for ArgoCD, Dashboard and Grafana 

Retrieve the required tokens and passwords:

```bash
argocd admin initial-password -n argocd
kubectl create token dashboard-admin
kubectl get secret -n monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

### 📊 Logs (Loki)
- **Grafana Source Endpoint:** http://loki.monitoring:3100

Manually push logs to Loki:

```bash
curl -H "Content-Type: application/json" \
  -s -X POST "http://localhost:3100/loki/api/v1/push" \
  --data-raw '{"streams": [{ "stream": { "foo": "bar2", "service_name": "test" }, "values": [ [ "'$(date +%s)000000000'", "hello" ] ] }]}'
```

### 🛠️ Debugging

Run a temporary pod for debugging network connections:

```bash
kubectl run mycurlpod -n monitoring --image=curlimages/curl -i --tty -- sh
nslookup loki.monitoring
```

---

##  📝 Todo

- [x] Expose without port-forwarding :
  - [x] Argo CD
  - [x] Dashboard UI
  - [x] Grafana
  - [x] nginx
- [x] Experiment Traefik 
  - `kubectl port-forward $(kubectl get pods --selector "app.kubernetes.io/name=traefik" --output=name) 8080:8080`
  - http://localhost:8080/dashboard/
- [ ] Add more monitoring :
  - ~~[ ] Mimir~~ (my computer isn't strong enough to start all the pods 🙁)
  - [ ] Prometheus
  - [ ] Tempo
- [ ] Replace Kube dashboard with kubewall
  - https://github.com/kubewall/kubewall
- [ ] Experiment with BookInfo App or demo : https://opentelemetry.io/docs/demo/kubernetes-deployment/
- [ ] Add more apps :
  - [ ] JS HTTP server
  - [ ] Rust HTTP server
