# Introduction 
Repositório que contêm toda a lógica do Kubernetes necessária para implantar e gerenciar os aplicativos em contêineres.

# Sistemas suportados
- Kubernetes
- ArgoCD
- Prometheus
- Grafana
- Airbyte
- Airflow
- Open Metadata

# Pré-requisitos
- [kubectl CLI](https://kubernetes.io/docs/tasks/tools/#kubectl)
- [kubectx](https://github.com/ahmetb/kubectx#installation)
- [helm](https://helm.sh/docs/intro/install/)
- [Argo CD]()

## Create namespaces
```bash
kubectl create namespace monitoring
kubectl create namespace cicd
kubectl create namespace ingestion-batch
kubectl create namespace orchestrator
```
## Add & update helm list repos
```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo add airbyte https://airbytehq.github.io/helm-charts
helm repo add apache-airflow https://airflow.apache.org

helm repo update
```

# Argo-cd
## install argo-cd
- [repo](https://artifacthub.io/packages/helm/argo/argo-cd)
- [alternative repo](https://github.com/argoproj/argo-helm)
```bash
helm install argocd argo/argo-cd --namespace cicd --version 3.26.8
```

## create a load balancer to argocd
```bash
kubectl patch svc argocd-server -n cicd -p '{"spec": {"type": "LoadBalancer"}}'
```

## retrieve load balancer ip
```bash
kubectl get services -l app.kubernetes.io/name=argocd-server,app.kubernetes.io/instance=argocd -o jsonpath="{.items[0].status.loadBalancer.ingress[0].ip}" -n cicd
```

## get password to log into argocd portal
```bash
ARGOCD_LB=<load balancer ip>
kubectl get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" -n cicd | base64 -d | xargs -t -I {} argocd login $ARGOCD_LB --username admin --password {} --insecure
```

## create cluster role binding for admin user [sa]
```bash
kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=system:serviceaccount:cicd:argocd-application-controller -n cicd
```

## register cluster
```bash
CLUSTER="cluster"
argocd cluster add $CLUSTER --in-cluster
```
## add repo into argo-cd repositories
```bash
REPOSITORY="http://..."
argocd repo add $REPOSITORY --username username --password password --port-forward
```


## get user and password to access grafana
```bash
kubectl get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" -n cicd | base64 -d; echo
```

# Aibyte
- [Documentation](https://docs.airbyte.com/deploying-airbyte/on-kubernetes-via-helm)
- [Repo](https://airbytehq.github.io/helm-charts)




# Remove old version of the chart and install the latest version:
```bash 
rm ./repository/helm-charts/airflow -rf |
helm pull airflow/airflow -d ./repository/helm-charts --untar
```
```yaml
    service:
        type: LoadBalancer
        port: 80
        annotations: {}
```
# Airbyte:
```bash
kubectl apply -f repository/app-manifests/ingestion-batch/airbyte.yaml
```

# Airflow
```bash
kubectl apply -f repository/app-manifests/orchestrator/airflow.yaml
```
