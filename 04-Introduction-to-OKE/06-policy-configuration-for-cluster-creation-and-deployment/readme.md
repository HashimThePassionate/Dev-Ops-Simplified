#  **Policy Configuration for Cluster Creation and Deployment**

<details>
<summary><strong>📖 Table of Contents</strong></summary>

- [**Policy Configuration for Cluster Creation and Deployment**](#policy-configuration-for-cluster-creation-and-deployment)
  - [Overview](#overview)
  - [Policy Configuration for OKE](#policy-configuration-for-oke)
    - [Administrator Group Permissions](#administrator-group-permissions)
    - [Policies for Non-Administrator Users](#policies-for-non-administrator-users)
      - [Policy Statement Structure](#policy-statement-structure)
  - [Required Policies for OKE Operations](#required-policies-for-oke-operations)
    - [Catch-All Policy for Cluster Resources](#catch-all-policy-for-cluster-resources)
    - [Scenario-Specific Policy Requirements](#scenario-specific-policy-requirements)
  - [Policies for Automatic Network Resource Configuration](#policies-for-automatic-network-resource-configuration)
  - [Optional Policies for Enhanced Control](#optional-policies-for-enhanced-control)
    - [Cloud Shell Policy](#cloud-shell-policy)
    - [Vault-Related Policy](#vault-related-policy)
    - [Cluster-Related Policies](#cluster-related-policies)
    - [Service Gateway Policy](#service-gateway-policy)
    - [Capacity Reservation Policy](#capacity-reservation-policy)
  - [Special Scenarios](#special-scenarios)
    - [Virtual Nodes and Virtual Node Pools](#virtual-nodes-and-virtual-node-pools)
    - [Data Encryption](#data-encryption)
    - [Kubernetes RBAC Authorization](#kubernetes-rbac-authorization)
  - [Summary](#summary)

</details>

## Overview

This README provides a comprehensive guide to configuring policies for Oracle Cloud Infrastructure (OCI) Container Engine for Kubernetes (OKE) cluster creation and deployment. It covers both required and optional policy statements necessary to enable user groups to perform various OKE-related actions, such as creating, updating, and managing clusters and node pools. The guide also addresses specific scenarios, including virtual nodes, encryption, and network resource management, ensuring effective access control and operational efficiency.

## Policy Configuration for OKE

Policies in OCI define permissions for user groups to interact with resources within a tenancy or specific compartments. This section outlines the policies required for OKE operations and optional policies for enhanced control.

### Administrator Group Permissions
- When a tenancy is created, an **Administrators group** is automatically established.
- Members of this group have broad permissions to manage all resources within the tenancy, including OKE clusters, without needing additional policies.

### Policies for Non-Administrator Users
For users who are not part of the Administrators group, specific policies must be created to grant permissions for OKE operations. These policies enable user groups to perform actions on resources within the tenancy or individual compartments.

#### Policy Statement Structure
- Replace placeholders in policy statements:
  - **`<group-name>`**: The name of the user group.
  - **`<location>`**: Either `tenancy` (for the root compartment) or a specific `compartment-name` (for an individual compartment).

## Required Policies for OKE Operations

To enable users to create, update, and delete OKE clusters and node pools, the following policy statements are required:

- **Manage Instance Family**:
  ```plaintext
  Allow group <group-name> to manage instance-family in <location>
  ```
- **Use Subnets**:
  ```plaintext
  Allow group <group-name> to use subnets in <location>
  ```
- **Read Virtual Network Family**:
  ```plaintext
  Allow group <group-name> to read virtual-network-family in <location>
  ```
- **Use Network Security Groups**:
  ```plaintext
  Allow group <group-name> to use network-security-groups in <location>
  ```
- **Use VNICs**:
  ```plaintext
  Allow group <group-name> to use vnics in <location>
  ```
- **Inspect Compartments**:
  ```plaintext
  Allow group <group-name> to inspect compartments in <location>
  ```
- **Use Private IPs**:
  ```plaintext
  Allow group <group-name> to use private-ips in <location>
  ```
- **Manage Public IPs**:
  ```plaintext
  Allow group <group-name> to manage public-ips in <location>
  ```

### Catch-All Policy for Cluster Resources
A single policy statement can grant comprehensive access to all cluster-related resources:
```plaintext
Allow group <group-name> to manage cluster-family in <location>
```
- **Effect**: Grants administrator-level privileges for all cluster-related resources in the specified location.

### Scenario-Specific Policy Requirements
- **VCN-Native Clusters** (Kubernetes API endpoint integrated within your VCN):
  - The `use private-ips` policy is always required.
  - The `manage public-ips` policy is required only if the cluster is configured with a public IP address.
- **Clusters with Public Kubernetes API Endpoint in Oracle-Managed Tenancy**:
  - Required policies: `use vnics`, `use private-ips`, and `manage public-ips`.

## Policies for Automatic Network Resource Configuration

When using the **Quick Create workflow** to automatically create and configure network resources during cluster creation, the following permissions are required:
- **Manage VCNs**:
  ```plaintext
  Allow group <group-name> to manage vcns in <location>
  ```
- **Manage Subnets**:
  ```plaintext
  Allow group <group-name> to manage subnets in <location>
  ```
- **Manage Internet Gateways**:
  ```plaintext
  Allow group <group-name> to manage internet-gateways in <location>
  ```
- **Manage NAT Gateways**:
  ```plaintext
  Allow group <group-name> to manage nat-gateways in <location>
  ```
- **Manage Route Tables**:
  ```plaintext
  Allow group <group-name> to manage route-tables in <location>
  ```
- **Manage Security Lists**:
  ```plaintext
  Allow group <group-name> to manage security-lists in <location>
  ```

## Optional Policies for Enhanced Control

In addition to required policies, optional policies can enhance control over OKE-related actions. These policies grant specific permissions for various scenarios.

### Cloud Shell Policy
- **Purpose**: Enables a user group to access clusters using Cloud Shell.
- **Policy Statement**:
  ```plaintext
  Allow group <group-name> to use cloud-shell in tenancy
  ```

### Vault-Related Policy
- **Purpose**: Allows users to select master encryption keys and vaults in the OCI Vault service when creating or modifying clusters via the Console.
- **Policy Statements**:
  ```plaintext
  Allow group <group-name> to read vaults in <location>
  Allow group <group-name> to read keys in <location>
  ```

### Cluster-Related Policies
- **Inspect Clusters**:
  ```plaintext
  Allow group <group-name> to inspect clusters in <location>
  ```
- **Use Cluster Node Pools**:
  ```plaintext
  Allow group <group-name> to use cluster-node-pools in <location>
  ```
- **Read Cluster Work Requests**:
  ```plaintext
  Allow group <group-name> to read cluster-work-requests in <location>
  ```

### Service Gateway Policy
- **Purpose**: Allows worker nodes to access other resources in the same region without exposing data to the public internet.
- **Policy Statement**:
  ```plaintext
  Allow group <group-name> to manage service-gateways in <location>
  ```

### Capacity Reservation Policy
- **Purpose**: Enables users to reserve compute capacity for OKE clusters.
- **Policy Statement**:
  ```plaintext
  Allow group <group-name> to use compute-capacity-reservation in <location>
  ```

## Special Scenarios

### Virtual Nodes and Virtual Node Pools
- Certain policies are essential for creating and utilizing clusters with virtual nodes and virtual node pools. These may include additional permissions for managing virtual node-specific resources (details covered in upcoming lessons).

### Data Encryption
- For encrypting data in boot volumes and block volumes using your own master encryption keys from the OCI Vault service, ensure the `read vaults` and `read keys` policies are in place.

### Kubernetes RBAC Authorization
- The **Kubernetes RBAC Authorizer** can be used for fine-grained access control within a cluster.
- Define Kubernetes RBAC roles and cluster roles to enforce specific permissions for users on particular clusters.

## Summary

This README has outlined the policy configuration for Oracle Cloud Infrastructure Container Engine for Kubernetes (OKE):
- **Administrator Group**: Automatically grants broad permissions for OKE operations.
- **Required Policies**: Essential for non-administrator users to manage clusters, node pools, and network resources.
- **Optional Policies**: Enhance control for specific scenarios, such as Cloud Shell access, vault integration, service gateways, and capacity reservations.
- **Special Scenarios**: Address virtual nodes, data encryption, and Kubernetes RBAC for fine-grained access control.
- **Location and Group Names**: Replace placeholders with the appropriate group name and location (tenancy or compartment) in policy statements.

By configuring these policies, you ensure that user groups have the necessary permissions to effectively create, manage, and deploy OKE clusters while maintaining security and operational efficiency.