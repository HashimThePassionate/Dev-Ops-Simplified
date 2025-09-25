# **Virtual Node and Node Pool Management and Resource Allocation in Oracle Container Engine for Kubernetes** (OKE)

This guide, presented by Mahendra Mehra, Senior Training Lead and Evangelist at Oracle University, covers the management of **virtual nodes** and **virtual node pools** in Oracle Container Engine for Kubernetes (OKE), focusing on creation, listing, updating, and deletion processes, as well as the allocation of CPU, memory, network, and storage resources for pods provisioned by virtual nodes. The content is tailored for Enhanced clusters, as virtual nodes are not supported in Basic clusters.

<details>
<summary>📋 Table of Contents</summary>

- [**Virtual Node and Node Pool Management and Resource Allocation in Oracle Container Engine for Kubernetes** (OKE)](#virtual-node-and-node-pool-management-and-resource-allocation-in-oracle-container-engine-for-kubernetes-oke)
  - [Prerequisites](#prerequisites)
  - [Virtual Node and Node Pool Management](#virtual-node-and-node-pool-management)
    - [1. Creating Virtual Node Pools](#1-creating-virtual-node-pools)
    - [2. Listing Virtual Node Pools](#2-listing-virtual-node-pools)
    - [3. Updating Virtual Node Pools](#3-updating-virtual-node-pools)
    - [4. Deleting Virtual Node Pools](#4-deleting-virtual-node-pools)
    - [5. Listing and Obtaining Details of Virtual Nodes](#5-listing-and-obtaining-details-of-virtual-nodes)
  - [Resource Allocation for Pods on Virtual Nodes](#resource-allocation-for-pods-on-virtual-nodes)
    - [1. CPU and Memory Allocation](#1-cpu-and-memory-allocation)
    - [2. Network Allocation](#2-network-allocation)
    - [3. Storage Allocation](#3-storage-allocation)
    - [4. Key Considerations](#4-key-considerations)
  - [Security Best Practices](#security-best-practices)
  - [Key Notes](#key-notes)
  - [Next Steps](#next-steps)

</details>

## Prerequisites
- An OKE **Enhanced cluster** in the `CTDOKE` compartment.
- IAM policies configured for virtual node usage (see previous lesson for details).
- Access to the OCI Console, OCI CLI, or API for cluster and node pool management.
- A Virtual Cloud Network (VCN) with private subnets for virtual nodes and pods.
- Familiarity with Kubernetes pod specifications for resource allocation.
- The current date and time: **04:49 AM PKT, Friday, September 26, 2025**.

## Virtual Node and Node Pool Management

### 1. Creating Virtual Node Pools
- **Requirement**: Virtual node pools can only be created in **Enhanced clusters**.
- **Methods**:
  - **OCI Console**:
    - Navigate to **Developer Services** > **Kubernetes Cluster (OKE)** > Select an Enhanced cluster (e.g., `OKE-private-bastion-demo`) or create a new one.
    - Go to **Node Pools** > **Create Node Pool**.
    - Select **Node Type**: `Virtual Nodes`.
    - Configure:
      - **Name**: e.g., `virtual-node-pool-1`.
      - **Node Count**: Number of virtual nodes (e.g., `3`).
      - **Placement**: Regional subnet (recommended) or specific availability domains (ADs) for high availability.
      - **Pod Shape**: e.g., `Pod.Standard.E3.Flex`.
      - **Subnet**: Private subnet for virtual nodes and pods (e.g., `10.0.2.0/24`).
      - **Network Security Groups (NSGs)**: Up to 5 NSGs for access control.
    - Click **Create Node Pool**.
  - **OCI CLI**:
    ```bash
    oci ce node-pool create --compartment-id <compartment-ocid> --cluster-id <cluster-ocid> --name virtual-node-pool-1 --node-type VIRTUAL_NODE --size 3 --placement-configurations '[{"availability-domain": "<AD>", "subnet-id": "<subnet-ocid>"}]' --shape Pod.Standard.E3.Flex
    ```
  - **API**: Use the `CreateNodePool` API with `nodeType` set to `VIRTUAL_NODE`.
- **Note**: Virtual nodes are automatically managed by OCI, handling upgrades and scaling.

### 2. Listing Virtual Node Pools
- **Purpose**: Provides visibility into virtual node pools within a cluster.
- **Methods**:
  - **OCI Console**:
    - Go to **Developer Services** > **Kubernetes Cluster (OKE)** > Select the cluster.
    - Under **Resources**, click **Node Pools**.
    - Virtual node pools are marked as `Virtual` in the **Node Type** column.
  - **OCI CLI**:
    ```bash
    oci ce node-pool list --compartment-id <compartment-ocid> --cluster-id <cluster-ocid>
    ```
  - **API**: Use the `ListNodePools` API.
- **Outcome**: Displays details such as pool name, node count, and node type.

### 3. Updating Virtual Node Pools
- **Purpose**: Adjusts node pool settings to meet evolving workload demands.
- **Editable Parameters**:
  - **Name**: Update for clarity (e.g., `virtual-node-pool-1` to `prod-virtual-pool`).
  - **Node Count**: Increase or decrease the number of virtual nodes (e.g., from 3 to 5).
  - **Placement**:
    - Change availability domains or fault domains for high availability.
    - Switch to a regional subnet if not already used.
  - **Virtual Node Communication**:
    - Update the **virtual node subnet** (must remain private).
    - Modify NSGs (up to 5) for access control.
  - **Pod Communication**:
    - Update the **pod subnet** (must be private, ideally the same as the virtual node subnet).
    - Adjust NSGs for pod traffic.
  - **Kubernetes Labels and Taints**:
    - Add or modify labels (e.g., `environment=production`) for workload organization.
    - Apply taints to control pod scheduling (e.g., `key=value:NoSchedule`).
- **Methods**:
  - **OCI Console**:
    - Go to **Node Pools** > Select the virtual node pool > **Edit**.
    - Modify desired parameters and save.
  - **OCI CLI**:
    ```bash
    oci ce node-pool update --node-pool-id <node-pool-ocid> --size 5 --kubernetes-labels '{"environment":"production"}' --taints '[{"key":"key1","value":"value1","effect":"NoSchedule"}]'
    ```
  - **API**: Use the `UpdateNodePool` API.

### 4. Deleting Virtual Node Pools
- **Process**:
  - **Permanent Action**: Deletion is irreversible; the node pool and its virtual nodes cannot be recovered.
  - **Cordon and Drain**:
    - **Cordon**: Prevents new pods from being scheduled on the nodes (`kubectl cordon <node-name>`).
    - **Drain**: Gracefully evicts existing pods, redistributing them to other nodes (`kubectl drain <node-name> --ignore-daemonsets`).
  - **Methods**:
    - **OCI Console**:
      - Go to **Node Pools** > Select the virtual node pool > **Delete**.
      - Confirm the action.
    - **OCI CLI**:
      ```bash
      oci ce node-pool delete --node-pool-id <node-pool-ocid> --force
      ```
    - **API**: Use the `DeleteNodePool` API.
- **Note**: Ensure workloads are migrated before deletion to avoid disruptions.

### 5. Listing and Obtaining Details of Virtual Nodes
- **Listing Virtual Nodes**:
  - **OCI Console**:
    - Go to **Developer Services** > **Kubernetes Cluster (OKE)** > Select the cluster > **Node Pools** > Select the virtual node pool > **Virtual Nodes**.
    - Lists all virtual nodes in the pool.
  - **OCI CLI**:
    ```bash
    oci ce node-pool get --node-pool-id <node-pool-ocid>
    ```
  - **API**: Use the `GetNodePool` API to retrieve node details.
- **Obtaining Details**:
  - **OCI Console**:
    - From the **Virtual Nodes** list, click a node to view details (e.g., status, placement, pod count).
  - **OCI CLI**:
    ```bash
    oci ce node get --node-id <node-ocid>
    ```
  - **API**: Use the `GetNode` API.
- **Note**: Virtual nodes are managed by OCI, so details focus on status and resource usage rather than manual configuration.

## Resource Allocation for Pods on Virtual Nodes

### 1. CPU and Memory Allocation
- **Pod-Level Allocation**:
  - Unlike managed nodes, resources are allocated at the **pod level** via the pod specification.
  - Specify **requests** and **limits** for CPU and memory in the pod spec:
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
            cpu: "0.5"
            memory: "1Gi"
          limits:
            cpu: "1"
            memory: "2Gi"
    ```
- **Allocation Rules**:
  - **Limit Specified, No Request**: OKE uses the limit value (e.g., `cpu: "1"`, `memory: "2Gi"`).
  - **Request Specified, No Limit**: OKE uses the request value (e.g., `cpu: "0.5"`, `memory: "1Gi"`).
  - **Both Request and Limit Specified**: OKE uses the limit value.
  - **Neither Specified**: OKE defaults to `0.125 OCPU` (125 millicores) and `0.5 GB` memory per container.
- **Minimum Requirements**:
  - Pods must request at least `0.125 OCPU` and `0.5 GB` memory.
- **Maximum Limits**:
  - Depends on the pod shape (e.g., `Pod.Standard.E3.Flex`, `Pod.Standard.E4.Flex`).
  - Maximum: `64 cores` (128 vCPUs) for AMD shapes.
- **Best Practice**:
  - Set **requests** and **limits** to the same value to align with OKE’s assured capacity model, ensuring predictable resource utilization.

### 2. Network Allocation
- **Bandwidth**:
  - Determined by the number of OCPUs allocated to the pod.
  - Allocation: **1 Gbps per OCPU**.
  - **Limits**:
    - **Minimum**: 1 Gbps.
    - **Maximum**: 40 Gbps.
  - Example: A pod with 2 OCPUs gets 2 Gbps bandwidth, capped at 40 Gbps for high-CPU pods.
- **Networking**:
  - Uses **VCN-Native Pod Networking**.
  - Pods are assigned IP addresses dynamically from the pod subnet when created.
  - Ensure pod subnet route tables include rules for a **NAT Gateway** (outbound internet) and **Service Gateway** (OCI services).

### 3. Storage Allocation
- **Default Allocation**:
  - Each pod gets **15 GB** of storage for the root filesystem of containers.
  - **Usage**:
    - Part of the 15 GB is consumed by container images.
    - The remainder is available for temporary application storage.
- **Note**: No additional persistent storage is allocated by default; use Persistent Volume Claims (PVCs) for persistent storage needs.

### 4. Key Considerations
- **Container Count**: Total resource requirements include all containers in the pod.
- **Kube-Proxy and Container Runtime**:
  - Consumes `0.25 GB` memory and negligible CPU per pod.
  - Account for this in resource planning.
- **Assured Capacity**: Virtual nodes guarantee resource availability, making consistent requests and limits critical.
- **Pod Shape**: Choose a shape (e.g., `Pod.Standard.E3.Flex`) that supports your workload’s CPU and memory needs.

## Security Best Practices
- **IAM Policies**: Ensure policies allow only necessary permissions (e.g., `manage cluster-virtual-node-pools` for users).
- **Private Subnets**: Use private subnets for virtual nodes and pods to minimize exposure.
- **Network Security Groups (NSGs)**:
  - Configure NSGs (up to 5) for granular control of pod and node traffic.
  - Example: Allow ingress to specific pod ports (e.g., `80`, `443`) from trusted sources.
- **Cordon and Drain**: Use before deleting node pools to avoid workload disruptions.
- **Monitoring**: Regularly review node pool and pod resource usage in the OCI Console.

## Key Notes
- **Enhanced Clusters Only**: Virtual nodes are exclusive to Enhanced clusters.
- **Serverless Management**: OCI handles scaling, upgrades, and patching, reducing operational overhead.
- **Pod-Level Resources**: Fine-grained control via pod specs optimizes resource usage.
- **Networking**: VCN-Native Pod Networking requires private subnets and specific route rules.
- **Official Documentation**: Refer to [OCI official documentation](https://docs.oracle.com) under “Container Engine for Kubernetes” > “Working with Virtual Nodes” for detailed instructions.

## Next Steps
- **Practical Demo**: Create and manage a virtual node pool, deploy pods, and monitor resource usage.
- **Scaling Strategies**: Explore automatic scaling with virtual nodes for dynamic workloads.
- **Troubleshooting**: Learn to diagnose resource allocation and networking issues with virtual nodes.

This guide provides a comprehensive overview of managing virtual nodes and node pools in OKE, along with detailed resource allocation strategies for pods. By following these steps, you can ensure a robust, efficient, and high-performing Kubernetes environment tailored to your application needs.