# **Demo: Creating and Configuring a Kubernetes Cluster Using Custom Create Workflow with Managed Nodes and Flannel CNI Plugin**

<details>
<summary><strong>📋 Table of Contents</strong></summary>

- [**Demo: Creating and Configuring a Kubernetes Cluster Using Custom Create Workflow with Managed Nodes and Flannel CNI Plugin**](#demo-creating-and-configuring-a-kubernetes-cluster-using-custom-create-workflow-with-managed-nodes-and-flannel-cni-plugin)
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
      - [1.8 Configure Security List Rules](#18-configure-security-list-rules)
        - [Security List for Kubernetes API Endpoint](#security-list-for-kubernetes-api-endpoint)
        - [Security Lists for Worker Nodes and Load Balancers](#security-lists-for-worker-nodes-and-load-balancers)
    - [2. Creating the Kubernetes Cluster](#2-creating-the-kubernetes-cluster)
    - [3. Accessing the Cluster](#3-accessing-the-cluster)
    - [4. Deploying an Application](#4-deploying-an-application)
  - [Key Notes](#key-notes)
  - [Next Steps](#next-steps)

</details>

<br/>

This guide demonstrates how to create and configure a Kubernetes cluster using the **Custom Create workflow** in Oracle Container Engine for Kubernetes (OKE) with managed nodes, leveraging the Flannel CNI plugin. The setup includes a public Kubernetes API endpoint, public load balancers, and private worker nodes within a Virtual Cloud Network (VCN). This demo covers the creation of required networking resources (VCN, internet gateway, NAT gateway, service gateway, route tables, security lists, and subnets), configuration of security list rules, cluster creation, managed node attachment, and application deployment. Bastion host setup is not covered in this demo but will be addressed in future lessons.



For detailed instructions, refer to the [OCI official documentation](https://docs.oracle.com) <br> Under "**Container Engine for Kubernetes**" > <br> "**Preparing for Container Engine for Kubernetes**" > <br>"**Example Network Resource Configurations**."

## Prerequisites
- An OCI account with permissions to create and manage resources in the desired compartment (e.g., `CTDOKE`).
- Familiarity with OCI networking concepts (VCN, subnets, gateways, route tables, security lists).
- Access to the OCI Console and Cloud Shell.

## Step-by-Step Process

### 1. Setting Up Networking Resources

Before creating the cluster, configure the necessary networking resources to support the Kubernetes API endpoint (public), worker nodes (private), and load balancers (public).

#### 1.1 Create a Virtual Cloud Network (VCN)
1. From the **OCI Console** home page, navigate to **Networking** > **Virtual Cloud Networks**.
2. Click **Create VCN**.
3. Configure the VCN:
   - **Name**: Provide a suitable name (e.g., `oke-vcn` or `acme-dev-vcn`).
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
   - **Name**: `routetable-kubernetes-api`.
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
Three security lists are required for the Kubernetes API endpoint, worker nodes, and load balancers.

1. Under **Resources**, select **Security Lists**.
2. Create security lists:
   - **Kubernetes API Endpoint**:
     - **Name**: `seclist-kubernetes-api`.
     - Leave rules empty initially.
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

#### 1.8 Configure Security List Rules

With subnets created, add ingress and egress rules to the associated security lists (`seclist-kubernetes-api`, `seclist-workernodes`, `seclist-loadbalancers`).

##### Security List for Kubernetes API Endpoint
1. Navigate to **Networking** > **Virtual Cloud Networks** > Select the VCN (`acme-dev-vcn`).
2. Under **Resources**, select **Security Lists** > `seclist-kubernetes-api`.
3. Add **Ingress Rules** (as per the OCI documentation):
   - **Rule 1: Worker Nodes to API Endpoint**
     - **Source CIDR**: `10.0.1.0/24` (worker node subnet CIDR).
     - **Protocol**: TCP.
     - **Destination Port**: `6443` (Kubernetes API endpoint port).
     - **Description**: Allow communication from worker nodes to Kubernetes API endpoint.
   - **Rule 2: Worker Nodes to Control Plane**
     - **Source CIDR**: `10.0.1.0/24`.
     - **Protocol**: TCP.
     - **Destination Port**: `12250`.
     - **Description**: Allow worker nodes to control plane communication.
   - **Rule 3: Path Discovery (Worker Nodes to API Endpoint)**
     - **Source CIDR**: `10.0.1.0/24`.
     - **Protocol**: ICMP.
     - **Type**: `3`.
     - **Code**: `5`.
     - **Description**: Path discovery from worker nodes to Kubernetes API endpoint.
   - **Rule 4: Internet Access to API Endpoint**
     - **Source CIDR**: `0.0.0.0/0` (open to the internet).
     - **Protocol**: TCP.
     - **Destination Port**: `6443`.
     - **Description**: Allow internet access to Kubernetes API endpoint.
4. Click **Add Ingress Rules** to apply.
5. Add **Egress Rules**:
   - **Rule 1: Control Plane to OCI Services**
     - **Destination Type**: Service.
     - **Destination Service**: All PHX Services in Oracle Services Network.
     - **Protocol**: TCP.
     - **Destination Port Range**: All.
     - **Description**: Allow Kubernetes control plane to communicate with OCI services.
   - **Rule 2: Path Discovery to OCI Services**
     - **Destination Type**: Service.
     - **Destination Service**: All PHX Services in Oracle Services Network.
     - **Protocol**: ICMP.
     - **Type**: `3`.
     - **Code**: `4`.
     - **Description**: Path discovery for OCI services.
   - **Rule 3: Control Plane to Worker Nodes**
     - **Destination CIDR**: `10.0.1.0/24`.
     - **Protocol**: TCP.
     - **Destination Port Range**: All.
     - **Description**: Allow Kubernetes control plane to communicate with worker nodes.
   - **Rule 4: Path Discovery to Worker Nodes**
     - **Destination CIDR**: `10.0.1.0/24`.
     - **Protocol**: ICMP.
     - **Type**: `3`.
     - **Code**: `4`.
     - **Description**: Path discovery from Kubernetes API endpoint to worker nodes.
6. Click **Add Egress Rules** to apply.

##### Security Lists for Worker Nodes and Load Balancers
- **Worker Nodes (`seclist-workernodes`)**:
  - Add ingress and egress rules as specified in the OCI documentation to allow communication with the Kubernetes API endpoint, load balancers, and OCI services.
  - Example rules include allowing traffic to/from the API endpoint and kube-proxy ports.
- **Load Balancers (`seclist-loadbalancers`)**:
  - **Ingress Rules**: Allow traffic on application-specific ports (e.g., `80` for HTTP, `443` for HTTPS).
    - **Source CIDR**: Can be `0.0.0.0/0` (internet) or a specific CIDR, depending on the application.
    - **Protocol**: TCP.
    - **Ports**: Application-dependent (e.g., `80`, `443`).
  - **Egress Rules**:
    - Allow communication to worker node ports and kube-proxy.
    - Example: Destination CIDR `10.0.1.0/24`, TCP, all ports.
- Note: Refer to the OCI documentation for precise rules for worker nodes and load balancers.

### 2. Creating the Kubernetes Cluster
1. Navigate to **Developer Services** > **Kubernetes Cluster (OKE)**.
2. Click **Create Cluster** > Select **Custom Create** > Click **Submit**.
3. Configure **Cluster Details**:
   - **Name**: Provide a suitable name (e.g., `oke-flannel-demo`).
   - **Compartment**: Select `CTDOKE`.
   - **Kubernetes Version**: Select the latest version (e.g., `1.27.2`).
   - **Advanced Options**: Skip for this demo (covered in separate lessons).
   - Click **Hide Advanced Options** > Click **Next**.

4. Configure **Network Setup**:
   - **Network Type**: Select **Flannel Overlay**.
   - **VCN**: Select `acme-dev-vcn`.
   - **Load Balancer Subnet**: Select `subnet-loadbalancers` (public regional subnet).
   - **Kubernetes API Endpoint Subnet**: Select `subnet-kubernetes-api` (public regional subnet).
   - **Network Security Group**: Leave unchecked (no NSG rules defined).
   - **Public IP for API Endpoint**: Check to assign a public IP, making the API endpoint internet-accessible.
   - **Advanced Options**:
     - **Kubernetes Service CIDR**: Optional; leave blank (must not overlap with VCN CIDR).
     - **Pod CIDR**: Optional; leave blank (must not overlap with VCN/subnet CIDRs, can be outside VCN CIDR).
   - Click **Next**.

5. Configure **Node Pool Details**:
   - **Pool Name**: `pool1`.
   - **Compartment**: `CTDOKE`.
   - **Kubernetes Version**: Default (matches cluster version, e.g., `1.27.2`).
   - **Node Placement**:
     - **Availability Domain**: Select one (e.g., AD-1).
     - **Subnet**: Select `subnet-workernodes` (private regional subnet).
     - **Fault Domains**: Optional; leave blank.
   - **Advanced Options**:
     - **Capacity Type**: Default to **On-Demand Capacity**.
     - Additional domains/subnets: Optional; use one entry for this demo.
   - **Shape and Image**:
     - **Shape**: Default `VM.Standard.E3.Flex`.
     - **Image**: Default OKE Worker Node Image (Oracle Linux 8).
   - **Node Pool Options**:
     - **Node Count**: Default `3`.
     - **Boot Volume**: Default settings (size and encryption).
     - **Cordon and Drain Settings**: Default.
     - **Initialization Script**: Optional; leave blank (must be in cloud-init format, e.g., `.yaml`).
     - **Kubernetes Labels**: Default (`name: pool1`); optionally add custom labels.
     - **Tags**: Optional; add node pool or worker node tags as needed.
     - **SSH Key**: Paste a public SSH key for worker node access (generated previously, e.g., `id_rsa.pub`).
   - Click **Next**.

6. **Review and Create**:
   - Verify details:
     - Cluster: Name, compartment, version, Flannel overlay, VCN, subnets.
     - Node Pool: `pool1`, 3 nodes, `VM.Standard.E3.Flex`, Oracle Linux 8.
   - **Cluster Type**: Check **Create as Basic Cluster** (no enhanced features used; uncheck for enhanced cluster).
   - Click **Create Cluster**.
   - The cluster enters **Creating** status, followed by node pool creation and attachment.

7. Monitor creation in **Work Requests**:
   - **Cluster Create**: In progress initially, then **Succeeded**.
   - **Node Pool Create**: Follows cluster creation, attaches 3 nodes.
   - Once complete, the cluster status changes to **Active** (green).

### 3. Accessing the Cluster
1. On the **Cluster Details** page, click **Access Cluster**.
2. Choose **Cloud Shell Access** (local access covered in future lessons).
3. Copy the provided command to configure the `kubeconfig` file for `kubectl`.
4. In **Cloud Shell**, paste and run the command:
   ```
   oci ce cluster create-kubeconfig --cluster-id <cluster-id> --file $HOME/.kube/config --region <region>
   ```
   - This configures the `kubeconfig` file for cluster access.
5. Verify connectivity:
   ```
   kubectl get nodes
   ```
   - Lists 3 worker nodes with IPs matching those in the node pool (`pool1`).

### 4. Deploying an Application
1. Create a deployment with 3 Nginx pods:
   ```
   kubectl create deployment mahindra-deployment --image=nginx --replicas=3
   ```
   - Confirms deployment creation.
2. Check deployment status:
   ```
   kubectl get deployments
   ```
   - Verifies 3/3 pods are running.
3. List pods:
   ```
   kubectl get pods
   ```
   - Shows 3 pods for `mahindra-deployment`.
4. Expose the deployment via a load balancer:
   ```
   kubectl expose deployment mahindra-deployment --type=LoadBalancer --name=lb-service
   ```
   - Creates a service (`lb-service`) of type LoadBalancer.
5. Check service status:
   ```
   kubectl get services
   ```
   - Initially, the external IP is `<pending>`.
   - Rerun until a public IP is assigned.
6. Access the Nginx web server:
   - Copy the external IP and open it in a browser.
   - Confirms the Nginx server is accessible via the load balancer.

## Key Notes
- **Flannel CNI Plugin**: Used for pod networking, with worker nodes in a private subnet for security. Enables pod networking without using VCN CIDR.
- **Network Configuration**:
  - **Public Subnets**: Host the Kubernetes API endpoint and load balancers, accessible from the internet.
  - **Private Subnet**: Hosts worker nodes, accessible only within the VCN.
  - **CIDR Blocks**: Ensure non-overlapping CIDR blocks (e.g., `10.0.0.0/30` for API endpoint, `10.0.1.0/24` for worker nodes, `10.0.2.0/24` for load balancers). Kubernetes service and pod CIDRs must also be non-overlapping.
- **Security Lists**: Rules are critical for controlling traffic to/from subnets. Tailor load balancer ingress rules (e.g., ports `80`, `443`) to application needs. Refer to the OCI documentation for specific rules.
- **Custom Create Workflow**: Offers fine-grained control over network and node configurations.
- **Cluster Type**: Basic cluster used (no enhanced features); select enhanced for additional capabilities.
- **Bastion Host**: Not covered in this demo but will be addressed in future lessons for accessing private resources.
- **OCI Documentation**: Essential for detailed guidance on networking setup, security list rules, and advanced configurations (see "Example Network Resource Configurations").
- **Accessing Private Nodes**: Requires a bastion host (covered in future lessons).

## Next Steps
- **Bastion Host Setup**: Future demos will cover creating and using a bastion host to access private resources.
- **Cluster Access**: Upcoming lessons will explore local access and advanced cluster management.
- **Enhanced Features**: Separate lessons will cover advanced options (e.g., virtual nodes, add-ons).

This demo successfully configured networking resources, security list rules, created a Kubernetes cluster with the Flannel CNI plugin, attached managed nodes, and deployed an Nginx application accessible via a load balancer. The cluster is now active and ready for use.