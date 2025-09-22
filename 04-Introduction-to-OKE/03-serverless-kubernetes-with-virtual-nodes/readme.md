#  **Managed Nodes vs. Virtual Nodes**

<details>
<summary><strong>📋 Table of Contents</strong></summary>

- [**Managed Nodes vs. Virtual Nodes**](#managed-nodes-vs-virtual-nodes)
  - [Overview](#overview)
  - [Scenario: Simplifying Node Management for the Pharmacy Application](#scenario-simplifying-node-management-for-the-pharmacy-application)
  - [Understanding Managed Nodes](#understanding-managed-nodes)
    - [What Are Managed Nodes?](#what-are-managed-nodes)
  - [Introduction to Virtual Nodes](#introduction-to-virtual-nodes)
    - [What Are Virtual Nodes?](#what-are-virtual-nodes)
    - [Key Benefits of Virtual Nodes](#key-benefits-of-virtual-nodes)

</details>

## Overview

This README explores a scenario involving a pharmacy application running on an Oracle Cloud Infrastructure (OCI) Container Engine for Kubernetes (OKE) cluster. It covers a discussion between Mo, a solution architect, and Tricia, a team lead, about the challenges of managing nodes and the benefits of using **virtual nodes** to simplify node management. The document provides a detailed comparison of **managed nodes** and **virtual nodes**, highlighting their features, benefits, and use cases.

## Scenario: Simplifying Node Management for the Pharmacy Application

Mo, the solution architect, discusses the pharmacy application running on an OCI OKE cluster with Tricia, the team lead. Tricia reports that the application is running smoothly but notes that managing nodes in the cluster requires manual work and specific expertise, which their team may lack. Mo suggests using **virtual nodes**, a feature of OKE that Tricia is unfamiliar with. Virtual nodes eliminate manual node management, reduce the need for specialized expertise, and make node management more efficient and cost-effective. Intrigued, Tricia asks Mo to provide more details about virtual nodes.

## Understanding Managed Nodes

Before diving into virtual nodes, it’s essential to understand **managed nodes**, which are the traditional approach to node management in OKE.

### What Are Managed Nodes?
- **Definition**: Managed nodes are compute instances (virtual machines or bare metal hosts) within your OCI tenancy, partially managed by you.
- **Responsibilities**:
  - You configure managed nodes to meet specific workload requirements.
  - You are responsible for upgrading Kubernetes on managed nodes and managing cluster capacity.
- **Components**:
  - **Kubelet**: Acts as the "host brain," managing the lifecycle of pods.
  - **Container Runtime**: Examples include CRI-O or containerd, responsible for running containers.
- **Workload Optimization**:
  - Managed nodes support specific workload requirements, such as GPUs, local NVMe drives, RDMA use cases, ARM or x86 architectures, and other scheduling needs.
  - By targeting workloads to specific nodes, you can optimize performance and resource utilization for your use case.

## Introduction to Virtual Nodes

**Virtual nodes** are a fully managed, highly available alternative to managed nodes, designed to simplify node management and provide a serverless Kubernetes experience.

### What Are Virtual Nodes?
- **Definition**: Virtual nodes are fully managed nodes that behave like real nodes in Kubernetes, built using the open-source **CNCF Virtual Kubernetes project**, which provides a translation layer between OCI and Kubernetes.
- **Unique Offering**: OCI is the first major cloud provider to offer a fully managed virtual Kubernetes product, delivering a serverless Kubernetes experience.
- **Configuration**: Virtual nodes are configured by customers and reside within a single availability and fault domain in OCI.
- **Components**:
  - **Pod Management**: Handled by the Virtual Kubernetes project, delegating lifecycle management of pods.
  - **Container Instance Management**: Leverages OCI’s serverless container instances for compute capacity.
- **Scalability**: Supports up to **1,000 pods per virtual node**, with plans to support more in the future.

### Key Benefits of Virtual Nodes
- **Fully Managed Experience**: Customers do