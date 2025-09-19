# **Kubernetes Bash Alias Commands**

<details>
<summary>Table of Contents</summary>

- [**Kubernetes Bash Alias Commands**](#kubernetes-bash-alias-commands)
  - [Prerequisites](#prerequisites)
  - [Bash Alias Commands](#bash-alias-commands)
    - [1. Setting Up a Basic Alias for `kubectl`](#1-setting-up-a-basic-alias-for-kubectl)
    - [2. Setting Up Aliases for Specific `kubectl` Commands](#2-setting-up-aliases-for-specific-kubectl-commands)
    - [3. Persisting Aliases Across Sessions](#3-persisting-aliases-across-sessions)
    - [4. Using Aliases to Delete Resources](#4-using-aliases-to-delete-resources)
  - [Running Commands in OCI Cloud Shell](#running-commands-in-oci-cloud-shell)
  - [Summary](#summary)

</details>

<br/>

This guide covers setting up and using bash aliases for `kubectl` commands to streamline operations in a Kubernetes cluster, demonstrated using the OCI Cloud Shell with an Oracle Kubernetes Engine (OKE) cluster (e.g., OKE-DEMO-1). Aliases provide shortcuts for frequently used commands, improving efficiency.

## Prerequisites
- A configured Kubernetes cluster (e.g., OKE-DEMO-1 in OCI).
- Access to OCI Cloud Shell with `kubectl` pre-configured.
- Existing resources in OKE-DEMO-1 (e.g., services like `myapp-cip-service`, pods, deployments from previous lessons).
- Familiarity with editing the `.bashrc` file in a Linux environment.

## Bash Alias Commands

### 1. Setting Up a Basic Alias for `kubectl`
- **Purpose**: Creates a shortcut (`k`) for the `kubectl` command to reduce typing.
- **Command**:
  ```bash
  alias k='kubectl'
  ```
- **Usage**:
  - Run the command in the Cloud Shell to set the alias for the current session.
  - Example: Test the alias:
    ```bash
    k get nodes
    ```
    - **Output**: Lists all nodes, equivalent to `kubectl get nodes`.
- **Details**: Replaces `kubectl` with `k` for any `kubectl` command, improving efficiency.

### 2. Setting Up Aliases for Specific `kubectl` Commands
- **Purpose**: Creates shortcuts for frequently used `kubectl` commands.
- **Example**:
  - Alias for `kubectl get nodes`:
    ```bash
    alias kgn='kubectl get nodes'
    ```
  - Run the alias:
    ```bash
    kgn
    ```
    - **Output**: Lists all nodes in the cluster (e.g., three nodes in OKE-DEMO-1).
- **Details**: Useful for repetitive commands, reducing the need to type full commands.

### 3. Persisting Aliases Across Sessions
- **Purpose**: Ensures aliases are available in every new terminal session.
- **Steps**:
  - Open the `.bashrc` file in the user’s home directory:
    ```bash
    cd ~
    ls -l
    nano .bashrc
    ```
  - Add aliases to the `.bashrc` file, e.g.:
    ```bash
    alias k='kubectl'
    alias kgn='kubectl get nodes'
    alias kgs='kubectl get service'
    alias kd='kubectl delete'
    ```
  - Save and exit the file (`Ctrl+O`, `Enter`, `Ctrl+X` in `nano`).
  - Apply changes immediately:
    ```bash
    source ~/.bashrc
    ```
- **Verification**:
  - Test an alias:
    ```bash
    kgs
    ```
    - **Output**: Lists all services in the current namespace (e.g., `kubernetes`, `myapp-np-service`, `myapp-lb-service`).
- **Details**:
  - Aliases stored in `.bashrc` persist across Cloud Shell sessions.
  - In Windows environments, similar functionality can be achieved using PowerShell profile functions.

### 4. Using Aliases to Delete Resources
- **Purpose**: Simplifies deletion of resources using an alias.
- **Example**:
  - Use the `kd` alias for `kubectl delete` to delete a service:
    ```bash
    kd service myapp-cip-service
    ```
    - **Output**: `service "myapp-cip-service" deleted`
  - Verify deletion:
    ```bash
    kgs
    ```
    - **Output**: Confirms `myapp-cip-service` is no longer listed.
- **Details**: Combines the alias (`kd`) with resource-specific commands for quick deletion.

## Running Commands in OCI Cloud Shell
1. **Access Cloud Shell**: Open the OCI Cloud Shell, pre-configured with `kubectl`.
2. **Set Up Aliases**:
   - Set temporary aliases for the current session (e.g., `alias k='kubectl'`).
   - Add persistent aliases to `~/.bashrc` for use across sessions.
3. **Execute Commands**:
   - Use aliases like `k`, `kgn`, `kgs`, or `kd` to run commands efficiently.
   - Clear the screen (`clear`) between commands for readability.
4. **Troubleshooting**:
   - If an alias doesn’t work, ensure it was added correctly to `.bashrc` and sourced (`source ~/.bashrc`).
   - Verify resource changes (e.g., service deletion) with `kgs` or other `kubectl` commands.
   - Check the current context with `k config current-context` to ensure commands target the correct cluster (e.g., OKE-DEMO-1).
5. **Notes**:
   - Aliases are session-specific unless added to `.bashrc`.
   - Choose short, intuitive alias names (e.g., `kgs` for `kubectl get service`) to avoid confusion.

## Summary
Bash aliases for `kubectl` commands (e.g., `k`, `kgn`, `kgs`, `kd`) streamline Kubernetes operations by reducing typing and improving efficiency. By setting aliases in the OCI Cloud Shell and persisting them in the `.bashrc` file, you can manage clusters, nodes, pods, services, and deletions with shortcuts. This guide concludes the multi-part series on `kubectl` commands, providing a comprehensive toolkit for managing an OKE cluster.