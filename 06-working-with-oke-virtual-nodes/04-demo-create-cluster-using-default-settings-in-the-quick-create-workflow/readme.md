# **Demo: Creating an OKE Cluster with Virtual Nodes Using Quick Create Workflow**

create an Oracle Container Engine for Kubernetes (OKE) cluster using **virtual nodes** and a **virtual node pool** with default settings via the **Quick Create workflow**. The demo includes configuring the cluster, verifying IAM policies, accessing the cluster, and testing pod creation with taints and tolerations. The Kubernetes API endpoint is placed in a **public subnet**, and the virtual nodes are in a **private regional subnet**.

## 📋 Table of Contents

<details>
<summary>Click to expand</summary>

- [**Demo: Creating an OKE Cluster with Virtual Nodes Using Quick Create Workflow**](#demo-creating-an-oke-cluster-with-virtual-nodes-using-quick-create-workflow)
  - [📋 Table of Contents](#-table-of-contents)
  - [Prerequisites](#prerequisites)
  - [IAM Policies for Virtual Nodes](#iam-policies-for-virtual-nodes)
  - [Step-by-Step Process](#step-by-step-process)
    - [1. Create the OKE Cluster](#1-create-the-oke-cluster)
    - [2. Access the Cluster](#2-access-the-cluster)
    - [3. Test Pod Creation](#3-test-pod-creation)
  - [Security Best Practices](#security-best-practices)
  - [Key Notes](#key-notes)
  - [Next Steps](#next-steps)

</details>

## Prerequisites
- **OCI Console** access with administrative privileges in the `CTDOKE` compartment or specific IAM policies for non-administrator users.
- **Region**: US West (Phoenix).
- **IAM Policies**: Mandatory policies for virtual nodes in the root compartment (see below).
- **Local Terminal**: Configured with OCI CLI and `kubectl` for cluster access.
- **Pod Specification Files**: YAML files for testing pod creation with tolerations.
- The current date and time: **04:53 AM PKT, Friday, September 26, 2025**.

## IAM Policies for Virtual Nodes
- **Mandatory Policies** (Root Compartment):
  ```plaintext
  Allow service OKE to manage container-instances in tenancy
  Allow service OKE to manage vnics in tenancy
  Allow group <GroupName> to manage cluster-virtual-node-pools in tenancy
  Allow group <GroupName> to read virtual-network-family in tenancy
  Allow group <GroupName> to manage vnics in tenancy
  ```
  - **Purpose**: Enables OKE to create container instances and VNICs for virtual nodes and allows users to manage virtual node pools.
  - **Location**: Create in the **root compartment** via **Identity & Security** > **Policies** > Select root compartment > **Create Policy**.
  - **Name**: e.g., `OKE-Virtual-Nodes-Policy`.
  - **Source**: Copy from [OCI documentation](https://docs.oracle.com) under “Container Engine for Kubernetes” > “Working with Virtual Nodes.”
- **Non-Administrator Users**:
  - Additional policies may be required in the target compartment (e.g., `CTDOKE`) for managing clusters and node pools (see previous lessons).
- **Demo Note**: The demo uses administrative privileges, granting full access to cluster resources.

## Step-by-Step Process

### 1. Create the OKE Cluster
1. **Navigate to OKE Console**:
   - In the OCI Console, go to **Developer Services** > **Kubernetes Cluster (OKE)**.
   - Ensure the `CTDOKE` compartment and **US West (Phoenix)** region are selected.
   - Click **Create Cluster**.

2. **Select Quick Create Workflow**:
   - Choose **Quick Create** (default) > Click **Submit**.

3. **Configure Cluster**:
   - **Name**: `OKE-VN-Public`.
     - Indicates virtual nodes (VN) and public Kubernetes API endpoint.
   - **Compartment**: `CTDOKE` (default).
   - **Kubernetes Version**: `1.27.2` (default).
   - **Kubernetes API Endpoint**: **Public Endpoint** (default).
   - **Node Type**: Select **Virtual Nodes**.
     - **Note**: Virtual nodes require the IAM policies listed above; without them, pod creation will fail.
   - **Node Count**: `3` (default, distributes nodes across availability domains or fault domains in a single-AD region).
   - **Pod Shape**: Select `Pod.Standard.E3.Flex` (default).
     - Available shapes: `Pod.Standard.E3.Flex`, `Pod.Standard.E4.Flex`, `Pod.Standard.A1.Flex` (depends on tenancy).
     - **Note**: CPU and memory are specified in pod specification files, not at the node pool level.
   - **Advanced Options**:
     - Add a **Taint**:
       - **Key**: `app`.
       - **Value**: `frontend`.
       - **Effect**: `NoSchedule`.
         - **NoSchedule**: Prevents pods without matching tolerations from being scheduled.
         - **NoExecute**: Evicts running pods without tolerations and prevents scheduling.
       - Purpose: Ensures only pods with matching tolerations (e.g., `app=frontend`) can run on these nodes.
     - Labels are optional for workload targeting but not used in this demo.
   - Click **Next**.

4. **Review and Create**:
   - Verify settings:
     - **Cluster Name**: `OKE-VN-Public`.
     - **Cluster Type**: Enhanced (required for virtual nodes).
     - **Kubernetes Version**: `1.27.2`.
     - **Node Pool**: Virtual node pool with 3 nodes, `Pod.Standard.E3.Flex` shape.
     - **Taint**: `app=frontend:NoSchedule`.
     - **Network**: Public API endpoint, private regional subnet for virtual nodes/pods.
   - Click **Create Cluster**.
   - Wait for the cluster status to change to **Active** (takes a few minutes).

### 2. Access the Cluster
1. **Generate kubeconfig File**:
   - In the OCI Console, go to the cluster (`OKE-VN-Public`) > Click **Access Cluster**.
   - Select **Cloud Shell Access** (available since the API endpoint is public).
   - Copy the `kubeconfig` command:
     ```bash
     oci ce cluster create-kubeconfig --cluster-id <cluster-ocid> --file $HOME/.kube/config --region us-phoenix-1
     ```
   - In Cloud Shell:
     ```bash
     oci ce cluster create-kubeconfig --cluster-id <cluster-ocid> --file $HOME/.kube/config --region us-phoenix-1
     ```
     - Confirms the `kubeconfig` file is merged into `~/.kube/config`.

2. **Verify Cluster Access**:
   - In Cloud Shell:
     ```bash
     kubectl get nodes
     ```
     - Expected Output:
       ```
       NAME                STATUS   ROLES        AGE   VERSION
       virtual-node-1      Ready    virtual      5m    v1.27.2
       virtual-node-2      Ready    virtual      5m    v1.27.2
       virtual-node-3      Ready    virtual      5m    v1.27.2
       ```
     - Confirms 3 virtual nodes in the virtual node pool, marked as `virtual` under the **ROLES** column.

### 3. Test Pod Creation
1. **Create a Deployment Without Tolerations**:
   - In Cloud Shell:
     ```bash
     kubectl create deployment nginx --image=nginx:latest
     ```
     - Output: `deployment.apps/nginx created`.
   - Check pod status:
     ```bash
     kubectl get pods
     ```
     - Output:
       ```
       NAME                     READY   STATUS    RESTARTS   AGE
       nginx-<hash>             0/1     Pending   0          1m
       ```
     - **Reason**: The pod is in `Pending` state due to the taint `app=frontend:NoSchedule` on the virtual nodes, with no matching toleration.
   - Describe the pod:
     ```bash
     kubectl describe pod nginx-<hash>
     ```
     - Output (Events):
       ```
       Events:
         Warning  FailedScheduling  0/3 nodes are available: 3 node(s) had untolerated taint {app: frontend}.
       ```

2. **Create a Pod with Matching Tolerations**:
   - Pod specification file (`virtual-pod-1.yaml`):
     ```yaml
     apiVersion: v1
     kind: Pod
     metadata:
       name: nginx-test
     spec:
       containers:
       - name: nginx
         image: nginx:latest
         resources:
           requests:
             cpu: "1"
             memory: "128Mi"
           limits:
             cpu: "2"
             memory: "512Mi"
       tolerations:
       - key: "app"
         operator: "Equal"
         value: "frontend"
         effect: "NoSchedule"
     ```
   - Apply the pod:
     ```bash
     cd ~/kubernetes-examples
     kubectl create -f virtual-pod-1.yaml
     ```
     - Output: `pod/nginx-test created`.
   - Check pod status:
     ```bash
     kubectl get pods
     ```
     - Output:
       ```
       NAME                     READY   STATUS              RESTARTS   AGE
       nginx-<hash>             0/1     Pending             0          2m
       nginx-test               0/1     ContainerCreating   0          10s
       ```
     - After a short wait:
       ```
       NAME                     READY   STATUS    RESTARTS   AGE
       nginx-<hash>             0/1     Pending   0          3m
       nginx-test               1/1     Running   0          1m
       ```
     - **Reason**: The `nginx-test` pod runs because it includes a toleration matching the taint (`app=frontend:NoSchedule`).

3. **Create a Pod with Non-Matching Tolerations**:
   - Pod specification file (`virtual-pod-2.yaml`):
     ```yaml
     apiVersion: v1
     kind: Pod
     metadata:
       name: mysql
     spec:
       containers:
       - name: mysql
         image: mysql:latest
         resources:
           requests:
             cpu: "1"
             memory: "128Mi"
           limits:
             cpu: "2"
             memory: "512Mi"
       tolerations:
       - key: "app"
         operator: "Equal"
         value: "backend"
         effect: "NoSchedule"
     ```
   - Apply the pod:
     ```bash
     kubectl create -f virtual-pod-2.yaml
     ```
     - Output: `pod/mysql created`.
   - Check pod status:
     ```bash
     kubectl get pods
     ```
     - Output:
       ```
       NAME                     READY   STATUS    RESTARTS   AGE
       nginx-<hash>             0/1     Pending   0          5m
       nginx-test               1/1     Running   0          2m
       mysql                    0/1     Pending   0          10s
       ```
     - Describe the pod:
       ```bash
       kubectl describe pod mysql
       ```
       - Output (Events):
         ```
         Events:
           Warning  FailedScheduling  0/3 nodes are available: 3 node(s) had untolerated taint {app: frontend}.
         ```
     - **Reason**: The `mysql` pod remains in `Pending` state because its toleration (`app=backend:NoSchedule`) does not match the taint (`app=frontend:NoSchedule`).

## Security Best Practices
- **IAM Policies**: Ensure the mandatory policies are in the root compartment; restrict non-administrator access to specific compartments.
- **Public API Endpoint**: Since the API endpoint is public, secure it with Network Security Groups (NSGs) or security lists to limit access (e.g., to specific CIDRs).
- **Private Subnets**: Virtual nodes and pods use private regional subnets for security.
- **Taints and Tolerations**: Use taints to control pod scheduling and prevent unauthorized workloads.
- **Resource Limits**: Set equal requests and limits in pod specs to align with OKE’s assured capacity model.
- **Monitoring**: Regularly check pod and node statuses in the OCI Console or with `kubectl`.

## Key Notes
- **Quick Create Workflow**: Automatically configures network resources (VCN, subnets, security lists) and a virtual node pool with default settings.
- **Virtual Nodes**: Supported only in Enhanced clusters, managed by OCI for scaling and upgrades.
- **Taints and Tolerations**:
  - **NoSchedule**: Prevents scheduling of pods without matching tolerations.
  - **NoExecute**: Evicts running pods without tolerations and prevents scheduling.
  - Taints are exclusive to virtual nodes in OKE for precise workload control.
- **Pod Specifications**:
  - CPU/memory requests and limits are mandatory for virtual nodes.
  - Minimum: `0.125 OCPU`, `0.5 GB` memory per container.
- **Networking**: Pods use VCN-Native Pod Networking; the pod subnet is private and ideally matches the virtual node subnet.
- **Official Documentation**: Refer to [OCI documentation](https://docs.oracle.com) under “Container Engine for Kubernetes” > “Working with Virtual Nodes” for setup and policy details.

## Next Steps
- **Scaling Demo**: Explore scaling virtual node pools and pods for dynamic workloads.
- **Advanced Taints/Tolerations**: Test complex taint scenarios and toleration configurations.
- **Resource Optimization**: Fine-tune pod specifications for CPU, memory, and network performance.

This demo successfully created an OKE cluster (`OKE-VN-Public`) with a virtual node pool using the Quick Create workflow, configured a taint (`app=frontend:NoSchedule`), and tested pod creation with matching and non-matching tolerations. The results demonstrate the importance of tolerations for scheduling pods on tainted virtual nodes.