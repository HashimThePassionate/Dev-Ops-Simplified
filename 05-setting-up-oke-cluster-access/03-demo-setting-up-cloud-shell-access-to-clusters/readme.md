# **Demo: Setting Up Cloud Shell Access to OKE Clusters**

This guide demonstrates how to configure **Cloud Shell access** to an Oracle Container Engine for Kubernetes (OKE) cluster, specifically the `OKE-PUBLIC-DEMO-1` cluster created in a previous demo. This cluster has a **public Kubernetes API endpoint** and worker nodes hosted in a public regional subnet, making it accessible via Cloud Shell. The process involves setting up a `kubeconfig` file and verifying cluster access using `kubectl`.

<details>
<summary><strong>📋 Table of Contents</strong></summary>

- [**Demo: Setting Up Cloud Shell Access to OKE Clusters**](#demo-setting-up-cloud-shell-access-to-oke-clusters)
  - [Prerequisites](#prerequisites)
  - [Step-by-Step Process](#step-by-step-process)
    - [1. Accessing the Cluster](#1-accessing-the-cluster)
    - [2. Verifying Cluster Access](#2-verifying-cluster-access)
    - [3. Performing Operations on the Cluster](#3-performing-operations-on-the-cluster)
  - [Key Notes](#key-notes)
  - [Next Steps](#next-steps)

</details>

## Prerequisites
- An OKE cluster (`OKE-PUBLIC-DEMO-1`) with a public Kubernetes API endpoint.
- Access to the OCI Console and Cloud Shell.
- Permissions to manage the cluster in the target compartment (e.g., `CTDOKE`).

## Step-by-Step Process

### 1. Accessing the Cluster
1. **Navigate to the OKE Console**:
   - In the OCI Console, go to **Developer Services** > **Kubernetes Cluster (OKE)**.
   - Select the `OKE-PUBLIC-DEMO-1` cluster.
   - Click **Access Cluster** to open the **Access Your Cluster** dialog box.

2. **Choose Cloud Shell Access**:
   - The dialog provides two options: **Cloud Shell Access** and **Local Access**.
   - Select **Cloud Shell Access** for this demo.

3. **Launch Cloud Shell**:
   - Click the **Launch Cloud Shell** button to open the Cloud Shell terminal.

4. **Configure the kubeconfig File**:
   - Copy the provided command from the dialog box (e.g., `oci ce cluster create-kubeconfig --cluster-id <cluster-ocid> --file $HOME/.kube/config --region <region>`).
   - Paste the command into the Cloud Shell and press **Enter**.
   - **Outcome**:
     - A `kubeconfig` file is created or updated at `~/.kube/config`.
     - The cluster’s configuration is added as a new context, with `current-context` set to `OKE-PUBLIC-DEMO-1`.
     - The command includes the cluster’s OCID, ensuring it targets the correct cluster.

### 2. Verifying Cluster Access
1. **Check Cluster Connectivity**:
   - Run the following command in Cloud Shell:
     ```bash
     kubectl get nodes
     ```
   - **Expected Output**: Lists the three nodes in the `OKE-PUBLIC-DEMO-1` cluster, confirming successful access.
   - Note: Cloud Shell comes pre-installed with `kubectl`, a command-line utility for interacting with Kubernetes clusters.

2. **Cross-Check Node Information**:
   - In the OCI Console, navigate to the cluster’s **Node Pools** under **Resources**.
   - Select **Pool 1** and verify the private IP addresses of the worker nodes.
   - Compare these IPs with the output of `kubectl get nodes` to ensure they match.

### 3. Performing Operations on the Cluster
1. **Create a Deployment**:
   - Use `kubectl` to create a deployment with three Nginx pods:
     ```bash
     kubectl create deployment mahendra-deployment --image=nginx:latest --replicas=3
     ```
   - **Outcome**: Creates a deployment named `mahindra-deployment` running the latest Nginx image with three replicas.

2. **Check Pod Status**:
   - Run:
     ```bash
     kubectl get pods
     ```
   - **Expected Output**: Lists three pods in the `Running` state, confirming the deployment is active.

## Key Notes
- **Public API Endpoint**: Cloud Shell access requires the Kubernetes API endpoint to have a public IP address, as configured in `OKE-PUBLIC-DEMO-1`.
- **kubeconfig File**:
  - Stored at `~/.kube/config` by default.
  - Adds the cluster as a new context, with `current-context` set to the target cluster.
  - If a `kubeconfig` file already exists, the new cluster configuration is appended.
- **kubectl**: Pre-installed in Cloud Shell, simplifying setup compared to local access.
- **Cluster OCID**: Ensures the `kubeconfig` command targets the correct cluster; verify it matches the cluster’s OCID in the OCI Console.
- **Security**: For MFA requirements or multiple CLI profiles, configure `OCI_CLI_PROFILE` and `--auth security_token` as needed (see previous lessons).
- **Official Documentation**: Refer to [OCI official documentation](https://docs.oracle.com) for detailed `kubeconfig` setup and troubleshooting.

## Next Steps
- **Local Shell Access**: Future demos will cover accessing the cluster from a local terminal, including for private API endpoints.
- **Advanced Operations**: Explore creating additional Kubernetes resources, exposing services, and managing node pools.
- **Bastion Host Access**: Upcoming lessons will demonstrate accessing private worker nodes using a bastion host.

This demo successfully configured Cloud Shell access to the `OKE-PUBLIC-DEMO-1` cluster, verified connectivity with `kubectl get nodes`, and deployed an Nginx application. Stay tuned for the next demo on additional access methods and cluster management.