# Overview of ConfigMap
In Kubernetes, ConfigMap is an API object used to store non-confidential data in key-value pairs.

ConfigMap allows users to decouple configuration artifacts from image content to keep containerized applications portable. Pods can consume ConfigMaps as environment variables, command line arguments, or as configuration files in a volume.

In CIS, Configuration artifacts of ConfigMap are agent-specific, meaning that configuration in ConfigMap differs from agent to agent. 

CIS v2.0 operates in two different agent modes: [AS3](https://clouddocs.f5.com/products/extensions/f5-appsvcs-extension/latest/) (Application Services Extensions) or CCCL (Common Controller Core Library).

>Note
This section covers ConfigMap for agent AS3 only.   
For agent CCCL, please refer to [f5-cccl github repo](https://github.com/f5devcentral/f5-cccl).  

CIS supports two types of AS3 ConfigMaps, more details explained in subsequent sections:

* User-defined AS3 ConfigMap
* Override AS3 Configmap

## What is AS3
F5 Application Services (AS3) Extensions use a declarative API, meaning AS3 Extension declarations describe the desired configuration state of a BIG-IP system.

When using AS3 Extensions, CIS sends declaration files using a single Rest API call.

The diagram below depicts the basic data model of the AS3 artifact.

![](https://clouddocs.f5.com/containers/latest/_images/config-map-diagram.png)

## What is AS3 ConfigMap
The AS3 ConfigMap hosts AS3 extensions, in JSON format, as a configuration artifact. CIS can manage and orchestrate BIG-IP declaratively through this ConfigMap.

In agent AS3 mode, CIS handles Ingress or Route resources by converting them into AS3 declarations before posting to BIG-IP. When AS3 ConfigMap is configured along with Ingress or Routes, CIS manages ConfigMap and Ingress (or) Routes AS3 declarations separately. While sending an AS3 declaration to BIG-IP, CIS will combine both of these AS3 declarations as a single declaration and POST it to BIG-IP.

>Important  
Ingress or Routes will always use the single partition (CIS managed partition) in CIS. But AS3 ConfigMap can have more than one partition, except CIS-managed partition. CIS will not process AS3 ConfigMap if configured in CIS-managed partition.

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: name-of-your-configmap
  namespace: default
  labels:
    f5type: virtual-server
    as3: "true"
data:
  template: |
    {
      "class": "AS3",
      "declaration": {
            "class": "ADC",
            "schemaVersion": "3.18.0",
            "id": "urn:uuid:33045210-3ab8-4636-9b2a-c98d22ab915d",
            "label": "http",
            "remark": "A1 Template",
            "k8s": {
               "class": "Tenant",
               "A1": {
                  "class": "Application",
                  "template": "generic",
                  "a1_80_vs": {
                        "class": "Service_HTTP",
                        "remark": "a1",
                        "virtualAddresses": [
                           "10.192.75.101"
                        ],
                        "pool": "web_pool"
                  },
                  "web_pool": {
                        "class": "Pool",
                        "monitors": [
                           "http"
                        ],
                        "members": [
                           {
                              "servicePort": 8080,
                              "serverAddresses": []
                           }
                        ]
                  }
               }
            }
      }
   }
```  
The image below depicts the example AS3 ConfigMap and its mapping with AS3 objects.

![](https://clouddocs.f5.com/containers/latest/_images/config-map-diagram2.png)

## Processing AS3 ConfigMap in CIS
As mentioned in the above figure, for CIS to process AS3 ConfigMap, ensure that below AS3 labels are populated.

Label | Value | Description
------------ | ------------- | -------------
as3 | true (or) false | When set to `true`, this tells CIS that this is a AS3 ConfigMap and processes it. When set to `false`, this tells CIS that you don’t want to usee AS3 ConfigMap, meaning you do not want CIS to process this ConfigMap temporarily until you reset this flag to true. If AS3 ConfigMap exists with flag flase, CIS will remove the respective configuration from BIG-IP. Meanwhile, you can make the necessary changes and save the ConfigMap within the K8S system.
f5type | virtual-server | This tells CIS that you want to create a virtual server on the BIG-IP device.

> Important  
Staging is not equivalent to deleting ConfigMap in CIS. CIS will save the previous active copy of ConfigMap before the ConfigMap AS3 label was set to false.

> Note  
Along with the above labels, to process an AS3 declaration, CIS uses goJsonSchema to validate AS3 declarations. If the JSON data structure does not conform with the schema, an error is logged.

To process AS3 ConfigMap, the highlighted CIS deployment configurations are required.  

[Required CIS deployment configurations portion](https://github.com/SalinaYu/EKS-and-F5-Intergration/blob/main/config/cis-deploy.yaml#L42)
```yaml
args: [
  "--bigip-username=$(BIGIP_USERNAME)",
  "--bigip-password=$(BIGIP_PASSWORD)",
  "--bigip-url=<ip_address-or-hostname>",
  "--bigip-partition=<name_of_partition>","--pool-member-type=<cis_deployment_type_nodeport_or_cluster>,
]
```
Continue reading [how to deploy ConfigMap](https://github.com/SalinaYu/EKS-and-F5-Intergration/blob/main/docs/ConfigMap%20Deployment.md) or
[go back to the main page](https://github.com/SalinaYu/EKS-and-F5-Intergration#quick-start).