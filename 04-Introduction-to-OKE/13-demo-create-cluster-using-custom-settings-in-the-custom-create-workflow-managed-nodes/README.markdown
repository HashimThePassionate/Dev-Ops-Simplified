# Demo: Continuing Cluster Creation with Custom Create Workflow and Managed Nodes (Flannel CNI Plugin)

This guide continues from the previous demo, where we configured networking resources (VCN, internet gateway, NAT gateway, service gateway, route tables, security lists, and subnets) for a Kubernetes cluster using the **Custom Create workflow** in Oracle Container Engine for Kubernetes (OKE). In this part, we add security list rules for the subnets, create the cluster with the Flannel CNI plugin, attach managed nodes, and demonstrate cluster access and deployment. The setup includes a public Kubernetes API endpoint, private worker nodes, and public load balancers.

For detailed instructions, refer to the [OCI official documentation](https://docs.oracle.com) under "Container Engine for Kubernetes" > "Preparing for Container Engine for Kubernetes" > "Example Network Resource Configurations."

## Prerequisites
- Completion of the previous demo, with networking resources (VCN, gateways, route tables, security lists, and subnets) configured.
- Permissions to manage resources in the target compartment (e.g., `CTDOKE`).
- Access to the OCI Console and Cloud Shell.

## Step-by-Step Process

### 1. Configuring Security List Rules

With subnets created for the Kubernetes API endpoint (public), worker nodes (private), and load balancers (public), the next step is to add ingress and egress rules to the associated security lists (`seclist-kubernetes-api`, `seclist-workernodes`, `seclist-loadbalancers`).

#### 1.1 Security List for Kubernetes API Endpoint
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

#### 1.2 Security Lists for Worker Nodes and Load Balancers
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
- Note: These rules are added off-screen in the demo to save time. Refer to the OCI documentation for precise rules.

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
- **Security List Rules**:
  - Critical for controlling traffic to/from subnets.
  - Tailor load balancer ingress rules (e.g., ports `80`, `443`) to application needs.
  - Refer to OCI documentation for precise rules for worker nodes and load balancers.
- **Flannel CNI Plugin**: Enables pod networking without using VCN CIDR, with private worker nodes for security.
- **Custom Create Workflow**: Offers fine-grained control over network and node configurations.
- **Cluster Type**: Basic cluster used (no enhanced features); select enhanced for additional capabilities.
- **Accessing Private Nodes**: Requires a bastion host (covered in future lessons).
- **CIDR Blocks**: Ensure non-overlapping CIDRs for Kubernetes services and pods.
- **OCI Documentation**: Essential for detailed security list rules and advanced configurations.

## Next Steps
- **Bastion Host Setup**: Future demos will cover creating and using a bastion host to access private resources.
- **Cluster Access**: Upcoming lessons will explore local access and advanced cluster management.
- **Enhanced Features**: Separate lessons will cover advanced options (e.g., virtual nodes, add-ons).

This demo successfully configured security list rules, created a Kubernetes cluster with the Flannel CNI plugin, attached managed nodes, and deployed an Nginx application accessible via a load balancer. The cluster is now active and ready for use.