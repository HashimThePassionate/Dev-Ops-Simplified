# **Setting Up OCI Bastion Service for OKE Cluster Access**

This guide, presented by Mahendra Mehra, Senior OCI Instructor at Oracle University, explains how to configure the **Oracle Cloud Infrastructure (OCI) Bastion service** to securely access an Oracle Container Engine for Kubernetes (OKE) cluster's **Kubernetes API endpoint** and **managed worker nodes**, particularly when they reside in **private subnets**. The process involves setting up a bastion, configuring network rules, defining IAM policies, and creating bastion sessions for cluster administrators and users.

<details>
<summary>📋 Table of Contents</summary>

- [**Setting Up OCI Bastion Service for OKE Cluster Access**](#setting-up-oci-bastion-service-for-oke-cluster-access)
  - [Overviewetting Up OCI Bastion Service for OKE Cluster Access\*\*](#overviewetting-up-oci-bastion-service-for-oke-cluster-access)
  - [Overview](#overview)
  - [Prerequisites](#prerequisites)
  - [Step-by-Step Process](#step-by-step-process)
    - [1. VCN Administrator Tasks](#1-vcn-administrator-tasks)
      - [1.1 Create a Bastion Subnet](#11-create-a-bastion-subnet)
      - [1.2 Configure Security Rules for Kubernetes API Endpoint](#12-configure-security-rules-for-kubernetes-api-endpoint)
      - [1.3 Configure Security Rules for Worker Nodes](#13-configure-security-rules-for-worker-nodes)
    - [2. Cluster Administrator Tasks](#2-cluster-administrator-tasks)
      - [2.1 Create a Bastion](#21-create-a-bastion)
      - [2.2 Enable Bastion Agent for Worker Nodes](#22-enable-bastion-agent-for-worker-nodes)
      - [2.3 Configure IAM Policies](#23-configure-iam-policies)
    - [3. Cluster User Tasks: Accessing Kubernetes API Endpoint](#3-cluster-user-tasks-accessing-kubernetes-api-endpoint)
      - [Steps](#steps)
    - [4. Cluster User Tasks: Accessing Managed Worker Nodes](#4-cluster-user-tasks-accessing-managed-worker-nodes)
      - [Steps](#steps-1)
  - [Security Best Practices](#security-best-practices)
  - [Key Notes](#key-notes)
  - [Next Steps](#next-steps)

</details>

## Overviewetting Up OCI Bastion Service for OKE Cluster Access**

This guide, presented by Mahendra Mehra, Senior OCI Instructor at Oracle University, explains how to configure the **Oracle Cloud Infrastructure (OCI) Bastion service** to securely access an Oracle Container Engine for Kubernetes (OKE) cluster’s **Kubernetes API endpoint** and **managed worker nodes**, particularly when they reside in **private subnets**. The process involves setting up a bastion, configuring network rules, defining IAM policies, and creating bastion sessions for cluster administrators and users.

## Overview
- **Purpose**: The OCI Bastion service provides a secure, controlled gateway for accessing private resources (e.g., Kubernetes API endpoints and worker nodes) within a Virtual Cloud Network (VCN) without exposing them to the public internet.
- **Use Cases**:
  - Accessing a private Kubernetes API endpoint using `kubectl`.
  - Performing administrative tasks (e.g., maintenance, log collection) on managed worker nodes via SSH.
- **Key Features**:
  - Deploys bastion hosts within the VCN as intermediaries for SSH connections.
  - Supports private endpoints for secure access to resources in private subnets.
  - Enforces granular access controls via IAM policies.
- **Roles**:
  - **VCN Administrator**: Configures VCN subnets, route tables, and security lists.
  - **Cluster Administrator**: Creates bastions, manages access to API endpoints and worker nodes, and sets IAM policies.
  - **Cluster User**: Creates bastion sessions to access the API endpoint or worker nodes.

## Prerequisites
- An OKE cluster (e.g., `OKE-PUBLIC-DEMO-1`) with the Kubernetes API endpoint and/or worker nodes in private subnets.
- Permissions to manage VCNs, bastions, clusters, and IAM policies in the target compartment (e.g., `CTDOKE`).
- Access to the OCI Console, Cloud Shell, or a local terminal with OCI CLI installed.
- For SSH access: A public/private SSH key pair, with the public key installed on the worker nodes during node pool creation.
- For `kubectl` access: A configured `kubeconfig` file for the cluster.

## Step-by-Step Process

### 1. VCN Administrator Tasks
To enable bastion access to the Kubernetes API endpoint and worker nodes, configure the necessary networking resources.

#### 1.1 Create a Bastion Subnet
- Create a new private subnet in the same VCN as the cluster (e.g., `acme-dev-vcn`).
  - Navigate to **Networking** > **Virtual Cloud Networks** > Select the VCN.
  - Under **Resources**, select **Subnets** > **Create Subnet**.
  - Configure:
    - **Name**: `subnet-bastion`.
    - **Subnet Type**: Regional (recommended).
    - **CIDR Block**: Non-overlapping with other subnets (e.g., `10.0.3.0/24`).
    - **Subnet Access**: Private Subnet.
    - **Route Table**: Create or select a route table with rules for bastion traffic (e.g., to OCI services or internet via NAT gateway).
    - **Security List**: Create a new security list (e.g., `seclist-bastion`).
    - **DNS Resolution**: Enable (default).
    - **DHCP Options**: Use default.
  - Click **Create Subnet**.
- **Note**: If a subnet already exists for bastion access (e.g., for API endpoint access), reuse it for worker node access.

#### 1.2 Configure Security Rules for Kubernetes API Endpoint
- **Bastion Subnet Security List (`seclist-bastion`)**:
  - Add an egress rule to allow traffic to the Kubernetes API endpoint:
    - Navigate to **Security Lists** > Select `seclist-bastion`.
    - Add Egress Rule:
      - **Destination CIDR**: `10.0.0.0/30` (API endpoint subnet CIDR).
      - **Protocol**: TCP.
      - **Destination Port**: `6443`.
      - **Description**: Allow bastion to Kubernetes API endpoint.
- **API Endpoint Subnet Security List**:
  - Add an ingress rule to allow traffic from the bastion subnet:
    - Navigate to **Security Lists** > Select the API endpoint’s security list (e.g., `seclist-kubernetes-api`).
    - Add Ingress Rule:
      - **Source CIDR**: `10.0.3.0/24` (bastion subnet CIDR).
      - **Protocol**: TCP.
      - **Destination Port**: `6443`.
      - **Description**: Allow bastion traffic to Kubernetes API endpoint.

#### 1.3 Configure Security Rules for Worker Nodes
- **Bastion Subnet Security List (`seclist-bastion`)**:
  - Add an egress rule for SSH traffic to worker nodes:
    - Add Egress Rule:
      - **Destination CIDR**: `10.0.1.0/24` (worker node subnet CIDR).
      - **Protocol**: TCP.
      - **Destination Port**: `22`.
      - **Description**: Allow bastion to worker nodes for SSH.
- **Worker Node Subnet Security List**:
  - Add an ingress rule to allow SSH traffic from the bastion subnet:
    - Navigate to **Security Lists** > Select the worker node’s security list (e.g., `seclist-workernodes`).
    - Add Ingress Rule:
      - **Source CIDR**: `10.0.3.0/24` (bastion subnet CIDR).
      - **Protocol**: TCP.
      - **Destination Port**: `22`.
      - **Description**: Allow bastion SSH traffic to worker nodes.

### 2. Cluster Administrator Tasks
Configure the bastion and IAM policies to enable access.

#### 2.1 Create a Bastion
- In the OCI Console:
  - Navigate to **Identity & Security** > **Bastion** > **Create Bastion**.
  - Configure:
    - **Name**: `oke-bastion`.
    - **Compartment**: `CTDOKE`.
    - **Target VCN**: `acme-dev-vcn`.
    - **Target Subnet**: `subnet-bastion`.
    - **CIDR Block Allow List**: `0.0.0.0/0` (all sources; restrict to specific IPs for better security, e.g., your organization’s IP range).
    - **Maximum Session TTL**: Default or adjust as needed (e.g., 3 hours).
  - Click **Create Bastion**.
- Note the bastion’s **OCID** and share it with cluster users.

#### 2.2 Enable Bastion Agent for Worker Nodes
- For SSH access to managed nodes:
  - Navigate to **Developer Services** > **Kubernetes Cluster (OKE)** > Select `OKE-PUBLIC-DEMO-1` > **Node Pools** > Select `Pool 1` > **Nodes**.
  - Select a node > Enable the **Bastion Agent** in the node details.
  - Wait up to 10 minutes for the agent status to show **Running**.

#### 2.3 Configure IAM Policies
- Ensure users have permissions to manage or use the bastion.
- **Example Policies**:
  - **Security Admins (Full Bastion Management)**:
    ```plaintext
    Allow group SecurityAdmins to manage bastion-family in tenancy
    Allow group SecurityAdmins to manage virtual-network-family in tenancy
    Allow group SecurityAdmins to read instance-family in tenancy
    Allow group SecurityAdmins to read instance-agent-plugins in tenancy
    Allow group SecurityAdmins to inspect work-requests in tenancy
    ```
  - **Bastion Users (Session Management)**:
    ```plaintext
    Allow group BastionUsers to use bastions in tenancy
    Allow group BastionUsers to read instances in tenancy
    Allow group BastionUsers to read virtual-network-family in tenancy
    Allow group BastionUsers to manage bastion-sessions in tenancy
    Allow group BastionUsers to read subnets in tenancy
    Allow group BastionUsers to read instance-agent-plugins in tenancy
    Allow group BastionUsers to read vnic-attachments in tenancy
    Allow group BastionUsers to read vnics in tenancy
    ```
  - **Restricted Access (e.g., Sales Admins)**:
    ```plaintext
    Allow group SalesAdmins to manage bastion-sessions in tenancy where target.resource.name = 'session_username'
    ```
    - Replace `session_username` with the OS username (e.g., `opc` for OKE nodes).
  - **Security Auditors (View-Only)**:
    ```plaintext
    Allow group SecurityAuditors to read bastion-family in tenancy
    ```
  - **Cluster Users (Kubernetes API Endpoint Access)**:
    ```plaintext
    Allow group ClusterUsers to manage bastion-sessions in compartment ABC where all {target.resource.subnet-id = '10.0.0.11/32', target.bastion-session.target-port = '6443'}
    ```
    - Restricts access to the API endpoint subnet (`10.0.0.11/32`) on port `6443`.

### 3. Cluster User Tasks: Accessing Kubernetes API Endpoint
Access the private Kubernetes API endpoint using `kubectl` via a bastion session.

#### Steps
1. **Generate kubeconfig File**:
   - If not already created, generate the `kubeconfig` file using OCI CLI:
     ```bash
     oci ce cluster create-kubeconfig --cluster-id <cluster-ocid> --file ~/.kube/config --region us-phoenix-1
     ```
   - Verify the file exists:
     ```bash
     cat ~/.kube/config
     ```

2. **Edit kubeconfig File**:
   - Locate the `server` field in the `kubeconfig` file (e.g., `server: https://<api-endpoint-ip>:6443`).
   - Replace `<api-endpoint-ip>` with the private IP of the Kubernetes API endpoint (e.g., `10.0.0.11`).
   - **Note**: Obtain the private IP from the cluster’s subnet details or `kubectl cluster-info` if already accessible.

3. **Create a Bastion Session**:
   - In the OCI Console:
     - Navigate to **Identity & Security** > **Bastion** > Select `oke-bastion`.
     - Click **Create Session**.
     - Configure:
       - **Session Type**: Managed SSH.
       - **Target Resource**: Select the cluster’s API endpoint private IP (e.g., `10.0.0.11`).
       - **Target Port**: `6443`.
       - **Session Name**: e.g., `api-endpoint-session`.
       - **SSH Key**: Not required for API endpoint access.
     - Click **Create Session**.
   - Alternatively, use OCI CLI:
     ```bash
     oci bastion session create-managed-ssh --bastion-id <bastion-ocid> --target-resource-id <cluster-ocid> --target-port 6443 --session-ttl-in-seconds 10800
     ```

4. **Obtain SSH Tunnel Command**:
   - In the OCI Console, select the session > Copy the **SSH Command** (e.g., `ssh -i <private-key> -L <local-port>:10.0.0.11:6443 <bastion-session-ip>`).
   - Modify the command:
     - Replace `<private-key>` with the path to your private key (if needed).
     - Replace `<local-port>` with a local port (e.g., `8443`).
     - Add `-N` for background execution.
   - Example:
     ```bash
     ssh -i ~/.ssh/id_rsa -L 8443:10.0.0.11:6443 -N <bastion-session-ip>
     ```

5. **Execute SSH Tunnel**:
   - Run the modified command in Cloud Shell or a local terminal.
   - Keep the terminal open to maintain the tunnel.

6. **Perform kubectl Operations**:
   - Update the `kubeconfig` file’s `server` field to use the local port:
     ```yaml
     server: https://localhost:8443
     ```
   - Run:
     ```bash
     kubectl get nodes
     ```
     - Expected output: Lists the cluster’s nodes, confirming access to the private API endpoint.

### 4. Cluster User Tasks: Accessing Managed Worker Nodes
Access managed worker nodes in a private subnet via SSH using a bastion session.

#### Steps
1. **Enable Bastion Agent**:
   - Ensure the bastion agent is enabled on the target worker node (see Cluster Administrator Tasks, step 2.2).

2. **Create a Bastion Session**:
   - In the OCI Console:
     - Navigate to **Identity & Security** > **Bastion** > Select `oke-bastion`.
     - Click **Create Session**.
     - Configure:
       - **Session Type**: Managed SSH.
       - **Target Resource**: Select the worker node’s compute instance (e.g., private IP `10.0.1.10`).
       - **Target Port**: `22`.
       - **Session Name**: e.g., `worker-node-session`.
       - **SSH Key**: Upload the public key matching the private key used during node pool creation.
     - Click **Create Session**.
   - Alternatively, use OCI CLI:
     ```bash
     oci bastion session create-managed-ssh --bastion-id <bastion-ocid> --target-resource-id <node-instance-ocid> --target-port 22 --session-ttl-in-seconds 10800
     ```

3. **Obtain SSH Tunnel Command**:
   - In the OCI Console, select the session > Copy the **SSH Command** (e.g., `ssh -i <private-key> opc@<node-private-ip>`).
   - Modify the command:
     - Replace `<private-key>` with the path to the private key used during node pool creation (e.g., `~/.ssh/id_rsa`).
     - Example:
       ```bash
       ssh -i ~/.ssh/id_rsa opc@10.0.1.10
       ```

4. **Execute SSH Command**:
   - Run the command in Cloud Shell or a local terminal.
   - If prompted, type `yes` to accept the host key.
   - Expected output: Connects to the worker node with the prompt `opc@<instance-name>`.

5. **Perform Node Operations**:
   - Run administrative commands (e.g., `journalctl` for logs, `df -h` for disk usage).
   - Exit the session:
     ```bash
     exit
     ```

## Security Best Practices
- **Restrict CIDR Allow List**: Use specific IP ranges in the bastion’s CIDR Block Allow List instead of `0.0.0.0/0`.
- **Granular IAM Policies**: Limit user access to specific resources (e.g., API endpoint only or specific nodes).
- **Session Timeout**: Set a reasonable session TTL (e.g., 3 hours) to automatically terminate unused sessions.
- **Private Key Security**: Ensure private keys have `chmod 600` permissions.
- **Monitor Bastion Usage**: Regularly review session activity in the OCI Console.

## Key Notes
- **Bastion Service**: Essential for secure access to private subnet resources without public exposure.
- **Private vs. Public Subnets**:
  - Private subnets require bastion or VCN peering; public subnets allow direct SSH with ingress rules.
  - This demo focuses on private subnets for enhanced security.
- **IAM Policies**: Critical for controlling access; tailor policies to specific roles (e.g., admins, users).
- **Bastion Agent**: Must be enabled and running for SSH access to worker nodes.
- **Official Documentation**: Refer to [OCI official documentation](https://docs.oracle.com) for Bastion service setup, IAM policies, and troubleshooting.

## Next Steps
- **Practical Demo**: Future lessons will provide a hands-on demo of creating bastion sessions using the OCI Console and CLI.
- **Advanced Access Control**: Explore fine-grained IAM policies and Network Security Groups.
- **Cluster Management**: Continue learning about managing OKE clusters and performing node maintenance.

This guide outlines the process of setting up the OCI Bastion service for secure access to an OKE cluster’s private Kubernetes API endpoint and managed worker nodes, covering VCN configuration, IAM policies, and session creation. Stay tuned for detailed demos in upcoming lessons.