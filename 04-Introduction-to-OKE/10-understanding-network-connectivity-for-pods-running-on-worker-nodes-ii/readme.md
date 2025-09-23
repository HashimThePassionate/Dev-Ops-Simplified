# **Exploring OCI VCN-Native Pod Networking CNI Plugin**

This document delves into the OCI VCN-Native Pod Networking CNI plugin and its role in optimizing pod networking within Kubernetes clusters, specifically when using Oracle Container Engine for Kubernetes (OKE).

<details>
<summary>📋 Table of Contents</summary>

- [**Exploring OCI VCN-Native Pod Networking CNI Plugin**](#exploring-oci-vcn-native-pod-networking-cni-plugin)
  - [Overview of OCI VCN-Native Pod Networking](#overview-of-oci-vcn-native-pod-networking)
    - [Dual Subnet Configuration](#dual-subnet-configuration)
  - [Configuring OCI VCN-Native Pod Networking](#configuring-oci-vcn-native-pod-networking)
    - [Kubernetes API Endpoint Subnet](#kubernetes-api-endpoint-subnet)
    - [Virtual Network Interface Cards (VNICs)](#virtual-network-interface-cards-vnics)
    - [Calculating Maximum Pods](#calculating-maximum-pods)
  - [Key Considerations](#key-considerations)
  - [Updating the OCI VCN-Native Pod Networking CNI Plugin](#updating-the-oci-vcn-native-pod-networking-cni-plugin)
    - [Applying Updates](#applying-updates)
  - [Conclusion](#conclusion)

</details>

## Overview of OCI VCN-Native Pod Networking

The OCI VCN-Native Pod Networking CNI plugin leverages Oracle Cloud Infrastructure (OCI) Virtual Cloud Network (VCN) capabilities to provide efficient and scalable pod networking. It utilizes a dual subnet configuration to enable robust communication within the cluster and with external services.

### Dual Subnet Configuration

Worker nodes in a cluster are connected to two specified subnets, creating a versatile network environment:
1. **Worker Node Subnet**:
   - Facilitates communication between the cluster control plane components (e.g., kube-apiserver, kube-controller-manager, kube-scheduler) and worker node processes (e.g., kubelet, kube-proxy).
   - Can be configured as:
     - **Private** or **public**.
     - **Regional subnet** or **availability domain-specific subnets** (recommended for high availability).

2. **Pod Subnet**:
   - Enables pod-to-pod communication and direct access to individual pods using private pod IP addresses.
   - Must be a **regional subnet** to support:
     - Communication within the same worker node.
     - Communication across nodes.
     - Interaction with OCI services via a **service gateway**.
     - Internet access through a **NAT gateway**.

## Configuring OCI VCN-Native Pod Networking

When creating a cluster with OKE and using the OCI VCN-Native Pod Networking CNI plugin, you must specify:
- **CIDR blocks** for the Kubernetes service.
- **Non-overlapping CIDR blocks** for the VCN and subnets compared to the Kubernetes service CIDR block.

### Kubernetes API Endpoint Subnet
- Requires only a small CIDR block, as the cluster needs just one IP address.
- A `/30` CIDR block is sufficient.

### Virtual Network Interface Cards (VNICs)
- Each worker node in the node pool has a **primary VNIC** with a primary IP address.
- Depending on the node pool's shape, worker nodes may have one or more **secondary VNICs**.
- VNICs reside in the subnets selected for pod communication.
- Each VNIC can be associated with up to **31 secondary IP addresses**.
- Pods running on a worker node obtain a secondary IP address from a VNIC for inbound and outbound communication.

### Calculating Maximum Pods
To calculate the maximum number of pods in a node pool for a specific shape:
- Each secondary VNIC can assign up to 31 secondary IP addresses.
- Use the equation:  
  *Maximum Pods = (Number of Secondary VNICs per Node) × 31*

## Key Considerations

1. **Compatibility**:
   - The OCI VCN-Native Pod Networking CNI plugin is compatible with both **virtual node pools** and **managed node pools**.
   - Requires Kubernetes version **1.22 or later**.
   - If using an OKE image as the base image for worker nodes, do not select an image released before **June 2022**.

2. **Service Mesh Support**:
   - Supported service mesh products include **Oracle Cloud Infrastructure Service Mesh**, **Istio**, and **Linkerd**.
   - Currently limited to **Oracle Linux 7** (Oracle Linux 8 support is planned).
   - Worker nodes must run Kubernetes **1.26 or later**.

3. **Security Rules**:
   - Specific **ingress** and **egress rules** are required for pod subnets and worker node subnets.
   - Refer to the [OCI official documentation](https://docs.oracle.com) for detailed information on configuring these security rules.

## Updating the OCI VCN-Native Pod Networking CNI Plugin

When VCN-native pod networking is specified as the cluster's network type, the cluster and its node pools initially use the latest version of the OCI VCN-Native Pod Networking CNI plugin. Updates are released periodically, and you can choose:
- **Automatic updates** by Oracle (default behavior).
- **Manual version selection**, where you are responsible for keeping the add-on up to date.

### Applying Updates
- Updates are applied only when worker nodes are rebooted.
- To check for pending updates, inspect the VCN-native IP CNI DaemonSet logs using the following command:
  ```
  kubectl logs -n kube-system -l app=oci-vcn-ip-native
  ```
- **Interpreting the Response**:
  - If there is output, updates are pending and will be applied during the next worker node reboot.
  - If there is no output, no updates are waiting to be applied.

## Conclusion

This lesson covered the OCI VCN-Native Pod Networking CNI plugin in depth, highlighting its dual subnet configuration, VNIC-based IP address allocation, and compatibility considerations. Combined with the previous exploration of the Flannel CNI plugin, this provides a comprehensive understanding of pod networking options in Kubernetes with OKE. For further details, refer to the OCI official documentation.