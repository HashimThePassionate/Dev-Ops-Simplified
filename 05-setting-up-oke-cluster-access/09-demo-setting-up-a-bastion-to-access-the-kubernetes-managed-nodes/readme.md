# **Demo: Setting Up an OCI Bastion for SSH Access to Managed Worker Nodes in a Private Subnet**

This guide, presented by Mahendra Mehra, Senior OCI Instructor at Oracle University, demonstrates how to configure an **Oracle Cloud Infrastructure (OCI) Bastion service** to provide secure **SSH access** to managed worker nodes in an Oracle Container Engine for Kubernetes (OKE) cluster hosted in a **private subnet**. The demo uses the `OKE-private-bastion-demo` cluster from the previous lesson, with both the Kubernetes API endpoint and worker nodes in private subnets. The bastion is hosted in the same private subnet as the worker nodes to simplify configuration.

<details>
<summary>Table of Contents</summary>

- [**Demo: Setting Up an OCI Bastion for SSH Access to Managed Worker Nodes in a Private Subnet**](#demo-setting-up-an-oci-bastion-for-ssh-access-to-managed-worker-nodes-in-a-private-subnet)
  - [Prerequisites](#prerequisites)
  - [Cluster Setup Recap](#cluster-setup-recap)
  - [Step-by-Step Process](#step-by-step-process)
    - [1. VCN Administrator Tasks: Configure Networking](#1-vcn-administrator-tasks-configure-networking)
    - [2. Cluster Administrator Tasks: Configure Bastion and Nodes](#2-cluster-administrator-tasks-configure-bastion-and-nodes)
    - [3. Cluster User Tasks: Create Bastion Session and Access Worker Node](#3-cluster-user-tasks-create-bastion-session-and-access-worker-node)
  - [Security Best Practices](#security-best-practices)
  - [Key Notes](#key-notes)
  - [Next Steps](#next-steps)

</details>

## Prerequisites
- An OKE cluster (`OKE-private-bastion-demo`) with:
  - Kubernetes API endpoint in a private subnet (e.g., `10.0.0.0/28`).
  - Worker nodes in a private subnet (e.g., `10.0.10.0/24`, 3 nodes with private IPs like `10.0.10.47`).
  - Load balancers in a public subnet.
- A public/private SSH key pair, with the public key installed on the worker nodes during node pool creation.
- OCI CLI and `kubectl` installed and configured on the local machine (Oracle Linux 8 in this demo).
- Administrative privileges in the `CTDOKE` compartment (or specific IAM policies for limited users; see previous lessons).
- Access to the OCI Console and a local terminal.

## Cluster Setup Recap
- **Cluster Name**: `OKE-private-bastion-demo`.
- **Compartment**: `CTDOKE`.
- **Kubernetes Version**: `1.27.2`.
- **API Endpoint**: Private subnet (`10.0.0.0/28`).
- **Worker Nodes**: Private subnet (`10.0.10.0/24`), 3 nodes, `VM.Standard.E3.Flex`, Oracle Linux 8.
- **Node Type**: Managed nodes.
- **SSH Key**: Public key provided during node pool creation.
- **Cluster Type**: Enhanced.
- **VCN**: `vcn-quick-OKE-PRIVATE-BASTION-DEMO`.

## Step-by-Step Process

### 1. VCN Administrator Tasks: Configure Networking
The bastion is hosted in the same private subnet as the worker nodes (`10.0.10.0/24`) to reuse existing infrastructure.

1. **Verify Subnets**:
   - In the OCI Console, go to **Networking** > **Virtual Cloud Networks** > Select the VCN (`vcn-quick-OKE-PRIVATE-BASTION-DEMO`).
   - Under **Subnets**, confirm:
     - **API Endpoint Subnet**: `oke-khapisubnet-...` (`10.0.0.0/28`, private).
     - **Worker Node Subnet**: `oke-nodesubnet-...` (`10.0.10.0/24`, private).
     - **Load Balancer Subnet**: `oke-lb-subnet-...` (public).
   - Copy the worker node subnet CIDR (`10.0.10.0/24`).

2. **Add Security Rules**:
   - Navigate to **Subnets** > Select `oke-nodesubnet-...` > Click the associated **Security List** (e.g., `seclist-workernodes`).
   - **Egress Rule (Bastion to Worker Nodes)**:
     - Click **Add Egress Rules**:
       - **Destination CIDR**: `10.0.10.0/24` (worker node subnet, same as bastion subnet).
       - **Protocol**: TCP.
       - **Destination Port**: `22`.
       - **Description**: Allow bastion to worker nodes traffic.
     - Click **Add Egress Rules**.
   - **Ingress Rule (Bastion to Worker Nodes)**:
     - Click **Add Ingress Rules**:
       - **Source CIDR**: `10.0.10.0/24` (bastion subnet, same as worker node subnet).
       - **Protocol**: TCP.
       - **Destination Port**: `22`.
       - **Description**: Allow bastion to worker nodes traffic.
     - Click **Add Ingress Rules**.
   - **Note**: Co-locating the bastion and worker nodes in the same subnet simplifies rules, as traffic is internal.

### 2. Cluster Administrator Tasks: Configure Bastion and Nodes
1. **Enable Bastion Agent on Worker Nodes**:
   - In the OCI Console, go to **Developer Services** > **Kubernetes Cluster (OKE)** > Select `OKE-private-bastion-demo` > **Node Pools** > Select `pool1` > **Nodes**.
   - For each of the three worker nodes:
     - Click the node (e.g., `gsq-0`, `gsq-1`, `gsq-2`).
     - Go to **Oracle Cloud Agent** tab > Locate **Bastion Plugin**.
     - Toggle to **Enable** the plugin.
   - Wait up to 10 minutes for the plugin status to change to **Running** for all nodes.
   - Verify:
     - Check each node’s **Oracle Cloud Agent** tab to confirm the Bastion Plugin status is **Running**.

2. **Create Bastion**:
   - Go to **Identity & Security** > **Bastion** > **Create Bastion**.
   - Configure:
     - **Name**: `bastion-oke-nodes`.
     - **Compartment**: `CTDOKE`.
     - **Target VCN**: `vcn-quick-OKE-PRIVATE-BASTION-DEMO`.
     - **Target Subnet**: `oke-nodesubnet-...` (worker node subnet).
     - **CIDR Block Allow List**: `0.0.0.0/0` (all sources; restrict to specific IPs for better security).
     - **Maximum Session TTL**: Default (180 minutes).
   - Click **Create Bastion**.
   - Wait for the bastion status to change to **Active**.

3. **Verify IAM Policies**:
   - The demo uses administrative privileges, granting full access to bastion and cluster resources.
   - For limited users, ensure IAM policies allow:
     - `manage bastions`, `manage bastion-sessions`, `read virtual-network-family`, `read instances`, etc. (see previous lessons).
   - Example policy for cluster users:
     ```plaintext
     Allow group ClusterUsers to manage bastion-sessions in compartment CTDOKE where all {target.resource.subnet-id = '10.0.10.0/24', target.bastion-session.target-port = '22'}
     ```

### 3. Cluster User Tasks: Create Bastion Session and Access Worker Node
1. **Create Bastion Session**:
   - In the OCI Console:
     - Go to **Identity & Security** > **Bastion** > Select `bastion-oke-nodes`.
     - Click **Create Session**.
     - Configure:
       - **Session Type**: Managed SSH Session.
       - **Session Name**: `worker-node-session-gsq0`.
       - **Username**: `opc` (default for Oracle Linux nodes).
       - **Compute Instance**: Select the worker node (e.g., `oke-private-bastion-demo-pool1-gsq-0`).
       - **SSH Public Key**: Paste the public key used during node pool creation:
         ```bash
         cat ~/.ssh/id_rsa.pub
         ```
         - Copy and paste into the console.
       - **Maximum Session TTL**: Default (180 minutes).
     - Click **Create Session**.
   - Wait for the session status to change to **Active**.

2. **Obtain SSH Command**:
   - Select the session (`worker-node-session-gsq0`) > Click the **Action Menu** > **Copy SSH Command**.
   - Paste into a text editor and modify:
     - Replace `<private-key>` with the path to the private key (e.g., `~/.ssh/id_rsa`).
     - Example:
       ```bash
       ssh -i ~/.ssh/id_rsa opc@10.0.10.47
       ```

3. **Execute SSH Command**:
   - In the local terminal:
     ```bash
     cd ~
     ssh -i ~/.ssh/id_rsa opc@10.0.10.47
     ```
   - If prompted, type `yes` to accept the host key.
   - Expected output: Connects to the worker node with the prompt `opc@oke-private-bastion-demo-pool1-gsq-0`.

4. **Perform Node Operations**:
   - Run administrative commands (e.g., `journalctl` for logs, `df -h` for disk usage).
   - Exit the session:
     ```bash
     exit
     ```

5. **Access Other Worker Nodes**:
   - Repeat steps 1–4 for other nodes (e.g., `gsq-1`, `gsq-2`):
     - Create a new session, selecting the appropriate compute instance.
     - Use the same public key and modify the SSH command with the target node’s private IP (e.g., `10.0.10.127`).

## Security Best Practices
- **Restrict CIDR Allow List**: Use a specific IP range (e.g., your organization’s IP) instead of `0.0.0.0/0` in the bastion’s CIDR Block Allow List.
- **IAM Policies**: Limit user access to specific nodes or subnets using granular policies.
- **Session Timeout**: Use the default TTL (180 minutes) or shorten it to minimize exposure.
- **Private Key Security**: Ensure `chmod 600 ~/.ssh/id_rsa` to restrict access.
- **Bastion Agent**: Verify the agent is running before creating sessions.
- **Monitor Sessions**: Regularly review active sessions in the OCI Console and terminate unused ones.

## Key Notes
- **Private Subnet Access**: The bastion enables SSH to worker nodes without public IP addresses, ensuring secure access within the VCN.
- **Subnet Reuse**: Hosting the bastion in the worker node subnet (`10.0.10.0/24`) simplifies security rules, as traffic is internal.
- **Bastion Agent**: Must be enabled and running on each target node for SSH access.
- **Node Username**: `opc` is the default for Oracle Linux-based OKE nodes.
- **Official Documentation**: Refer to [OCI official documentation](https://docs.oracle.com) for Bastion service, OKE cluster setup, and IAM policies.

## Next Steps
- **Advanced Configurations**: Explore using Network Security Groups for finer-grained access control.
- **Node Maintenance**: Learn to perform tasks like log collection and troubleshooting on worker nodes.
- **Troubleshooting**: Address common issues with bastion sessions and SSH access in future lessons.

This demo successfully configured an OCI Bastion to provide SSH access to managed worker nodes in the `OKE-private-bastion-demo` cluster’s private subnet, established a session, and connected to a node. Users can repeat the process for other nodes as needed, ensuring secure and controlled access.