# **Prerequisites to OKE Cluster**

<details>
<summary>📋 Table of Contents</summary>

- [**Prerequisites to OKE Cluster**](#prerequisites-to-oke-cluster)
  - [Overview](#overview)
  - [Access to an Oracle Cloud Infrastructure Tenancy](#access-to-an-oracle-cloud-infrastructure-tenancy)
  - [Sufficient Resource Quotas](#sufficient-resource-quotas)
    - [Compute Instance Quota](#compute-instance-quota)
    - [Block Volume Quota](#block-volume-quota)
    - [Load Balancer Quota](#load-balancer-quota)
    - [VCN and Subnet Quota](#vcn-and-subnet-quota)
  - [Ready-to-Deploy Compartment](#ready-to-deploy-compartment)
  - [Configuring Network Resources](#configuring-network-resources)
  - [Policies for Cluster Management](#policies-for-cluster-management)
  - [Tools for Accessing the OKE Cluster](#tools-for-accessing-the-oke-cluster)
    - [Kubernetes Command-Line Tool (kubectl)](#kubernetes-command-line-tool-kubectl)
    - [Kubeconfig File Configuration](#kubeconfig-file-configuration)
    - [Permissions](#permissions)
  - [Summary](#summary)

</details>

## Overview

This README outlines the essential prerequisites for setting up and using Oracle Cloud Infrastructure Container Engine for Kubernetes (OKE). It covers access to an OCI tenancy, resource quotas, compartment setup, network resource configuration, policies, and tools for cluster access. Understanding these prerequisites ensures a smooth deployment and management of OKE clusters. Throughout this document, each aspect is detailed to guide you through the preparation process.

## Access to an Oracle Cloud Infrastructure Tenancy

The first step is gaining access to an Oracle Cloud Infrastructure (OCI) tenancy, which refers to your Oracle Cloud account. OKE is available in all Oracle Cloud Infrastructure regions, allowing you to leverage its capabilities regardless of your location. Your tenancy serves as the foundation for your OKE journey.

## Sufficient Resource Quotas

It's essential to have sufficient quotas (also known as service limits) on various resources. Below are the key resource quotas to consider:

### Compute Instance Quota
- Depends on the chosen compute shape, determined by the number of compute cores and memory required.
- For a Kubernetes cluster with managed nodes and managed node pools: Requires at least one compute instance.
- For a highly available cluster in a region with three availability domains: Requires at least three compute instances (one in each domain).

### Block Volume Quota
- If creating Kubernetes persistent volumes, ensure sufficient quota in each availability domain.
- Persistent volume claims should request a minimum of 50 gigabytes.

### Load Balancer Quota
- Required if creating a load balancer to distribute traffic between nodes running services in the Kubernetes cluster.
- Ensure sufficient quota is available within the region.

### VCN and Subnet Quota
- Sufficient quotas must be available in the region when creating clusters.

## Ready-to-Deploy Compartment

A dedicated compartment within your tenancy is needed for deployment. This compartment houses essential network resources, including:
- Virtual Cloud Network (VCN)
- Subnets
- Internet Gateways
- Route Tables
- Security Lists

If the compartment does not exist, create it. Network resources can reside in the root compartment, but as a best practice:
- If multiple teams will create clusters, create a dedicated compartment for each team to ensure organization and simplify management.

## Configuring Network Resources

Configure network resources within the compartment, including:
- Virtual Cloud Networks
- Subnets
- Internet Gateways
- Route Tables
- Security Lists

These must be set up correctly in each region where clusters will be created and deployed. Best practice:
- Use regional subnets to simplify failover across availability domains.

When creating a new cluster:
- OKE can automatically create and configure these network resources.
- Alternatively, specify existing, correctly configured network resources.

## Policies for Cluster Management

To create and manage clusters effectively, you must belong to one of the following:
- **Tenancy Administrators Group**: Provides necessary permissions to oversee OKE.
- **A Group with Appropriate Policies**: Grants specific OKE permissions (covered in upcoming lessons).

Policies are crucial for granting and managing permissions, so ensure you are in the right group or have the appropriate policies in place.

## Tools for Accessing the OKE Cluster

To perform Kubernetes operations on a cluster, the following are required:

### Kubernetes Command-Line Tool (kubectl)
- Use the installation included in CloudShell or set up a local installation.
- Accessing a cluster using kubectl will be explored later.

### Kubeconfig File Configuration
- Configure your own kubeconfig file for the cluster (covered in upcoming lessons).
- Each user must set up their own kubeconfig file; you cannot use one configured by another user.

### Permissions
- Appropriate permissions must be in place for effective cluster interaction.
- Access control and policies will be detailed in upcoming lessons.

## Summary

This README has detailed the prerequisites for Oracle Cloud Infrastructure Container Engine for Kubernetes (OKE):
- Access to an OCI tenancy available in all regions.
- Sufficient quotas for compute instances, block volumes, load balancers, VCNs, and subnets.
- A dedicated compartment for network resources, with best practices for multi-team environments.
- Proper configuration of network resources, with options for automatic setup.
- Membership in appropriate groups or policies for permissions.
- Essential tools like kubectl, kubeconfig files, and permissions for cluster access.

By addressing these prerequisites, you will be well-prepared to deploy and manage OKE clusters effectively.