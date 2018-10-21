
# Amazon EKS Bootcamp

Amazon EKS Bootcamp walkthrough repository.

This bootcamp provides instructions to create, manage, and scale a Kubernetes cluster on Amazon EKS, and aslo the know-how on Kubernetes cluster.

## Getting Started - Creating EKS Cluster

### eksctl

- [Create your EKS cluster with eksctl via Cloud9](https://github.com/pahud/amazon-eks-workshop/blob/master/00-getting-started/create-eks-with-eksctl.md)

- [Customize your EKS nodegroup](https://github.com/pahud/amazon-eks-workshop/blob/master/01-nodegroup/customize-nodegroup.md)

### Terraform

- [Template with on-demand nodes](https://github.com/KYPan0818/amazon-eks-bootcamp/00-getting-started/examples/terraform)

- [Vishwakarma from getamis](https://github.com/getamis/vishwakarma)

## Monitor

### Cluster Monitoring

- [Cluster Monitoring](https://github.com/KYPan0818/bootcamp-eks/blob/master/04-monitor/)  

## Scaling

### Horizontal Pod Autoscaler

 - [Horizontal Pod Autoscaler (HPA)](https://github.com/KYPan0818/bootcamp-eks/blob/master/05-scaling/hpa.md)

## DevOps

### Continuous Deployment to Kubernetes services using AWS Code-Services

- [AWS Code Pipeline with EKS](https://github.com/ivan-lin1993/eks-codedeploy-demo)

## Authentication between IAM & Kubernetes

### Kubernetes RBAC

Role-based access control (RBAC) is a method of regulating access to computer or network resources based on the roles of individual users within an enterprise.

Reference - [Kubernetes RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

### AWS IAM Authenticator

A tool to use AWS IAM credentials to authenticate to a Kubernetes cluster. The initial work on this tool was driven by Heptio.

Reference - [AWS IAM Authenticator](https://github.com/kubernetes-sigs/aws-iam-authenticator)
  
## Reference

- Pahud - [**amazon-eks-workshop**](https://github.com/pahud/amazon-eks-workshop)

- AWS Sample - [**aws-workshop-for-kubernetes**](https://github.com/aws-samples/aws-workshop-for-kubernetes)