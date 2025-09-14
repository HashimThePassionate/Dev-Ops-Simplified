# **Introduction to Oracle Cloud Infrastructure Registry (OCIR)**

<details>
<summary><strong>📋 Table of Contents</strong></summary>

- [**Introduction to Oracle Cloud Infrastructure Registry (OCIR)**](#introduction-to-oracle-cloud-infrastructure-registry-ocir)
  - [Overview](#overview)
  - [What is OCIR?](#what-is-ocir)
  - [Key Advantages of OCIR](#key-advantages-of-ocir)
  - [Key Concepts](#key-concepts)
    - [Container Images](#container-images)
    - [Container Registry](#container-registry)
    - [Repository](#repository)
    - [Key Terms](#key-terms)
  - [Getting Started](#getting-started)
  - [Summary](#summary)

</details>

## Overview
This guide introduces **Oracle Cloud Infrastructure Registry (OCIR)**, an Oracle-managed registry designed to simplify the development-to-production workflow for containerized applications. It covers the key features, benefits, and concepts of OCIR, making it an essential tool for developers managing container images.

## What is OCIR?
**OCIR** is a highly available and scalable Docker registry that allows developers to store, share, and manage container images. It supports both private and public registries, enabling secure internal use or public access for pulling images from repositories. Key characteristics include:

- **Compliance**: Adheres to Open Container Initiative (OCI) standards, supporting Docker images, manifest lists (multi-architecture images for ARM, AMD64, etc.), and Helm charts.
- **Security**: Offers private access through a service gateway, ensuring resources within a Virtual Cloud Network (VCN) in the same region can access OCIR without public internet exposure.
- **Integration**: Seamlessly integrates with Oracle Container Engine for Kubernetes (OKE) and OCI Identity and Access Management (IAM) for authentication.

## Key Advantages of OCIR
- **Integration with Kubernetes**: Works cohesively with Oracle Container Engine for Kubernetes for streamlined container management.
- **Flexible Access Control**: Supports private or public registries, giving administrators control over accessibility.
- **Regional Availability**: Enables efficient image pulling from the same region as deployments, leveraging OCI's infrastructure for high performance, availability, and low-latency operations.
- **Anywhere Access**: Allows image operations via container CLI from cloud, on-premises, or personal laptops.
- **Repository Quotas**: Each tenancy can have up to 500 repositories per region, with a cumulative storage limit of 500 GB and up to 100,000 images per repository. Charges apply only for stored images.

## Key Concepts

### Container Images
A **container image** is a read-only template containing instructions for creating a container. It includes the application and its dependencies.

### Container Registry
OCIR is an OCI-compliant registry that stores artifacts like Docker images, manifest lists (multi-architecture images), and Helm charts.

### Repository
A **repository** is a named collection of related images grouped for convenience. Each image in a repository is identified by a unique **tag** (e.g., version number or string like `latest`).

Example:
```
hashim_project/acme-web-app:4.6.1
hashim_project/acme-web-app:4.6.2
```
- **Repository Name**: `hashim_project/acme-web-app`
- **Tags**: `4.6.1`, `4.6.2`

Repositories can be private or public, and users require an OCI username and authentication token to push/pull images.

### Key Terms
- **Region Key**: Identifies the OCIR region (e.g., `iad` for US East Ashburn, `phx` for US West Phoenix).
- **Tenancy Namespace**: An auto-generated, immutable alphanumeric string unique to each tenancy. Find it in the OCI console under Profile > Tenancy > Object Storage Namespace.
  ```
  Example: ansh81vru1zp
  ```
- **Repository Name**: A unique name for a repository within a tenancy, which may include slashes (e.g., `project01/acme-web-app`). Slashes do not indicate a hierarchical structure but are part of the name string.
  ```
  Example: project01/acme-web-app
  ```
- **Registry Identifier**: Combines region key and tenancy namespace.
  ```
  Structure: <region-key>.ocir.io/<tenancy-namespace>
  Example: iad.ocir.io/ansh81vru1zp
  ```
- **Image Name**: Combines repository name and tag, separated by a colon.
  ```
  Structure: <repository-name>:<tag>
  Example: project01/acme-web-app:4.6.3
  ```
- **Image Path**: A fully qualified path to an image in a registry.
  ```
  Structure: <region-key>.ocir.io/<tenancy-namespace>/<repository-name>:<tag>
  Examples:
  iad.ocir.io/ansh81vru1zp/project01/acme-web-app:4.6.3
  phx.ocir.io/ansh81vru1zp/my-hello-app:latest
  ```

## Getting Started
To interact with OCIR:
1. **Set Up Authentication**: Use an OCI username and authentication token for push/pull operations.
2. **Access OCIR**: Use the container CLI (e.g., Docker CLI) to push/pull images from OCIR repositories.
3. **Check Regional Availability**: Refer to Oracle's official documentation for region keys specific to your region.

## Summary
Oracle Cloud Infrastructure Registry (OCIR) is a powerful tool for managing containerized workflows, offering seamless integration with Kubernetes, robust security, regional accessibility, and efficient image management. By understanding key concepts like images, repositories, and tags, developers can leverage OCIR to streamline application development and deployment.