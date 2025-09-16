# **Configuring OCI Container Instances**

<details>
<summary>📋 Table of Contents</summary>

- [**Configuring OCI Container Instances**](#configuring-oci-container-instances)
  - [Introduction](#introduction)
  - [Configuration Sections](#configuration-sections)
    - [1. Basic Details and Networking](#1-basic-details-and-networking)
      - [Networking Configuration](#networking-configuration)
    - [2. Advanced Options](#2-advanced-options)
    - [3. Configuring Containers](#3-configuring-containers)
      - [Container Name](#container-name)
      - [Image Selection](#image-selection)
      - [Environment Variables](#environment-variables)
      - [Resource Allocation](#resource-allocation)
      - [Startup Options](#startup-options)
      - [Security Settings](#security-settings)
    - [4. Key Considerations](#4-key-considerations)
  - [Use Case for Alex](#use-case-for-alex)
  - [Summary](#summary)

</details>

<br/>

This guide details the configuration process for Oracle Cloud Infrastructure (OCI) Container Instances, focusing on the creation and configuration of container instances and their containers. It covers networking, advanced options, container settings, resource allocation, startup options, security settings, and key considerations for effective management. This is ideal for users like Alex who need to run isolated web applications or RESTful APIs without managing servers.

## Introduction

OCI Container Instances provide a serverless, secure, and flexible platform for running containerized applications. This guide walks through the key configuration sections, including networking, container settings, and security, to help users set up container instances efficiently.

## Configuration Sections

### 1. Basic Details and Networking
The **Basic Details** section in the OCI Console allows you to configure essential settings for a container instance.

#### Networking Configuration
- **Virtual Cloud Network (VCN) and Subnet**:
  - Specify the VCN and subnet where the container instance will be created.
  - Options:
    - Use an existing VCN and subnet.
    - Create a new VCN or subnet.
    - Enter the OCID of an existing subnet.
- **Public IP Address**:
  - Optionally assign a public IPv4 address if the subnet is public, enabling internet access to the container instance.
- **Advanced Options** (accessible by clicking **Show Advanced Options**):
  - **Network Security Groups (NSGs)**: Control traffic to/from the container instance.
  - **Private IPv4 Address**: Specify a private IP within the subnet.
  - **DNS and Hostname**: Assign a DNS record and hostname for the container instance within the VCN.

### 2. Advanced Options
Under the **Advanced Options** tab, configure additional settings:
- **Graceful Shutdown Timeout**: Set the time (in seconds) the container instance waits for the OS to shut down before powering off.
- **Container Restart Policy**:
  - **Always**: Containers restart automatically, even after successful exits (ideal for critical services like web servers).
  - **Never**: Containers do not restart, regardless of exit reason (suitable for one-time tasks).
  - **On Failure**: Containers restart only if they exit with an error (useful for tasks requiring successful completion).
- **Tags**:
  - Add **free-form tags** or **defined tags** to organize and manage resources.
  - Requires appropriate IAM permissions for tagging.

### 3. Configuring Containers
Each container within a container instance requires specific configurations.

#### Container Name
- Enter a name for the container (does not need to be unique, as the OCID is the unique identifier).
- **Best Practice**: Avoid including confidential information in the name for security reasons.

#### Image Selection
- **OCI Container Registry (OCIR)**:
  - Use Oracle’s managed registry to store, share, and manage container images.
- **External Registry** (e.g., Docker Hub):
  - Specify the registry hostname, repository name, and tags (if any).
  - Authentication options:
    - **None**: For public registries.
    - **Basic**: Provide username and password.
    - **OCI Vault Secret**: Use OCI Vault to securely store credentials for private registries.
      - Ideal for pulling images from private repositories securely.
      - Store credentials as secrets in OCI Vault and grant the container instance permission to read them (see IAM policies in previous guides).

#### Environment Variables
- Set environment variables to customize container execution.
  - Examples:
    - `NGINX_PORT` for NGINX behavior.
    - `MYSQL_PASSWORD` for MySQL configuration.

#### Resource Allocation
- **Resource Tab**:
  - By default, containers can use all available resources (vCPUs and memory) of the container instance.
  - To restrict resources:
    - Enable **resource throttling**.
    - Specify resource limits in **absolute values** (e.g., 2 vCPUs, 4 GB memory) or **percentages** (e.g., 50% of available resources).
  - Allows splitting resources among multiple containers for balanced performance.

#### Startup Options
- **Startup Options Tab**:
  - Configure:
    - **Working Directory**: Set the container’s working directory.
    - **Command**: Specify the entrypoint command.
    - **Command Arguments**: Provide additional arguments for the command.
  - Customizes the startup behavior of containers.

#### Security Settings
- **Security Tab**:
  - **Read-Only Root Filesystem**:
    - Restricts write access to the container’s root filesystem, preventing unauthorized modifications to critical system files.
  - **Run as Non-Root User**:
    - Ensures the container runs without root privileges, reducing the impact of potential security vulnerabilities.
    - Specify a **User ID** and **Group ID** (between 1 and 65535; 0 is reserved for root).
  - **Linux Capabilities**:
    - Add or drop specific Linux capabilities to enhance security.
    - By default, containers launch with standard capabilities; drop unnecessary ones to minimize the attack surface.

### 4. Key Considerations
- **IAM Policies**:
  - Assign appropriate IAM policies to user groups managing container instances (e.g., `manage compute-container-family` in the target compartment).
  - Ensure policies allow access to OCIR, networking resources, and Vault Secrets if needed (see previous guides).
- **Container Limits**:
  - A maximum of **60 containers** can be created per container instance.
- **Management**:
  - Container instances can be started, stopped, or restarted.
  - Billing pauses when a container instance is stopped (using standard shapes).
- **Metrics**:
  - Monitor health, capacity, and performance (e.g., CPU, memory, storage usage) via the **Container Instance Details** page in the OCI Console.
- **Ephemeral Storage**:
  - Data on ephemeral storage is **lost** when a container instance is deleted.
  - Monitor storage usage to ensure sufficient space for images, logs, and other data.

## Use Case for Alex
For Alex, who wants to test isolated web applications or RESTful APIs:
- **Networking**: Use a public subnet with a public IP for internet-accessible APIs or a private subnet with NSGs for secure internal access.
- **Restart Policy**: Set to `Always` for web servers or `On Failure` for testing tasks.
- **Image Source**: Pull images from OCIR (private repositories with Vault Secrets) or Docker Hub (public images).
- **Resources**: Throttle resources for multiple containers (e.g., an API and a sidecar) to optimize performance.
- **Security**: Enable read-only root filesystem and non-root user to enhance security for public-facing applications.

## Summary
This guide covered the configuration of OCI Container Instances, including:
- **Networking**: Configuring VCN, subnet, public IP, NSGs, DNS, and hostname.
- **Advanced Options**: Setting graceful shutdown timeout, restart policy, and tags.
- **Container Configuration**: Specifying name, image source (OCIR or external), environment variables, resource limits, startup options, and security settings.
- **Key Considerations**: IAM policies, container limits, management, metrics, and ephemeral storage.

These configurations enable users like Alex to deploy containerized applications efficiently and securely without managing infrastructure. For detailed setup instructions, refer to the [OCI Container Instances Documentation](https://docs.oracle.com/en-us/iaas/Content/ContEng/Concepts/contengabout.htm).

Thank you for exploring OCI Container Instances!