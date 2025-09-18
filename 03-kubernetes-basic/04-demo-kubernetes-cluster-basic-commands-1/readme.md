# **Kubernetes Basic Commands**

<details>
<summary>Table of Contents</summary>

- [**Kubernetes Basic Commands**](#kubernetes-basic-commands)
  - [Prerequisites](#prerequisites)
  - [Kubernetes Commands Overview](#kubernetes-commands-overview)
    - [1. `kubectl cluster-info`](#1-kubectl-cluster-info)
    - [2. `kubectl version`](#2-kubectl-version)
    - [3. `kubectl api-resources`](#3-kubectl-api-resources)
    - [4. `kubectl api-versions`](#4-kubectl-api-versions)
  - [Running Commands in OCI Cloud Shell](#running-commands-in-oci-cloud-shell)
  - [Summary](#summary)

</details>

## Prerequisites
To follow along with these Kubernetes commands, you need:
- A functional Kubernetes cluster (e.g., an Oracle Kubernetes Engine (OKE) cluster configured in the OCI console).
- **kubectl** installed and configured to interact with the cluster (e.g., via OCI Cloud Shell, which comes pre-configured with kubectl).
- Access to a specific cluster (e.g., OKE-DEMO-1 in this case).

This guide assumes you have two Kubernetes clusters available and are using the OKE-DEMO-1 cluster to execute commands.

## Kubernetes Commands Overview

Below are the basic `kubectl` commands for managing and inspecting a Kubernetes cluster, demonstrated using the OCI Cloud Shell.

### 1. `kubectl cluster-info`
- **Purpose**: Displays endpoint information for the master (control plane) and services in the Kubernetes cluster.
- **Usage**: Run in the Cloud Shell to view details about the cluster's control plane and services.
- **Example Output**:
  - Shows the addresses where the Kubernetes master and services are running.
- **Command**:
  ```bash
  kubectl cluster-info
  ```

### 2. `kubectl version`
- **Purpose**: Displays the Kubernetes version for both the client (kubectl) and the server (Kubernetes cluster).
- **Details**:
  - **Client Version**: The version of the kubectl client installed locally (e.g., 1.26).
  - **Server Version**: The version of the Kubernetes server the cluster is running (e.g., 1.27).
  - Note: The client and server versions can differ by ±1. Beyond this, a warning may be displayed.
- **Command**:
  ```bash
  kubectl version
  ```

### 3. `kubectl api-resources`
- **Purpose**: Lists all available API resources in the Kubernetes cluster that can be used for operations.
- **Details**:
  - Displays resources such as `persistentvolumes`, `persistentvolumeclaims`, `namespaces`, `nodes`, `pods`, `resourcequotas`, `secrets`, and more.
- **Command**:
  ```bash
  kubectl api-resources
  ```

### 4. `kubectl api-versions`
- **Purpose**: Displays the API versions supported by the Kubernetes cluster.
- **Details**:
  - Shows available API versions for Kubernetes resources (e.g., `v1`, `v1beta1`).
  - Helps in selecting the appropriate API version for your workloads.
- **Command**:
  ```bash
  kubectl api-versions
  ```

## Running Commands in OCI Cloud Shell
1. **Open Cloud Shell**:
   - Access the OCI Cloud Shell, which is pre-configured with `kubectl` for interacting with your OKE cluster.
2. **Set Up Cluster Access**:
   - Ensure the Cloud Shell is configured to access the OKE-DEMO-1 cluster.
3. **Execute Commands**:
   - Run the commands above in the Cloud Shell terminal.
   - Use `clear` to clean the terminal screen between commands for better readability.
4. **Maximize Terminal**:
   - Maximize the Cloud Shell terminal for a better working experience.

## Summary
These basic `kubectl` commands (`cluster-info`, `version`, `api-resources`, `api-versions`) are essential for inspecting and managing a Kubernetes cluster. They provide insights into the cluster's configuration, versions, and available resources, enabling you to interact effectively with your OKE cluster in the OCI console.