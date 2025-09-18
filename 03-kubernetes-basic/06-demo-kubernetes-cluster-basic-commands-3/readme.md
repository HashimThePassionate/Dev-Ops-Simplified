# **Kubernetes Pod Management Commands**

This guide covers essential `kubectl` commands for managing pods in a Kubernetes cluster, demonstrated using the OCI Cloud Shell with an Oracle Kubernetes Engine (OKE) cluster (e.g., OKE-DEMO-1). These commands help inspect, troubleshoot, and interact with pods.

## Table of Contents
- [**Kubernetes Pod Management Commands**](#kubernetes-pod-management-commands)
  - [Table of Contents](#table-of-contents)
  - [Prerequisites](#prerequisites)
  - [Pod Management Commands](#pod-management-commands)
    - [1. `kubectl get pods`](#1-kubectl-get-pods)
    - [2. `kubectl describe pod <pod-name>`](#2-kubectl-describe-pod-pod-name)
    - [3. `kubectl logs <pod-name>`](#3-kubectl-logs-pod-name)
    - [4. `kubectl exec <pod-name> -- <command>`](#4-kubectl-exec-pod-name----command)
    - [5. `kubectl exec -it <pod-name> -- /bin/bash`](#5-kubectl-exec--it-pod-name----binbash)
  - [Running Commands in OCI Cloud Shell](#running-commands-in-oci-cloud-shell)
  - [Summary](#summary)

## Prerequisites
- A configured Kubernetes cluster (e.g., OKE-DEMO-1 in OCI).
- Access to OCI Cloud Shell with `kubectl` pre-configured.
- Pods running in the cluster (e.g., pods labeled with `app=nginx`).

## Pod Management Commands

### 1. `kubectl get pods`
- **Purpose**: Lists all pods in the current namespace, providing their names and basic status.
- **Usage**: Use this command to identify pod names for further operations.
- **Command**:
  ```bash
  kubectl get pods
  ```
- **Example**: Run to view available pods, then select a pod name for detailed inspection.

### 2. `kubectl describe pod <pod-name>`
- **Purpose**: Provides detailed information about a specific pod.
- **Details**:
  - Includes namespace, pod name, service account, IP address, container ID, image details (e.g., `nginx:latest` from `docker.io`), and container runtime (e.g., CRI-O in OCI).
  - Shows events related to pod creation, such as image pulling, container creation, and start time (e.g., pulling `nginx:latest` in 5 seconds).
  - Useful for troubleshooting and debugging pod issues.
- **Command**:
  ```bash
  kubectl describe pod <pod-name>
  ```
- **Example**:
  ```bash
  kubectl describe pod my-deployment-xyz
  ```

- **Using Label Selectors**:
  - To describe multiple pods with a specific label (e.g., `app=nginx`):
  - **Command**:
    ```bash
    kubectl describe pod -l app=nginx
    ```
  - **Output**: Displays details for all pods matching the label (e.g., four pods with `app=nginx`).

### 3. `kubectl logs <pod-name>`
- **Purpose**: Retrieves logs from a specific pod for troubleshooting and debugging.
- **Details**:
  - Shows log output to identify errors, exceptions, or application-specific information.
  - Use the `-f` flag to follow logs in real-time, monitoring runtime behavior or ongoing processes.
- **Commands**:
  - Basic log retrieval:
    ```bash
    kubectl logs <pod-name>
    ```
  - Real-time log streaming:
    ```bash
    kubectl logs <pod-name> -f
    ```
- **Example**:
  ```bash
  kubectl logs my-deployment-xyz
  kubectl logs my-deployment-xyz -f
  ```
- **Note**: With `-f`, the terminal remains active, displaying new logs until interrupted (e.g., with `Ctrl+C`).

### 4. `kubectl exec <pod-name> -- <command>`
- **Purpose**: Runs arbitrary commands inside a pod’s container to inspect its filesystem, environment variables, or perform operations.
- **Example**:
  - List contents of a directory (e.g., `/app`):
    ```bash
    kubectl exec <pod-name> -- ls /app
    ```
  - If the directory doesn’t exist (e.g., `/app`), it returns an error like `cannot access /app: no such file or directory`.
  - List contents of the root directory:
    ```bash
    kubectl exec <pod-name> -- ls /
    ```
- **Output**: Shows available directories (e.g., no `app` directory, but others like `bin`, `etc`).

### 5. `kubectl exec -it <pod-name> -- /bin/bash`
- **Purpose**: Opens an interactive shell session inside a pod’s container.
- **Details**:
  - Provides a root prompt to execute commands directly within the container.
  - Useful for inspecting or modifying the container’s filesystem.
- **Command**:
  ```bash
  kubectl exec -it <pod-name> -- /bin/bash
  ```
- **Example Workflow**:
  - Run the command to access the shell:
    ```bash
    kubectl exec -it my-deployment-xyz -- /bin/bash
    ```
  - Inside the shell, list directories:
    ```bash
    ls
    ```
  - Create a directory (e.g., `mahendra`):
    ```bash
    mkdir mahendra
    ls
    ```
  - Exit the shell:
    ```bash
    exit
    ```
  - Verify changes by running:
    ```bash
    kubectl exec my-deployment-xyz -- ls /
    ```
  - **Output**: Confirms the `mahendra` directory exists in the container’s root directory.

## Running Commands in OCI Cloud Shell
1. **Access Cloud Shell**: Open the OCI Cloud Shell, pre-configured with `kubectl`.
2. **Select a Pod**: Use `kubectl get pods` to list pods and copy a pod name.
3. **Execute Commands**: Run the above commands, clearing the screen (`clear`) between commands for clarity.
4. **Troubleshooting**:
   - Use `kubectl describe pod` for detailed pod events to debug issues.
   - Use `kubectl logs` to check for application errors.
   - Use `kubectl exec` to inspect or modify the container environment.

## Summary
These `kubectl` commands (`get pods`, `describe pod`, `logs`, `exec`) are critical for managing and troubleshooting pods in a Kubernetes cluster. They allow you to inspect pod details, retrieve logs, execute commands, and access interactive shells, enabling effective monitoring and debugging of containerized applications in an OKE cluster.