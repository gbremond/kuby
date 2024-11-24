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

kubectl port-forward svc/argocd-server -n argocd 8081:443 &
```

## Dashboard 

```bash
kubectl -n monitoring port-forward svc/dashboard-kong-proxy 8443:443 &

kubectl -n default create token dashboard-admin
```

## Deploy hello world
```bash
helm repo add examples https://helm.github.io/examples

helm install ahoy examples/hello-world
```

## Access it

```bash 
export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=hello-world,app.kubernetes.io/instance=ahoy" -o jsonpath="{.items[0].metadata.name}")
export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
echo "Visit http://127.0.0.1:8080 to use your application"
kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT
```

## Alloy

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

helm install --namespace monitoring alloy grafana/alloy  -f monitoring/alloy/values.yaml
```

## Loki
```bash
helm install --namespace monitoring loki grafana/loki -f monitoring/loki/values.yaml
```
Send logs to : http://loki-gateway.monitoring.svc.cluster.local/loki/api/v1/push

Grafana source : http://loki-gateway.monitoring.svc.cluster.local/

## Grafana

```bash
helm install --namespace monitoring grafana grafana/grafana 

kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
export POD_NAME=$(kubectl get pods --namespace monitoring -l "app.kubernetes.io/name=grafana,app.kubernetes.io/instance=grafana" -o jsonpath="{.items[0].metadata.name}")
kubectl --namespace monitoring port-forward $POD_NAME 3000
```