# **Basic and Enhanced Clusters**

<details>
<summary><strong>📋 Table of Contents</strong></summary>

- [**Basic and Enhanced Clusters**](#basic-and-enhanced-clusters)
  - [Overview](#overview)
  - [Cluster Types](#cluster-types)
    - [Basic Clusters](#basic-clusters)
    - [Enhanced Clusters](#enhanced-clusters)
      - [Virtual Node Support](#virtual-node-support)
      - [Granular Add-On Management](#granular-add-on-management)
      - [Increased Scalability](#increased-scalability)
      - [Enhanced Security with Workload Identity](#enhanced-security-with-workload-identity)
      - [Financially Backed SLA](#financially-backed-sla)
  - [Features Not Supported by Basic Clusters](#features-not-supported-by-basic-clusters)
  - [Key Considerations for Cluster Creation](#key-considerations-for-cluster-creation)
    - [Default Cluster Type](#default-cluster-type)
    - [Upgrading and Downgrading](#upgrading-and-downgrading)
  - [Summary](#summary)

</details>

## Overview

The Oracle Cloud Infrastructure Container Engine for Kubernetes (OKE) offers two cluster types: **Basic Clusters** and **Enhanced Clusters**. This README provides a comprehensive overview of both cluster types, their features, differences, and key considerations for creating and managing them. As OKE continues to evolve, understanding these cluster types helps you choose the right option for your cloud-native application needs.

## Cluster Types

When creating a new cluster with OKE, you must specify the cluster type as either **Basic** or **Enhanced**. Below is a detailed comparison of the two.

### Basic Clusters

Basic Clusters provide the core functionality of Kubernetes and OKE, making them suitable for standard workloads. Key characteristics include:

- **Core Functionality**: Supports all essential Kubernetes and OKE features.
- **Service Level Objective (SLO)**: Oracle guarantees a certain level of availability, but there is no financially backed Service Level Agreement (SLA). This means no monetary compensation is provided if the availability target is not met.
- **Limitations**: Does not support advanced features available in Enhanced Clusters (detailed below).

### Enhanced Clusters

Enhanced Clusters include all features of Basic Clusters plus additional capabilities designed for more complex and demanding workloads. Key features include:

#### Virtual Node Support
- **Simplified Infrastructure Management**: Virtual nodes eliminate the need to manually scale, upgrade, or troubleshoot worker nodes, allowing you to focus on your applications.
- **Scalability**: Ideal for large clusters with many nodes requiring frequent updates or scaling, reducing operational overhead.

#### Granular Add-On Management
- **Flexible Configuration**: Deploy and configure both essential add-ons (e.g., CoreDNS, kube-proxy) and optional add-ons (e.g., Kubernetes Dashboard).
- **Customization Options**:
  - Enable or disable specific add-ons.
  - Select specific add-on versions.
  - Opt-in or opt-out of Oracle-managed automatic updates.
  - Apply add-on-specific customizations to tailor the cluster to your application needs.
- **Lifecycle Management**: Oracle manages the lifecycle of add-ons, simplifying deployment and maintenance.

#### Increased Scalability
- **More Worker Nodes**: Provision more worker nodes in a single cluster to support larger workloads, improving resource utilization and reducing operational overhead.
- **Efficiency**: Fewer, larger environments are easier to secure, monitor, upgrade, and manage, potentially lowering costs.
- **Limits**: Review the OKE limits documentation for maximum worker node counts and additional considerations for large managed node clusters.

#### Enhanced Security with Workload Identity
- **OCI IAM Policies**: Define policies that authorize specific Kubernetes pods to make OCI API calls and access OCI resources.
- **Service Account Integration**: Scope policies to Kubernetes service accounts associated with application pods, enabling secure, direct API access based on defined permissions.
- **OCI Audit**: Automatically tracks all API calls made by Kubernetes workloads, adding an extra layer of security and accountability.

#### Financially Backed SLA
- **Kubernetes API Server Uptime**: Enhanced Clusters come with a financially backed SLA for Kubernetes API server uptime and availability. If performance falls below the stated SLA, compensation is provided, ensuring high availability and performance.

## Features Not Supported by Basic Clusters

Basic Clusters do not support the following features available in Enhanced Clusters:
- Virtual node pools.
- Fine-grained cluster add-on deployment and configuration.
- Workload identity for enhanced security.
- Increased numbers of worker nodes.
- Financially backed SLA for Kubernetes API server uptime.

## Key Considerations for Cluster Creation

When creating a cluster, keep the following points in mind:

### Default Cluster Type
- **Console**:
  - By default, new clusters are created as **Enhanced Clusters** unless you explicitly select **Basic Cluster**.
  - If no enhanced features are selected during creation, you can opt to create a Basic Cluster.
- **CLI or API**:
  - By default, new clusters are created as **Basic Clusters** unless you explicitly specify an Enhanced Cluster.

### Upgrading and Downgrading
- **Enhanced Clusters**: You can create an Enhanced Cluster and add enhanced features later, even if none were initially selected.
- **Basic to Enhanced**: You can upgrade a Basic Cluster to an Enhanced Cluster at any time.
- **Enhanced to Basic**: Downgrading from an Enhanced Cluster to a Basic Cluster is not supported.

## Summary

This README has outlined the differences between **Basic Clusters** and **Enhanced Clusters** in Oracle Cloud Infrastructure Container Engine for Kubernetes (OKE):
- **Basic Clusters**: Provide core Kubernetes and OKE functionality with a service level objective but no financially backed SLA.
- **Enhanced Clusters**: Offer all Basic Cluster features plus advanced capabilities like virtual nodes, granular add-on management, workload identity, increased scalability, and a financially backed SLA.

When choosing a cluster type, consider your workload requirements, scalability needs, and security priorities to select the option that best aligns with your application goals.