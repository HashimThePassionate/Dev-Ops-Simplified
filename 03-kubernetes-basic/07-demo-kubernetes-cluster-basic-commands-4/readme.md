# **Kubernetes Deployment Management Commands**

<details>
<summary>Table of Contents</summary>

- [**Kubernetes Deployment Management Commands**](#kubernetes-deployment-management-commands)
  - [Prerequisites](#prerequisites)
  - [Deployment Management Commands](#deployment-management-commands)
    - [1. `kubectl get deployments`](#1-kubectl-get-deployments)
    - [2. `kubectl get deployment <name> -o yaml`](#2-kubectl-get-deployment-name--o-yaml)
    - [3. `kubectl scale deployment <name> --replicas=<number>`](#3-kubectl-scale-deployment-name---replicasnumber)
    - [4. `kubectl rollout status deployment/<name>`](#4-kubectl-rollout-status-deploymentname)
    - [5. `kubectl set image deployment/<name> <container-name>=<image>`](#5-kubectl-set-image-deploymentname-container-nameimage)
  - [Running Commands in OCI Cloud Shell](#running-commands-in-oci-cloud-shell)
  - [Summary](#summary)

</details>

<br/>

This guide covers `kubectl` commands for managing deployments in a Kubernetes cluster, demonstrated using the OCI Cloud Shell with an Oracle Kubernetes Engine (OKE) cluster (e.g., OKE-DEMO-1). These commands help list, inspect, scale, and update deployments.

## Prerequisites
- A configured Kubernetes cluster (e.g., OKE-DEMO-1 in OCI).
- Access to OCI Cloud Shell with `kubectl` pre-configured.
- Existing deployments (e.g., `my-deployment` with 4 replicas and `my-deployment-1` with 2 replicas, as created in previous lessons).

## Deployment Management Commands

### 1. `kubectl get deployments`
- **Purpose**: Lists all deployments in the current namespace.
- **Command**:
  ```bash
  kubectl get deployments
  ```
- **Output**:
  - Displays deployments in the default namespace (e.g., `my-deployment` with 4 replicas, `my-deployment-1` with 2 replicas).
  - Includes details like the number of ready replicas and the age of each deployment.
- **Usage**: Run to view the current state of deployments in the cluster.

### 2. `kubectl get deployment <name> -o yaml`
- **Purpose**: Retrieves detailed information about a specific deployment in YAML format.
- **Command**:
  ```bash
  kubectl get deployment my-deployment -o yaml
  ```
- **Output**: Provides a detailed YAML description of the deployment, including its configuration, replicas, and container details.
- **Usage**: Useful for inspecting the full configuration of a deployment like `my-deployment`.

### 3. `kubectl scale deployment <name> --replicas=<number>`
- **Purpose**: Scales the number of replicas in a deployment, replica set, or stateful set.
- **Command**:
  ```bash
  kubectl scale deployment my-deployment --replicas=6
  ```
- **Example Workflow**:
  - Check current deployment state:
    ```bash
    kubectl get deployments
    ```
    - Output: Shows `my-deployment` with 4 replicas.
  - Scale to 6 replicas:
    ```bash
    kubectl scale deployment my-deployment --replicas=6
    ```
    - Output: `deployment.apps/my-deployment scaled`
  - Verify scaling:
    ```bash
    kubectl get deployments
    kubectl get pods
    ```
    - Output: Confirms `my-deployment` now has 6 pods running.
- **Details**: Allows scaling up or down based on the specified replica count.

### 4. `kubectl rollout status deployment/<name>`
- **Purpose**: Checks the current status of a deployment’s rollout.
- **Command**:
  ```bash
  kubectl rollout status deployment/my-deployment
  ```
- **Output**: `deployment "my-deployment" successfully rolled out`
- **Usage**: Verifies that changes to the deployment (e.g., scaling or image updates) have been applied successfully.

### 5. `kubectl set image deployment/<name> <container-name>=<image>`
- **Purpose**: Updates the container image for a deployment inline.
- **Command**:
  ```bash
  kubectl set image deployment/my-deployment-1 nginx=nginx:1.24.0
  ```
- **Example Workflow**:
  - Update the image for `my-deployment-1` (created inline in previous lessons):
    ```bash
    kubectl set image deployment/my-deployment-1 nginx=nginx:1.24.0
    ```
    - Output: `deployment.apps/my-deployment-1 image updated`
  - Check rollout status:
    ```bash
    kubectl rollout status deployment/my-deployment-1
    ```
    - Output: `deployment "my-deployment-1" successfully rolled out`
  - Verify the image update:
    ```bash
    kubectl get pods
    kubectl describe pod <pod-name>
    ```
    - Select a pod name from `my-deployment-1` (e.g., via `kubectl get pods`).
    - Run `kubectl describe pod <pod-name>` to confirm the container is using `nginx:1.24.0`.
- **Details**: Updates the container image (e.g., from `nginx:latest` to `nginx:1.24.0`) without modifying a manifest file.

## Running Commands in OCI Cloud Shell
1. **Access Cloud Shell**: Open the OCI Cloud Shell, pre-configured with `kubectl`.
2. **Execute Commands**:
   - Run the commands above to manage deployments.
   - Clear the screen (`clear`) between commands for readability.
3. **Verify Changes**:
   - Use `kubectl get deployments` and `kubectl get pods` to confirm replica counts.
   - Use `kubectl describe pod` to verify image updates.
4. **Troubleshooting**:
   - Check rollout status with `kubectl rollout status` to ensure updates are applied.
   - Inspect pod details with `kubectl describe pod` for debugging.

## Summary
These `kubectl` commands (`get deployments`, `get deployment -o yaml`, `scale deployment`, `rollout status`, `set image`) enable you to manage deployments in a Kubernetes cluster. They allow listing deployments, inspecting details, scaling replicas, checking rollout status, and updating container images. Demonstrated in the OCI Cloud Shell, these commands are essential for managing and updating workloads in an OKE cluster.