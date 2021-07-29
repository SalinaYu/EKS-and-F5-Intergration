 # How to deploy ConfigMap
 ## Prerequisites 
* CIS version 2.0 or newer
* CIS uses AS3 declarative API. You will need the AS3 extension v3.18 or newer installed on BIG-IP before using the AS3 extension of CIS.

You can find the required YAML files in the [user-defined-configmap respository on GitHub](https://github.com/F5Networks/k8s-bigip-ctlr/tree/master/docs/config_examples/configmap).

CIS uses the partition defined in the CIS configuration by default to communicate with the BIG-IP system when adding static ARPs and forwarding entries for VXLAN. CIS managed partitions `<partition>` and should not be used in ConfigMap as tenants. If CIS is deployed with `--bigip-partition=cis`, then `<cis>` is not supposed to be used as a tenant in the AS3 declaration.

## CIS Service Discovery
There are 7 steps to deploy ConfigMap **after** you have complete the prerequisites above:  
1. Update K8S Service configurations.
2. Make sure the backend application that is going to be served by the BIG-IP is running.
3. Prepare K8S ConfigMap deployment.
4. AS3 ConfigMap becomes available for processing (behind the scene).
5. CIS performs service discovery (behind the scene).
6. After completing service discovery, CIS modifies the AS3 declaration by appending the discovered endpoints (behind the scene). 
7. BIG-IP will be configured and ready to process traffic.

This procedural diagram depicts Service Discovery while processing ConfigMap in CIS (The diagram is followed by detailed 7 steps):

![](https://clouddocs.f5.com/containers/latest/_images/config-map-diagram-quickstart1.png)

> Important  
From the above figure, for ConfigMap to properly function in CIS, configure the same tenant, application, and pool details for steps 1 and 3.

**Detailed Steps**:  
1\. Prepare and deploy the desired service (NodePort in this example) in Kubernetes. Make sure the below labels are configured:
* `cis.f5.com/as3-tenant: Tenant-1`
* `cis.f5.com/as3-app: APP1`
* `cis.f5.com/as3-pool: web_pool`

2\. Prepare and deploy the backend application deployment that is going to be served by BIG-IP.

3\. Prepare and deploy the AS3 deployment inside an AS3 ConfigMap template in Kubernetes. Make sure the tenant, application, and pools are configured with the same values as in Step 1.
* `Tenant Class: Tenant-1`
* `Application Class: APP1`
* `Pool Class:  web_pool`

4\. After the AS3 ConfigMap becomes available for processing, CIS decodes the AS3 declaration and extracts tenant (`Tenant-1`), Application (`APP1`) and Pool (`web_pool`) details.

5\. CIS performs service discovery using extracted tenant (`Tenant-1`), Application (`APP1`) and Pool (`web_pool`) details, and fetches service endpoints, in this example `10.105.126.114:80`.

6\. After completing Service discovery, CIS modifies the AS3 declaration by appending the discovered endpoints. CIS only modifies these two values in the AS3 declaration:
* serverAddresses array
* servicePort value

7\. CIS posts the generated AS3 declaration to the BIG-IP system to begin processing traffic.

> Important
From the above figure, for ConfigMap to properly function in CIS, configure the same tenant, application, and pool details for step 1 and step 3.

## Supported operations of AS3 ConfigMap in CIS
CIS processes when a ConfigMap is created, modified, or deleted in Kubernetes.

The sections below explain the detailed operations of AS3 ConfigMap in CIS for creation, modification, and deletion.

### Create AS3 ConfigMap
You can use  the command `$ kubectl apply -f <as3_configmap_file_in_yaml>` to create ConfigMap.

### Modify AS3 ConfigMap
You can use the command `$ kubectl apply -f <as3_configmap_file_in_yaml>` to Modify ConfigMap.

### Delete AS3 ConfigMap
You can use the command `$ kubectl delete -f <as3_configmap_file_in_yaml>` to delete ConfigMap.

[Go back to the main page](https://github.com/SalinaYu/EKS-and-F5-Intergration#quick-start).