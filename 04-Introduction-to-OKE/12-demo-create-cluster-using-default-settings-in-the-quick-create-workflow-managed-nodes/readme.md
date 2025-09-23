# **Demo: Creating a Kubernetes Cluster Using Quick Create Workflow with Managed Nodes**

This guide demonstrates how to create a Kubernetes cluster using the **Quick Create workflow** in Oracle Container Engine for Kubernetes (OKE) with managed nodes, leveraging default settings. The demo is performed within the Oracle Cloud Infrastructure (OCI) console.

<details>
<summary><strong>📋 Table of Contents</strong></summary>

- [**Demo: Creating a Kubernetes Cluster Using Quick Create Workflow with Managed Nodes**](#demo-creating-a-kubernetes-cluster-using-quick-create-workflow-with-managed-nodes)
  - [Prerequisites](#prerequisites)
  - [Step-by-Step Process](#step-by-step-process)
    - [1. Accessing the Kubernetes Cluster Creation Page](#1-accessing-the-kubernetes-cluster-creation-page)
    - [2. Initiating Cluster Creation](#2-initiating-cluster-creation)
    - [3. Configuring Cluster Details](#3-configuring-cluster-details)
      - [Cluster Name](#cluster-name)
      - [Kubernetes Version](#kubernetes-version)
      - [Kubernetes API Endpoint](#kubernetes-api-endpoint)
      - [Node Type](#node-type)
      - [Kubernetes Worker Nodes](#kubernetes-worker-nodes)
      - [Shape and Image](#shape-and-image)
      - [Node Count](#node-count)
      - [Advanced Options](#advanced-options)
      - [Cluster Type](#cluster-type)
    - [4. Reviewing and Creating the Cluster](#4-reviewing-and-creating-the-cluster)
    - [5. Monitoring Cluster Creation](#5-monitoring-cluster-creation)
    - [6. Verifying Node Pool Creation](#6-verifying-node-pool-creation)
  - [Key Notes](#key-notes)
  - [Next Steps](#next-steps)

</details>

## Prerequisites
- An OCI account with appropriate permissions.
- Access to a compartment where you can create and manage resources (e.g., `CTDOKE` compartment used in this demo).

## Step-by-Step Process

### 1. Accessing the Kubernetes Cluster Creation Page
1. Log in to your **OCI Console**.
2. Click the **Navigation Menu** (hamburger icon) in the top-left corner.
3. Navigate to **Developer Services** > **Kubernetes Cluster (OKE)** under the "Containers & Artifacts" section.
4. From the **List Scope**, select the compartment where you have permissions to work (e.g., `CTDOKE`).
   - Note: This demo assumes administrator privileges for the selected compartment.

### 2. Initiating Cluster Creation
1. On the **Cluster List** page, click **Create Cluster**.
2. In the **Create Cluster** dialog box, choose between:
   - **Quick Create**: Uses default settings to create a cluster and its resources.
   - **Custom Create**: Allows fine-grained control over configurations.
3. Select **Quick Create** (default) and click **Submit**.
   - A summary of the new resources to be created (e.g., VCN, subnets, internet gateways, route tables, security lists) is displayed.

### 3. Configuring Cluster Details
On the **Cluster Create** page, you can accept default configurations or customize them. Below are the key configuration options:

#### Cluster Name
- Default: Auto-generated.
- Example: Change to `Public-Demo-1` to reflect that managed nodes will be in a public subnet.
- Ensure the cluster is created in the intended compartment (e.g., `CTDOKE`).

#### Kubernetes Version
- Specifies the Kubernetes version for control plane and worker nodes.
- Default: `1.27.2` (used in this demo).
- Alternative: Select a different version from the dropdown list if needed.

#### Kubernetes API Endpoint
- Defines access to the cluster’s Kubernetes API endpoint:
  - **Private Endpoint**: Creates a regional private subnet, assigns a private IP address.
  - **Public Endpoint**: Creates a public regional subnet, assigns both public and private IP addresses.
- Default: **Public Endpoint** (used in this demo).

#### Node Type
- Specifies the type of worker nodes in the first node pool:
  - **Managed**: You are responsible for managing worker nodes.
  - **Virtual**: Covered in future demos.
- Selection: **Managed** (used in this demo).

#### Kubernetes Worker Nodes
- Defines access to worker nodes:
  - **Private Workers**: Hosted in a private regional subnet, assigned private IP addresses (recommended for security).
  - **Public Workers**: Hosted in a public regional subnet, assigned both public and private IP addresses.
- Selection: **Public Workers** (used in this demo for demonstration purposes).

#### Shape and Image
- **Shape**: Determines CPU and memory allocation for worker nodes.
  - Default: A flexible shape allowing customization of CPU and memory.
  - View supported shapes in your tenancy by clicking the shape list.
  - Selection: Default shape (used in this demo).
- **Image**: Defines the operating system and software for the worker nodes.
  - **OKE Worker Node Image**: Recommended, built on Oracle Linux with pre-configured software for Kubernetes.
  - **Platform Image**: Oracle Linux only, requiring OKE to install necessary software on first boot.
  - Selection: **OKE Worker Node Image (Oracle Linux 8)** (default, used in this demo).
  - To change: Click **Change Image** and browse available images.

#### Node Count
- Specifies the number of worker nodes in the node pool.
- Nodes are distributed evenly across availability domains (or fault domains in single-availability-domain regions).
- Default: 3 nodes (used in this demo).

#### Advanced Options
- Click **Show Advanced Options** to configure:
  - **Boot Volume**: Covered in later demos.
  - **Image Verification**: Covered in later demos.
  - **SSH Key**: Add an SSH key for accessing managed nodes.
  - **Labels**: Optional, for targeting workloads to specific node pools.
- **Adding an SSH Key**:
  1. Open the **Cloud Shell** in the OCI Console.
  2. Navigate to the default SSH directory: `cd ~/.ssh`.
  3. Generate an SSH key pair: `ssh-keygen`.
     - Accept the default file name (`id_rsa`).
     - Leave the passphrase blank (press Enter twice).
  4. View the public key: `cat ~/.ssh/id_rsa.pub`.
  5. Copy the public key.
  6. In the console, select **Paste a Public Key** and paste the key.
     - This key is installed on all worker nodes for SSH access.
  - Selection: Add SSH key (used in this demo for future SSH access demos).

#### Cluster Type
- By default, an **Enhanced Cluster** is created with additional features.
- To create a **Basic Cluster**, check the box for **Create as Basic Cluster**.
- Selection: **Basic Cluster** (used in this demo).

### 4. Reviewing and Creating the Cluster
1. On the **Review** page, verify all configurations:
   - Cluster name: `Public-Demo-1`.
   - Cluster type: Basic.
   - Kubernetes version: `1.27.2`.
   - Network details: Public API endpoint, public worker nodes.
   - Node pool details: Managed nodes, 3 nodes, OKE image (Oracle Linux 8).
2. Click **Create Cluster**.
   - The console initiates creation of network resources (VCN, internet gateways, route tables, security lists, subnets), followed by the cluster and node pool.

### 5. Monitoring Cluster Creation
- The cluster initially appears with a **Creating** status in the console.
- Check progress in the **Work Requests** section:
  - **Cluster Create**: Indicates cluster creation status.
  - **Node Pool Create**: Indicates node pool creation and attachment status.
- Once complete, the cluster status changes to **Active** (green).
- Example: The demo shows the cluster transitioning to **Active** with one node pool containing three nodes running Kubernetes `1.27.2`.

### 6. Verifying Node Pool Creation
- Navigate to the **Work Requests** section to confirm the **Node Pool Create** operation is **Succeeded**.
- Return to the **Cluster Details** page to verify the node pool is attached and the cluster is fully operational.

## Key Notes
- **Public vs. Private Workers**:
  - Public workers are used in this demo for simplicity but are less secure.
  - Private workers are recommended for production to limit internet exposure.
- **SSH Key**: Added to enable SSH access to managed nodes in future demos, particularly for public subnet nodes.
- **Node Distribution**: Nodes are automatically distributed across availability domains (or fault domains in single-AD regions) for high availability.
- **Basic vs. Enhanced Cluster**: Basic clusters are simpler and sufficient for this demo, while enhanced clusters offer additional features.

## Next Steps
- **Cluster Access**: Future demos will cover setting up cluster access using Cloud Shell and a local terminal.
- **SSH to Managed Nodes**: Upcoming demos will demonstrate connecting to managed nodes in public subnets using the SSH key configured in this demo.

This demo successfully created a Kubernetes cluster named `Public-Demo-1` using the Quick Create workflow with managed nodes in a public subnet. Refer to the OCI Console and official documentation for further details.