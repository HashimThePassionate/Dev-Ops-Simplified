# **Accessing a Kubernetes Cluster Using kubectl**

This guide, presented by Mahendra Mehra, Senior OCI Instructor at Oracle University, explains how to manage and access a Kubernetes cluster created with Oracle Container Engine for Kubernetes (OKE) using the `kubectl` command-line tool. It focuses on configuring the `kubeconfig` file, handling authentication, and addressing multi-factor authentication (MFA) requirements.

<details>
<summary><strong>📋 Table of Contents</strong></summary>

- [**Accessing a Kubernetes Cluster Using kubectl**](#accessing-a-kubernetes-cluster-using-kubectl)
  - [Overview of kubectl and kubeconfig](#overview-of-kubectl-and-kubeconfig)
    - [Key Features of kubeconfig](#key-features-of-kubeconfig)
  - [Configuring kubeconfig for Cluster Access](#configuring-kubeconfig-for-cluster-access)
    - [1. Setting Up kubeconfig](#1-setting-up-kubeconfig)
    - [2. Handling Multiple CLI Profiles](#2-handling-multiple-cli-profiles)
    - [3. Multi-Factor Authentication (MFA) Considerations](#3-multi-factor-authentication-mfa-considerations)
    - [4. Using Service Accounts for Non-User Access](#4-using-service-accounts-for-non-user-access)
  - [Key Notes](#key-notes)
  - [Next Steps](#next-steps)

</details>

## Overview of kubectl and kubeconfig

- **kubectl**: A command-line tool for managing Kubernetes clusters, available pre-installed in OCI Cloud Shell or installable locally.
- **kubeconfig File**: A configuration file that enables `kubectl` to communicate with a specific cluster. It is typically stored in the user's home directory at `~/.kube/config`.

### Key Features of kubeconfig
- **Multi-Cluster Support**: Organizes access to multiple clusters through different **contexts**. The **current-context** specifies the active cluster for `kubectl` operations.
- **OCI Command Integration**: Includes an OCI CLI command that dynamically generates short-lived, user-specific authentication tokens for cluster access.
- **User-Specific Tokens**: Tokens are non-shareable, requiring each user to have their own `kubeconfig` file.

## Configuring kubeconfig for Cluster Access

### 1. Setting Up kubeconfig
- The `kubeconfig` file contains details about the cluster, user, and authentication method.
- To generate a `kubeconfig` file for an OKE cluster:
  1. In the OCI Console, navigate to **Developer Services** > **Kubernetes Cluster (OKE)**.
  2. Select the cluster and click **Access Cluster**.
  3. Choose **Cloud Shell Access** or **Local Access** and copy the provided command (e.g., `oci ce cluster create-kubeconfig --cluster-id <cluster-id> --file $HOME/.kube/config --region <region>`).
  4. Run the command in Cloud Shell or a local terminal to create/update the `kubeconfig` file.

### 2. Handling Multiple CLI Profiles
- If using multiple OCI CLI profiles (e.g., for different tenancies), specify the profile for authentication token generation.
- **Steps**:
  1. Identify the profile name in your OCI CLI configuration file (`~/.oci/config`).
  2. Set the `OCI_CLI_PROFILE` environment variable before running `kubectl` commands:
     ```bash
     export OCI_CLI_PROFILE=<profile-name>
     ```
     - Replace `<profile-name>` with the actual profile name (e.g., `DEFAULT`, `TENANCY1`).
  3. Verify cluster connectivity:
     ```bash
     kubectl get nodes
     ```
     - Returns a list of nodes in the cluster, confirming successful access.

### 3. Multi-Factor Authentication (MFA) Considerations
- If an IAM policy restricts cluster access to MFA-verified users, additional configuration is required.
- **Steps**:
  1. Update the `kubeconfig` file to include MFA parameters in the OCI CLI command:
     - Add `--profile <profile-name> --auth security_token` to the `args` section of the `user` configuration.
     - Example `kubeconfig` snippet:
       ```yaml
       users:
       - name: user-oke
         user:
           exec:
             apiVersion: client.authentication.k8s.io/v1beta1
             command: oci
             args:
             - ce
             - cluster
             - create-kubeconfig
             - --cluster-id <cluster-id>
             - --region <region>
             - --profile <profile-name>
             - --auth security_token
       ```
     - Replace `<profile-name>` with the MFA-verified user’s profile name from `~/.oci/config`.
  2. Set the `OCI_CLI_PROFILE` environment variable:
     ```bash
     export OCI_CLI_PROFILE=<profile-name>
     ```
  3. Ensure the user is MFA-verified (e.g., via OCI Console or CLI).
  4. Verify cluster access:
     ```bash
     kubectl get nodes
     ```
     - If unauthorized, confirm MFA verification and correct profile settings.

### 4. Using Service Accounts for Non-User Access
- For non-user processes (e.g., CI/CD tools) accessing the cluster:
  - Create a **Kubernetes Service Account**.
  - Generate a service account token and add it to the `kubeconfig` file.
  - Example `kubeconfig` snippet for a service account:
    ```yaml
    users:
    - name: ci-cd-user
      user:
        token: <service-account-token>
    ```
  - Refer to Kubernetes documentation for service account creation and token generation.

## Key Notes
- **Non-Shareable kubeconfig**: Each user requires a unique `kubeconfig` file due to user-specific, short-lived tokens.
- **Cloud Shell vs. Local Access**:
  - **Cloud Shell**: Pre-installed with `kubectl` and OCI CLI, ideal for quick access.
  - **Local Access**: Requires local installation of `kubectl` and OCI CLI; covered in future lessons.
- **MFA Restrictions**: Ensure the user is MFA-verified if required by IAM policies to avoid "unauthorized" errors.
- **OCI CLI Profile**: Critical for multi-tenancy environments; always specify the correct profile for token generation.
- **Official Documentation**: Refer to [OCI official documentation](https://docs.oracle.com) for detailed `kubeconfig` setup and troubleshooting.

## Next Steps
- **Upcoming Lessons**: Explore practical methods for cluster access using:
  - **Cloud Shell Access**: Configuring and using `kubectl` in OCI Cloud Shell.
  - **Local Shell Access**: Setting up `kubectl` on a local machine.
- **Advanced Configurations**: Future lessons will cover service accounts, advanced `kubeconfig` setups, and cluster management.

This guide has laid the foundation for accessing an OKE cluster using `kubectl` by configuring the `kubeconfig` file, handling multiple CLI profiles, and addressing MFA requirements. Stay tuned for hands-on demos in the next lessons.