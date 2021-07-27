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
- AS3: 3.18+ must be installed on the BIG-IP system.
- Create a BIG-IP partition to manage Kubernetes 
<br/>