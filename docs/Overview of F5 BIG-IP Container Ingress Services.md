# Overview of F5 BIG-IP Container Ingress Services

F5 BIG-IP Container Ingress Services (CIS) lets you manage your F5 BIG-IP device from Kubernetes or OpenShift using either environment’s native CLI/API.

The F5 BIG-IP CIS (`k8s-bigip-ctlr`) is a cloud-native connector that can use either Kubernetes or OpenShift as a BIG-IP orchestration platform.

CIS watches the [Kubernetes API](https://kubernetes.io/docs/concepts/overview/kubernetes-api/) for specially formatted resources, and updates the BIG-IP system configuration accordingly.

![](https://clouddocs.f5.com/containers/latest/_images/what-is-cis-diagram.png)  

**Features:**  
* Dynamically create, and manage BIG-IP objects.
* Forward traffic from the BIG-IP device to Kubernetes clusters via [NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#nodeport) or [ClusterIP](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types).
* Support [F5 AS3 Extension](https://clouddocs.f5.com/products/extensions/f5-appsvcs-extension/latest/) declarations.  

## Deployment Options
These options are configured using 'pool-member-type' parameter in CIS deployment.

### NodePort
This section documents K8S with integration of CIS and BIG-IP using NodePort configuration. Benefits of NodePort are:

* It works in any environment (no requirement for SDN)
* No persistence/visibility to backend Pod
* Can be deployed for “static” workloads (not ideal)

Similar to Docker, BIG-IP communicates with an ephemeral port, but in this case the kube-proxy keeps track of the backend Pod (container). This works well, but the downside is that you have an additional layer of load balancing with the kube-proxy.

![](https://clouddocs.f5.com/containers/latest/_images/nodeport-diagram.png)

When using NodePort, pool members represent the kube-proxy service on the node. BIG-IP needs a local route to the nodes. There is no need for VXLAN tunnels or Calico. BIG-IP can dynamically ARP for the Kube-proxy running on node.

### ClusterIP
This section documents K8S with integration of CIS and BIG-IP using clusterIP configuration. Benefits of clusterIP are:

* Requires ability to route to Pod
* Flannel VXLAN, OpenShift VXLAN
* Alternately Pod routable through network, for example:
  * Calico BGP
  * Public Cloud network  

The BIG-IP CIS also supports a cluster mode where Ingress traffic bypasses the Kube-proxy and routes traffic directly to the pod. This requires that the BIG-IP have the ability to route to the pod. This could be by using an overlay network that F5 supports (Flannel VXLAN, or OpenShift VXLAN). Leave the kube-proxy intact (no changes to underlying Kubernetes infrastructure).

![](https://clouddocs.f5.com/containers/latest/_images/clusterip-diagram.png)  


Continue reading [how to Install F5 BIG-IP CIS](https://github.com/SalinaYu/EKS-and-F5-Intergration/blob/main/docs/CIS%20Installation.md) or
[back to the main page](https://github.com/SalinaYu/EKS-and-F5-Intergration#quick-start)