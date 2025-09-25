# **Demo: Connecting to Managed Nodes in Public Subnets Using SSH**

This guide, presented by Mahendra Mehra, Senior OCI Instructor at Oracle University, demonstrates how to connect to managed worker nodes in an Oracle Container Engine for Kubernetes (OKE) cluster hosted in a **public subnet** using SSH. The demo uses the `OKE-PUBLIC-DEMO-1` cluster created in a previous lesson, which has its Kubernetes API endpoint and worker nodes in a public subnet, with a public SSH key installed on the nodes.

## Table of Contents

<details>
<summary>Click to expand</summary>

- [**Demo: Connecting to Managed Nodes in Public Subnets Using SSH**](#demo-connecting-to-managed-nodes-in-public-subnets-using-ssh)
  - [Table of Contents](#table-of-contents)
  - [Prerequisites](#prerequisites)
  - [Step-by-Step Process](#step-by-step-process)
    - [1. Verify Cluster and SSH Key Setup](#1-verify-cluster-and-ssh-key-setup)
    - [2. Verify SSH Utility](#2-verify-ssh-utility)
    - [3. Check Security Configuration](#3-check-security-configuration)
    - [4. Obtain Worker Node Public IP](#4-obtain-worker-node-public-ip)
    - [5. Connect to a Worker Node via SSH](#5-connect-to-a-worker-node-via-ssh)
    - [6. Connect to Another Worker Node](#6-connect-to-another-worker-node)
  - [Security Best Practices](#security-best-practices)
  - [Key Notes](#key-notes)
  - [Next Steps](#next-steps)

</details>

## Prerequisites
- An OKE cluster (`OKE-PUBLIC-DEMO-1`) with managed nodes in a public subnet and a public SSH key installed during node pool creation.
- Access to the OCI Console and Cloud Shell (used in this demo; local terminal also applicable).
- The private SSH key corresponding to the public key installed on the nodes, stored in `~/.ssh/id_rsa` or a custom location.
- Permissions to manage the cluster, node pools, and networking resources in the `CTDOKE` compartment.
- An ingress rule allowing SSH traffic (port `22`) to the worker nodes.

## Step-by-Step Process

### 1. Verify Cluster and SSH Key Setup
- **Cluster Details**:
  - Cluster: `OKE-PUBLIC-DEMO-1` in the `CTDOKE` compartment.
  - Node Pool: One node pool with three managed worker nodes.
  - Worker Nodes: Hosted in a public subnet with public IP addresses.
  - SSH Key: A public SSH key was provided during node pool creation and installed on all worker nodes.
- **Check Private Key**:
  - In Cloud Shell, navigate to the SSH directory:
    ```bash
    cd ~/.ssh
    ls
    ```
    - Confirm the presence of `id_rsa` (private key) and `id_rsa.pub` (public key).
  - Verify permissions on the private key:
    ```bash
    ls -al id_rsa
    ```
    - Expected output: Permissions should be `rw-------` (e.g., `600`), allowing read/write access only for the owner.
    - Set permissions if needed:
      ```bash
      chmod 600 ~/.ssh/id_rsa
      ```
    - **Note**: Incorrect permissions (e.g., accessible to other users) cause the SSH utility to ignore the key, resulting in connection errors.

### 2. Verify SSH Utility
- Confirm SSH is available in Cloud Shell:
  ```bash
  ssh -v
  ```
  - Expected output: Displays OpenSSH version information, confirming the SSH utility is installed.
- **Note**: Cloud Shell comes pre-installed with SSH, making it ideal for this demo. For local terminals, ensure an SSH client is installed.

### 3. Check Security Configuration
- **Ingress Rule for SSH**:
  - Worker nodes in a public subnet require an ingress rule to allow SSH traffic on port `22`.
  - In the OCI Console:
    - Navigate to **Networking** > **Virtual Cloud Networks** > Select the VCN.
    - Under **Resources**, select **Subnets** > Select the worker node subnet (e.g., `oke-nodesubnet-...`).
    - Click the associated **Security List**.
    - Verify or add an ingress rule:
      - **Source CIDR**: `0.0.0.0/0` (all sources; restrict to specific IPs for better security, e.g., your machine’s IP).
      - **Protocol**: TCP.
      - **Destination Port**: `22`.
      - **Description**: Allow SSH access to worker nodes.
    - **Note**: For clusters created with the **Quick Create workflow**, this rule is auto-generated. For **Custom Create**, manually add it if missing.

### 4. Obtain Worker Node Public IP
- **Option 1: Using kubectl**:
  - Run:
    ```bash
    kubectl get nodes -o wide
    ```
    - Output: Lists the three worker nodes with their **EXTERNAL-IP** (public IP) and **INTERNAL-IP** (private IP).
    - Example:
      ```
      NAME           STATUS   ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP
      node-1         Ready    <none>   1d    v1.27.2   10.0.1.10     129.146.32.57
      node-2         Ready    <none>   1d    v1.27.2   10.0.1.11     129.146.32.58
      node-3         Ready    <none>   1d    v1.27.2   10.0.1.12     129.146.32.59
      ```
    - Note the **EXTERNAL-IP** for the target node (e.g., `129.146.32.57`).

- **Option 2: Using OCI Console**:
  - Navigate to **Developer Services** > **Kubernetes Cluster (OKE)** > Select `OKE-PUBLIC-DEMO-1`.
  - Go to **Node Pools** > Select `Pool 1` > **Nodes** tab.
  - Click a node to view its details in the **Compute Instances** page.
  - Note the **Public IPv4 Address** (e.g., `129.146.32.57`).
  - **Note**: The console may only show private IPs in some views; clicking a node reveals the public IP.

### 5. Connect to a Worker Node via SSH
- **Default SSH Command** (private key in `~/.ssh/id_rsa`):
  ```bash
  ssh opc@<node-ip-address>
  ```
  - Replace `<node-ip-address>` with the public IP (e.g., `ssh opc@129.146.32.57`).
  - **User**: `opc` is the default user for OKE worker nodes.
  - If prompted, type `yes` to accept the host key.

- **Non-Default Private Key Location**:
  - If the private key is not in `~/.ssh/id_rsa`:
    ```bash
    ssh -i /path/to/private_key opc@<node-ip-address>
    ```
    - Example: `ssh -i ~/.ssh/custom_key opc@129.146.32.57`.

- **Verify Connection**:
  - Upon successful connection, the prompt changes to `opc@<instance-name>`.
  - Confirm the instance name matches the node (e.g., ends with `bhq-0`).
  - Check in the OCI Console under **Compute Instances** to verify the instance name.

- **Exit SSH Session**:
  ```bash
  exit
  ```
  - Closes the connection and returns to the Cloud Shell.

### 6. Connect to Another Worker Node
- Repeat the process for another node:
  - Use a different public IP (e.g., `129.146.32.58`).
  - Example:
    ```bash
    ssh opc@129.146.32.58
    ```
    - Type `yes` if prompted.
  - Verify the prompt reflects the new node’s instance name.

## Security Best Practices
- **Restrict Ingress Rule**: Use a specific source CIDR (e.g., your machine’s IP) instead of `0.0.0.0/0` to limit SSH access.
- **Private Key Permissions**: Ensure `chmod 600` on the private key to prevent unauthorized access.
- **Temporary Access**: Add/remove the SSH ingress rule as needed to minimize exposure.
- **Monitor Access**: Regularly review security list rules and SSH key usage.

## Key Notes
- **Public Subnet Access**: Nodes in public subnets have public IPs, enabling direct SSH access with proper security rules.
- **Quick vs. Custom Create**:
  - Quick Create: Automatically adds the SSH ingress rule.
  - Custom Create: Requires manual addition of the rule.
- **Virtual Nodes**: Not accessible via SSH.
- **Cloud Shell**: Pre-installed with SSH and `kubectl`, simplifying access for this demo.
- **kubectl vs. Console**: Both methods for retrieving IPs are reliable; `kubectl` is faster for command-line users.
- **Official Documentation**: Refer to [OCI official documentation](https://docs.oracle.com) under "Container Engine for Kubernetes" for detailed SSH configuration.

## Next Steps
- **Private Subnet Access**: Future