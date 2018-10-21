
# Horizontal Pod Autoscaler

The Horizontal Pod Autoscaler automatically scales the number of pods in a replication controller, deployment or replica set based on observed CPU utilization (or, with  [custom metrics](https://git.k8s.io/community/contributors/design-proposals/instrumentation/custom-metrics-api.md)  support, on some other application-provided metrics). Note that Horizontal Pod Autoscaling does not apply to objects that can’t be scaled, for example, DaemonSets.  

## Prerequisites
Make your Amazon EKS cluster is running on plateform version **"eks.2"**.
![eks-plateform-version](https://github.com/KYPan0818/bootcamp-eks/blob/master/images/eks-plateform-version.jpg)

## Setup for example
### Install the metrics-server
git clone and deploy metrics-server
```
~$ git clone https://github.com/kubernetes-incubator/metrics-server
~$ kubectl apply -f metrics-server/deploy/1.8+/
clusterrole.rbac.authorization.k8s.io "system:aggregated-metrics-reader" created
clusterrolebinding.rbac.authorization.k8s.io "metrics-server:system:auth-delegator" created
rolebinding.rbac.authorization.k8s.io "metrics-server-auth-reader" created
apiservice.apiregistration.k8s.io "v1beta1.metrics.k8s.io" created
serviceaccount "metrics-server" created
deployment.extensions "metrics-server" created
service "metrics-server" created
clusterrole.rbac.authorization.k8s.io "system:metrics-server" created
clusterrolebinding.rbac.authorization.k8s.io "system:metrics-server" created
~$ kubectl get po -n kube-system
```
> Confirm metrics-server pods are deployed into namespace 'kube-system'

check `/apis/metrics.k8s.io/v1beta1/nodes`
```
~$ kubectl get --raw "/apis/metrics.k8s.io/v1beta1/nodes" | jq .
{
  "kind": "NodeMetricsList",
  "apiVersion": "metrics.k8s.io/v1beta1",
  "metadata": {
    "selfLink": "/apis/metrics.k8s.io/v1beta1/nodes"
  },
  "items": [...]
}
```
### Create HPA sample
create  sample deployment and service
```
~$ kubectl run php-apache --image=k8s.gcr.io/hpa-example --requests=cpu=200m --expose --port=80
service "php-apache" created
deployment.apps "php-apache" created
~$ kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
deployment.apps "php-apache" autoscaled
~$ kubectl get hpa
NAME         REFERENCE               TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   <unknown>/50%   1         10        0          13s
```
> probably will see \<unknown\>/50% for 1-2 minutes, and then should be able to see 0%/50% like as above
 
### Test it! 
Get into the container and take a pressure on it
```
~$ kubectl run -i --tty load-generator --image=busybox /bin/sh
If you don't see a command prompt, try pressing enter.
/ # while true; do wget -q -O- http://php-apache.default.svc.cluster.local; done
```
![hpa-validate](https://github.com/KYPan0818/bootcamp-eks/blob/master/images/hpa-validate.png)
Open another terminate to validate HPA status
```
~$ kubectl get hpa
```
![hpa-result](https://github.com/KYPan0818/bootcamp-eks/blob/master/images/hpa-result.png)
```
~$ kubectl get deployment php-apache
```
![hpa-result-deploy](https://github.com/KYPan0818/bootcamp-eks/blob/master/images/hpa-result-deploy.png)

## Setup for custom metric

Still WIP...

## Reference
Kubernetes
- [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
- [Horizontal Pod Autoscaler Walkthrough](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)
- [Custom Metrics](		https://github.com/kubernetes/community/blob/master/contributors/design-proposals/instrumentation/custom-metrics-api.md)

AWS Open Source Blog
- [Introducing Horizontal Pod Autoscaling for Amazon EKS](https://aws.amazon.com/blogs/opensource/horizontal-pod-autoscaling-eks/)