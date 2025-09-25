# **Comparing Virtual Nodes and Managed Nodes in Oracle Container Engine for Kubernetes (OKE)**

This guide, presented by Mahendra Mehra, Senior Training Lead and Evangelist at Oracle University, compares **virtual nodes** and **managed nodes** in Oracle Container Engine for Kubernetes (OKE). It highlights key differences in management, resource allocation, load balancing, pod networking, scaling, and pricing to help users choose the appropriate node type for their Kubernetes workloads.

<details>
<summary><strong>📋 Table of Contents</strong></summary>

- [**Comparing Virtual Nodes and Managed Nodes in Oracle Container Engine for Kubernetes (OKE)**](#comparing-virtual-nodes-and-managed-nodes-in-oracle-container-engine-for-kubernetes-oke)
  - [Overview](#overview)
  - [Key Differences](#key-differences)
    - [1. Management Approach](#1-management-approach)
    - [2. Resource Allocation](#2-resource-allocation)
    - [3. Load Balancing](#3-load-balancing)
    - [4. Pod Networking](#4-pod-networking)
    - [5. Scaling](#5-scaling)
    - [6. Pricing](#6-pricing)
    - [7. Additional Considerations](#7-additional-considerations)
  - [Summary Table](#summary-table)
  - [Choosing Between Virtual and Managed Nodes](#choosing-between-virtual-and-managed-nodes)
  - [Key Notes](#key-notes)
  - [Next Steps](#next-steps)

</details>

## Overview
When creating a node pool in OKE, you can choose between **managed nodes** and **virtual nodes**. Each type offers distinct features, affecting how you manage, scale, and pay for your Kubernetes cluster. Below is a detailed comparison of their characteristics.

## Key Differences

### 1. Management Approach
- **Managed Nodes**:
  - **User Responsibility**: Users manage the nodes, including configuration, Kubernetes upgrades, and security patching.
  - **Flexibility**: Allows customization of nodes to meet specific requirements.
  - **Cluster Types**: Supported in both **Basic** and **Enhanced** clusters.
- **Virtual Nodes**:
  - **Serverless Experience**: OCI manages the nodes, handling Kubernetes upgrades and security patches while respecting application availability.
  - **Cluster Types**: Supported only in **Enhanced** clusters.
  - **Benefit**: Reduces operational overhead, as OCI automates node management.

### 2. Resource Allocation
- **Managed Nodes**:
  - **Node Pool Level**: CPU and memory requirements are specified at the node pool level during creation.
  - **Configuration**: Users define resources (e.g., shape, CPU, memory) for the entire node pool.
- **Virtual Nodes**:
  - **Pod Level**: CPU and memory requirements are specified in the pod specification using **requests** and **limits**.
  - **Benefit**: Offers fine-grained resource control, optimizing usage for individual pods.

### 3. Load Balancing
- **Managed Nodes**:
  - **Load Balancing**: Traffic is distributed among worker nodes (e.g., `node-ip:nodeport`).
  - **Security Lists**: Users may need to manually configure security list rules for load balancers, depending on the setup.
- **Virtual Nodes**:
  - **Load Balancing**: Traffic is distributed directly to pod IP addresses (e.g., `pod-ip:nodeport`).
  - **Security Lists**: Load balancer security list management is not enabled; users must manually configure security rules.
  - **Configuration**: Load balancers assign a node port and route traffic to pod IPs, requiring specific network setup.

### 4. Pod Networking
- **Managed Nodes**:
  - **CNI Plugins**: Supports both **VCN-Native Pod Networking** and **Flannel CNI** plugins.
  - **Flexibility**: Users can choose the networking model based on their needs.
- **Virtual Nodes**:
  - **CNI Plugin**: Only supports **VCN-Native Pod Networking**.
  - **VNIC Usage**: Each virtual node has a single VNIC; IP addresses are not pre-allocated before pod creation.
  - **CNI Visibility**: The VCN-Native Pod Networking CNI plugin does not appear as running in the `kube-system` namespace.
  - **Network Requirements**: Pod subnet route tables must include rules for a **NAT Gateway** and **Service Gateway** to ensure connectivity.

### 5. Scaling
- **Managed Nodes**:
  - **User Responsibility**: Users manually adjust cluster capacity by changing the number of node pools or nodes.
  - **Autoscaling**: Optional **node pool autoscaling** can be enabled to automatically scale nodes and pods based on demand.
  - **Control**: Provides flexibility but requires active management.
- **Virtual Nodes**:
  - **Automated Scaling**: OCI manages cluster capacity, automatically scaling virtual node pools.
  - **Capacity**: Each virtual node supports up to **500 pods**; users can increase the number of virtual nodes or node pools to scale up.
  - **Benefit**: Eliminates operational overhead for capacity management.

### 6. Pricing
- **Managed Nodes**:
  - **Cost Model**: Charged based on the compute instances (e.g., `VM.Standard.E3.Flex`) allocated to the node pool, regardless of actual pod usage.
  - **Predictability**: Fixed cost based on instance size and count.
- **Virtual Nodes**:
  - **Cost Model**: Charged based on the exact compute resources (CPU and memory) consumed by each pod.
  - **Efficiency**: Pay-as-you-go model, potentially reducing costs for variable workloads.

### 7. Additional Considerations
- **SSH Access**:
  - **Managed Nodes**: Support SSH access for administrative tasks (e.g., log collection, maintenance) if a public SSH key is provided during node pool creation.
  - **Virtual Nodes**: Do not support SSH access, as they are fully managed by OCI.
- **Use Cases**:
  - **Managed Nodes**: Ideal for workloads requiring fine-tuned control, custom configurations, or SSH access for troubleshooting.
  - **Virtual Nodes**: Suited for serverless, scalable applications where operational simplicity and automatic management are priorities.

## Summary Table

| **Feature**               | **Managed Nodes**                                                                 | **Virtual Nodes**                                                                 |
|---------------------------|-----------------------------------------------------------------------------------|-----------------------------------------------------------------------------------|
| **Management**            | User-managed; requires manual upgrades and patching                              | OCI-managed; automatic upgrades and patching                                      |
| **Cluster Types**         | Basic and Enhanced clusters                                                     | Enhanced clusters only                                                           |
| **Resource Allocation**   | Node pool level (CPU/memory for entire pool)                                     | Pod level (requests/limits in pod spec)                                          |
| **Load Balancing**        | Between worker nodes (`node-ip:nodeport`)                                        | Between pods (`pod-ip:nodeport`)                                                 |
| **Security Lists**        | May require manual configuration                                                 | Always requires manual security rule configuration                               |
| **Pod Networking**        | Supports VCN-Native and Flannel CNI                                              | Only VCN-Native; single VNIC; requires NAT/Service Gateway rules                 |
| **Scaling**               | Manual or autoscaling; user adjusts node pools/nodes                             | Automatic scaling; up to 500 pods per virtual node                               |
| **Pricing**               | Per compute instance                                                            | Per pod resource consumption                                                     |
| **SSH Access**            | Supported with public key                                                       | Not supported                                                                    |

## Choosing Between Virtual and Managed Nodes
- **Managed Nodes**:
  - **Pros**:
    - Greater control over node configuration.
    - Supports SSH for maintenance and troubleshooting.
    - Compatible with both Basic and Enhanced clusters.
    - Flexible networking options (VCN-Native or Flannel).
  - **Cons**:
    - Higher operational overhead (manual upgrades, capacity management).
    - Fixed compute costs based on instance allocation.
  - **Use Cases**: Workloads requiring custom configurations, SSH access, or specific CNI plugins.
- **Virtual Nodes**:
  - **Pros**:
    - Serverless experience with minimal management overhead.
    - Automatic scaling and upgrades.
    - Cost-efficient for variable workloads (pay per pod resource).
    - Simplified pod-level resource allocation.
  - **Cons**:
    - Limited to Enhanced clusters.
    - No SSH access.
    - Restricted to VCN-Native networking.
  - **Use Cases**: Scalable, serverless applications with dynamic workloads and minimal administrative needs.

## Key Notes
- **Decision Factors**: Choose based on your application’s requirements for control, scalability, cost, and operational complexity.
- **Hybrid Approach**: You can mix managed and virtual node pools in an Enhanced cluster to balance control and automation.
- **Official Documentation**: Refer to [OCI official documentation](https://docs.oracle.com) under "Container Engine for Kubernetes" for detailed guidance on node pool configuration.

## Next Steps
- **Practical Demo**: Explore creating and managing node pools with virtual and managed nodes in OKE.
- **Scaling Strategies**: Learn how to implement autoscaling for managed nodes or leverage virtual node scaling.
- **Networking Configurations**: Dive deeper into VCN-Native networking and security rule setups for virtual nodes.

By understanding these differences, you can select the node type that best aligns with your Kubernetes workload requirements, balancing control, scalability, and cost.