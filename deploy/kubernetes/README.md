# Installing sock-shop on Kubernetes

See the [documentation](https://microservices-demo.github.io/deployment/kubernetes-minikube.html) on how to deploy Sock Shop using Minikube.

## Kubernestes manifests

There are 2 sets of manifests for deploying Sock Shop on Kubernetes: one in the [manifests directory](manifests/), and complete-demo.yaml. The complete-demo.yaml is a single file manifest
made by concatenating all the manifests from the manifests directory, so please regenerate it when changing files in the manifests directory.

## Monitoring

All monitoring is performed by prometheus. All services expose a `/metrics` endpoint. All services have a Prometheus Histogram called `request_duration_seconds`, which is automatically appended to create the metrics `_count`, `_sum` and `_bucket`.

The manifests for the monitoring are spread across the [manifests-monitoring](./manifests-monitoring) and [manifests-alerting](./manifests-alerting/) directories.

To use them, please run `kubectl create -f <path to directory>`.

### What's Included?

* Sock-shop grafana dashboards
* Alertmanager with 500 alert connected to slack
* Prometheus with config to scrape all k8s pods, connected to local alertmanager.

### Ports

Grafana will be exposed on the NodePort `31300` and Prometheus is exposed on `31090`. If running on a real cluster, the easiest way to connect to these ports is by port forwarding in a ssh command:
```
ssh -i $KEY -L 3000:$NODE_IN_CLUSTER:31300 -L 9090:$NODE_IN_CLUSTER:31090 ubuntu@$BASTION_IP
```
Where all the pertinent information should be entered. Grafana and Prometheus will be available on `http://localhost:3000` or `:9090`.

If on Minikube, you can connect via the VM IP address and the NodePort.


## Kubernetes 1.17.3
    cd microservices-demo/deploy/kubernetes 
    kubectl create namespace sock-shop
    kubectl convert -f . | kubectl create -f -
    watch kubectl get pods -n sock-shop
### wait for all in status Running
    kubectl describe svc front-end -n sock-shop

## [Minikube](https://kubernetes.io/docs/setup/learning-environment/minikube/)

     minikube start --kubernetes-version v1.15.10 --vm-driver=virtualbox

#### Adding cluster to gitlab
http://gitlab.com/help/user/project/clusters/index.md#adding-an-existing-kubernetes-cluster

1. Proxy
    kubectl proxy --accept-hosts='.*' --address='192.168.1.86'

### Gitlab Requirement
2. Create ClusterRole
apiVersion: v1
kind: ServiceAccount
metadata:
  name: gitlab-admin
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: gitlab-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: gitlab-admin
  namespace: kube-system

    
3. Get certificate
    kubectl get secret
    
    kubectl get secret <secret name> -o jsonpath="{['data']['ca\.crt']}" | base64 --decode
    
4. Get token
    kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep gitlab-admin | awk '{print $1}')
    
5. Create private docker registry credentials inside kubernetes cluster
    kubectl create secret docker-registry regcred --docker-server=registry.gitlab.com --docker-username=tomasz.szuster@billennium.com --docker-password=-V3i9V1DuT-8dz6RXxmr --docker-email=tomasz.szuster@billennium.com