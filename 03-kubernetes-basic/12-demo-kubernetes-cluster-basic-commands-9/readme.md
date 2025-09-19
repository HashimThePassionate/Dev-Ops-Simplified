# **Kubernetes Node Operation Commands**

<details>
<summary>Table of Contents</summary>

- [**Kubernetes Node Operation Commands**](#kubernetes-node-operation-commands)
  - [Prerequisites](#prerequisites)
  - [Node Operation Commands](#node-operation-commands)
    - [1. `kubectl get nodes`](#1-kubectl-get-nodes)
    - [2. `kubectl describe node <node-name>`](#2-kubectl-describe-node-node-name)
    - [3. `kubectl get pods -o wide | grep <node-ip>`](#3-kubectl-get-pods--o-wide--grep-node-ip)
  - [Running Commands in OCI Cloud Shell](#running-commands-in-oci-cloud-shell)
  - [Summary](#summary)

</details>

<br/>

This guide covers `kubectl` commands for managing and inspecting nodes in a Kubernetes cluster, demonstrated using the OCI Cloud Shell with an Oracle Kubernetes Engine (OKE) cluster (e.g., OKE-DEMO-1). These commands focus on retrieving node details, resource allocations, and pod distributions across nodes.

## Prerequisites
- A configured Kubernetes cluster (e.g., OKE-DEMO-1 in OCI).
- Access to OCI Cloud Shell with `kubectl` pre-configured.
- A node pool in OKE-DEMO-1 with three worker nodes (e.g., with IPs `10.0.10.24`, `10.0.10.35`, `10.0.10.95`).
- Existing deployments (e.g., `my-deployment` with 3 pods, as created in previous lessons).

## Node Operation Commands

### 1. `kubectl get nodes`
- **Purpose**: Lists all worker nodes in the Kubernetes cluster.
- **Command**:
  ```bash
  kubectl get nodes
  ```
- **Output**: Displays the three nodes in the OKE-DEMO-1 node pool (e.g., with IPs `10.0.10.24`, `10.0.10.35`, `10.0.10.95`).
- **Usage**:
  - Verifies the nodes available in the cluster.
  - Matches the node pool configuration in the OCI console, which shows three worker nodes.

### 2. `kubectl describe node <node-name>`
- **Purpose**: Displays detailed resource allocation information for a specific node.
- **Command**:
  ```bash
  kubectl describe node <node-name>
  ```
- **Output**:
  - Includes summary columns for:
    - **Requests**: CPU and memory requested by pods on the node.
    - **Limits**: CPU and memory limits set for pods on the node.
  - Shows the total resource requests and limits for all pods running on the node.
- **Usage**:
  - Run for each node (e.g., `10.0.10.24`) to inspect resource utilization.
  - Example:
    ```bash
    kubectl describe node 10.0.10.24
    ```

### 3. `kubectl get pods -o wide | grep <node-ip>`
- **Purpose**: Lists pods running on a specific node, filtered by the node’s IP address.
- **Command**:
  ```bash
  kubectl get pods -o wide | grep <node-ip>
  ```
- **Example Workflow**:
  - List pods for node `10.0.10.24`:
    ```bash
    kubectl get pods -o wide | grep 10.0.10.24
    ```
    - Output: Shows one pod (e.g., `my-deployment-xyz-fkbnw`) running on node `10.0.10.24`, with details like age (e.g., 11 minutes).
  - List pods for node `10.0.10.35`:
    ```bash
    kubectl get pods -o wide | grep 10.0.10.35
    ```
    - Output: Shows another pod (e.g., `my-deployment-xyz-gddk9`) running on node `10.0.10.35`.
  - List pods for node `10.0.10.95`:
    ```bash
    kubectl get pods -o wide | grep 10.0.10.95
    ```
    - Output: Shows a third pod (e.g., `my-deployment-xyz-abc123`) running on node `10.0.10.95`.
- **Verification**:
  - Compare with all pods:
    ```bash
    kubectl get pods -o wide
    ```
    - Output: Confirms three pods from `my-deployment` are evenly distributed across the three nodes:
      - `my-deployment-xyz-fkbnw` on `10.0.10.24`
      - `my-deployment-xyz-gddk9` on `10.0.10.35`
      - `my-deployment-xyz-abc123` on `10.0.10.95`
- **Details**: The `-o wide` flag provides additional details like node IPs, and `grep` filters pods by node IP.

## Running Commands in OCI Cloud Shell
1. **Access Cloud Shell**: Open the OCI Cloud Shell, pre-configured with `kubectl`.
2. **Execute Commands**:
   - Run `kubectl get nodes` to list all nodes.
   - Use `kubectl describe node` to check resource allocations.
   - Run `kubectl get pods -o wide | grep <node-ip>` to view pods per node.
   - Clear the screen (`clear`) between commands for readability.
3. **Troubleshooting**:
   - Verify node details in the OCI console under node pools to confirm the number of nodes (e.g., three worker nodes).
   - Use `kubectl get pods -o wide` to ensure pods are distributed as expected across nodes.
   - Check resource usage with `kubectl describe node` for potential resource bottlenecks.
4. **Notes**:
   - Ensure you are in the correct context (e.g., OKE-DEMO-1) using `kubectl config current-context`.
   - The Kubernetes scheduler distributes pods across nodes for load balancing, as seen in the even distribution of `my-deployment` pods.

## Summary
These `kubectl` commands (`get nodes`, `describe node`, `get pods -o wide | grep`) enable you to manage and inspect node operations in a Kubernetes cluster. They allow listing nodes, viewing resource allocations, and checking pod distribution across nodes. Demonstrated in the OCI Cloud Shell, these commands are essential for monitoring and managing worker nodes in an OKE cluster.