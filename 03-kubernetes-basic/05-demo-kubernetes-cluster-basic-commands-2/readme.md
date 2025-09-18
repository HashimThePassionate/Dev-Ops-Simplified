# **Kubernetes Resource Management Commands**

This guide covers essential `kubectl` commands for creating, updating, and managing resources in a Kubernetes cluster, demonstrated using the OCI Cloud Shell with an Oracle Kubernetes Engine (OKE) cluster (e.g., OKE-DEMO-1). These commands focus on deployments, pods, and namespaces.

<details>
<summary>Table of Contents</summary>

- [**Kubernetes Resource Management Commands**](#kubernetes-resource-management-commands)
  - [Prerequisites](#prerequisites)
  - [Sample Deployment Manifest](#sample-deployment-manifest)
  - [Resource Management Commands](#resource-management-commands)
    - [1. `kubectl create -f <file-name>`](#1-kubectl-create--f-file-name)
    - [2. `kubectl create deployment <name> --image=<image> --replicas=<number>`](#2-kubectl-create-deployment-name---imageimage---replicasnumber)
    - [3. `kubectl apply -f <file-name>`](#3-kubectl-apply--f-file-name)
    - [4. `kubectl get pods`](#4-kubectl-get-pods)
    - [5. `kubectl get pods -l <label>`](#5-kubectl-get-pods--l-label)
    - [6. `kubectl create namespace <name>`](#6-kubectl-create-namespace-name)
    - [7. `kubectl get namespace`](#7-kubectl-get-namespace)
    - [8. `kubectl get all --all-namespaces`](#8-kubectl-get-all---all-namespaces)
  - [Running Commands in OCI Cloud Shell](#running-commands-in-oci-cloud-shell)
  - [Summary](#summary)

</details>

## Prerequisites
- A configured Kubernetes cluster (e.g., OKE-DEMO-1 in OCI).
- Access to OCI Cloud Shell with `kubectl` pre-configured.
- A sample deployment manifest file (e.g., `deployment.yaml`) in a directory like `kubernetes_examples`.

## Sample Deployment Manifest
Below is an example of a deployment manifest file (`deployment.yaml`) used in the commands.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

This manifest defines a deployment named `my-deployment` with:
- 2 replicas.
- Containers using the `nginx:latest` image.
- Labels (`app=nginx`) for filtering.
- Port 80 exposed.

## Resource Management Commands

### 1. `kubectl create -f <file-name>`
- **Purpose**: Creates resources (e.g., deployments, pods) in the Kubernetes cluster from a manifest file.
- **Usage**: Specify the YAML file containing the resource configuration.
- **Command**:
  ```bash
  kubectl create -f deployment.yaml
  ```
- **Example**:
  ```bash
  kubectl create -f kubernetes_examples/deployment.yaml
  ```
- **Output**: `deployment.apps/my-deployment created`
- **Details**: Creates the deployment `my-deployment` with 2 replicas, as defined in the YAML file.

### 2. `kubectl create deployment <name> --image=<image> --replicas=<number>`
- **Purpose**: Creates a deployment using inline configuration without a manifest file.
- **Usage**: Specify the deployment name, container image, and number of replicas directly in the command.
- **Command**:
  ```bash
  kubectl create deployment my-deployment-1 --image=nginx:latest --replicas=2
  ```
- **Output**: `deployment.apps/my-deployment-1 created`
- **Details**: Creates a deployment named `my-deployment-1` with 2 replicas of the `nginx:latest` image.

### 3. `kubectl apply -f <file-name>`
- **Purpose**: Creates or updates resources from a manifest file or standard input.
- **Usage**: Useful for updating existing resources (e.g., changing the number of replicas in a deployment).
- **Example Workflow**:
  - Edit the `deployment.yaml` file to change `replicas` from 2 to 4.
  - Run:
    ```bash
    kubectl apply -f kubernetes_examples/deployment.yaml
    ```
  - **Output**: `deployment.apps/my-deployment configured`
  - Verify the update:
    ```bash
    kubectl get pods
    ```
  - **Output**: Shows 4 pods running for `my-deployment`.
- **Note**: Warnings may appear but can often be ignored if the update is successful.

### 4. `kubectl get pods`
- **Purpose**: Lists all pods in the current namespace.
- **Command**:
  ```bash
  kubectl get pods
  ```
- **Output**: Displays pods from both deployments (e.g., `my-deployment` and `my-deployment-1`), each with their respective replicas (e.g., 2 pods for `my-deployment-1`, 4 pods for `my-deployment` after applying the update).
- **Details**: Helps verify the state of pods after creating or updating deployments.

### 5. `kubectl get pods -l <label>`
- **Purpose**: Filters pods based on labels.
- **Usage**: Use the `-l` flag with a key-value pair to list pods matching the label.
- **Command**:
  ```bash
  kubectl get pods -l app=nginx
  ```
- **Example**:
  - If `my-deployment` has the label `app=nginx` (as defined in the YAML), this command lists its 4 pods.
  - Pods from `my-deployment-1` (created inline without labels) are excluded.
- **Output**: Shows only pods with the label `app=nginx` (e.g., 4 pods from `my-deployment`).

### 6. `kubectl create namespace <name>`
- **Purpose**: Creates a namespace to isolate resources within the cluster.
- **Usage**: Namespaces separate resources, such as for development and production environments.
- **Commands**:
  - Create a development namespace:
    ```bash
    kubectl create namespace development
    ```
    - **Output**: `namespace/development created`
  - Create a production namespace:
    ```bash
    kubectl create namespace production
    ```
    - **Output**: `namespace/production created`
- **Details**: Resources created without a specified namespace are placed in the `default` namespace.

### 7. `kubectl get namespace`
- **Purpose**: Lists all namespaces in the cluster.
- **Command**:
  ```bash
  kubectl get namespace
  ```
- **Output**: Displays all namespaces, including:
  - User-created namespaces (e.g., `development`, `production`).
  - System namespaces (e.g., `kube-system`, `default`).
- **Details**: Helps verify the creation of new namespaces and view system namespaces used by Kubernetes.

### 8. `kubectl get all --all-namespaces`
- **Purpose**: Lists all resources in the cluster across all namespaces.
- **Command**:
  ```bash
  kubectl get all --all-namespaces
  ```
- **Output**: Includes:
  - Deployments (e.g., `my-deployment`, `my-deployment-1` in the `default` namespace).
  - Replica sets.
  - Pods.
  - Services (e.g., `coredns` in the `kube-system` namespace).
  - Other resources like `dns-autoscaler` in `kube-system`.
- **Details**: Provides a comprehensive view of all resources, including those in system namespaces like `kube-system`.

## Running Commands in OCI Cloud Shell
1. **Access Cloud Shell**: Open the OCI Cloud Shell, pre-configured with `kubectl`.
2. **Navigate to Manifest Directory**:
   - Switch to the directory containing your manifest file (e.g., `kubernetes_examples`).
   - Example: `cd kubernetes_examples`
3. **Edit Manifest Files**:
   - Use the OCI code editor to view or modify YAML files (e.g., `deployment.yaml`).
   - Save changes before applying.
4. **Execute Commands**:
   - Run the commands above, clearing the screen (`clear`) for readability.
   - Verify results with `kubectl get pods` or `kubectl get namespace`.
5. **Troubleshooting**:
   - Check pod status with `kubectl get pods`.
   - Inspect resource details with `kubectl describe` (from previous lessons).
   - Use label selectors to filter resources for targeted inspection.

## Summary
These `kubectl` commands (`create`, `apply`, `get pods`, `get pods -l`, `create namespace`, `get namespace`, `get all --all-namespaces`) enable you to create, update, and manage Kubernetes resources like deployments, pods, and namespaces. By using manifest files or inline configurations, you can define and modify resources efficiently, while namespaces help organize and isolate environments. These commands, demonstrated in the OCI Cloud Shell, are critical for managing an OKE cluster.