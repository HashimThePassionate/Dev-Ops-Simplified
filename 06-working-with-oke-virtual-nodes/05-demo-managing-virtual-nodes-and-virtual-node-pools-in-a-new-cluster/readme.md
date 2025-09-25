# **Demo: Managing Virtual Nodes and Virtual Node Pools in an OKE Cluster**

This guide, presented by Mahendra Mehra, Senior Training Lead and Evangelist at Oracle University, demonstrates how to manage **virtual nodes** and **virtual node pools** in an Oracle Container Engine for Kubernetes (OKE) cluster. The demo uses the existing `OKE-VN-PUBLIC-2` Enhanced cluster, created with virtual nodes, to show how to add, update, delete, and list virtual node pools and nodes. The focus is on scaling the cluster and managing resources effectively using the OCI Console.

<details>
<summary><strong>📋 Table of Contents</strong></summary>

- [**Demo: Managing Virtual Nodes and Virtual Node Pools in an OKE Cluster**](#demo-managing-virtual-nodes-and-virtual-node-pools-in-an-oke-cluster)
  - [Prerequisites](#prerequisites)
  - [Step-by-Step Process](#step-by-step-process)
    - [1. Add a New Virtual Node Pool](#1-add-a-new-virtual-node-pool)
    - [2. View Virtual Node Pool Details](#2-view-virtual-node-pool-details)
    - [3. Update a Virtual Node Pool](#3-update-a-virtual-node-pool)
    - [4. Delete a Virtual Node Pool](#4-delete-a-virtual-node-pool)
    - [5. Verify Virtual Nodes](#5-verify-virtual-nodes)
  - [Security Best Practices](#security-best-practices)
  - [Key Notes](#key-notes)
  - [Next Steps](#next-steps)

</details>

## Prerequisites
- An OKE **Enhanced cluster** named `OKE-VN-PUBLIC-2` in the `CTDOKE` compartment, running Kubernetes version `1.27.2`.
- **Virtual Node Pool**: Existing pool (`pool1`) with 3 virtual nodes, using `Pod.Standard.E3.Flex` shape and a taint (`app=frontend:NoSchedule`).
- **IAM Policies**: Mandatory policies for virtual nodes in the root compartment (as configured in the previous demo).
  ```plaintext
  Allow service OKE to manage container-instances in tenancy
  Allow service OKE to manage vnics in tenancy
  Allow group <GroupName> to manage cluster-virtual-node-pools in tenancy
  Allow group <GroupName> to read virtual-network-family in tenancy
  Allow group <GroupName> to manage vnics in tenancy
  ```
- **Networking**: Public Kubernetes API endpoint, private regional subnet for virtual nodes and pods (recommended to be the same).
- Administrative privileges in the `CTDOKE` compartment (or specific policies for non-administrator users).
- Access to the OCI Console.
- The current date and time: **04:57 AM PKT, Friday, September 26, 2025**.

## Step-by-Step Process

### 1. Add a New Virtual Node Pool
1. **Navigate to Node Pools**:
   - In the OCI Console, go to **Developer Services** > **Kubernetes Cluster (OKE)** > Select `OKE-VN-PUBLIC-2`.
   - Under **Resources**, click **Node Pools**.
   - Verify the existing node pool (`pool1`):
     - **Node Type**: Virtual.
     - **Total Worker Nodes**: 3.
   - Click **Add Node Pool**.

2. **Configure New Node Pool**:
   - **Name**: `pool2`.
   - **Compartment**: `CTDOKE` (default).
   - **Node Type**: Select **Virtual Nodes**.
   - **Node Count**: `1` (to demonstrate a small pool).
   - **Pod Shape**: Select `Pod.Standard.E4.Flex` (different from `pool1`’s `E3.Flex`).
   - **Pod Communication**:
     - **Pod Subnet**: Select the private regional subnet (e.g., `oke-vnsubnet-...`).
     - **Note**: Oracle recommends using the same subnet for pods and virtual nodes.
   - **Virtual Node Communication**:
     - **Node Subnet**: Select the same private regional subnet as the pod subnet.
   - **Node Placement Configuration**:
     - **Availability Domain**: Select one AD (e.g., `AD-1`).
     - **Fault Domain**: Select a fault domain (e.g., `FD-1`).
     - **Note**: Since only one node is specified, a single AD and FD are sufficient. For multiple nodes, distribute across ADs for high availability.
   - Click **Add**.
   - **Note**: The taint (`app=frontend:NoSchedule`) from the cluster creation is inherited unless overridden.

3. **Monitor Creation**:
   - Check **Work Requests** in the cluster details page.
   - Verify the `VIRTUALNODEPOOL-CREATE` operation is in progress.
   - Wait for the status to change to **Completed** (takes a few minutes).
   - Return to **Node Pools**:
     - Confirm `pool2` is listed with:
       - **Node Type**: Virtual.
       - **Total Worker Nodes**: 1.
       - **Shape**: `Pod.Standard.E4.Flex`.

### 2. View Virtual Node Pool Details
1. **Access Node Pool Details**:
   - In **Node Pools**, click `pool1` or `pool2`.
   - Details displayed:
     - **Node Type**: Virtual.
     - **Node Pool ID**: Unique OCID.
     - **Placement Configuration**: AD (e.g., `AD-1`), subnet, fault domain.
     - **Shape**: `Pod.Standard.E3.Flex` (for `pool1`) or `Pod.Standard.E4.Flex` (for `pool2`).
     - **Taints**: `app=frontend:NoSchedule`.
     - **Tags**: Creator, creation date, principal type.

2. **List Virtual Nodes**:
   - In **Node Pools**, select `pool1` > Click **Virtual Nodes** (or the worker nodes link).
   - Displays:
     - **Node State**: Active.
     - **Kubernetes Version**: `1.27.2`.
     - **Node ID**: Unique OCID.
     - **Subnet**: Private regional subnet.
     - **Placement**: AD and fault domain.
   - Repeat for `pool2` (shows 1 virtual node).

### 3. Update a Virtual Node Pool
1. **Edit Node Pool**:
   - In **Node Pools**, select `pool1` > Click **Edit**.
   - Editable fields:
     - **Name**: e.g., Change to `prod-virtual-pool-1`.
     - **Node Count**: Increase from 3 to 4 to scale up.
     - **Placement Configuration**: Add or modify ADs/fault domains.
     - **Subnets**: Update pod or virtual node subnets (must remain private).
     - **Labels/Taints**: Add new labels (e.g., `environment=production`) or modify taints.
   - For this demo:
     - Change **Node Count** to `4`.
     - Leave other settings unchanged.
   - Click **Save Changes**.

2. **Monitor Update**:
   - Check **Work Requests** for the `VIRTUALNODEPOOL-UPDATE` operation.
   - Wait for the status to change to **Completed**.
   - Verify in **Node Pools**:
     - `pool1` now shows **Total Worker Nodes**: 4.
   - Click **Virtual Nodes** to list all 4 nodes, confirming the new node is active.

### 4. Delete a Virtual Node Pool
1. **Initiate Deletion**:
   - In **Node Pools**, select `pool2` > Click **Delete Node Pool**.
   - Confirm by entering the pool name (`pool2`).
   - Click **Show Advanced Options**:
     - **Eviction Grace Period**: Default is 60 minutes; change to `30` minutes for faster termination.
     - **Force Terminate After Grace Period**: Enable to ensure nodes are terminated even if not fully drained.
   - Click **Delete**.

2. **Monitor Deletion**:
   - Check **Work Requests** for the `VIRTUALNODEPOOL-DELETE` operation.
   - Verify the status changes to **Deleting** in **Node Pools**.
   - Wait for completion (takes a few minutes).
   - Confirm `pool2` is removed from the **Node Pools** list.

3. **Cordon and Drain**:
   - **Cordon**: Prevents new pods from scheduling on nodes (automatic during deletion).
   - **Drain**: Gracefully evicts pods, redistributing them to other nodes.
   - **Note**: The eviction grace period (e.g., 30 minutes) ensures workloads are migrated before termination. Enabling **Force Terminate** ensures deletion proceeds even if draining fails.

### 5. Verify Virtual Nodes
1. **List Virtual Nodes**:
   - In **Node Pools**, select `pool1` > Click **Virtual Nodes**.
   - Confirms 4 active virtual nodes with:
     - **Kubernetes Version**: `1.27.2`.
     - **Subnet**: Private regional subnet.
     - **Placement**: Distributed across ADs/fault domains.
   - Click a node to view detailed information (e.g., node ID, status).

## Security Best Practices
- **IAM Policies**: Verify mandatory policies are in place in the root compartment; restrict non-administrator access to specific compartments.
- **Private Subnets**: Ensure virtual node and pod subnets are private for security.
- **Network Security Groups (NSGs)**: Use NSGs (up to 5) for granular control of pod and node traffic.
- **Cordon and Drain**: Always use during node pool deletion to minimize workload disruptions.
- **Taints**: Leverage taints (e.g., `app=frontend:NoSchedule`) to control pod scheduling.
- **Monitoring**: Regularly check node pool and node statuses in the OCI Console.

## Key Notes
- **Enhanced Clusters Only**: Virtual nodes are exclusive to Enhanced clusters.
- **Automatic Management**: OCI handles scaling, upgrades, and patching for virtual nodes.
- **Subnet Recommendation**: Use the same private regional subnet for virtual nodes and pods to simplify networking.
- **Taints and Tolerations**: Inherited from cluster creation unless overridden at the node pool level.
- **Scaling**: Add/remove node pools or adjust node counts to scale the cluster; virtual nodes support up to 500 pods per node.
- **Official Documentation**: Refer to [OCI documentation](https://docs.oracle.com) under “Container Engine for Kubernetes” > “Working with Virtual Nodes” for detailed instructions.

## Next Steps
- **Pod Deployment**: Deploy pods with matching tolerations to test scheduling on the updated node pool.
- **Autoscaling**: Explore automatic scaling of virtual node pools for dynamic workloads.
- **Advanced Management**: Experiment with labels, taints, and NSG configurations for fine-grained control.

This demo successfully demonstrated adding a new virtual node pool (`pool2`), updating an existing pool (`pool1`) by scaling up to 4 nodes, deleting a node pool, and listing virtual nodes in the `OKE-VN-PUBLIC-2` cluster. These steps showcase effective management of virtual nodes and node pools in OKE.