# Demo: **Connecting to Managed Nodes in an OKE Cluster Using SSH**

<details>
<summary>📋 Table of Contents</summary>

- [Demo: **Connecting to Managed Nodes in an OKE Cluster Using SSH**](#demo-connecting-to-managed-nodes-in-an-oke-cluster-using-ssh)
  - [Overview](#overview)
  - [Prerequisites](#prerequisites)
  - [Connecting to Managed Nodes in a Public Subnet](#connecting-to-managed-nodes-in-a-public-subnet)
    - [Steps](#steps)
    - [Key Notes](#key-notes)
  - [Connecting to Managed Nodes in a Private Subnet](#connecting-to-managed-nodes-in-a-private-subnet)
    - [Overview](#overview-1)
    - [Steps (High-Level, Detailed in Future Lessons)](#steps-high-level-detailed-in-future-lessons)
  - [Security Best Practices](#security-best-practices)
  - [Key Notes](#key-notes-1)
  - [Next Steps](#next-steps)

</details>

This guide, presented by Mahendra Mehra, Senior OCI Instructor at Oracle University, explains how to connect to managed worker nodes in an Oracle Container Engine for Kubernetes (OKE) cluster using SSH. It covers accessing nodes in both public and private subnets, with a focus on security considerations and configuration steps.

## Overview
- **Purpose**: Accessing managed worker nodes in an OKE cluster is often necessary for maintenance, configuration inspection, log collection, or troubleshooting.
- **Security Note**: SSH access increases node vulnerability, requiring careful configuration to minimize exposure.
- **SSH Key Requirement**: A public SSH key must have been provided during the creation of the managed node pool, installed on all worker nodes in the pool.
- **Node Types**: Only **managed nodes** support SSH access; **virtual nodes** do not.
- **Access Methods**: Depend on whether worker nodes are in a **public subnet** (direct SSH) or **private subnet** (requires OCI Bastion service or Cloud Shell private access).

## Prerequisites
- An OKE cluster with managed nodes (e.g., `OKE-PUBLIC-DEMO-1` from previous demos).
- A public/private SSH key pair, with the public key installed in the node pool during creation.
- Permissions to manage the cluster and networking resources in the OCI Console.
- A Unix-like local terminal (e.g., Linux, macOS, Solaris) with SSH utilities.
- For private subnet nodes: Access to OCI Cloud Shell or Bastion service (covered in later lessons).

## Connecting to Managed Nodes in a Public Subnet

### Steps
1. **Verify SSH Key Setup**:
   - Ensure a public SSH key was provided during node pool creation (e.g., in the `OKE-PUBLIC-DEMO-1` demo).
   - The corresponding private key must be accessible on your local machine (e.g., `~/.ssh/id_rsa`).

2. **Configure Ingress Rule for SSH**:
   - For nodes in a public subnet, add an ingress rule to allow SSH traffic on port `22`.
   - In the OCI Console:
     - Navigate to **Networking** > **Virtual Cloud Networks** > Select the VCN.
     - Under **Resources**, select **Security Lists** or **Network Security Groups** associated with the worker node subnet.
     - Add an ingress rule:
       - **Source CIDR**: `0.0.0.0/0` (all sources; restrict to specific CIDRs for better security).
       - **Protocol**: TCP.
       - **Destination Port**: `22`.
       - **Description**: Allow SSH access to worker nodes.

3. **Obtain Node Public IP**:
   - Option 1: Use `kubectl`:
     ```bash
     kubectl get nodes -o wide
     ```
     - Note the **EXTERNAL-IP** or **INTERNAL-IP** (public IP for public subnet nodes).
   - Option 2: Use the OCI Console:
     - Go to **Developer Services** > **Kubernetes Cluster (OKE)** > Select the cluster.
     - Navigate to **Node Pools** > Select the pool (e.g., `Pool 1`) > **Nodes** tab.
     - Note the public IP addresses of the worker nodes.

4. **Connect via SSH**:
   - Default SSH command (if private key is in `~/.ssh/id_rsa`):
     ```bash
     ssh opc@<node-ip-address>
     ```
     - Replace `<node-ip-address>` with the public IP from step 3.
   - If the private key is in a non-default location:
     ```bash
     ssh -i /path/to/private_key opc@<node-ip-address>
     ```
     - Example: `ssh -i ~/.ssh/custom_key opc@192.0.2.1`.

5. **Alternative: Configure SSH Config File**:
   - Add the private key to the SSH configuration file for streamlined access.
   - Edit `~/.ssh/config` (or system-wide `/etc/ssh/ssh_config`):
     ```plaintext
     Host oke-node
         HostName <node-ip-address>
         User opc
         IdentityFile /path/to/private_key
     ```
     - Replace `<node-ip-address>` and `/path/to/private_key` as needed.
   - Connect using:
     ```bash
     ssh oke-node
     ```

6. **Set Private Key Permissions**:
   - Ensure the private key file is secure:
     ```bash
     chmod 600 /path/to/private_key
     ```
     - Restricts access to the owner only (read/write permissions).
     - Incorrect permissions (e.g., accessible to other users) cause SSH to ignore the key.

### Key Notes
- **Security**: Restrict the ingress rule’s source CIDR to specific IPs (e.g., your local machine’s IP) instead of `0.0.0.0/0` for enhanced security.
- **Public Subnet Access**: Direct SSH access is possible due to public IP addresses assigned to nodes.
- **User**: The default user for OKE nodes is `opc`.

## Connecting to Managed Nodes in a Private Subnet

### Overview
- Worker nodes in private subnets have **private IP addresses only** and are accessible only within the VCN.
- Direct SSH from a local machine requires VCN peering or a bastion host.
- Oracle recommends using the **OCI Bastion service** for secure external access (covered in later lessons).
- Alternative: Use **Cloud Shell private access** for SSH without exposing nodes to the public internet.

### Steps (High-Level, Detailed in Future Lessons)
1. **Set Up OCI Bastion Service**:
   - Create a bastion in the OCI Console under **Identity & Security** > **Bastion**.
   - Configure a session to access the private IP of the worker node.
   - Use the provided SSH command with a temporary key pair.

2. **Use Cloud Shell Private Access**:
   - Configure Cloud Shell to access the VCN’s private subnet (requires VCN configuration).
   - Run SSH commands from Cloud Shell to the node’s private IP.

3. **Obtain Node Private IP**:
   - Use:
     ```bash
     kubectl get nodes -o wide
     ```
     - Note the **INTERNAL-IP** for private subnet nodes.
   - Alternatively, check the OCI Console under **Node Pools** > **Nodes**.

4. **SSH Command**:
   - Example (via Bastion or Cloud Shell):
     ```bash
     ssh -i /path/to/private_key opc@<node-private-ip>
     ```
     - Requires Bastion session or Cloud Shell configuration.

## Security Best Practices
- **Enable/Disable SSH Access**: Enable SSH only when needed (e.g., by adding/removing ingress rules) to minimize exposure.
- **Restrict Access**: Use specific source CIDRs in ingress rules and secure private key files (`chmod 600`).
- **Bastion for Private Nodes**: Prefer the OCI Bastion service for secure, temporary access to private subnet nodes.
- **Key Management**: Store private keys securely and avoid sharing them.

## Key Notes
- **Public vs. Private Subnets**:
  - Public subnet nodes are directly accessible with proper ingress rules.
  - Private subnet nodes require Bastion or Cloud Shell, ensuring no public exposure.
- **kubectl for IP Retrieval**: Use `kubectl get nodes -o wide` to quickly find node IPs.
- **OCI Console**: Provides an alternative way to view node IPs and manage configurations.
- **Official Documentation**: Refer to [OCI official documentation](https://docs.oracle.com) for detailed SSH setup and Bastion service configuration.

## Next Steps
- **Bastion Service Setup**: Future lessons will cover using the OCI Bastion service for accessing private subnet nodes.
- **Advanced Troubleshooting**: Explore node maintenance, log collection, and configuration inspection.
- **Cluster Management**: Continue learning about managing OKE clusters with `kubectl`.

This guide provides a comprehensive overview of connecting to managed OKE nodes using SSH, with practical steps for public subnet nodes and an introduction to private subnet access. Stay tuned for detailed demos on the OCI Bastion service.