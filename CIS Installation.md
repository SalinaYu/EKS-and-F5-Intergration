# CIS Installation  
## Overview  
CIS can be configured in multiple ways depending on the customer scenario.  

CIS can be deployed on Kubernetes platform.  

In this project, we are deploying CIS on Kubernetes platform using EKS (Elastic Kubernetes Service) on AWS and have standalone BIG-IP running in the same Amazon VPC (Virtual Private Cloud). CIS is utilizing ConfigMap to configure the BIG-IP.  

---

## Prerequisites  
There are the mandatory requirements for deploying CIS:
<br/>
- Kubernetes Cluster must be up and running.
- You must have a fully active/licensed BIG-IP.
- AS3: 3.18+ must be installed on the BIG-IP system.
- You need the credentials set, username and password, for the BIG-IP.
<br/>

---

## Installing CIS
There are 4 steps to install CIS **after** you have complete the prerequisites above:  
1. Create K8S Secrets.
2. Create K8S Service Account.
3. Create K8S Cluster Role and Cluster Role Binding.
4. Create the K8S CIS Deployment.

Here are the steps that you need to take with sample scripts and example yaml files:

1\. Add BIG-IP Credentials as K8S Secrets. Replace `<username>` and `<password>` with your own credentials. 
```
kubectl create secret generic bigip-login -n kube-system --from-literal=username=<username> --from-literal=password=<password>
```
2\. Create a service account for deploying CIS. In the example below, the Service Account is named `bigip-ctlr`.
```
kubectl create serviceaccount bigip-ctlr -n kube-system
```
3\. Create a Cluster Role and Cluster Role Binding on the Kubernetes Cluster using the examples below.

The example below shows the broadest supported permission set. You can narrow the permissions down to specific resources, namespaces, etc. to suit your needs. See the [Kubernetes RBAC documentation](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) for more information.

```yaml
# for use in k8s clusters only
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: bigip-ctlr-clusterrole
rules:
- apiGroups: ["", "extensions", "networking.k8s.io"]
  resources: ["nodes", "services", "endpoints", "namespaces", "ingresses", "pods", "ingressclasses"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["", "extensions", "networking.k8s.io"]
  resources: ["configmaps", "events", "ingresses/status", "services/status"]
  verbs: ["get", "list", "watch", "update", "create", "patch"]
- apiGroups: ["cis.f5.com"]
  resources: ["virtualservers","virtualservers/status", "tlsprofiles", "transportservers", "ingresslinks", "externaldnss"]
  verbs: ["get", "list", "watch", "update", "patch"]
- apiGroups: ["fic.f5.com"]
  resources: ["f5ipams", "f5ipams/status"]
  verbs: ["get", "list", "watch", "update", "create", "patch", "delete"]
- apiGroups: ["apiextensions.k8s.io"]
  resources: ["customresourcedefinitions"]
  verbs: ["get", "list", "watch", "update", "create", "patch"]
- apiGroups: ["", "extensions"]
  resources: ["secrets"]
  verbs: ["get", "list", "watch"]


---

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: bigip-ctlr-clusterrole-binding
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: bigip-ctlr-clusterrole
subjects:
- apiGroup: ""
  kind: ServiceAccount
  name: bigip-ctlr
  namespace: kube-system
```