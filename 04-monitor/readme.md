# EKS Cluster Monitoring

## Introduction
This provides demonstrate how to monitor a Kubernetes cluster 

1.  Kubernetes Dashboard
2.  Heapster
3.  Prometheus
4. CNI Metric Monitor 
5. Datadog - WIP

## Kubernetes Dashboard
Kubernetes Dashboard is a general purpose, web-based UI for Kubernetes clusters. It allows users to manage applications running in the cluster and troubleshoot them, as well as manage the cluster itself.
Reference - [Kubernetes Dashboard](https://github.com/kubernetes/dashboard)

### Getting Started
Deploy Dashboard
```
~$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
secret "kubernetes-dashboard-certs" created
serviceaccount "kubernetes-dashboard" created
role.rbac.authorization.k8s.io "kubernetes-dashboard-minimal" created
rolebinding.rbac.authorization.k8s.io "kubernetes-dashboard-minimal" created
deployment.apps "kubernetes-dashboard" created
service "kubernetes-dashboard" created
```
Deploy heapster to enable container cluster monitoring and performance analysis on your cluster
```
~$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/influxdb/heapster.yaml
serviceaccount "heapster" created
deployment.extensions "heapster" created
service "heapster" created
```
Deploy the influxdb backend for heapster to your cluster
```
~$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/influxdb/influxdb.yaml
deployment.extensions "monitoring-influxdb" created
service "monitoring-influxdb" created
```
Create the heapster cluster role binding for the dashboard
```
~$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/rbac/heapster-rbac.yaml
clusterrolebinding.rbac.authorization.k8s.io "heapster" created
```
Create a service account
```
~$ kubectl apply -f example/dashboard/dashboard_eks-admin-service-account.yaml
serviceaccount "eks-admin" created
```
Apply the cluster role binding to your cluster
```
~$ kubectl apply -f example/dashboard/dashboard_eks-admin-cluster-role-binding.yaml
clusterrolebinding.rbac.authorization.k8s.io "eks-admin" created
```
Retrieve an _`<authentication_token>`_ for the `eks-admin` service account
```					
~$ kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep eks-admin | awk '{print $1}')
```
![dashboard-token](https://github.com/KYPan0818/bootcamp-eks/blob/master/images/dashboard-token.jpg)
Start the **kubectl proxy** to access dashboard via Cloud9 outsource port 8080
```
~$ kubectl proxy --address 0.0.0.0 --accept-hosts '.*' --port 8080
Starting to serve on [::]:8080...

```
Open the following link with a web browser to access the dashboard endpoint
`
https://ENVIRONMENT_ID.vfs.cloud9.REGION_ID.amazonaws.com/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/
`
> You can get the cloud9 env URL by clicking "Preview" on the topside negative bar 

Choose **Token**, paste the _`<authentication_token>`_ output from the previous step into the **Token** field, and choose **SIGN IN**
![dashboard-login](https://github.com/KYPan0818/bootcamp-eks/blob/master/images/dashboard-login.png)

Welcome to Kubernetes Dashboard
![dashboard-console](https://github.com/KYPan0818/bootcamp-eks/blob/master/images/dashboard-console.png)

## Heapster 
[Heapster](https://github.com/kubernetes/heapster) is a metrics aggregator and processor. It is installed as a cluster-wide pod. It gathers monitoring and events data for all containers on each node by talking to the Kubelet. Kubelet itself fetches this data from [cAdvisor](https://github.com/google/cadvisor). This data is persisted in a time series database [InfluxDB](https://github.com/influxdata/influxdb) for storage. The data is then visualized using a [Grafana](http://grafana.org/) dashboard, or can be viewed in Kubernetes Dashboard.

### Installation
Deploy Heapster, InfluxDB and Grafana dashboard from templates
> Templates refer form *[Heapster, InfluxDB and Grafana](https://github.com/aws-samples/aws-workshop-for-kubernetes/tree/master/02-path-working-with-clusters/201-cluster-monitoring/templates/heapster)*

```
~$ kubectl apply -f example/heapster/
deployment.apps "monitoring-grafana" created
service "monitoring-grafana" created
clusterrolebinding.rbac.authorization.k8s.io "heapster" created
serviceaccount "heapster" created
deployment.apps "heapster" created
service "heapster" created
deployment.apps "monitoring-influxdb" created
service "monitoring-influxdb" created
```

Start the **kubectl proxy** to access dashboard via Cloud9 outsource port 8080
```
~$ kubectl proxy --address 0.0.0.0 --accept-hosts '.*' --port 8080
Starting to serve on [::]:8080...

```
Open the following link with a web browser to access Grafana dashboard
`
https://ENVIRONMENT_ID.vfs.cloud9.REGION_ID.amazonaws.com/api/v1/namespaces/kube-system/services/monitoring-grafana/proxy/?orgId=1
`
> You can get the cloud9 env URL by clicking "Preview" on the topside negative bar

### Grafana Dashboard
On the Grafana home page, change the dashboard to _`Cluster`_
![heapster-grafana-home](https://github.com/KYPan0818/bootcamp-eks/blob/master/images/heapster-grafana-home.jpg)
The cluster dashboard looks like below
![heapster-grafana](https://github.com/KYPan0818/bootcamp-eks/blob/master/images/heapster-grafana.png)
You can also see the resource utilization of every pod in the cluster on the _`Pods`_ dashboard

After the deployment of Heapster, Kubernetes Dashboard now shows additional graphs such as CPU and Memory utilization for pods and nodes, and other workloads like below
![heapster-dashboard-updated](https://github.com/KYPan0818/bootcamp-eks/blob/master/images/heapster-dashboard-updated.png)
> The same URL link on Kubernetes dashboard section

### Cleanup Heapster
Remove installed components
```	
kubectl delete -f example/heapster/
```

## Prometheus
[Prometheus](http://prometheus.io/) is an open-source systems monitoring and alerting toolkit. Prometheus collects metrics from monitored targets by scraping metrics from HTTP endpoints on these targets.
Prometheus will be managed by the  [Kubernetes Operator](https://github.com/coreos/prometheus-operator/)  - This operator uses  [Custom Resources](https://kubernetes.io/docs/concepts/api-extension/custom-resources/)  to extend the Kubernetes API and add custom resources such as  `Prometheus`,  `ServiceMonitor`  and  `Alertmanager`.
Prometheus is able to dynamically scrape new targets by adding a  [ServiceMonitor](https://github.com/coreos/prometheus-operator/blob/master/Documentation/user-guides/running-exporters.md)  - we have included a couple of them to scrape  `kube-controller-manager`,  `kube-scheduler`,  `kube-state-metrics`,  `kubelet`  and  `node-exporter`.
[Node exporter](https://github.com/prometheus/node_exporter)  is a Prometheus exporter for hardware and OS metrics exposed by *NIX kernels.  [kube-state-metrics](https://github.com/kubernetes/kube-state-metrics)  is a simple service that listens to the Kubernetes API server and generates metrics about the state of the objects.

### Installation
Deploy Prometheus, Node exporter and Grafana dashboard from templates, and wait until the Prometheus Operator has started
> Templates refer form *[Prometheus, Node exporter and Grafana](https://github.com/aws-samples/aws-workshop-for-kubernetes/tree/master/02-path-working-with-clusters/201-cluster-monitoring/templates/prometheus)*

Deploy Prometheus Operator
```
~$ kubectl apply -f example/prometheus/prometheus-bundle.yaml
namespace "monitoring" created
clusterrolebinding.rbac.authorization.k8s.io "prometheus-operator" created
clusterrole.rbac.authorization.k8s.io "prometheus-operator" created
serviceaccount "prometheus-operator" created
deployment.apps "prometheus-operator" created
~$ kubectl rollout status deployment/prometheus-operator -n monitoring
deployment "prometheus-operator" successfully rolled out
```

Deploy Prometheus Custom Resource, Service Monitors, Cluster Roles and Bindings (RBAC)
```
~$ kubectl apply -f example/prometheus/prometheus.yaml
serviceaccount "kube-state-metrics" created
clusterrole.rbac.authorization.k8s.io "kube-state-metrics" created
clusterrolebinding.rbac.authorization.k8s.io "kube-state-metrics" created
service "kube-scheduler-prometheus-discovery" created
service "kube-controller-manager-prometheus-discovery" created
daemonset.apps "node-exporter" created
service "node-exporter" created
deployment.apps "kube-state-metrics" created
service "kube-state-metrics" created
prometheus.monitoring.coreos.com "prometheus" created
servicemonitor.monitoring.coreos.com "prometheus-operator" created
servicemonitor.monitoring.coreos.com "kube-apiserver" created
servicemonitor.monitoring.coreos.com "kubelet" created
servicemonitor.monitoring.coreos.com "kube-controller-manager" created
servicemonitor.monitoring.coreos.com "kube-scheduler" created
servicemonitor.monitoring.coreos.com "kube-state-metrics" created
servicemonitor.monitoring.coreos.com "node-exporter" created
alertmanager.monitoring.coreos.com "main" created
secret "alertmanager-main" created
~$ kubectl get po -n monitoring
NAME                                   READY     STATUS    RESTARTS   AGE
alertmanager-main-0                    2/2       Running   0          2m
alertmanager-main-1                    2/2       Running   0          1m
alertmanager-main-2                    2/2       Running   0          1m
kube-state-metrics-668c4cddfb-p4425    2/2       Running   0          2m
node-exporter-c8cls                    1/1       Running   0          2m
node-exporter-jckzp                    1/1       Running   0          2m
node-exporter-tmgkt                    1/1       Running   0          2m
node-exporter-vhq7m                    1/1       Running   0          2m
prometheus-operator-66f55f7f9f-zgch4   1/1       Running   0          6m
prometheus-prometheus-0                2/2       Running   0          2m
prometheus-prometheus-1                2/2       Running   0          2m
```
### Prometheus Dashboard
Prometheus is now scraping metrics from the different scraping targets
```
~$ kubectl port-forward $(kubectl get po -l prometheus=prometheus -n monitoring -o jsonpath={.items[0].metadata.name}) 8080:9090 -n monitoring
```
All targets should be shown as `UP` by accessing 
`
https://ENVIRONMENT_ID.vfs.cloud9.REGION_ID.amazonaws.com/targets
`
![prometheus-dashboard-targets](https://github.com/KYPan0818/bootcamp-eks/blob/master/images/prometheus-dashboard-targets.png)

### Setup for Grafana

Install Grafana 
```
~$ kubectl apply -f example/prometheus/grafana-bundle.yaml  
secret "grafana-credentials" created
service "grafana" created
configmap "grafana-dashboards-0" created
deployment.apps "grafana" created
~$ kubectl rollout status deployment/grafana -n monitoring
deployment "grafana" successfully rolled out
```

### Grafana Dashboard
Start the **kubectl port-forward** to access Grafana dashboard via Cloud9 outsource port 8080
```
~$ kubectl port-forward $(kubectl get pod -l app=grafana -o jsonpath={.items[0].metadata.name} -n monitoring) 8080:3000 -n monitoring
Forwarding from 127.0.0.1:8080 -> 3000
Forwarding from [::1]:8080 -> 3000

```
Open the following link with a web browser to access Grafana dashboard
`
https://ENVIRONMENT_ID.vfs.cloud9.REGION_ID.amazonaws.com/
`
> You can get the cloud9 env URL by clicking "Preview" on the topside negative bar

The complete list of Grafana dashboards is available using the search button at the top, and access various metrics using these dashboards
![prometheus-grafana-dashboard](https://github.com/KYPan0818/bootcamp-eks/blob/master/images/prometheus-grafana-dashboard.png)

### Cleanup Prometheus
Remove installed components
```
kubectl delete -f example/prometheus/grafana-bundle.yaml
kubectl delete -f example/prometheus/prometheus.yaml
kubectl delete -f example/prometheus/prometheus-bundle.yaml
```

## CNI Metrics Helper
The Amazon Elastic Container Service for Kubernetes ([EKS](https://aws.amazon.com/eks/)) uses the [VPC CNI plugin](https://aws.amazon.com/blogs/opensource/vpc-cni-plugin-v1-1-available/) for pod networking. The plugin runs as a DaemonSet and is responsible for assigning an IP address to pods.

### Setup steps
- AWS Open Source Blog - [CNI Metrics Helper](https://aws.amazon.com/tw/blogs/opensource/cni-metrics-helper/)

## Datadog
[Datadog](https://www.datadoghq.com/) is a monitoring service for cloud-scale applications, providing monitoring of servers, databases, tools, and services, through a SaaS-based data analytics platform. It gives a unified view of an entire stack, allowing to seamlessly monitor metrics, application traces as well as logs.

### *Setup - WIP*

## Reference

aws-workshop-for-kubernetes 

- [201-cluster-monitoring](https://github.com/aws-samples/aws-workshop-for-kubernetes/tree/master/02-path-working-with-clusters/201-cluster-monitoring)
- [207-cluster-monitoring-with-datadog](https://github.com/aws-samples/aws-workshop-for-kubernetes/tree/master/02-path-working-with-clusters/207-cluster-monitoring-with-datadog)

AWS EKS documentation
- [dashboard-tutorial](https://docs.aws.amazon.com/eks/latest/userguide/dashboard-tutorial.html)