# kuby

A simple experiment with Kubernetes, ArgoCD and the Grafana stack.

# Prerequisites

- a Kubernetes cluster
- kubectl 
- helm 
- argocd CLI

# Instructions

## Argo 

```bash
kubectl create namespace argocd

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

kubectl -n argocd  port-forward svc/argocd-server 8081:443 &
```

## Dashboard 

```bash
kubectl -n monitoring port-forward svc/dashboard-kong-proxy 8443:443 &

kubectl create token dashboard-admin
```

## hello-world

```bash 
kubectl -n apps port-forward svc/nginx-hello-world 8082:80 &
```

## Grafana

```bash
kubectl get secret -n monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

kubectl -n monitoring port-forward svc/grafana 3000:80 &
```

## Loki
```bash
 kubectl -n monitoring port-forward svc/loki 3100 &
```

Endpoint for Grafana source : http://loki.monitoring:3100

Push logs :
```bash
curl -H "Content-Type: application/json" \
  -s -X POST "http://localhost:3100/loki/api/v1/push" \
  --data-raw '{"streams": [{ "stream": { "foo": "bar2", "service_name": "test" }, "values": [ [ "'$(date +%s)000000000'", "hello" ] ] }]}'
```

## Access all

```bash
kubectl -n argocd  port-forward svc/argocd-server 8081:443 &
kubectl -n monitoring port-forward svc/dashboard-kong-proxy 8443:443 &
kubectl -n monitoring port-forward svc/grafana 3000:80 &
kubectl -n monitoring port-forward  svc/loki 3100 &
kubectl -n apps port-forward svc/nginx-hello-world 8082:80 &
```

## Debug 

```bash
kubectl run mycurlpod -n monitoring --image=curlimages/curl -i --tty -- sh
nslookup loki.
```