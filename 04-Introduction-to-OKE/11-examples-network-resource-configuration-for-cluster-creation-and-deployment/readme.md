# **Network Resource Configuration for Cluster Creation and Deployment**

<details>
<summary>Table of contents</summary>

- [**Network Resource Configuration for Cluster Creation and Deployment**](#network-resource-configuration-for-cluster-creation-and-deployment)
  - [Accessing Official Documentation](#accessing-official-documentation)
  - [Example 1: Flannel CNI Plugin with Private Kubernetes API Endpoint, Private Worker Nodes, and Public Load Balancers](#example-1-flannel-cni-plugin-with-private-kubernetes-api-endpoint-private-worker-nodes-and-public-load-balancers)
    - [Overview](#overview)
    - [Network Resources](#network-resources)
    - [Route Tables](#route-tables)
    - [Security Rules](#security-rules)
  - [Example 2: Flannel CNI Plugin with Private Kubernetes API Endpoint, Private Worker Nodes, and Public Load Balancers](#example-2-flannel-cni-plugin-with-private-kubernetes-api-endpoint-private-worker-nodes-and-public-load-balancers)
    - [Overview](#overview-1)
    - [Key Differences from Example 1](#key-differences-from-example-1)
    - [Network Resources](#network-resources-1)
    - [Route Tables and Security Rules](#route-tables-and-security-rules)
  - [Example 3: OCI VCN-Native Pod Networking CNI Plugin with Public Kubernetes API Endpoint, Private Worker Nodes, Private Pod Subnet, and Public Load Balancers](#example-3-oci-vcn-native-pod-networking-cni-plugin-with-public-kubernetes-api-endpoint-private-worker-nodes-private-pod-subnet-and-public-load-balancers)
    - [Overview](#overview-2)
    - [Key Feature](#key-feature)
    - [Network Resources](#network-resources-2)
    - [Route Tables](#route-tables-1)
    - [Security Rules](#security-rules-1)
  - [Example 4: OCI VCN-Native Pod Networking CNI Plugin with Private Kubernetes API Endpoint, Private Worker Nodes, Private Pod Subnet, and Public Load Balancers](#example-4-oci-vcn-native-pod-networking-cni-plugin-with-private-kubernetes-api-endpoint-private-worker-nodes-private-pod-subnet-and-public-load-balancers)
    - [Overview](#overview-3)
    - [Key Differences from Example 3](#key-differences-from-example-3)
    - [Network Resources](#network-resources-3)
    - [Route Tables and Security Rules](#route-tables-and-security-rules-1)
  - [Key Considerations](#key-considerations)
  - [Conclusion](#conclusion)

</details>

<br/>


This document explores network resource configuration for creating highly available Kubernetes clusters in a region with three availability domains using the **custom create workflow** in Oracle Container Engine for Kubernetes (OKE). It covers four example configurations, detailing the setup of subnets, route tables, and security rules for different networking scenarios using the Flannel and OCI VCN-Native Pod Networking CNI plugins. For comprehensive details, refer to the [OCI official documentation](https://docs.oracle.com) under the "Container Engine for Kubernetes" section, specifically "Preparing for Container Engine for Kubernetes" > "Example Network Resource Configurations."

## Accessing Official Documentation
To find detailed networking setups:
1. Visit [docs.oracle.com](https://docs.oracle.com).
2. Search for "Container Engine for Kubernetes" or navigate to the relevant section.
3. Expand "Preparing for Container Engine for Kubernetes" and select "Example Network Resource Configurations" to view detailed configurations for each example.

## Example 1: Flannel CNI Plugin with Private Kubernetes API Endpoint, Private Worker Nodes, and Public Load Balancers

### Overview
- **Kubernetes API Endpoint**: Hosted in a **public subnet** with a public IP address, accessible from the internet.
- **Worker Nodes**: Hosted in a **private subnet** within the VCN for enhanced security.
- **Load Balancers**: Hosted in a **public subnet**, accessible from the internet.
- **Bastion Host**: Hosted in a **private subnet** to enable SSH access to worker nodes or the Kubernetes API endpoint.

### Network Resources
- **VCN**: Required to host all resources.
- **Internet Gateway**: Enables internet access for public subnets.
- **NAT Gateway**: Facilitates outbound internet access for private subnets.
- **Service Gateway**: Enables communication with OCI services.
- **DHCP Option**: Set to type "Internet and VCN Resolver."
- **Subnets**:
  - **Public Subnet** for Kubernetes API endpoint.
  - **Private Subnet** for worker nodes.
  - **Public Subnet** for load balancers.
  - **Private Subnet** for bastion host.

### Route Tables
- **Public API Endpoint Subnet**:
  - Route Rule: Destination CIDR `0.0.0.0/0`, Target: Internet Gateway.
- **Private Worker Node Subnet**:
  - Route Rule 1: Destination CIDR `0.0.0.0/0`, Target: NAT Gateway (for internet traffic).
  - Route Rule 2: Destination OCI Services, Target: Service Gateway.
- **Public Load Balancer Subnet**:
  - Route Rule: Destination CIDR `0.0.0.0/0`, Target: Internet Gateway.
- **Private Bastion Subnet**: Configured as needed for SSH access.

### Security Rules
- **Ingress and Egress Rules**: Must be configured for each subnet (Kubernetes API endpoint, worker nodes, load balancers, and bastion) to control communication. Refer to the official documentation for specific rules.

## Example 2: Flannel CNI Plugin with Private Kubernetes API Endpoint, Private Worker Nodes, and Public Load Balancers

### Overview
- **Kubernetes API Endpoint**: Hosted in a **private subnet** for added security, not exposed to the internet.
- **Worker Nodes**: Hosted in a **private subnet** within the VCN.
- **Load Balancers**: Hosted in a **public subnet**, accessible from the internet.
- **Bastion Host**: Hosted in a **private subnet** for SSH access to private resources.

### Key Differences from Example 1
- The Kubernetes API endpoint is moved to a private subnet to enhance security and control, ensuring it is not directly accessible from the internet.
- Access to the API endpoint and worker nodes is facilitated via the bastion host.

### Network Resources
- Similar to Example 1, including VCN, Internet Gateway, NAT Gateway, Service Gateway, and DHCP Option.
- **Subnets**:
  - **Private Subnet** for Kubernetes API endpoint.
  - **Private Subnet** for worker nodes.
  - **Public Subnet** for load balancers.
  - **Private Subnet** for bastion host.

### Route Tables and Security Rules
- Configured similarly to Example 1, with adjustments for the private Kubernetes API endpoint subnet. Refer to the official documentation for detailed route rules and security lists (ingress and egress rules).

## Example 3: OCI VCN-Native Pod Networking CNI Plugin with Public Kubernetes API Endpoint, Private Worker Nodes, Private Pod Subnet, and Public Load Balancers

### Overview
- **Kubernetes API Endpoint**: Hosted in a **public subnet**, accessible from the internet.
- **Worker Nodes**: Hosted in a **private subnet** within the VCN.
- **Pod Subnet**: Hosted in a **private subnet**, enabling pod-to-pod communication and direct access via private pod IP addresses.
- **Load Balancers**: Hosted in a **public subnet**, accessible from the internet.
- **Bastion Host**: Hosted in a **private subnet** for SSH access.

### Key Feature
- The **pod subnet** supports pod communication and direct access through private pod IP addresses, leveraging the OCI VCN-Native Pod Networking CNI plugin.

### Network Resources
- **VCN**, **Internet Gateway**, **NAT Gateway**, **Service Gateway**, and **DHCP Option** (Internet and VCN Resolver).
- **Subnets**:
  - **Public Subnet** for Kubernetes API endpoint.
  - **Private Subnet** for worker nodes.
  - **Private Subnet** for pods.
  - **Public Subnet** for load balancers.
  - **Private Subnet** for bastion host.

### Route Tables
- **Public API Endpoint Subnet**:
  - Route Rule: Destination CIDR `0.0.0.0/0`, Target: Internet Gateway.
- **Private Worker Node Subnet**:
  - Route Rule 1: Destination CIDR `0.0.0.0/0`, Target: NAT Gateway.
  - Route Rule 2: Destination OCI Services, Target: Service Gateway.
- **Private Pod Subnet**:
  - Configured similarly to worker node subnet for internet and OCI service access.
- **Public Load Balancer Subnet**:
  - Route Rule: Destination CIDR `0.0.0.0/0`, Target: Internet Gateway.
- **Private Bastion Subnet**: Configured as needed.

### Security Rules
- Ingress and egress rules must be configured for all subnets, including the pod subnet. Refer to the official documentation for specific rules.

## Example 4: OCI VCN-Native Pod Networking CNI Plugin with Private Kubernetes API Endpoint, Private Worker Nodes, Private Pod Subnet, and Public Load Balancers

### Overview
- **Kubernetes API Endpoint**: Hosted in a **private subnet**, not exposed to the internet.
- **Worker Nodes**: Hosted in a **private subnet** within the VCN.
- **Pod Subnet**: Hosted in a **private subnet** for pod communication.
- **Load Balancers**: Hosted in a **public subnet**, accessible from the internet.
- **Bastion Host**: Hosted in a **private subnet** for SSH access to private resources.

### Key Differences from Example 3
- The Kubernetes API endpoint is in a private subnet, enhancing security by limiting direct internet exposure.
- Access to private resources (API endpoint, worker nodes, and pod subnet) is facilitated via the bastion host.

### Network Resources
- Similar to Example 3, including VCN, Internet Gateway, NAT Gateway, Service Gateway, and DHCP Option.
- **Subnets**:
  - **Private Subnet** for Kubernetes API endpoint.
  - **Private Subnet** for worker nodes.
  - **Private Subnet** for pods.
  - **Public Subnet** for load balancers.
  - **Private Subnet** for bastion host.

### Route Tables and Security Rules
- Configured similarly to Example 3, with adjustments for the private Kubernetes API endpoint subnet. Detailed route rules and security lists (ingress and egress) are provided in the official documentation.

## Key Considerations
- **Ingress and Egress Rules**: The connectors in diagrams represent the ingress and egress rules that must be configured for each subnet to control communication. These are critical for secure and functional networking.
- **Bastion Host**: Used in all examples to provide secure SSH access to private resources.
- **Custom Create Workflow**: These examples are tailored for the custom create workflow, allowing fine-grained control over network configurations.
- **Official Documentation**: Provides detailed configurations for VCN, subnets, route tables, and security lists for each example. Always refer to it for precise setup instructions.

## Conclusion
The four examples demonstrate different approaches to configuring network resources for highly available Kubernetes clusters using the Flannel and OCI VCN-Native Pod Networking CNI plugins. These configurations balance accessibility, security, and scalability:
- **Examples 1 and 2** use the Flannel CNI plugin, with varying levels of exposure for the Kubernetes API endpoint.
- **Examples 3 and 4** leverage the OCI VCN-Native Pod Networking CNI plugin, introducing a pod subnet for enhanced pod communication.

These examples serve as a starting point. For detailed insights and comprehensive setup instructions, refer to the [OCI official documentation](https://docs.oracle.com) under "Container Engine for Kubernetes" > "Preparing for Container Engine for Kubernetes" > "Example Network Resource Configurations."