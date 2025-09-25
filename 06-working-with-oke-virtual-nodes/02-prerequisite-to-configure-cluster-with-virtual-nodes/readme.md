# **Configuring IAM Policies and Setting Up Virtual Nodes in Oracle Container Engine for Kubernetes** (OKE)

This guide, presented by Mahendra Mehra, Senior Training Lead and Evangelist at Oracle University, details the **Identity and Access Management (IAM) policies** required for using **virtual nodes** in Oracle Container Engine for Kubernetes (OKE) and the process for creating, configuring, and setting up networking for virtual nodes and virtual node pools. The focus is on enabling both tenancy administrators and non-administrator users to work with virtual nodes in **Enhanced clusters**.

<details>
<summary><strong>📋 Table of Contents</strong></summary>

- [**Configuring IAM Policies and Setting Up Virtual Nodes in Oracle Container Engine for Kubernetes** (OKE)](#configuring-iam-policies-and-setting-up-virtual-nodes-in-oracle-container-engine-for-kubernetes-oke)
  - [Prerequisites](#prerequisites)
  - [Required IAM Policies](#required-iam-policies)
    - [1. Mandatory Policy for All Users](#1-mandatory-policy-for-all-users)
    - [2. Policies for Non-Administrator Users](#2-policies-for-non-administrator-users)
  - [Creating and Configuring Virtual Nodes and Virtual Node Pools](#creating-and-configuring-virtual-nodes-and-virtual-node-pools)
    - [1. Creating Virtual Node Pools](#1-creating-virtual-node-pools)
    - [2. Configuring Virtual Node Pools](#2-configuring-virtual-node-pools)
    - [3. Network Setup for Virtual Nodes](#3-network-setup-for-virtual-nodes)
    - [4. Additional Notes](#4-additional-notes)
  - [Security Best Practices](#security-best-practices)
  - [Key Notes](#key-notes)
  - [Next Steps](#next-steps)

</details>

## Prerequisites
- An OKE **Enhanced cluster** (virtual nodes are not supported in Basic clusters).
- Administrative access to the tenancy or specific IAM policies for non-administrator users.
- A Virtual Cloud Network (VCN) with private subnets for virtual nodes and pods.
- Access to the OCI Console, OCI CLI, or API for cluster and node pool creation.
- Familiarity with Kubernetes pod specifications for resource allocation.
- The current date and time: **04:46 AM PKT, Friday, September 26, 2025**.

## Required IAM Policies

### 1. Mandatory Policy for All Users
- **Purpose**: Enables the OKE service to create container instances with VNICs connected to a subnet in the tenancy’s VCN for virtual nodes.
- **Scope**: Must be created in the **root compartment** of the tenancy.
- **Policy Statement**:
  ```plaintext
  Allow service OKE to manage container-instances in tenancy
  Allow service OKE to manage vnics in tenancy
  ```
- **Instructions**:
  - Copy the policy statements from the [OCI official documentation](https://docs.oracle.com) under “Container Engine for Kubernetes” > “Working with Virtual Nodes.”
  - In the OCI Console:
    - Go to **Identity & Security** > **Policies** > Select the root compartment.
    - Click **Create Policy**.
    - Name: e.g., `OKE-Virtual-Nodes-Service-Policy`.
    - Paste the statements and create the policy.
- **Applicability**: Required for both tenancy administrators and non-administrator users.

### 2. Policies for Non-Administrator Users
- **Purpose**: Grants non-administrator users permissions to manage virtual node pools, read network resources, and manage VNICs.
- **Policy Statements**:
  ```plaintext
  Allow group <GroupName> to manage cluster-virtual-node-pools in compartment <CompartmentName>
  Allow group <GroupName> to read virtual-network-family in compartment <CompartmentName>
  Allow group <GroupName> to manage vnics in compartment <CompartmentName>
  ```
  - Replace `<GroupName>` with the user group (e.g., `ClusterUsers`).
  - Replace `<CompartmentName>` with the target compartment (e.g., `CTDOKE`).
- **Instructions**:
  - Copy from the OCI documentation under “Working with Virtual Nodes.”
  - Create the policy in the target compartment:
    - Go to **Identity & Security** > **Policies** > Select the compartment (e.g., `CTDOKE`).
    - Click **Create Policy**.
    - Name: e.g., `Virtual-Nodes-User-Policy`.
    - Paste the statements and create the policy.
- **Note**: Administrators with full tenancy access do not need these additional policies, as their privileges cover these actions.

## Creating and Configuring Virtual Nodes and Virtual Node Pools

### 1. Creating Virtual Node Pools
- **Requirement**: Virtual nodes are only supported in **Enhanced clusters**.
- **Methods**:
  - **OCI Console**:
    - Navigate to **Developer Services** > **Kubernetes Cluster (OKE)** > Select or create an Enhanced cluster.
    - Go to **Node Pools** > **Create Node Pool**.
    - Select **Virtual Nodes** as the node type.
  - **OCI CLI**:
    ```bash
    oci ce node-pool create --compartment-id <compartment-ocid> --cluster-id <cluster-ocid> --name <node-pool-name> --node-type VIRTUAL_NODE --size <node-count>
    ```
  - **API**: Use the `CreateNodePool` API with `nodeType` set to `VIRTUAL_NODE`.
- **Key Parameters**:
  - **Node Count**: Specify the number of virtual nodes (e.g., 3).
  - **Placement**: Choose availability domains (ADs) for high availability or use a regional subnet (recommended).
  - **Pod Shape**: Select a compute shape (e.g., `Pod.Standard.E3.Flex`) supported by OKE and available in your tenancy.
    - Only shapes compatible with OKE and your tenancy are displayed.
    - Align the shape with your application’s CPU and memory requirements.

### 2. Configuring Virtual Node Pools
- **Node Count**:
  - Defines the number of virtual nodes in the pool.
  - Example: Set to 3 for distribution across multiple availability domains.
- **Placement**:
  - **Regional Subnet**: Recommended for optimal performance and availability.
  - **Availability Domains**: Distribute nodes across ADs for high availability (e.g., one node per AD in a region with three ADs).
- **Pod Shape**:
  - Determines the processor type for pods running on virtual nodes.
  - Example: `Pod.Standard.E3.Flex` for flexible CPU/memory allocation.
  - Ensure the shape supports your application’s workload requirements.
- **Resource Allocation**:
  - Specify CPU and memory in the **pod specification** (not at the node pool level).
  - Example Pod Spec:
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: example-pod
    spec:
      containers:
      - name: example-container
        image: nginx
        resources:
          requests:
            cpu: "1"
            memory: "1Gi"
          limits:
            cpu: "2"
            memory: "2Gi"
    ```
  - **Note**: Precision in defining requests and limits ensures optimal resource utilization.

### 3. Network Setup for Virtual Nodes
- **Pod Communication**:
  - **CNI Plugin**: Only **VCN-Native Pod Networking** is supported.
  - **Pod Subnet**:
    - Must be a **private regional subnet** (e.g., `10.0.2.0/24`).
    - Oracle recommends using the same subnet for pods and virtual nodes.
    - Configure route table rules for:
      - **NAT Gateway**: For outbound internet access.
      - **Service Gateway**: For access to OCI services (e.g., Object Storage).
  - **Security Rules**:
    - Use **Network Security Groups (NSGs)** instead of security lists for finer-grained control.
    - Maximum of 5 NSGs per subnet.
    - Example NSG Rule:
      - **Ingress**: Allow traffic from specific sources to pod ports.
      - **Egress**: Allow pods to communicate with external services.
    - Example:
      ```plaintext
      Ingress Rule: Source CIDR: 10.0.0.0/16, Protocol: TCP, Destination Port: 80
      Egress Rule: Destination CIDR: 0.0.0.0/0, Protocol: All
      ```

- **Virtual Node Communication**:
  - **Virtual Node Subnet**:
    - Must be a **private subnet** (regional recommended, e.g., `10.0.2.0/24`).
    - Must be distinct from load balancer subnets (if used).
    - Oracle recommends using the same subnet as the pod subnet.
  - **VNIC Allocation**:
    - Each virtual node uses a single VNIC.
    - IP addresses are dynamically allocated when pods are created, not pre-allocated.
  - **Route Table Rules**:
    - Ensure rules for NAT Gateway and Service Gateway are defined.

### 4. Additional Notes
- **Load Balancing**:
  - Traffic is distributed to pod IP addresses (`pod-ip:nodeport`) rather than node IPs.
  - Manual configuration of security rules is required for load balancers.
- **Scaling**:
  - Virtual node pools automatically scale up to **500 pods per virtual node**.
  - Increase the number of virtual nodes or node pools for additional capacity.
- **SSH Access**:
  - Virtual nodes do not support SSH access, as they are fully managed by OCI.
- **Official Documentation**:
  - Refer to [OCI official documentation](https://docs.oracle.com) under “Container Engine for Kubernetes” > “Working with Virtual Nodes” for policy statements and detailed setup instructions.

## Security Best Practices
- **IAM Policies**: Use least-privilege principles, restricting non-administrator users to specific compartments and resources.
- **Private Subnets**: Ensure pod and virtual node subnets are private to minimize exposure.
- **NSGs**: Prefer NSGs over security lists for granular access control.
- **CIDR Restrictions**: Limit NSG rules to specific source/destination CIDRs for pod and node communication.
- **Monitoring**: Regularly audit IAM policies and NSG rules in the OCI Console.

## Key Notes
- **Enhanced Clusters Only**: Virtual nodes require Enhanced clusters due to their serverless architecture.
- **Pod-Level Resources**: Specifying CPU/memory in pod specs allows precise resource allocation.
- **Networking Simplicity**: Using the same private subnet for pods and virtual nodes reduces complexity.
- **Automatic Management**: OCI handles upgrades, patching, and scaling for virtual nodes, reducing operational overhead.

## Next Steps
- **Practical Demo**: Create a virtual node pool and deploy a sample application using the OCI Console or CLI.
- **Advanced Networking**: Explore NSG configurations and load balancer setups for virtual nodes.
- **Performance Optimization**: Learn to fine-tune pod specifications for optimal resource usage.

This guide provides a comprehensive overview of the IAM policies and setup process for virtual nodes in OKE, ensuring secure and efficient cluster operations. By following these steps, you can deploy a resilient, high-performing Kubernetes environment tailored to your application needs.