# **Kubernetes Edit Pod and Deployment Commands**

<details>
<summary>Table of Contents</summary>

- [**Kubernetes Edit Pod and Deployment Commands**](#kubernetes-edit-pod-and-deployment-commands)
  - [Prerequisites](#prerequisites)
  - [Edit Pod and Deployment Commands](#edit-pod-and-deployment-commands)
    - [1. `kubectl get pods`](#1-kubectl-get-pods)
    - [2. `kubectl edit pod <pod-name>`](#2-kubectl-edit-pod-pod-name)
    - [3. `kubectl get pod <pod-name> -o yaml > <file-name>`](#3-kubectl-get-pod-pod-name--o-yaml--file-name)
    - [4. `kubectl edit deployment <deployment-name>`](#4-kubectl-edit-deployment-deployment-name)
  - [Running Commands in OCI Cloud Shell](#running-commands-in-oci-cloud-shell)
  - [Summary](#summary)

</details>

<br/>

This guide covers `kubectl` commands for editing pods and deployments in a Kubernetes cluster, demonstrated using the OCI Cloud Shell with an Oracle Kubernetes Engine (OKE) cluster (e.g., OKE-DEMO-1). These commands allow you to modify pod and deployment configurations and extract pod definitions.

## Prerequisites
- A configured Kubernetes cluster (e.g., OKE-DEMO-1 in OCI).
- Access to OCI Cloud Shell with `kubectl` pre-configured.
- Existing deployments and pods (e.g., `my-deployment` with 6 replicas and `my-deployment-1` with 2 replicas, as created in previous lessons).

## Edit Pod and Deployment Commands

### 1. `kubectl get pods`
- **Purpose**: Lists all pods in the current namespace to identify pod names for editing.
- **Command**:
  ```bash
  kubectl get pods
  ```
- **Output**: Displays all pods (e.g., 8 pods, including those from `my-deployment` and `my-deployment-1`).
- **Usage**: Run to select a pod name for editing or inspection.

### 2. `kubectl edit pod <pod-name>`
- **Purpose**: Edits the specification of a specific pod in a VI editor.
- **Command**:
  ```bash
  kubectl edit pod <pod-name>
  ```
- **Example**:
  ```bash
  kubectl edit pod my-deployment-abc123-zc76d
  ```
- **Details**:
  - Opens the pod specification in a VI editor.
  - Editable fields include:
    - Container image
    - Init container image
    - `activeDeadlineSeconds`
    - `spec.tolerations`
  - Changes to other fields may not be supported or may require deployment-level edits.
- **Workflow**:
  - Run the command to open the VI editor.
  - Inspect the pod specification but avoid making changes unless necessary (e.g., exit without saving for now).
  - Save and exit (`:wq`) to apply changes, or exit without saving (`:q!`).

### 3. `kubectl get pod <pod-name> -o yaml > <file-name>`
- **Purpose**: Extracts the pod definition in YAML format and saves it to a file.
- **Command**:
  ```bash
  kubectl get pod <pod-name> -o yaml > my-pod.yaml
  ```
- **Example**:
  ```bash
  kubectl get pod my-deployment-abc123-zc76d -o yaml > my-pod.yaml
  ```
- **Details**:
  - Exports the full pod specification to a file (e.g., `my-pod.yaml`).
  - Unlike `kubectl edit pod`, this allows you to save the specification for reference or modification.
- **Verification**:
  - Open the file to confirm the pod specification:
    ```bash
    nano my-pod.yaml
    ```
  - Output: Displays the pod’s YAML configuration, including namespace, container details, and status.

### 4. `kubectl edit deployment <deployment-name>`
- **Purpose**: Edits the specification of a deployment in a VI editor.
- **Command**:
  ```bash
  kubectl edit deployment my-deployment
  ```
- **Example Workflow**:
  - List deployments:
    ```bash
    kubectl get deployments
    ```
    - Output: Shows `my-deployment` (e.g., with 6 replicas) and `my-deployment-1`.
  - Edit the deployment:
    ```bash
    kubectl edit deployment my-deployment
    ```
    - Opens the deployment specification in a VI editor.
    - Modify a field, e.g., change `replicas` from 6 to 3:
      - Enter insert mode (`i`), update `replicas: 3`, exit insert mode (`Esc`), save and exit (`:wq`).
  - Verify the rollout:
    ```bash
    kubectl rollout status deployment/my-deployment
    ```
    - Output: `deployment "my-deployment" successfully rolled out`
  - Confirm the change:
    ```bash
    kubectl get deployments
    kubectl get pods
    ```
    - Output: Shows `my-deployment` with 3 replicas and 3 running pods.
- **Details**:
  - Edits apply to the pod template within the deployment.
  - Changes trigger automatic deletion and creation of pods to reflect the new configuration.
  - Editable fields include replicas, container images, labels, and other pod template properties.

## Running Commands in OCI Cloud Shell
1. **Access Cloud Shell**: Open the OCI Cloud Shell, pre-configured with `kubectl`.
2. **Execute Commands**:
   - Run `kubectl get pods` to list pods and select a pod name.
   - Use `kubectl edit pod` or `kubectl get pod -o yaml` to modify or extract pod specifications.
   - Use `kubectl edit deployment` to update deployment configurations.
   - Clear the screen (`clear`) between commands for readability.
3. **Troubleshooting**:
   - Verify pod changes with `kubectl get pods` or `kubectl describe pod`.
   - Check deployment rollout status with `kubectl rollout status` to ensure updates are applied.
   - Inspect saved YAML files with `nano` or other editors for accuracy.
4. **Notes**:
   - Direct pod edits are limited; prefer deployment edits for pod template changes to ensure consistency.
   - Deployment edits automatically manage pod lifecycle, unlike direct pod edits.

## Summary
These `kubectl` commands (`get pods`, `edit pod`, `get pod -o yaml`, `edit deployment`) enable you to edit and manage pod and deployment configurations in a Kubernetes cluster. They allow you to modify pod images or tolerations, extract pod definitions for reference, and update deployment properties like replicas. Demonstrated in the OCI Cloud Shell, these commands are essential for fine-tuning workloads in an OKE cluster.