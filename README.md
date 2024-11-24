# kuby

A simple experiment with Kubernetes, ArgoCD and the Grafana stack.

# Prerequisites

- a Kubernetes cluster
- kubectl 
- argocd CLI

# Instructions

## Install Argo CD 

```bash
kubectl create namespace argocd

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Create application referencing your repository.

## Access all

```bash
kubectl -n argocd port-forward svc/argocd-server 8081:443 &
kubectl -n monitoring port-forward svc/dashboard-kong-proxy 8443:443 &
kubectl -n monitoring port-forward svc/grafana 3000 &
kubectl -n monitoring port-forward svc/loki 3100 &
kubectl -n apps port-forward svc/nginx-hello-world 8082 &
```

Get secrets for Dashboard and Grafana :
```bash
kubectl create token dashboard-admin
kubectl get secret -n monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

### Logs (Loki)
Endpoint for Grafana source : http://loki.monitoring:3100

Push logs manually :
```bash
curl -H "Content-Type: application/json" \
  -s -X POST "http://localhost:3100/loki/api/v1/push" \
  --data-raw '{"streams": [{ "stream": { "foo": "bar2", "service_name": "test" }, "values": [ [ "'$(date +%s)000000000'", "hello" ] ] }]}'
```

### Debug 

```bash
kubectl run mycurlpod -n monitoring --image=curlimages/curl -i --tty -- sh
nslookup loki.monitoring
```