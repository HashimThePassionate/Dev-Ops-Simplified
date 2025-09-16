# **Getting Started with OCI Container Instances: Creating a WordPress Site**

This guide demonstrates how to create and configure an Oracle Cloud Infrastructure (OCI) Container Instance to run a fully functional WordPress site with both a WordPress container (pulled from Docker Hub) and a MySQL container (pulled from OCI Container Registry, OCIR). It walks through the setup process using the OCI Console, focusing on basic details, networking, and advanced configuration options.

<details>
<summary>📋 Table of Contents</summary>

- [**Getting Started with OCI Container Instances: Creating a WordPress Site**](#getting-started-with-oci-container-instances-creating-a-wordpress-site)
  - [Introduction](#introduction)
  - [Prerequisites](#prerequisites)
  - [Step-by-Step Instructions](#step-by-step-instructions)
    - [1. Accessing the Container Instances Service](#1-accessing-the-container-instances-service)
    - [2. Creating a Container Instance](#2-creating-a-container-instance)
    - [3. Configuring Placement](#3-configuring-placement)
    - [4. Selecting a Compute Shape](#4-selecting-a-compute-shape)
    - [5. Configuring Networking](#5-configuring-networking)
    - [6. Advanced Container Instance Options](#6-advanced-container-instance-options)
    - [7. Next Steps (Not Covered in Demo Snippet)](#7-next-steps-not-covered-in-demo-snippet)
  - [Key Considerations](#key-considerations)
  - [Use Case for Alex](#use-case-for-alex)
  - [Summary](#summary)

</details>

## Introduction

OCI Container Instances provide a serverless platform to run containerized workloads without managing infrastructure. In this demo, we will:
- Create a container instance in the `CTDOKE` compartment.
- Configure it to run a WordPress site using a WordPress container from Docker Hub and a MySQL container from OCIR.
- Set up networking, compute shapes, and advanced options for optimal performance and accessibility.

This setup is ideal for users like Alex, who need to test isolated web applications like WordPress without managing servers or using Kubernetes.

## Prerequisites
- **OCI Account**: Access to the OCI Console with appropriate permissions to create container instances.
- **Compartment**: A compartment (e.g., `CTDOKE`) where the container instance will be created.
- **VCN and Subnet**: An existing Virtual Cloud Network (VCN) and public subnet for networking.
- **Container Images**:
  - WordPress image from Docker Hub.
  - MySQL image in OCIR (e.g., `oci-repo-public/mysql:latest` from previous demos).
- **IAM Policies**: Permissions to manage `compute-container-family`, use `virtual-network-family`, and read OCIR repositories (see previous guides).

## Step-by-Step Instructions

### 1. Accessing the Container Instances Service
1. Sign in to the OCI Console.
2. Click the **Hamburger Menu** (top-left corner).
3. Navigate to **Developer Services > Container Instances**.
4. In the **Compartment** section, select the `CTDOKE` compartment.

### 2. Creating a Container Instance
1. Click the **Create Container Instance** button to open the creation wizard.
2. In the **Basic Details** section:
   - **Name**: Enter a name for the container instance (e.g., `wordpress-demo`). Remove any timestamp for simplicity.
   - **Compartment**: Ensure `CTDOKE` is selected.

### 3. Configuring Placement
- **Availability Domain**: Choose an availability domain (e.g., `AD-1`) or accept the default.
- **Fault Domain**:
  - Click **Show Advanced Options**.
  - Select **Let Oracle Choose the Best Fault Domain** (default) or choose a specific fault domain for high availability.
  - For this demo, use the default: **Let Oracle Choose the Best Fault Domain**.

### 4. Selecting a Compute Shape
- **Default Shape**: `CI.Standard.E4.Flex` with 1 OCPU and 16 GB memory.
- **Customize Shape**:
  - Click **Change Shape** to open the **Browse All Shapes** window.
  - Available shapes:
    - **AMD**: `CI.Standard.E4.Flex` (1–8 OCPUs standard, up to 64 with extended OCPUs), `CI.Standard.E3.Flex`.
    - **ARM**: `CI.Standard.A1.Flex` (1–16 OCPUs standard, higher with extended OCPUs).
  - **Extended OCPUs**:
    - Enable for high-demand workloads requiring more cores than standard limits.
    - Note: Extended OCPU instances may take longer to create.
  - For this demo:
    - Select **CI.Standard.A1.Flex** (ARM-based).
    - Set **OCPUs** to `4`.
    - Adjust **Memory** to `12 GB` (default is 24 GB for 4 OCPUs; each OCPU supports up to 6 GB).
    - Click **Select Shape** to confirm.

### 5. Configuring Networking
- **Virtual Cloud Network (VCN)**:
  - Select an existing VCN (e.g., `CI-Demo`) or create a new one.
- **Subnet**:
  - Choose an existing public subnet to allow internet access.
- **Public IP Address**:
  - Check **Assign a Public IPv4 Address** to make the WordPress site accessible from the internet.
- **Advanced Options** (click **Show Advanced Options**):
  - **Network Security Groups (NSGs)**: Optionally configure NSGs to control traffic.
  - **Source/Destination Check**: Enabled by default; VNIC drops packets where it is not the source or destination.
  - **Private IPv4 Address**: Optionally specify a private IP within the subnet.
  - **DNS and Hostname**: Optionally set DNS records and a hostname for the container instance.
  - For this demo, skip advanced networking options and click **Hide Advanced Options**.

### 6. Advanced Container Instance Options
- **Graceful Shutdown Timeout**:
  - Set to `30 seconds` to allow containers to exit gracefully when the instance is terminated.
- **Container Restart Policy**:
  - Select **On Failure** to restart containers only if they exit with an error, suitable for ensuring the WordPress and MySQL containers recover from failures.
  - Other options:
    - **Always**: Restarts containers regardless of exit status (ideal for critical services).
    - **Never**: No restarts (suitable for one-time tasks).
- For this demo, keep **On Failure** as the restart policy.

### 7. Next Steps (Not Covered in Demo Snippet)
The demo snippet ends at clicking **Next**, which typically leads to the **Container Configuration** section. Based on previous guides, the next steps would include:
- **Adding Containers**:
  - Add two containers: one for WordPress (from Docker Hub) and one for MySQL (from OCIR).
- **Container Configuration**:
  - **Names**: Assign names (e.g., `wordpress-container`, `mysql-container`).
  - **Images**:
    - WordPress: Specify `docker.io/library/wordpress:latest` (public registry, no authentication).
    - MySQL: Specify `phx.ocir.io/<tenancy-namespace>/oci-repo-public/mysql:latest` (use OCI Vault secrets for private repository authentication if needed).
  - **Environment Variables**:
    - For WordPress: Set variables like `WORDPRESS_DB_HOST` (point to `localhost` or `127.0.0.1` for MySQL on the same instance), `WORDPRESS_DB_USER`, `WORDPRESS_DB_PASSWORD`, and `WORDPRESS_DB_NAME`.
    - For MySQL: Set variables like `MYSQL_ROOT_PASSWORD` and `MYSQL_DATABASE`.
  - **Resource Limits**: Optionally throttle CPU and memory to balance resources between containers.
  - **Startup Options**: Configure working directory, commands, or arguments if needed.
  - **Security**: Enable read-only root filesystem and non-root user for enhanced security.
- **Tags**: Add free-form or defined tags for resource management.
- **Create**: Review settings and click **Create** to launch the container instance.

## Key Considerations
- **IAM Policies**:
  - Ensure policies allow:
    - `manage compute-container-family` in the `CTDOKE` compartment.
    - `use virtual-network-family` for networking.
    - `read repos` in tenancy for OCIR image pulls.
    - Access to OCI Vault secrets if pulling from private OCIR repositories.
- **Ephemeral Storage**:
  - Ensure images (WordPress and MySQL) are <7.5 GB to fit within the 15 GB ephemeral storage limit.
  - Monitor storage usage via the **Container Instance Details** page.
- **Networking**:
  - Use a public subnet with a public IP for WordPress accessibility.
  - Configure NSGs to restrict traffic (e.g., allow HTTP/HTTPS for WordPress).
- **Container Communication**:
  - WordPress and MySQL containers on the same instance communicate via `localhost` or `127.0.0.1`.

## Use Case for Alex
For Alex, who wants to test isolated web applications:
- **WordPress Site**: This setup creates a fully functional WordPress site with MySQL, accessible publicly via a public IP.
- **Minimal Management**: No need to manage servers or Kubernetes; OCI handles the infrastructure.
- **Security**: Use non-root users and read-only filesystems for both containers to enhance security.
- **Flexibility**: The `CI.Standard.A1.Flex` shape with 4 OCPUs and 12 GB memory supports a small-scale WordPress deployment.

## Summary
This guide walked through the initial steps to create an OCI Container Instance for a WordPress site:
- Selected the `CTDOKE` compartment and named the instance `wordpress-demo`.
- Configured placement with default availability and fault domain settings.
- Chose the `CI.Standard.A1.Flex` shape with 4 OCPUs and 12 GB memory.
- Set up networking with an existing VCN (`CI-Demo`), public subnet, and public IP.
- Configured advanced options like a 30-second graceful shutdown timeout and `On Failure` restart policy.

The next steps involve configuring the WordPress and MySQL containers, which can be completed based on the container configuration guidelines from previous demos. For detailed instructions, refer to the [OCI Container Instances Documentation](https://docs.oracle.com/en-us/iaas/Content/ContEng/Concepts/contengabout.htm).

Thank you for following along!