# solarwinds-kubernetes-integration

Ensure your Kubernetes environment is set up: 
https://github.com/ObserveX/Kubernetes-cluster-setup

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

### 1) Using a port-forward:

Run the following command to create a port-forward from your local machine to the prometheus-k8s service:
```
kubectl port-forward service/prometheus-k8s 9090:9090 -n monitoring
```
Open your web browser and go to http://localhost:9090 to access the Prometheus console.

### 2) Using a NodePort service:

Create a new NodePort service to expose the prometheus-k8s service on a public IP address and port:
```
kubectl expose service prometheus-k8s --type=NodePort --name=prometheus-nodeport -n monitoring
```
Get the NodePort allocated for the prometheus-nodeport service:
```
kubectl get service prometheus-nodeport -n monitoring
```
Open your web browser and go to http://public-ip-address:nodeport to access the Prometheus console
  
Note: If you are using a cloud-based Kubernetes service such as GKE, EKS, or AKS, you may need to configure firewall rules to allow incoming traffic to the NodePort. Also, make sure to replace monitoring with the actual namespace where the Prometheus server is deployed in case it's different.

## Troubleshooting Prometheus Errors
Navigate to Prometheus console. Go to Status -->  Targets (Check if there are any errors)
Note: Most of the errors are usually related to target endpoints being not reachable from the Prometheus server. 

Check logs for the pod having issues
```
kubectl logs <pod-name> -n <namespace>
```
Where pod-name is the name of the kube-controller-manager pod, which can be obtained from the output of kubectl get pods -n namespace

From the logs above, we can see that kube-controller-manager, kube-scheduler and etcd are listening on localhost(127.0.0.1)
Edit the config file for these components to bind on non loopback IP addresses(0.0.0.0)
  
check below config files. 
```
/etc/kubernetes/manifests/kube-scheduler.yaml
/etc/kubernetes/manifests/kube-controller-manager.yaml
/etc/kubernetes/manifests/kube-apiserver.yaml
/etc/kubernetes/manifests/etcd.yaml
```
## Add Kubernestes Cluster into SolarWinds Monitoring

1) Copy or create an API token (Ingestion type), found in the settings area of SolarWinds Observability. 

2) Create a kubectl secret with the following command, replacing <YourK8sNamespace> with the namespace of your Kubernetes cluster and <YourApiToken> with the API token from the previous step.
```
kubectl create secret generic solarwinds-api-token -n <YourK8sNamespace> --from-literal=SOLARWINDS_API_TOKEN=<YourApiToken>
```
Create a values.yaml file with the following helm values, 
```
otel:
  endpoint: <YourOTelEndpoint>:443
  metrics:
    prometheus: 
      url: <YourPrometheusURL>
cluster:
  name: <YourClusterName>
  uid: <YourUniqueId>
```
replacing:
```
<YourOTelEndpoint> with your organization's OTel endpoint
<YourPrometheusURL> with the URL of the Prometheus server
<YourUniqueId> with a unique ID that follows the following format: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
<YourClusterName> with the display name for the cluster entity in SolarWinds Observability
```
Deploy the helm chart with the values.yaml file with the following command, replacing <YourK8sNamespace> with the namespace of your Kubernetes cluster.
```
helm repo add solarwinds https://helm.solarwinds.com && helm install -f values.yaml swo-k8s-collector solarwinds/swo-k8s-collector --namespace <YourK8sNamespace> --create-namespace
```
Once collected K8s data is sent to SolarWinds Observability, Kubernetes cluster and Kubernetes node entities are created and available in the Entity Explorer and Kubernetes Overview.
  
## More on SolarWinds k8s OTEL Configuration (Still to explore)
https://github.com/solarwinds/swi-k8s-opentelemetry-collector
