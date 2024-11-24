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
kubectl -n apps port-forward svc/nginx-hello-world 8082:80
```

## Grafana

```bash
kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

kubectl -n monitoring port-forward svc/grafana 3000:80 &
```

## Loki
```bash
helm install --namespace monitoring loki grafana/loki -f monitoring/loki/values_old.yaml
```
Send logs to : http://loki-gateway.monitoring.svc.cluster.local/loki/api/v1/push

Grafana source : http://loki-gateway.monitoring.svc.cluster.local/

