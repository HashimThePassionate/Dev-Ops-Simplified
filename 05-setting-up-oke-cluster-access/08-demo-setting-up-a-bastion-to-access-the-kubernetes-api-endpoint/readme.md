# **Demo: Setting Up an OCI Bastion to Access a Private Kubernetes API Endpoint**

This guide, presented by Mahendra Mehra, Senior OCI Instructor at Oracle University, demonstrates how to configure an **Oracle Cloud Infrastructure (OCI) Bastion service** to securely access the **Kubernetes API endpoint** of an OKE cluster hosted in a **private subnet**. The demo uses a new cluster, `OKE-private-bastion-demo`, with both the API endpoint and worker nodes in private subnets and load balancers in a public subnet. The bastion is hosted in the same subnet as the API endpoint to simplify configuration.

<details>
<summary><strong>📋 Table of Contents</strong></summary>

- [**Demo: Setting Up an OCI Bastion to Access a Private Kubernetes API Endpoint**](#demo-setting-up-an-oci-bastion-to-access-a-private-kubernetes-api-endpoint)
  - [Prerequisites](#prerequisites)
  - [Cluster Setup Overview](#cluster-setup-overview)
  - [Step-by-Step Process](#step-by-step-process)
    - [1. Create the OKE Cluster](#1-create-the-oke-cluster)
    - [2. VCN Administrator Tasks: Configure Networking](#2-vcn-administrator-tasks-configure-networking)
    - [3. Cluster Administrator Tasks: Create Bastion](#3-cluster-administrator-tasks-create-bastion)
    - [4. Cluster User Tasks: Create Bastion Session and Access Cluster](#4-cluster-user-tasks-create-bastion-session-and-access-cluster)

</details>

## Prerequisites
- An OKE cluster (`OKE-private-bastion-demo`) with:
  - Kubernetes API endpoint in a private subnet (e.g., `10.0.0.0/28`).
  - Worker nodes in a private subnet (e.g., `10.0.1.0/24`).
  - Load balancers in a public subnet.
- A public/private SSH key pair, with the public key provided during node pool creation.
- OCI CLI and `kubectl` installed and configured on the local machine.
- Administrative privileges in the `CTDOKE` compartment (or specific IAM policies for limited users; see previous lessons).
- Access to the OCI Console and a local terminal (Oracle Linux 8 in this demo).

## Cluster Setup Overview
- **Cluster Name**: `OKE-private-bastion-demo`.
- **Compartment**: `CTDOKE`.
- **Kubernetes Version**: `1.27.2`.
- **API Endpoint**: Private subnet (`10.0.0.0/28`).
- **Worker Nodes**: Private subnet (`10.0.1.0/24`), 3 nodes, `VM.Standard.E3.Flex` shape, Oracle Linux 8.
- **Load Balancers**: Public subnet.
- **Node Type**: Managed nodes.
- **SSH Key**: Public key provided during node pool creation.
- **Cluster Type**: Enhanced (default for Quick Create with private endpoints).

## Step-by-Step Process

### 1. Create the OKE Cluster
1. **Navigate to OKE Console**:
   - In the OCI Console, go to **Developer Services** > **Kubernetes Cluster (OKE)**.
   - Ensure the `CTDOKE` compartment is selected.
   - Click **Create Cluster** > Select **Quick Create** > Click **Submit**.

2. **Configure Cluster**:
   - **Name**: `OKE-private-bastion-demo`.
   - **Compartment**: `CTDOKE`.
   - **Kubernetes Version**: `1.27.2` (default).
   - **Kubernetes API Endpoint**: Select **Private Endpoint**.
   - **Node Type**: Select **Managed Nodes**.
   - **Node Placement**: Select **Private Workers**.
   - **Shape**: `VM.Standard.E3.Flex` (default).
   - **Image**: Oracle Linux 8 (default).
   - **Node Count**: `3`.
   - **Advanced Options**:
     - Paste the public SSH key for worker node access:
       - In the local terminal:
         ```bash
         cd ~/.ssh
         cat id_rsa.pub
         ```
         - Copy the public key content.
       - In the OCI Console, paste the key in the **SSH Key** field.
     - Leave default labels and other settings.
   - Click **Next** > Review details > Select **Create as Enhanced Cluster** > Click **Create Cluster**.
   - Wait for the cluster status to change to **Active** (takes a few minutes).

3. **Verify Private Endpoint**:
   - In the **Cluster Details** page, note the **Kubernetes API Private Endpoint IP** (e.g., `10.0.0.11`).
   - Attempt to access the cluster:
     - Click **Access Cluster** > Note that **Cloud Shell Access** is unavailable due to the private endpoint.
     - Select **Local Access** and copy the `kubeconfig` command:
       ```bash
       oci ce cluster create-kubeconfig --cluster-id <cluster-ocid> --file $HOME/.kube/config --region us-phoenix-1
       ```
     - In the local terminal:
       ```bash
       cd ~
       oci ce cluster create-kubeconfig --cluster-id <cluster-ocid> --file $HOME/.kube/config --region us-phoenix-1
       ```
     - Try:
       ```bash
       kubectl get nodes
       ```
       - Expected output: Error (e.g., `no route to host`), confirming the private endpoint is inaccessible without a bastion.

### 2. VCN Administrator Tasks: Configure Networking
The bastion is hosted in the same private subnet as the Kubernetes API endpoint (`10.0.0.0/28`) to reuse existing infrastructure.

1. **Verify Subnets**:
   - In the OCI Console, go to **Networking** > **Virtual Cloud Networks** > Select the VCN (e.g., `oke-vcn-private-bastion-demo`).
   - Under **Subnets**, confirm:
     - **API Endpoint Subnet**: `oke-khapisubnet-...` (`10.0.0.0/28`, private).
     - **Worker Node Subnet**: `oke-nodesubnet-...` (`10.0.1.0/24`, private).
     - **Load Balancer Subnet**: `oke-lb-subnet-...` (public).
   - Copy the API endpoint subnet CIDR (`10.0.0.0/28`).

2. **Add Security Rules**:
   - **Egress Rule (Bastion to API Endpoint)**:
     - Navigate to **Subnets** > Select `oke-khapisubnet-...` > Click the associated **Security List** (e.g., `seclist-kubernetes-api`).
     - Click **Add Egress Rules**:
       - **Destination CIDR**: `10.0.0.0/28` (same as API endpoint subnet, as bastion is co-located).
       - **Protocol**: TCP.
       - **Destination Port**: `6443`.
       - **Description**: Allow bastion to Kubernetes API endpoint communication.
     - Click **Add Egress Rules**.
   - **Ingress Rule (Bastion to API Endpoint)**:
     - Click **Ingress Rules** > **Add Ingress Rules**:
       - **Source CIDR**: `10.0.0.0/28` (bastion subnet, same as API endpoint subnet).
       - **Protocol**: TCP.
       - **Destination Port**: `6443`.
       - **Description**: Allow bastion to Kubernetes API endpoint communication.
     - Click **Add Ingress Rules**.
   - **Note**: Co-locating the bastion and API endpoint in the same subnet simplifies rules, as traffic remains internal.

### 3. Cluster Administrator Tasks: Create Bastion
1. **Create Bastion**:
   - In the OCI Console, go to **Identity & Security** > **Bastion** > **Create Bastion**.
   - Configure:
     - **Name**: `bastion-oke`.
     - **Compartment**: `CTDOKE`.
     - **Target VCN**: `oke-vcn-private-bastion-demo`.
     - **Target Subnet**: `oke-khapisubnet-...` (same as API endpoint subnet).
     - **CIDR Block Allow List**: `0.0.0.0/0` (all sources; restrict to specific IPs for better security).
     - **Maximum Session TTL**: Default (180 minutes).
   - Click **Create Bastion**.
   - Wait for the bastion status to change to **Active**.

2. **Verify IAM Policies**:
   - The demo uses administrative privileges, granting full access to bastion and cluster resources.
   - For limited users, ensure IAM policies allow:
     - `manage bastions`, `manage bastion-sessions`, `read virtual-network-family`, `read instances`, etc. (see previous lessons).
   - Example policy for cluster users:
     ```plaintext
     Allow group ClusterUsers to manage bastion-sessions in compartment CTDOKE where all {target.resource.subnet-id = '10.0.0.0/28', target.bastion-session.target-port = '6443'}
     ```

### 4. Cluster User Tasks: Create Bastion Session and Access Cluster
1. **Edit kubeconfig File**:
   - In the local terminal:
     ```bash
     cd ~/.kube
     vi config
     ```
   - Locate the `server` field (e.g., `server: https://10.0.0.11:6443`).
   - Replace the IP with `localhost` or `127.0.0.1`:
     ```yaml
     server: https://127.0.0.1:6443
     ```
   - Save and exit.

2. **Create Bastion Session**:
   - In the OCI Console:
     - Go to **Identity & Security** > **Bastion** > Select `bastion-oke`.
     - Click **Create Session**.
     - Configure:
       - **Session Type**: SSH Port Forwarding Session.
       - **Session Name**: `bastionokesession`.
       - **Target Resource**: Select **IP Address** and enter the API endpoint’s private IP (e.g., `10.0.0.11`).
       - **Port**: