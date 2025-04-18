# ELK Stack (Elasticsearch, Filebeat, Kibana) to Collect Logs from Applications On Kubernetes Cluster
In this video, we use the ELK Stack to collect logs of applications deployed on a Kubernetes cluster. 


## Requirements
1. Docker Desktop (on Mac & Windows) or Docker Engine (on Linux)
2. Kubectl
3. Minikube

## Check Requirements
```sh 
docker --version
```
```sh
kubectl version --client
```
```sh
minikube version
```

## Start Minikube Cluster
```sh
minikube start --cpus=4 --memory=4096 --driver=docker
```
```sh
kubectl get nodes
```

## Deploy Sample Applications in demo-app Namespace
```sh
kubectl create namespace demo-apps
```
```sh
kubectl apply -f app1.yaml
```
```sh
kubectl apply -f app2.yaml
```

## Deploy Elasticsearch in logging Namespace
```sh
kubectl create namespace logging
```
```sh
kubectl apply -f elasticsearch-updated.yaml
```

### Verify Elasticsearch Pod & PVC
```sh
kubectl get pods -n logging
```
```sh
kubectl get pvc -n logging
```
```sh
kubectl get pv -n logging
```

## Deploy Kibana in logging Namespace
```sh
kubectl apply -f kibana.yaml
```

### Verify Kibana 
```sh
kubectl get pods -n logging
```

### Expose Kibana UI with Minikube Tunnel (Open URL in browser)
```sh
minikube service kibana -n logging --url
```
 
## Deploy Filebeat as a DaemonSet
### Download Filebeat Kubernetes Manifest from Elastic
```sh
curl -L -O https://raw.githubusercontent.com/elastic/beats/7.17/deploy/kubernetes/filebeat-kubernetes.yaml
```

### Modify filebeat-kubernetes.yaml file 
1. Update ELASTICSEARCH_HOST with = http://elasticsearch.logging.svc.cluster.local:9200
2. Add the namespace "demo-apps" to the Filebeat pod annotations if you want namespace-specific logs.
3. But Filebeat is running as a DaemonSet to it has access to all pods across all namespaces via its ClusterRole.

### Deploy Filebeat
```sh
kubectl apply -f filebeat-kubernetes-updated.yaml
```

### On Kibana
1. Explore on My Own
2. Click Home Left Panel
3. Go to Stack Management
4. Click Index Patterns - create an index pattern name e.g: filebeat-*
5. Select @timestamp in Timestamp field
6. Click create index pattern.
7. Go to Discover on the left panel of homepage to see logs from app1 and app2.


### Verify the Logs from the apps
```sh
kubectl logs app1 -n demo-apps | tail
```
```sh
kubectl logs app2 -n demo-apps | head
```

### Verify it from Kibana UI
1. On the left panel (under Available fields)
2. Scroll down to the bottom to see e.g. log message, log.file.path, etc. 
3. Click to examine them.


## Now, Deploy and Log an NGINX application
```sh
kubectl apply -f nginx-deployment.yaml
```

### Expose Nginx with Minikube Tunnel (Open URL in browser)
```sh
minikube service nginx-service -n demo-apps --url
```

### Create some Filters in Kibana to examine Nginx logs
1. Add filter
2. Field = kubernetes.labels.app, Operator = is, Value = nginx & Save. (You may have to change timestamp next to the "Refresh button" to see some logs)



## Clean UP
```sh
kubectl delete ns logging
```
```sh
kubectl delete ns demo-apps
```
```sh
minikube stop
```
```sh
minikube delete --all
```
