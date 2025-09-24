# **Demo: Setting Up Local Access to an OKE Cluster**

<details>
<summary>📋 Table of Contents</summary>

- [**Demo: Setting Up Local Access to an OKE Cluster**](#demo-setting-up-local-access-to-an-oke-cluster)
  - [Prerequisites](#prerequisites)
  - [Step-by-Step Process](#step-by-step-process)
    - [1. Overview of Steps](#1-overview-of-steps)
    - [2. Install and Configure OCI CLI](#2-install-and-configure-oci-cli)
    - [3. Install kubectl](#3-install-kubectl)
    - [4. Configure kubeconfig File](#4-configure-kubeconfig-file)
    - [5. Verify Cluster Access](#5-verify-cluster-access)
  - [Key Notes](#key-notes)
  - [Next Steps](#next-steps)

</details>

<br/>

This guide demonstrates how to configure **local access** to an Oracle Container Engine for Kubernetes (OKE) cluster, specifically the `OKE-PUBLIC-DEMO-1` cluster, which has a **public Kubernetes API endpoint** and worker nodes in a **private subnet**. The process involves setting up the Oracle Cloud Infrastructure (OCI) CLI, installing `kubectl`, generating and uploading an API signing key pair, configuring a `kubeconfig` file, and verifying cluster access from a local terminal running Oracle Linux 8.

## Prerequisites
- An OKE cluster (`OKE-PUBLIC-DEMO-1`) with a public Kubernetes API endpoint.
- A local terminal (Oracle Linux 8 in this demo; adjust for other operating systems).
- Permissions to manage the cluster and user settings in the OCI Console.
- Access to the OCI Console.

## Step-by-Step Process

### 1. Overview of Steps
To set up local access to the OKE cluster:
1. Generate an API signing key pair and upload the public key to the OCI Console.
2. Install and configure the OCI CLI.
3. Install `kubectl`.
4. Configure the `kubeconfig` file.
5. Verify cluster access with `kubectl`.

### 2. Install and Configure OCI CLI
1. **Check if OCI CLI is Installed**:
   - Run:
     ```bash
     oci -v
     ```
   - If not installed, proceed with installation.

2. **Install OCI CLI (Oracle Linux 8)**:
   - Ensure prerequisites:
     ```bash
     sudo dnf -y install oraclelinux-developer-release-el8
     ```
     - If already up-to-date, proceed.
   - Install OCI CLI:
     ```bash
     sudo dnf install python36-oci-cli
     ```
     - Confirm installation:
       ```bash
       oci -v
       ```
       - Expected output: `3.23.2` (or later).

3. **Configure OCI CLI**:
   - Run:
     ```bash
     oci setup config
     ```
   - Provide the following:
     - **Config File Location**: Accept default (`~/.oci/config`).
     - **User OCID**:
       - In the OCI Console, go to **Profile** > **My Profile** > Copy **User OCID**.
       - Paste into the terminal.
     - **Tenancy OCID**:
       - In the OCI Console, go to **Profile** > **Tenancy** > Copy **Tenancy OCID**.
       - Paste into the terminal.
     - **Region**: Select the region index (e.g., `50` for `us-phoenix-1`).
     - **Generate API Signing Key Pair**: Enter `Y` to generate a new key pair.
     - **Key Directory**: Accept default (`~/.oci/`).
     - **Key Name**: Accept default (`oci_api_key`).
     - **Passphrase**: Leave blank (press Enter).
   - **Outcome**: Creates `oci_api_key.pem` (private key) and `oci_api_key_public.pem` (public key) in `~/.oci/`.

4. **Upload Public Key to OCI Console**:
   - View the public key:
     ```bash
     cat ~/.oci/oci_api_key_public.pem
     ```
     - Copy the key content.
   - In the OCI Console:
     - Go to **Profile** > **My Profile** > **API Keys** > **Add API Key**.
     - Select **Paste Public Key** and paste the key.
     - Click **Add** and review the configuration file preview.
     - Close the dialog.
   - Verify the configuration file (`~/.oci/config`):
     ```bash
     cat ~/.oci/config
     ```
     - Confirms user OCID, tenancy OCID, region, and API key fingerprint.

5. **Test OCI CLI**:
   - Run:
     ```bash
     oci os ns get
     ```
     - Expected output: Returns the namespace (e.g., `intoraclerohit`), confirming CLI configuration.

### 3. Install kubectl
1. **Check if kubectl is Installed**:
   - Run:
     ```bash
     kubectl
     ```
     - If not found, proceed with installation.

2. **Install kubectl (Oracle Linux)**:
   - Refer to the [Kubernetes documentation](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/).
   - Download the latest release:
     ```bash
     curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
     ```
   - Make the binary executable:
     ```bash
     chmod +x ./kubectl
     ```
   - Move to a directory in the PATH:
     ```bash
     sudo mv ./kubectl /usr/local/bin/kubectl
     ```
   - Verify installation:
     ```bash
     kubectl version --client
     ```
     - Expected output: Displays version (e.g., `1.28.4`). Ensure it aligns with the cluster’s Kubernetes version (e.g., `1.27.2`).

### 4. Configure kubeconfig File
1. **Navigate to the OKE Cluster**:
   - In the OCI Console, go to **Developer Services** > **Kubernetes Cluster (OKE)** > Select `OKE-PUBLIC-DEMO-1`.
   - Click **Access Cluster** > Select **Local Access**.

2. **Create kubeconfig Directory**:
   - In the local terminal:
     ```bash
     mkdir -p $HOME/.kube
     ```

3. **Generate kubeconfig File**:
   - Copy the provided command from the **Local Access** dialog (e.g., `oci ce cluster create-kubeconfig --cluster-id <cluster-ocid> --file $HOME/.kube/config --region us-phoenix-1`).
   - Paste and run in the terminal:
     ```bash
     oci ce cluster create-kubeconfig --cluster-id <cluster-ocid> --file $HOME/.kube/config --region us-phoenix-1
     ```
   - **Outcome**: Creates `~/.kube/config` with the cluster’s configuration.
   - Verify the file:
     ```bash
     cd ~/.kube
     ls
     cat config
     ```
     - Confirms the cluster OCID (e.g., ending in `DPQ`) matches the cluster in the OCI Console.

### 5. Verify Cluster Access
1. **Check Cluster Connectivity**:
   - Run:
     ```bash
     kubectl get nodes
     ```
   - **Expected Output**: Lists the worker nodes in the cluster, confirming successful access.

2. **Create a Deployment**:
   - Create a deployment with two Nginx pods:
     ```bash
     kubectl create deployment mahendra-nginx --image=nginx --replicas=2
     ```
     - Confirms deployment creation.
   - Check pod status:
     ```bash
     kubectl get pods
     ```
     - Expected output: Lists two pods in the `Running` state.

## Key Notes
- **Public API Endpoint**: This demo assumes the Kubernetes API endpoint is in a public subnet, allowing direct access from the local terminal.
- **Private API Endpoint**: Requires a bastion host (covered in later demos) and VCN peering for local access.
- **kubectl Version**: Ensure compatibility with the cluster’s Kubernetes version (e.g., `1.28.4` for a `1.27.2` cluster).
- **OCI CLI Version**: Must be `2.24.0` or later (this demo uses `3.23.2`).
- **API Signing Key**: Essential for OCI CLI authentication; the public key must be uploaded to the user’s profile.
- **Official Documentation**:
  - [OCI CLI Installation](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/sdkconfig.htm).
  - [kubectl Installation](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/).
  - [OKE Cluster Access](https://docs.oracle.com) under "Container Engine for Kubernetes."
- **MFA Considerations**: If IAM policies require MFA, configure `OCI_CLI_PROFILE` and `--auth security_token` in the `kubeconfig` file (see previous lessons).

## Next Steps
- **Bastion Host Access**: Future demos will cover accessing clusters with private API endpoints using the OCI Bastion service.
- **Advanced Cluster Management**: Explore deploying and managing additional Kubernetes resources.
- **Troubleshooting**: Refer to OCI documentation for resolving access issues.

This demo successfully configured local access to the `OKE-PUBLIC-DEMO-1` cluster, verified connectivity with `kubectl get nodes`, and deployed an Nginx application with two pods. Stay tuned for further demos on advanced access and management.