# **Network Resource Configuration for Custom OKE Cluster Setup** 

<details>
<summary><strong>📋 Table of Contents</strong></summary>

- [**Network Resource Configuration for Custom OKE Cluster Setup**](#network-resource-configuration-for-custom-oke-cluster-setup)
  - [Overview](#overview)
  - [Key Network Resources](#key-network-resources)
    - [VCN Configuration](#vcn-configuration)
    - [Subnet Configuration](#subnet-configuration)
    - [Internet Gateway Configuration](#internet-gateway-configuration)
    - [NAT Gateway Configuration](#nat-gateway-configuration)
    - [Service Gateway Configuration](#service-gateway-configuration)
    - [Route Table Configuration](#route-table-configuration)
    - [DHCP Options Configuration](#dhcp-options-configuration)
    - [Security Rule Configuration](#security-rule-configuration)
  - [Accessing Official Documentation](#accessing-official-documentation)
  - [Summary](#summary)

</details>

## Overview

This README provides a comprehensive guide to configuring key network resources for a custom Oracle Cloud Infrastructure (OCI) Container Engine for Kubernetes (OKE) cluster setup. It covers the configuration of Virtual Cloud Network (VCN), subnets, internet gateways, NAT gateways, service gateways, route tables, DHCP options, and security rules (via Network Security Groups or Security Lists). These resources are critical for ensuring smooth deployment and operation of OKE clusters.

## Key Network Resources

### VCN Configuration

The **Virtual Cloud Network (VCN)** forms the foundation of your custom OKE cluster. Proper configuration is essential for accommodating all cluster components.

- **CIDR Block**:
  - Define a spacious CIDR block (e.g., `10.0.0.0/16`) to support:
    - Kubernetes API endpoint
    - Worker nodes
    - Pods
    - Load balancers
- **Subnets**:
  - Set up an adequate number of subnets for:
    - Worker nodes
    - Load balancers
    - Kubernetes API endpoints
    - Pods
  - **Best Practice**: Use **regional subnets** to simplify failover across availability domains.
- **Security Rules**:
  - Configure security rules in **Network Security Groups (NSGs)** or **Security Lists** for each subnet to protect the cluster and network.
- **DNS Resolution**:
  - Enable DNS resolution for the VCN to ensure efficient resolution of DNS names.
- **Gateways**:
  - Include an **Internet Gateway**, **NAT Gateway**, or both, depending on whether subnets are public or private.
- **Route Table**:
  - A well-structured route table is required to direct network traffic effectively when gateways are used.

### Subnet Configuration

Subnets are critical for organizing and isolating cluster components. The number and type of subnets depend on your cluster and deployment needs.

- **Minimum Subnets**:
  - **Kubernetes API Endpoint Subnet**: For the cluster’s control plane.
  - **Worker Node Subnet**: For managed or virtual nodes.
  - **Load Balancer Subnets** (optional, 1 or 2): Can be regional or availability domain-specific.
  - **Pod Subnet**: For Kubernetes pods.
  - **Bastion Subnet** (optional): For secure administrative access.
- **Best Practice**: Use **regional subnets** to simplify failover across availability domains.
- **CIDR Block Considerations**:
  - Ensure subnet CIDR blocks do not overlap with the CIDR blocks specified for pods within the cluster.
- **Node Type Variations**:
  - Configuration requirements for worker node and pod subnets may differ when using **virtual nodes** versus **managed nodes**.

### Internet Gateway Configuration

An **Internet Gateway** is essential for clusters with public subnets requiring internet access.

- **Purpose**:
  - Enables components like worker nodes, load balancers, or Kubernetes API endpoints in public subnets to interact with the internet.
- **Configuration**:
  - Specify the Internet Gateway as the target for the destination CIDR block `0.0.0.0/0` in the route table.
- **Use Case**: Required for public subnets to serve as the entry and exit point for internet-bound traffic.

### NAT Gateway Configuration

A **NAT Gateway** is critical for private subnets that require outbound internet access without direct exposure.

- **Purpose**:
  - Allows components in private subnets (e.g., worker nodes, Kubernetes API endpoints, or pods) to initiate outbound internet connections securely.
- **Configuration**:
  - Specify the NAT Gateway as the target for the destination CIDR block `0.0.0.0/0` in the route table.
- **Use Case**: Enhances security for private subnets by providing controlled outbound internet access.

### Service Gateway Configuration

A **Service Gateway** enables private subnets to access Oracle services without exposing data to the public internet.

- **Purpose**:
  - Required when using private subnets for worker nodes, Kubernetes API endpoints, or pods, especially with the **OCI VCN-Native Pod Networking CNI Plugin** (details covered in upcoming lessons).
- **Configuration**:
  - Create the Service Gateway in the same VCN and compartment as the subnets for worker nodes, Kubernetes API endpoints, or pods.
  - Select the **All <region> Services in Oracle Services Network** option.
  - Specify the Service Gateway as the target for **All <region> Services in Oracle Services Network** in a route table rule.
- **Use Case**: Ensures secure access to Oracle services within the region.

### Route Table Configuration

Route tables direct network traffic for various subnets in your OKE cluster.

- **Worker Node Subnets**:
  - **Public Subnets**: Configure a route rule targeting the **Internet Gateway** for destination CIDR block `0.0.0.0/0`.
  - **Private Subnets**:
    - Configure a route rule targeting the **Service Gateway** for **All <region> Services in Oracle Services Network**.
    - Configure a route rule targeting the **NAT Gateway** for destination CIDR block `0.0.0.0/0`.
- **Kubernetes API Endpoint Subnets**:
  - Same routing rules as worker node subnets for both public and private subnets.
- **Load Balancer Subnets**:
  - **Public Subnets**: Configure a route rule targeting the **Internet Gateway** for destination CIDR block `0.0.0.0/0`.
  - **Private Subnets**: Route table can be left empty.
- **Pod Subnets** (with OCI VCN-Native Pod Networking CNI Plugin):
  - Pods typically use private subnets.
  - Configure:
    - A route rule targeting the **Service Gateway** for **All <region> Services in Oracle Services Network**.
    - A route rule targeting the **NAT Gateway** for destination CIDR block `0.0.0.0/0`.
  - Public pod subnets do not require configuration.

### DHCP Options Configuration

Proper DHCP configuration ensures the VCN’s domain name system (DNS) functions correctly.

- **Default Settings**:
  - Use the default DNS type: **Internet and VCN Resolver**.
  - This setting is suitable for most OKE cluster deployments.
- **Purpose**: Ensures efficient DNS resolution within the VCN.

### Security Rule Configuration

Security rules protect the cluster and network, defined in **Network Security Groups (NSGs)** or **Security Lists**.

- **Configuration Options**:
  - **Network Security Groups (NSGs)** (Recommended):
    - Selected for node pools, Kubernetes API endpoints, pods (with VCN-Native Pod Networking), and load balancers during cluster creation.
  - **Security Lists**:
    - Associated with subnets for worker nodes, Kubernetes API endpoints, pods, and load balancers.
- **Union of Rules**:
  - If both NSGs and Security Lists are used, all rules from both are applied (forming a union).
- **Component-Specific Requirements**:
  - Worker nodes, Kubernetes API endpoints, pods, and load balancers have distinct security rule requirements.
  - Add extra rules as needed to meet specific use case requirements.
- **Documentation**:
  - Due to the extensive nature of security rules, refer to the official OCI documentation for detailed guidance.
  - Navigate to the **Preparing for Container Engine for Kubernetes** section, specifically the **Network Resource Configuration for Cluster Creation and Deployment** page.
  - Includes specific rules for:
    - Kubernetes API endpoints (ingress and egress rules)
    - Worker nodes
    - Pod subnets
    - Bastion subnets (based on use case)

## Accessing Official Documentation

- **Location**: Official OCI Documentation > **Container Engine for Kubernetes** > **Preparing for Container Engine for Kubernetes** > **Network Resource Configuration for Cluster Creation and Deployment**.
- **Content**: Provides a comprehensive breakdown of required security rules, including ingress and egress rules for all components.

## Summary

This README has detailed the key network resources required for configuring a custom OKE cluster:
- **VCN Configuration**: Define a spacious CIDR block, regional subnets, security rules, DNS resolution, and gateways.
- **Subnet Configuration**: Set up subnets for Kubernetes API endpoints, worker nodes, load balancers, pods, and optionally bastion hosts, ensuring non-overlapping CIDR blocks.
- **Internet Gateway**: Enable internet access for public subnets.
- **NAT Gateway**: Provide secure outbound internet access for private subnets.
- **Service Gateway**: Allow private subnets to access Oracle services securely.
- **Route Tables**: Configure routing rules for public and private subnets, targeting Internet, NAT, or Service Gateways as needed.
- **DHCP Options**: Use default Internet and VCN Resolver settings for DNS.
- **Security Rules**: Define in NSGs (recommended) or Security Lists, with component-specific requirements available in OCI documentation.

By properly configuring these network resources, you ensure a secure, efficient, and scalable custom OKE cluster deployment.