# CIS Installation  
## Overview  
CIS can be configured in multiple ways depending on the customer scenario.  

CIS can be deployed on Kubernetes platform.  

In this project, we are deploying CIS in [**NodePort**](https://github.com/SalinaYu/EKS-and-F5-Intergration/blob/main/Overview%20of%20F5%20BIG-IP%20Container%20Ingress%20Services.md#nodeport) mode on Kubernetes platform using EKS (Elastic Kubernetes Service) on AWS and have standalone BIG-IP running in the same Amazon VPC (Virtual Private Cloud). CIS is utilizing ConfigMap to configure the BIG-IP.  

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
3\. Create a **Cluster Role** and **Cluster Role Binding** on the Kubernetes Cluster using the examples below.

The example below shows the broadest supported permission set. You can narrow the permissions down to specific resources, namespaces, etc. to suit your needs. See the [Kubernetes RBAC documentation](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) for more information.

[k8s-rbac.yaml](https://github.com/SalinaYu/EKS-and-F5-Intergration/blob/main/k8s-rbac.yaml)

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
Push this configuration with the command:
```
kubectl apply -f k8s-rbac.yaml
```
> Important  
you can substitute a Role and RoleBinding if your Controller does not need access to the entire Cluster.

4\. Create a CIS deployment using cis-deploy.yaml as shown below. 

You need to update the example file with the correct information, for example: bigip-url `<ip_address-or-hostname>`, bigip-partition `<name_of_partition>`,pool-member-type `<cis_deployment_type_nodeport_or_cluster>`, etc.

[cis-deploy.yaml](https://github.com/SalinaYu/EKS-and-F5-Intergration/blob/main/cis-deploy.yaml)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: k8s-bigip-ctlr-deployment
  namespace: kube-system
spec:
# DO NOT INCREASE REPLICA COUNT
  replicas: 1
  selector:
    matchLabels:
      app: k8s-bigip-ctlr-deployment
  template:
    metadata:
      labels:
        app: k8s-bigip-ctlr-deployment
    spec:
      # Name of the Service Account bound to a Cluster Role with the required
      # permissions
      containers:
        - name: k8s-bigip-ctlr
          image: "f5networks/k8s-bigip-ctlr:latest"
          env:
            - name: BIGIP_USERNAME
              valueFrom:
                secretKeyRef:
                # Replace with the name of the Secret containing your login
                # credentials
                  name: bigip-login
                  key: username
            - name: BIGIP_PASSWORD
              valueFrom:
                secretKeyRef:
                # Replace with the name of the Secret containing your login
                # credentials
                  name: bigip-login
                  key: password
          command: ["/app/bin/k8s-bigip-ctlr"]
          args: [
            # See the k8s-bigip-ctlr documentation for information about
            # all config options
            # https://clouddocs.f5.com/containers/latest/
            "--bigip-username=$(BIGIP_USERNAME)",
            "--bigip-password=$(BIGIP_PASSWORD)",
            "--bigip-url=<ip_address-or-hostname>",
            "--bigip-partition=<name_of_partition>",
            "--pool-member-type=<cis_deployment_type_nodeport_or_cluster>",
            "--insecure=true",
            # for secure communication provide the internal ca certificates using config-map with below option and remove insecure parameter
            #"--trusted-certs-cfgmap=<namespace/configmap>",
            ]
      serviceAccount: bigip-ctlr
      serviceAccountName: bigip-ctlr
```
Push this configuration with the command:
```
kubectl apply -f cis-deploy.yaml
```
