# solarwinds-kubernetes-integration

Ensure your Kubernetes environment is set up: 
https://github.com/soultechie/Kubernetes-cluster-setup

## Install Prometheus on your K8s cluster 
Reference: 
```
https://github.com/prometheus-operator/prometheus-operator#quickstart
https://github.com/prometheus-operator/kube-prometheus
https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack
```
Here we will use prometheus-community/kube-prometheus-stack. Make sure to install Helm.
Get Helm Repository Info
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```
Install Helm Chart
```
helm install [RELEASE_NAME] prometheus-community/kube-prometheus-stack
helm install prometheus prometheus-community/kube-prometheus-stack
```
Verify the prometheus pods
```
kubectl get pods --all-namespaces
```
## Access Prometheus 
To access the Prometheus console, you can use either of the following methods:

1) Using a port-forward:

Run the following command to create a port-forward from your local machine to the prometheus-k8s service:
```
kubectl port-forward service/prometheus-k8s 9090:9090 -n monitoring
```
Open your web browser and go to http://localhost:9090 to access the Prometheus console.

2) Using a NodePort service:

Create a new NodePort service to expose the prometheus-k8s service on a public IP address and port:
```
kubectl expose service prometheus-k8s --type=NodePort --name=prometheus-nodeport -n monitoring
```
Get the NodePort allocated for the prometheus-nodeport service:
```
kubectl get service prometheus-nodeport -n monitoring
```
Open your web browser and go to http://public-ip-address:nodeport to access the Prometheus console, where <public-ip-address> is the public IP address of any node in the cluster and <nodeport> is the NodePort allocated for the prometheus-nodeport service.
  
Note: If you are using a cloud-based Kubernetes service such as GKE, EKS, or AKS, you may need to configure firewall rules to allow incoming traffic to the NodePort. Also, make sure to replace monitoring with the actual namespace where the Prometheus server is deployed in case it's different.
  
Navigate to Prometheus console. Go to Status -->  Targets (Check if there are any errors)
Note: Most of the errors are usually related to target endpoints being not reachable from the Prometheus server. 
  
  From the logs above, we can see that kube-controller-manager, kube-scheduler and etcd are listening on localhost(127.0.0.1)
  Edit the config file for these components to bind on non loopback IP addresses(0.0.0.0)
  
  
If there are any issues related to following components: kube-scheduler, kube-controller-manager and etcd, check below config files. 
```
/etc/kubernetes/manifests/kube-scheduler.yaml
/etc/kubernetes/manifests/kube-controller-manager.yaml
/etc/kubernetes/manifests/kube-apiserver.yaml
/etc/kubernetes/manifests/etcd.yaml
```
