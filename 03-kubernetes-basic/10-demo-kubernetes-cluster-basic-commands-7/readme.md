# **Kubernetes Configuration and Debugging Commands**

<details>
<summary>Table of Contents</summary>

- [**Kubernetes Configuration and Debugging Commands**](#kubernetes-configuration-and-debugging-commands)
  - [Prerequisites](#prerequisites)
  - [Configuration and Debugging Commands](#configuration-and-debugging-commands)
    - [1. `kubectl config view`](#1-kubectl-config-view)
    - [2. `kubectl config get-contexts`](#2-kubectl-config-get-contexts)
    - [3. `kubectl config current-context`](#3-kubectl-config-current-context)
    - [4. `kubectl config use-context <context-name>`](#4-kubectl-config-use-context-context-name)
    - [5. `kubectl get events`](#5-kubectl-get-events)
  - [Running Commands in OCI Cloud Shell](#running-commands-in-oci-cloud-shell)
  - [Summary](#summary)

</details>

This guide covers `kubectl` commands for configuring and debugging resources in a Kubernetes cluster, demonstrated using the OCI Cloud Shell with an Oracle Kubernetes Engine (OKE) cluster (e.g., OKE-DEMO-1 and OKE-DEMO-2). These commands help manage cluster configurations, switch contexts, and view events for troubleshooting.

## Prerequisites
- A configured Kubernetes cluster (e.g., OKE-DEMO-1 and OKE-DEMO-2 in OCI).
- Access to OCI Cloud Shell with `kubectl` pre-configured.
- `kubectl` configured to access multiple clusters (e.g., OKE-DEMO-1 and OKE-DEMO-2).
- Existing resources in OKE-DEMO-1 (e.g., deployments, pods, services from previous lessons), while OKE-DEMO-2 has no user-created resources.

## Configuration and Debugging Commands

### 1. `kubectl config view`
- **Purpose**: Displays the configuration details of the Kubernetes cluster(s) `kubectl` is configured to access.
- **Command**:
  ```bash
  kubectl config view
  ```
- **Output**:
  - Shows cluster details, including:
    - Server URL
    - Cluster name
    - Cluster ID (e.g., ending with `pk`)
    - Context name
  - Matches the cluster ID visible in the OCI console for the current cluster (e.g., OKE-DEMO-1).
- **Usage**: Verify the configuration of the cluster you are connected to.

### 2. `kubectl config get-contexts`
- **Purpose**: Lists all available contexts (representing clusters) that `kubectl` can connect to.
- **Command**:
  ```bash
  kubectl config get-contexts
  ```
- **Output**:
  - Displays contexts for all configured clusters (e.g., OKE-DEMO-1 and OKE-DEMO-2).
  - A star (`*`) indicates the current context (e.g., OKE-DEMO-1 with context name ending in `ypq`).
  - Includes context names, cluster names, and associated users.
- **Usage**: Identify available clusters and the currently active context.

### 3. `kubectl config current-context`
- **Purpose**: Displays the name of the current context (the cluster `kubectl` is interacting with).
- **Command**:
  ```bash
  kubectl config current-context
  ```
- **Output**: Shows the current context name (e.g., context for OKE-DEMO-1 ending with `ypq`).
- **Usage**: Confirms the active cluster; typically redundant as `kubectl config get-contexts` shows the current context with a star.

### 4. `kubectl config use-context <context-name>`
- **Purpose**: Switches the active context to a different cluster.
- **Command**:
  ```bash
  kubectl config use-context <context-name>
  ```
- **Example Workflow**:
  - List contexts:
    ```bash
    kubectl config get-contexts
    ```
    - Output: Shows contexts for OKE-DEMO-1 (e.g., ending with `ypq`) and OKE-DEMO-2 (e.g., ending with `v4a`).
  - Switch to OKE-DEMO-2:
    ```bash
    kubectl config use-context <context-name-ending-with-v4a>
    ```
    - Output: `Switched to context "<context-name-ending-with-v4a>"`
  - Verify the switch:
    - Check the OCI console to confirm the cluster ID for OKE-DEMO-2 ends with `v4a`.
    - Run:
      ```bash
      kubectl get all
      ```
      - Output: Shows only the default `kubernetes` service (type `ClusterIP`) in OKE-DEMO-2, as no user-created resources exist.
  - Switch back to OKE-DEMO-1:
    ```bash
    kubectl config use-context <context-name-ending-with-ypq>
    ```
    - Output: `Switched to context "<context-name-ending-with-ypq>"`
    - Run:
      ```bash
      kubectl get all
      ```
      - Output: Lists all resources (pods, deployments, services) in OKE-DEMO-1.
- **Details**: Allows seamless switching between clusters for managing multiple environments.

### 5. `kubectl get events`
- **Purpose**: Displays all events for resources in the current namespace of the cluster.
- **Command**:
  ```bash
  kubectl get events
  ```
- **Output**: Lists events for all resources (e.g., pod creation, scaling, service exposure) in the current cluster (e.g., OKE-DEMO-1).
- **Usage**: Useful for debugging and tracking the history of resource changes in the cluster.

## Running Commands in OCI Cloud Shell
1. **Access Cloud Shell**: Open the OCI Cloud Shell, pre-configured with `kubectl`.
2. **Execute Commands**:
   - Run `kubectl config view` to check cluster configuration.
   - Use `kubectl config get-contexts` to list available clusters and identify the current context.
   - Switch contexts with `kubectl config use-context` to manage different clusters.
   - Run `kubectl get events` to view resource events for troubleshooting.
   - Clear the screen (`clear`) between commands for readability.
3. **Troubleshooting**:
   - Verify the current context with `kubectl config current-context` or `kubectl config get-contexts`.
   - Use `kubectl get all` to confirm the resources available in the active cluster.
   - Check events with `kubectl get events` to diagnose issues with resource creation or updates.
4. **Notes**:
   - Ensure `kubectl` is configured to access all relevant clusters (e.g., OKE-DEMO-1 and OKE-DEMO-2).
   - The star in `kubectl config get-contexts` output clearly indicates the active cluster, making `kubectl config current-context` optional.

## Summary
These `kubectl` commands (`config view`, `config get-contexts`, `config current-context`, `config use-context`, `get events`) enable you to configure and debug Kubernetes clusters. They allow you to view cluster configurations, manage multiple cluster contexts, switch between clusters, and inspect resource events for troubleshooting. Demonstrated in the OCI Cloud Shell, these commands are critical for managing and debugging multi-cluster environments in an OKE setup.