# **Creating Kubernetes Clusters Using Console Workflows**

<details>
<summary>📋 Table of Contents</summary>

- [**Creating Kubernetes Clusters Using Console Workflows**](#creating-kubernetes-clusters-using-console-workflows)
  - [Overview](#overview)
  - [Prerequisites](#prerequisites)
  - [Console Workflow Options](#console-workflow-options)
    - [Quick Create Workflow](#quick-create-workflow)
    - [Custom Create Workflow](#custom-create-workflow)
  - [Alternative Methods for Cluster Creation](#alternative-methods-for-cluster-creation)
    - [CLI (Command Line Interface)](#cli-command-line-interface)
    - [API](#api)
    - [OCI Resource Manager and Terraform Modules](#oci-resource-manager-and-terraform-modules)
  - [Accessing Documentation](#accessing-documentation)
  - [Summary](#summary)

</details>

## Overview

This README provides a detailed guide on creating Kubernetes clusters using Oracle Cloud Infrastructure (OCI) Container Engine for Kubernetes (OKE) through console workflows. It covers the **Quick Create** and **Custom Create** workflows, as well as alternative methods like CLI, API, and Terraform modules. Whether you are a tenancy administrator or a user with appropriate permissions, this guide walks you through the steps to configure and deploy OKE clusters efficiently.

## Prerequisites

To create an OKE cluster, you must:

- Be a member of the tenancy’s **Administrators group** or belong to a group with the necessary permissions granted through policies (as detailed in previous lessons).
- Have access to an OCI tenancy with sufficient resource quotas and a configured compartment (refer to earlier READMEs for details).

## Console Workflow Options

OKE provides two primary console workflows for creating Kubernetes clusters: **Quick Create** and **Custom Create**. Each offers distinct advantages depending on your needs.

### Quick Create Workflow

The **Quick Create workflow** is designed for rapid deployment with minimal configuration, using default settings to get your cluster up and running quickly.

- **Key Features**:
  - Automatically creates necessary network resources, including:
    - Regional subnets for the Kubernetes API endpoint.
    - Regional subnets for worker nodes.
    - Regional subnets for load balancers (public by default).
  - Allows specification of whether subnets for the Kubernetes API endpoint and worker nodes are **public** or **private**.
  - Requires minimal user input, enabling cluster creation with just a few clicks.
- **Permissions**:
  - You must belong to a group with policies granting permissions to manage clusters and network resources (e.g., `manage vcns`, `manage subnets`, etc.).
- **Use Case**: Ideal for users who need a quick setup with default configurations and minimal customization.

### Custom Create Workflow

The **Custom Create workflow** offers maximum control over cluster configuration, allowing you to tailor settings to your specific requirements.

- **Key Features**:
  - Customize cluster properties, including:
    - **Encryption**: Configure encryption settings to meet security needs.
    - **Network Resources**: Specify existing network resources (e.g., public or private subnets for the Kubernetes API endpoint, worker nodes, and load balancers).
  - Flexibility in node pool creation:
    - Define node pools during cluster creation.
    - Or, create a cluster without node pools and add them later.
  - **Node Pool-Free Cluster**:
    - Creating a cluster without node pools is advantageous when installing network policy providers like **Calico**.
    - **Best Practice**: Set up the cluster without node pools initially to avoid recreating pods after installing Calico or similar providers, ensuring a smoother deployment.
- **Use Case**: Suitable for advanced users or scenarios requiring specific configurations, such as custom network setups or delayed node pool creation.

## Alternative Methods for Cluster Creation

In addition to console workflows, OKE clusters can be created using other methods for more advanced or programmatic control.

### CLI (Command Line Interface)

- **Overview**: The OCI CLI provides commands to configure and manage OKE resources, including cluster creation.
- **Access**: Refer to the official OCI documentation for CLI options.
  - Example: Use the `oci ce cluster create` command to create a cluster.
- **Details**: The CLI documentation outlines specific commands and parameters for managing OKE resources.

### API

- **Overview**: Use the OCI API’s `CreateCluster` operation to programmatically create clusters.
- **Supported SDKs**: Available for multiple programming languages, including Java, Python, and more.
- **Access**: Detailed instructions are provided in the OCI API documentation, accessible via links in the official documentation.

### OCI Resource Manager and Terraform Modules

- **Overview**: Use OCI Resource Manager stacks or Oracle Terraform modules to automate OKE cluster setup.
- **Terraform Modules**:
  - Available on the OCI GitHub page, these modules simplify cluster configuration.
  - Enable easy setup with pre-defined configurations, saving time and effort.
- **Use Case**: Ideal for infrastructure-as-code (IaC) enthusiasts or teams managing multiple clusters programmatically.

## Accessing Documentation

- **Official OCI Documentation**:
  - Provides comprehensive details on creating and managing OKE clusters via Console, CLI, and API.
  - Includes links to:
    - CLI commands for OKE resource management.
    - API operations (e.g., `CreateCluster`) with SDK-specific examples.
- **GitHub**: Oracle’s GitHub page hosts Terraform modules for OKE, streamlining cluster setup.

## Summary

This README has outlined the methods for creating OKE clusters:

- **Quick Create Workflow**: A fast, default-driven approach that automatically configures network resources, ideal for rapid deployment.
- **Custom Create Workflow**: Offers maximum flexibility to tailor cluster properties, including encryption, network resources, and node pool configurations, with a best practice of creating node pool-free clusters for network policy providers like Calico.
- **Alternative Methods**: CLI, API, and Terraform modules provide programmatic options for advanced users.
- **Resources**: Official OCI documentation and GitHub Terraform modules provide detailed guidance and tools for cluster creation.

By choosing the appropriate workflow or method, you can create OKE clusters that align with your project’s requirements, balancing speed, customization, and automation.