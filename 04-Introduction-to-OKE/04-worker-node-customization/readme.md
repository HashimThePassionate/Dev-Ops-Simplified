# **Worker Node Customization**

<details>
<summary>📋 Table of Contents</summary>

- [**Worker Node Customization**](#worker-node-customization)
  - [Overview](#overview)
  - [Customizing Worker Nodes](#customizing-worker-nodes)
    - [Creation Methods](#creation-methods)
    - [Node Types](#node-types)
  - [Supported Images for Managed Nodes](#supported-images-for-managed-nodes)
    - [Platform Images](#platform-images)
    - [OKE Images](#oke-images)
    - [Custom Images](#custom-images)
  - [Supported Shapes for Managed Nodes](#supported-shapes-for-managed-nodes)
    - [Unsupported Shapes for Managed Nodes](#unsupported-shapes-for-managed-nodes)
  - [Supported Shapes for Virtual Nodes](#supported-shapes-for-virtual-nodes)
    - [Checking Supported Shapes](#checking-supported-shapes)
  - [Summary](#summary)

</details>

## Overview

This README provides a detailed guide on customizing worker nodes when creating a cluster using Oracle Cloud Infrastructure Container Engine for Kubernetes (OKE). It covers the flexibility to configure worker nodes, including selecting operating system images and compute shapes, and explores the types of images supported for managed nodes, as well as supported shapes for both managed and virtual nodes. This customization empowers development teams to tailor OKE clusters to meet specific project and workload requirements.

## Customizing Worker Nodes

When creating an OKE cluster, you can customize worker nodes by specifying two key elements:

1. **Operating System Image**:
   - The image serves as a template for the worker node's virtual hard drive.
   - It determines the operating system and other software installed on the node.

2. **Shape**:
   - The shape defines the number of CPUs and the amount of memory allocated to each compute instance.
   - This ensures the worker node meets your specific performance and resource requirements.

This customization allows you to tailor your OKE cluster to your exact needs, enhancing the efficiency of building, deploying, and managing cloud-native applications.

### Creation Methods
You can define and create OKE clusters using:
- **Console**: An intuitive interface for cluster configuration.
- **REST API**: A programmatic option for advanced control.

This flexibility is particularly valuable for development teams working on cloud-native applications.

### Node Types
You can specify whether applications run on:
- **Virtual Nodes**: Fully managed, serverless nodes.
- **Managed Nodes**: Partially managed nodes within your OCI tenancy.

OKE efficiently provisions both node types within your existing Oracle Cloud Infrastructure (OCI) tenancy, ensuring adaptability to your project and workload requirements.

## Supported Images for Managed Nodes

OKE supports provisioning managed nodes using a selection of Oracle Linux images provided by OCI. These images are categorized into three types: **Platform Images**, **OKE Images**, and **Custom Images**.

### Platform Images
- **Description**: Supplied by Oracle, these images contain only an Oracle Linux operating system.
- **Provisioning Process**:
  - When a compute instance hosting a managed node boots for the first time, OKE downloads, installs, and configures all required software.
- **Naming Convention**:
  - Platform image names may or may not include a CPU architecture reference and do not include "OKE" in their names.
  - Examples: `Oracle-Linux-8.5`, `Oracle-Linux-7.9`.
- **How to Check Supported Images**:
  - Use the CLI command: `oci ce node-pool-options get --node-pool-option-id all`.
  - Review the supported images in the `data sources` section of the response.

### OKE Images
- **Description**: Built on platform images, OKE images are optimized for managed nodes, pre-configured with necessary settings and software for a seamless experience.
- **Benefits**:
  - Significantly reduces provisioning time compared to platform or custom images (by more than half compared to platform images).
- **How to Access**:
  - **Console**: Select "OKE Images" as the image source in the "Browse All Images" window during the cluster creation workflow.
  - **CLI**: Use the command: `oci ce node-pool-options get --node-pool-option-id all` and review the `data sources` section.
- **Naming Convention**:
  - Format: `<platform-image-name>-OKE-<kubernetes-version>-<OKE-build-number>`.
  - Example: `Oracle-Linux-8.7-Gen2-GPU-OKE-1.25.4-549`.
- **Compatibility**:
  - When creating or updating node pools, the OKE image’s Kubernetes version must match the node pool’s Kubernetes version.

### Custom Images
- **Description**: Allow you to design worker node images tailored to specific requirements, based on supported platform or OKE images.
- **Features**:
  - Typically include an Oracle Linux operating system.
  - Can incorporate additional customizations, configurations, and software present when the image was created.
- **Provisioning**:
  - Use the CLI or API to specify the OCID of the custom image when creating a node pool.
  - Example Command: `oci ce node-pool create --node-image-id <OCID-of-custom-image>`.
- **Requirements**:
  - Must be based on Oracle Linux images returned by the `oci ce node-pool-options get` command.
  - Requires access to a yum repository (public or internal).
- **Considerations**:
  - OKE installs Kubernetes on top of custom images, which may alter certain kernel configurations.
  - For best support and compatibility, use the most up-to-date base image when creating custom images.

## Supported Shapes for Managed Nodes

Choosing the right shape for managed nodes is critical for optimal performance and resource allocation. OKE supports the following shapes for managed nodes:
- **Flexible Shapes**:
  - All flexible shapes except burstable instances (e.g., `VM.Standard.E3.Flex`).
- **Bare Metal Shapes**:
  - Standard shapes.
  - GPU shapes.
- **HPC Shapes**:
  - Excluding those in RDMA networks.
- **VM Shapes**:
  - Standard shapes.
  - GPU shapes.
  - Dense I/O shapes.

### Unsupported Shapes for Managed Nodes
The following shapes are not supported:
- Dedicated VM host shapes.
- Micro VM shapes.
- HPC shapes on bare metal instances in RDMA networks.
- Flexible shapes for burstable instances (e.g., `VM.Standard.E3.Flex`).

## Supported Shapes for Virtual Nodes

Virtual nodes offer flexibility in managing Kubernetes clusters, and specific shapes are supported to meet workload needs:
- **Supported Shapes**:
  - `Pod.Standard.A1.Flex`
  - `Pod.Standard.E3.Flex`
  - `Pod.Standard.E4.Flex`
- **Note**: All other shapes are not supported for virtual nodes.

### Checking Supported Shapes
To view supported shapes for both managed and virtual nodes in your tenancy:
- Use the CLI command: `oci ce node-pool-options get --node-pool-option-id all`.
- Review the `data shapes` section of the response for available shapes.

## Summary

This README has outlined the customization options for worker nodes in Oracle Cloud Infrastructure Container Engine for Kubernetes (OKE):
- **Customization**: Specify operating system images and compute shapes to tailor worker nodes to your needs.
- **Image Types**:
  - **Platform Images**: Oracle Linux images requiring software installation at boot.
  - **OKE Images**: Pre-configured for faster provisioning and optimized for managed nodes.
  - **Custom Images**: Tailored images based on Oracle Linux for specific requirements.
- **Shapes**:
  - Managed nodes support a variety of flexible, bare metal, HPC, and VM shapes, with some exclusions.
  - Virtual nodes support specific flexible shapes (`Pod.Standard.A1.Flex`, `Pod.Standard.E3.Flex`, `Pod.Standard.E4.Flex`).
- **Management**: Use the Console or REST API to create and configure clusters, with CLI commands to check supported images and shapes.

By leveraging these customization options, you can optimize your OKE clusters for performance, scalability, and compatibility with your cloud-native applications.