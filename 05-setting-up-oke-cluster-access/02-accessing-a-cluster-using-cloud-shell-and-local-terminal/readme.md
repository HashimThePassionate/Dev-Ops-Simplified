# **Accessing Kubernetes Clusters Using kubectl: Cloud Shell and Local Shell**

This guide, presented by Mahendra Mehra, Senior OCI Instructor at Oracle University, explores two methods for accessing Oracle Container Engine for Kubernetes (OKE) clusters using `kubectl`: **Cloud Shell Access** and **Local Shell Access**. It provides step-by-step instructions for configuring access and verifying connectivity.

<details>
<summary><strong>📋 Table of Contents</strong></summary>

- [**Accessing Kubernetes Clusters Using kubectl: Cloud Shell and Local Shell**](#accessing-kubernetes-clusters-using-kubectl-cloud-shell-and-local-shell)
  - [Prerequisites](#prerequisites)
  - [1. Cloud Shell Access](#1-cloud-shell-access)
    - [Steps](#steps)
    - [Key Notes](#key-notes)
  - [2. Local Shell Access](#2-local-shell-access)
    - [Steps](#steps-1)
    - [Key Notes](#key-notes-1)
  - [Conclusion](#conclusion)

</details>

## Prerequisites
- An OKE cluster created in Oracle Cloud Infrastructure (OCI).
- Permissions to access the cluster and manage resources in the target compartment.
- For Cloud Shell: Access to the OCI Console and Cloud Shell.
- For Local Shell: Local installation of `kubectl` and OCI CLI, and network connectivity to the cluster.

## 1. Cloud Shell Access

Cloud Shell provides a pre-configured environment with `kubectl` and OCI CLI, ideal for clusters with a **public Kubernetes API endpoint**.

### Steps
1. **Navigate to the OKE Console**:
   - In the OCI Console, go to **Developer Services** > **Kubernetes Cluster (OKE)**.
   - Select your cluster and click **Access Cluster**.
   - In the **Access Your Cluster** dialog, choose **Cloud Shell Access**.

2. **Configure the kubeconfig File**:
   - Copy the provided command (e.g., `oci ce cluster create-kubeconfig --cluster-id <cluster-id> --file $HOME/.kube/config --region <region>`).
   - Run the command in Cloud Shell to generate or update the `kubeconfig` file.
   - **Behavior**:
     - If a `kubeconfig` file exists at the specified location (default: `~/.kube/config`), the cluster is added as a new context.
     - The `current-context` is set to the new cluster.

3. **Custom kubeconfig Location** (Optional):
   - To use a non-default location or filename for the `kubeconfig` file:
     - Specify the path in the `oci ce cluster create-kubeconfig` command (e.g., `--file /custom/path/config`).
     - Set the `KUBECONFIG` environment variable:
       ```bash
       export KUBECONFIG=/custom/path/config
       ```

4. **Verify Cluster Access**:
   - Run:
     ```bash
     kubectl get nodes
     ```
   - Expected Output: A list of nodes in the cluster, confirming successful access.

### Key Notes
- **Public IP Requirement**: Cloud Shell access requires the Kubernetes API endpoint to have a public IP address.
- **Context Management**: Multiple clusters are managed as contexts in the `kubeconfig` file, with `current-context` determining the active cluster.
- **OCI CLI Profile**: If using multiple profiles, set `OCI_CLI_PROFILE` to the appropriate profile name before running `kubectl` commands (see previous lessons).

## 2. Local Shell Access

Local Shell access is suitable for clusters with **private or public Kubernetes API endpoints**, provided the local network is peered with the cluster’s VCN for private endpoints.

### Steps
1. **Generate an API Signing Key Pair**:
   - Create an API signing key pair (public/private key) for OCI CLI authentication.
   - Upload the public key to the OCI Console:
     - Navigate to **Identity & Security** > **Users** > Select your user > **API Keys** > **Add API Key**.
     - Upload the `.pub` file or paste its contents.
   - Save the private key securely (e.g., `~/.oci/oci_api_key.pem`).

2. **Install and Configure OCI CLI**:
   - Install the OCI CLI on your local machine (follow [OCI CLI installation guide](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/sdkconfig.htm)).
   - Configure the CLI:
     ```bash
     oci setup config
     ```
     - Provide the user OCID, tenancy OCID, region, and path to the API key pair.
     - Update the CLI configuration file (`~/.oci/config`) with the correct profile.

3. **Install kubectl**:
   - Install `kubectl` locally (see [Kubernetes documentation](https://kubernetes.io/docs/tasks/tools/)).
   - Ensure the `kubectl` version is compatible with the cluster’s Kubernetes version (e.g., within one minor version of the control plane, such as `1.27.x` for a `1.27.2` cluster).

4. **Set Up the kubeconfig File**:
   - Use the OCI CLI to generate the `kubeconfig` file:
     ```bash
     oci ce cluster create-kubeconfig --cluster-id <cluster-id> --file ~/.kube/config --region <region>
     ```
   - For a non-default location:
     - Specify the path in the command (e.g., `--file /custom/path/config`).
     - Set the `KUBECONFIG` environment variable:
       ```bash
       export KUBECONFIG=/custom/path/config
       ```

5. **Verify Cluster Access**:
   - Check the `kubectl` and cluster versions:
     ```bash
     kubectl version
     ```
     - Displays the local `kubectl` version and the cluster’s control plane version.
   - List cluster nodes:
     ```bash
     kubectl get nodes
     ```
     - Returns a list of nodes, confirming successful access.

### Key Notes
- **Private API Endpoint**: Requires VCN peering between your local network and the cluster’s VCN for access.
- **kubectl Version Compatibility**: Align the `kubectl` version with the cluster’s Kubernetes version to avoid compatibility issues.
- **OCI CLI Configuration**: Ensure the CLI profile includes the correct API key and region for token generation.
- **MFA Considerations**: If IAM policies require MFA, update the `kubeconfig` file with `--auth security_token` and set `OCI_CLI_PROFILE` to the MFA-verified profile (see previous lessons).

## Conclusion
This guide covered two methods for accessing OKE clusters using `kubectl`:
- **Cloud Shell Access**: Ideal for public API endpoints, leveraging pre-installed tools and simple `kubeconfig` configuration.
- **Local Shell Access**: Suitable for both public and private endpoints, requiring local setup of `kubectl`, OCI CLI, and API keys.

Both methods involve configuring a `kubeconfig` file and verifying access with `kubectl get nodes`. Future lessons will explore advanced access scenarios, such as service accounts and managing multiple clusters.

For additional details, refer to the [OCI official documentation](https://docs.oracle.com) under "Container Engine for Kubernetes."