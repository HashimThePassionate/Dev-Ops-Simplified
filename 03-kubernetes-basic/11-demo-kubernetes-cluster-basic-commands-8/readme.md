# **Kubernetes Delete Commands**

This guide covers `kubectl` commands for deleting resources in a Kubernetes cluster, demonstrated using the OCI Cloud Shell with an Oracle Kubernetes Engine (OKE) cluster (e.g., OKE-DEMO-1). These commands allow you to remove pods, deployments, and namespaces, either individually or by label selectors.

<details>
<summary>Table of Contents</summary>

- [**Kubernetes Delete Commands**](#kubernetes-delete-commands)
  - [Prerequisites](#prerequisites)
  - [Delete Commands](#delete-commands)
    - [1. `kubectl delete pod -l <label>`](#1-kubectl-delete-pod--l-label)
    - [2. `kubectl delete pod --all`](#2-kubectl-delete-pod---all)
    - [3. `kubectl delete deployment <name>`](#3-kubectl-delete-deployment-name)
    - [4. `kubectl delete namespace <name1> <name2>`](#4-kubectl-delete-namespace-name1-name2)
  - [Running Commands in OCI Cloud Shell](#running-commands-in-oci-cloud-shell)
  - [Summary](#summary)

</details>

<br/>

## Prerequisites
- A configured Kubernetes cluster (e.g., OKE-DEMO-1 in OCI).
- Access to OCI Cloud Shell with `kubectl` pre-configured.
- Existing resources in OKE-DEMO-1, including:
  - Deployments (e.g., `my-deployment` with 3 replicas, `my-deployment-1` with 2 replicas).
  - Pods with labels (e.g., `app=nginx` for `my-deployment`).
  - Namespaces (e.g., `development`, `production`).

## Delete Commands

### 1. `kubectl delete pod -l <label>`
- **Purpose**: Deletes pods matching a specific label selector.
- **Command**:
  ```bash
  kubectl delete pod -l app=nginx
  ```
- **Details**:
  - Uses the `-l` flag to target pods with a specific label (e.g., `app=nginx`).
  - Deletes all pods matching the label (e.g., pods in `my-deployment`).
  - If pods are managed by a deployment, the replica set recreates them to maintain the desired number of replicas (e.g., 3 pods for `my-deployment`).
- **Example Workflow**:
  - Run the delete command:
    ```bash
    kubectl delete pod -l app=nginx
    ```
    - Output: Deletes all pods with `app=nginx` (e.g., pods in `my-deployment`).
  - Verify pods:
    ```bash
    kubectl get pods
    ```
    - Output: Shows new pods created by the `my-deployment` replica set to replace the deleted ones, maintaining 3 replicas.
- **Usage**: Useful for deleting multiple pods matching a label without affecting the deployment’s replica set.

### 2. `kubectl delete pod --all`
- **Purpose**: Deletes all pods in the current namespace, including initialized and uninitialized pods.
- **Command**:
  ```bash
  kubectl delete pod --all
  ```
- **Details**:
  - Removes all pods, regardless of their state.
  - Pods managed by deployments or replica sets will be recreated to meet the desired replica count.
- **Note**: Use with caution, as this affects all pods in the namespace.

### 3. `kubectl delete deployment <name>`
- **Purpose**: Deletes a specific deployment in the cluster.
- **Command**:
  ```bash
  kubectl delete deployment my-deployment-1
  ```
- **Example Workflow**:
  - Run the delete command:
    ```bash
    kubectl delete deployment my-deployment-1
    ```
    - Output: `deployment.apps/my-deployment-1 deleted`
  - Verify deployments:
    ```bash
    kubectl get deployments
    ```
    - Output: Lists only remaining deployments (e.g., `my-deployment`).
- **Details**: Deletes the deployment and its associated pods and replica sets.

### 4. `kubectl delete namespace <name1> <name2>`
- **Purpose**: Deletes one or more namespaces in the cluster.
- **Command**:
  ```bash
  kubectl delete namespace development production
  ```
- **Example Workflow**:
  - Run the delete command:
    ```bash
    kubectl delete namespace development production
    ```
    - Output: `namespace "development" deleted` and `namespace "production" deleted`
  - Verify namespaces:
    ```bash
    kubectl get namespace
    ```
    - Output: Shows remaining namespaces (e.g., `default`, `kube-system`), with `development` and `production` removed.
- **Details**: Deletes the specified namespaces and all resources within them (e.g., pods, services, deployments).

## Running Commands in OCI Cloud Shell
1. **Access Cloud Shell**: Open the OCI Cloud Shell, pre-configured with `kubectl`.
2. **Execute Commands**:
   - Run `kubectl delete` commands to remove pods, deployments, or namespaces.
   - Clear the screen (`clear`) between commands for readability.
3. **Troubleshooting**:
   - Verify deletions with `kubectl get pods`, `kubectl get deployments`, or `kubectl get namespace`.
   - For pods managed by deployments, check `kubectl get pods` to confirm recreation by the replica set.
   - Ensure you are in the correct context (e.g., OKE-DEMO-1) using `kubectl config current-context`.
4. **Notes**:
   - Deleting pods managed by a deployment does not permanently remove them due to replica sets.
   - Deleting a namespace removes all its resources; use cautiously.

## Summary
These `kubectl` delete commands (`delete pod -l`, `delete pod --all`, `delete deployment`, `delete namespace`) enable you to remove resources from a Kubernetes cluster. You can delete pods by label, all pods in a namespace, specific deployments, or entire namespaces. Demonstrated in the OCI Cloud Shell, these commands are essential for cleaning up resources in an OKE cluster, with considerations for automatic pod recreation by deployments.