# **Demo: Creating a Kubernetes Cluster Using Custom Create Workflow with Managed Nodes and Flannel CNI Plugin**

<details>
<summary>📚 Table of Contents</summary>

- [**Demo: Creating a Kubernetes Cluster Using Custom Create Workflow with Managed Nodes and Flannel CNI Plugin**](#demo-creating-a-kubernetes-cluster-using-custom-create-workflow-with-managed-nodes-and-flannel-cni-plugin)
  - [Prerequisites](#prerequisites)
  - [Step-by-Step Process](#step-by-step-process)
    - [1. Setting Up Networking Resources](#1-setting-up-networking-resources)
      - [1.1 Create a Virtual Cloud Network (VCN)](#11-create-a-virtual-cloud-network-vcn)
      - [1.2 Create an Internet Gateway](#12-create-an-internet-gateway)
      - [1.3 Create a NAT Gateway](#13-create-a-nat-gateway)
      - [1.4 Create a Service Gateway](#14-create-a-service-gateway)
      - [1.5 Create Route Tables](#15-create-route-tables)
        - [Route Table for Kubernetes API Endpoint](#route-table-for-kubernetes-api-endpoint)
        - [Route Table for Private Worker Nodes](#route-table-for-private-worker-nodes)
        - [Route Table for Public Load Balancers](#route-table-for-public-load-balancers)
      - [1.6 Create Security Lists](#16-create-security-lists)
      - [1.7 Create Subnets](#17-create-subnets)
        - [Subnet for Kubernetes API Endpoint](#subnet-for-kubernetes-api-endpoint)
        - [Subnet for Private Worker Nodes](#subnet-for-private-worker-nodes)
        - [Subnet for Load Balancers](#subnet-for-load-balancers)
      - [1.8 Next Steps for Security Lists](#18-next-steps-for-security-lists)
    - [2. Creating the Cluster Using Custom Create Workflow](#2-creating-the-cluster-using-custom-create-workflow)
  - [Key Notes](#key-notes)
  - [Next Steps](#next-steps)

</details>

<br/>


This guide demonstrates how to create a Kubernetes cluster using the **Custom Create workflow** in Oracle Container Engine for Kubernetes (OKE) with managed nodes, leveraging the Flannel CNI plugin. The setup includes a public Kubernetes API endpoint, public load balancers, and private worker nodes within a Virtual Cloud Network (VCN). This demo covers the creation of required networking resources (VCN, internet gateway, NAT gateway, service gateway, route tables, security lists, and subnets) before creating the cluster. Bastion host setup is not covered in this demo but will be addressed in future lessons.

For detailed instructions, refer to the [OCI official documentation](https://docs.oracle.com) under "Container Engine for Kubernetes" > "Preparing for Container Engine for Kubernetes" > "Example Network Resource Configurations."

## Prerequisites
- An OCI account with permissions to create and manage resources in the desired compartment.
- Familiarity with OCI networking concepts (VCN, subnets, gateways, route tables, security lists).
- Access to the OCI Console.

## Step-by-Step Process

### 1. Setting Up Networking Resources

Before creating the cluster, configure the necessary networking resources to support the Kubernetes API endpoint (public), worker nodes (private), and load balancers (public).

#### 1.1 Create a Virtual Cloud Network (VCN)
1. From the **OCI Console** home page, navigate to **Networking** > **Virtual Cloud Networks**.
2. Click **Create VCN**.
3. Configure the VCN:
   - **Name**: Provide a suitable name (e.g., `oke-vcn`).
   - **CIDR Block**: `10.0.0.0/16` (provides a large IP address range).
   - **DNS Resolution**: Ensure the checkbox is selected.
4. Click **Create VCN**.
   - Wait for the VCN to become **Available**.

#### 1.2 Create an Internet Gateway
1. In the VCN details page, under **Resources**, select **Internet Gateways**.
2. Click **Create Internet Gateway**.
3. Configure:
   - **Name**: Provide a name (e.g., `oke-internet-gateway`).
   - Leave other settings as default.
4. Click **Create Internet Gateway**.
   - Verify the gateway is **Available**.

#### 1.3 Create a NAT Gateway
1. Under **Resources**, select **NAT Gateways**.
2. Click **Create NAT Gateway**.
3. Configure:
   - **Name**: Provide a name (e.g., `oke-nat-gateway`).
   - Leave other settings as default.
4. Click **Create NAT Gateway**.

#### 1.4 Create a Service Gateway
1. Under **Resources**, select **Service Gateways**.
2. Click **Create Service Gateway**.
3. Configure:
   - **Name**: Provide a name (e.g., `oke-service-gateway`).
   - **Services**: Select **All PHX Services in Oracle Services Network** (for Phoenix region; adjust for other regions as needed).
4. Click **Create Service Gateway**.

#### 1.5 Create Route Tables
Three route tables are required for the Kubernetes API endpoint (public), worker nodes (private), and load balancers (public).

##### Route Table for Kubernetes API Endpoint
1. Under **Resources**, select **Route Tables**.
2. Click **Create Route Table**.
3. Configure:
   - **Name**: `routetable-kubernetes-api` (or as per documentation).
   - **Route Rule**:
     - **Target Type**: Internet Gateway.
     - **Destination CIDR Block**: `0.0.0.0/0` (open access to the internet).
     - **Target Internet Gateway**: Select the created internet gateway (`oke-internet-gateway`).
4. Click **Create**.

##### Route Table for Private Worker Nodes
1. Click **Create Route Table**.
2. Configure:
   - **Name**: `routetable-workernodes`.
   - **Route Rule 1** (Internet traffic):
     - **Target Type**: NAT Gateway.
     - **Destination CIDR Block**: `0.0.0.0/0`.
     - **Target NAT Gateway**: Select the created NAT gateway (`oke-nat-gateway`).
   - **Route Rule 2** (OCI services):
     - **Target Type**: Service Gateway.
     - **Destination Service**: All PHX Services in Oracle Services Network.
     - **Target Service Gateway**: Select the created service gateway (`oke-service-gateway`).
3. Click **Create**.

##### Route Table for Public Load Balancers
1. Click **Create Route Table**.
2. Configure:
   - **Name**: `routetable-loadbalancers`.
   - **Route Rule**:
     - **Target Type**: Internet Gateway.
     - **Destination CIDR Block**: `0.0.0.0/0`.
     - **Target Internet Gateway**: Select the created internet gateway (`oke-internet-gateway`).
3. Click **Create**.

#### 1.6 Create Security Lists
Three security lists are required for the Kubernetes API endpoint, worker nodes, and load balancers. For this demo, create empty security lists initially; rules will be added after subnet creation.

1. Under **Resources**, select **Security Lists**.
2. Create security lists:
   - **Kubernetes API Endpoint**:
     - **Name**: `seclist-kubernetes-api`.
     - Leave rules empty for now.
     - Click **Create Security List**.
   - **Private Worker Nodes**:
     - **Name**: `seclist-workernodes`.
     - Leave rules empty.
     - Click **Create Security List**.
   - **Load Balancers**:
     - **Name**: `seclist-loadbalancers`.
     - Leave rules empty.
     - Click **Create Security List**.

#### 1.7 Create Subnets
Three subnets are required: one for the Kubernetes API endpoint (public), one for worker nodes (private), and one for load balancers (public).

##### Subnet for Kubernetes API Endpoint
1. Under **Resources**, select **Subnets**.
2. Click **Create Subnet**.
3. Configure:
   - **Name**: `subnet-kubernetes-api`.
   - **Subnet Type**: Regional (recommended).
   - **CIDR Block**: `10.0.0.0/30` (provides 4 IP addresses, sufficient for the API endpoint).
   - **Subnet Access**: Public Subnet.
   - **Route Table**: Select `routetable-kubernetes-api`.
   - **DNS Resolution**: Ensure checked.
   - **DHCP Options**: Select default DHCP options for the VCN.
   - **Security List**: Select `seclist-kubernetes-api`.
4. Click **Create Subnet**.
   - Wait for the subnet to become **Available**.

##### Subnet for Private Worker Nodes
1. Click **Create Subnet**.
2. Configure:
   - **Name**: `subnet-workernodes`.
   - **Subnet Type**: Regional (recommended).
   - **CIDR Block**: `10.0.1.0/24` (provides 256 IP addresses, sufficient for this demo).
   - **Subnet Access**: Private Subnet.
   - **Route Table**: Select `routetable-workernodes`.
   - **DNS Resolution**: Ensure checked.
   - **DHCP Options**: Select default DHCP options.
   - **Security List**: Select `seclist-workernodes`.
3. Click **Create Subnet**.

##### Subnet for Load Balancers
1. Click **Create Subnet**.
2. Configure:
   - **Name**: `subnet-loadbalancers`.
   - **Subnet Type**: Regional (recommended).
   - **CIDR Block**: `10.0.2.0/24` (provides 256 IP addresses).
   - **Subnet Access**: Public Subnet.
   - **Route Table**: Select `routetable-loadbalancers`.
   - **DNS Resolution**: Ensure checked.
   - **DHCP Options**: Select default DHCP options.
   - **Security List**: Select `seclist-loadbalancers`.
3. Click **Create Subnet**.

#### 1.8 Next Steps for Security Lists
- Security lists currently have no rules. Ingress and egress rules must be added to control traffic for each subnet (Kubernetes API endpoint, worker nodes, and load balancers).
- Refer to the [OCI official documentation](https://docs.oracle.com) for specific ingress and egress rules required for each subnet. These will be covered in the next part of the demo.

### 2. Creating the Cluster Using Custom Create Workflow
The next steps involve creating the Kubernetes cluster and attaching managed nodes, but these are not covered in this part of the demo due to time constraints. The process typically includes:
1. Navigating to **Developer Services** > **Kubernetes Cluster (OKE)**.
2. Selecting **Create Cluster** > **Custom Create**.
3. Configuring:
   - Cluster name and compartment.
   - Kubernetes version (e.g., `1.27.2`).
   - Network settings: Select the created VCN, subnets, and Flannel CNI plugin.
   - Node pool settings: Managed nodes, private subnet (`subnet-workernodes`), node count, shape, and image (e.g., OKE Worker Node Image with Oracle Linux 8).
4. Reviewing and creating the cluster.

These steps will be detailed in the next part of the demo, along with adding security list rules.

## Key Notes
- **Flannel CNI Plugin**: Used for pod networking, with worker nodes in a private subnet for security.
- **Network Configuration**:
  - **Public Subnets**: Host the Kubernetes API endpoint and load balancers, accessible from the internet.
  - **Private Subnet**: Hosts worker nodes, accessible only within the VCN.
  - **CIDR Blocks**: Ensure non-overlapping CIDR blocks (e.g., `10.0.0.0/30` for API endpoint, `10.0.1.0/24` for worker nodes, `10.0.2.0/24` for load balancers).
- **Security Lists**: Rules must be added post-subnet creation to control traffic. Refer to the OCI documentation for specific rules.
- **Bastion Host**: Not covered in this demo but will be addressed in future lessons for accessing private resources.
- **Documentation**: The [OCI official documentation](https://docs.oracle.com) provides detailed guidance on networking setup for this configuration (see "Example Network Resource Configurations").

## Next Steps
- **Add Security List Rules**: Configure ingress and egress rules for the Kubernetes API endpoint, worker nodes, and load balancer subnets.
- **Complete Cluster Creation**: Use the Custom Create workflow to create the cluster and attach managed nodes.
- **Bastion Setup**: Future demos will cover setting up a bastion host for accessing private resources.
- **Cluster Access**: Upcoming lessons will demonstrate connecting to the cluster and nodes.

This demo has laid the foundation by configuring the necessary networking resources for a Kubernetes cluster with the Flannel CNI plugin. Stay tuned for the continuation, where security list rules and cluster creation will be completed.