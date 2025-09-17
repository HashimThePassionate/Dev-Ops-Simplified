# **Deploying a WordPress Site with OCI Container Instances**
<details>
<summary>📋 Table of Contents</summary>

- [**Deploying a WordPress Site with OCI Container Instances**](#deploying-a-wordpress-site-with-oci-container-instances)
  - [Introduction](#introduction)
  - [Prerequisites](#prerequisites)
  - [Step-by-Step Instructions](#step-by-step-instructions)
    - [1. Configuring the MySQL Container](#1-configuring-the-mysql-container)
      - [Container Name](#container-name)
      - [Image Selection](#image-selection)
      - [Environment Variables](#environment-variables)
      - [Advanced Options](#advanced-options)
        - [Resources Tab](#resources-tab)
        - [Startup Options Tab](#startup-options-tab)
        - [Security Tab](#security-tab)
      - [Finalize MySQL Container](#finalize-mysql-container)
    - [2. Reviewing and Creating the Container Instance](#2-reviewing-and-creating-the-container-instance)
    - [3. Monitoring Instance Creation](#3-monitoring-instance-creation)
    - [4. Verifying Network Configuration](#4-verifying-network-configuration)
    - [5. Accessing and Installing WordPress](#5-accessing-and-installing-wordpress)
  - [Key Considerations](#key-considerations)
  - [Use Case for Alex](#use-case-for-alex)
  - [Summary](#summary)

</details>

<br/>
**Deploying a WordPress Site with OCI Container Instances**

This guide continues the setup of an Oracle Cloud Infrastructure (OCI) Container Instance to deploy a fully functional WordPress site with a WordPress container (from Docker Hub) and a MySQL container (from OCI Container Registry, OCIR). It covers configuring the MySQL container, reviewing settings, launching the instance, and verifying the WordPress site’s accessibility via a public IP address. This setup is ideal for users like Alex who want to test isolated web applications without managing servers.

## Introduction

In this demo, we complete the configuration of a container instance named `wordpress-demo` in the `CTDOKE` compartment, using the `CI.Standard.A1.Flex` shape (4 OCPUs, 12 GB memory). The WordPress container was configured in the previous session, and now we add the MySQL container, finalize the setup, and verify the WordPress site’s functionality.

## Prerequisites
- **OCI Console Access**: Signed in with permissions to manage container instances in the `CTDOKE` compartment.
- **Container Instance Setup**:
  - Name: `wordpress-demo`.
  - Shape: `CI.Standard.A1.Flex`, 4 OCPUs, 12 GB memory.
  - Networking: Public subnet in `CI-Demo` VCN, public IP assigned, 30-second graceful shutdown, `On Failure` restart policy.
  - WordPress container: Configured with `docker.io/library/wordpress:latest`, environment variables, and 50% resource throttling (2 OCPUs, 6 GB memory).
- **IAM Policies**:
  - `manage compute-container-family` in `CTDOKE`.
  - `use virtual-network-family` for networking.
  - `read repos` in tenancy for OCIR image pulls.
- **Container Images**:
  - WordPress: `docker.io/library/wordpress:latest` (public, Docker Hub).
  - MySQL: `phx.ocir.io/<tenancy-namespace>/oci-repo-public/mysql:8.0.31` (public OCIR repository).
- **Networking**: Security list configured to allow inbound traffic on port 80 for WordPress.

## Step-by-Step Instructions

### 1. Configuring the MySQL Container
In the **Container Configuration** section of the OCI Container Instance creation wizard:

#### Container Name
- **Name**: Set to `mysql`.
- **Note**: The name does not need to be unique (OCID is the unique identifier). Avoid confidential information.

#### Image Selection
1. Click **Select Image**.
2. Choose **OCI Container Registry**.
3. Verify the MySQL image in OCIR:
   - Navigate to **Developer Services > Container Registry** in a new tab.
   - Select the `CTDOKE` compartment.
   - Confirm the public repository `oci-repo-public/mysql` contains the image `mysql:8.0.31`.
4. Back in the wizard, specify:
   - **Repository**: `mysql`.
   - **Image**: `8.0.31`.
   - Full path: `phx.ocir.io/<tenancy-namespace>/oci-repo-public/mysql:8.0.31` (replace `<tenancy-namespace>` with your tenancy namespace, e.g., `intoraclerohit`).
5. **Registry Credentials**:
   - Select **None** since the repository is public.
   - For private repositories, use **OCI Vault Secret** and ensure IAM policies allow access (e.g., `Allow dynamic-group ContainerInstanceDynamicGroup to read repos in tenancy`).
6. Click **Select Image** to confirm.

#### Environment Variables
Set environment variables to initialize the MySQL database (based on [Docker Hub MySQL documentation](https://hub.docker.com/_/mysql)):
1. Add variables:
   - **MYSQL_ROOT_PASSWORD**: `wordpress` (matches WordPress configuration).
   - **MYSQL_DATABASE**: `wordpress`.
   - **MYSQL_USER**: `wordpress`.
   - **MYSQL_PASSWORD**: `wordpress`.
2. Click **Add Another Variable** for each variable.

#### Advanced Options
Click **Show Advanced Options**.

##### Resources Tab
- Enable **Resource Throttling**.
- Select **Percentage** and set to `50%` for both vCPUs and memory.
- Result: MySQL container limited to 2 OCPUs and 6 GB memory, matching the WordPress container’s allocation.

##### Startup Options Tab
- Override the MySQL container’s entrypoint argument:
  - **Command Arguments**: `--default-authentication-plugin=mysql_native_password`.
  - **Purpose**: Ensures compatibility with older MySQL clients (e.g., WordPress) using the `mysql_native_password` plugin.
  - **Note**: This plugin is not recommended for production due to security concerns; use alternatives like `caching_sha2_password` in production.
- Leave other options (working directory, command) as default.

##### Security Tab
- Leave **Read-Only Root Filesystem** and **Run as Non-Root User** unchecked (run as root, default User ID/Group ID = 0).
- Leave **Linux Capabilities** blank (use default capabilities).

#### Finalize MySQL Container
- With environment variables, resource throttling (50%), and the authentication plugin configured, the MySQL container is ready.

### 2. Reviewing and Creating the Container Instance
1. Click **Next** to review settings:
   - **Container Instance**:
     - Name: `wordpress-demo`.
     - Shape: `CI.Standard.A1.Flex`, 4 OCPUs, 12 GB memory.
     - Availability Domain: `PHX-AD-1`.
     - Networking: `CI-Demo` VCN, public subnet, public IPv4 address.
   - **Container 1 (WordPress)**:
     - Image: `docker.io/library/wordpress:latest`.
     - Environment Variables: `WORDPRESS_DB_HOST=127.0.0.1`, `WORDPRESS_DB_USER=wordpress`, `WORDPRESS_DB_PASSWORD=wordpress`, `WORDPRESS_DB_NAME=wordpress`.
     - Resources: 50% (2 OCPUs, 6 GB memory).
   - **Container 2 (MySQL)**:
     - Image: `phx.ocir.io/<tenancy-namespace>/oci-repo-public/mysql:8.0.31`.
     - Environment Variables: `MYSQL_ROOT_PASSWORD=wordpress`, `MYSQL_DATABASE=wordpress`, `MYSQL_USER=wordpress`, `MYSQL_PASSWORD=wordpress`.
     - Resources: 50% (2 OCPUs, 6 GB memory).
     - Command Argument: `--default-authentication-plugin=mysql_native_password`.
2. Click **Create** to launch the container instance.

### 3. Monitoring Instance Creation
- The container instance enters the **Creating** state.
- Check the **Containers** section in the **Container Instance Details** page:
  - Confirms two containers (`wordpress`, `mysql`) with 2 OCPUs and 6 GB memory each.
- Wait a few seconds for the instance to become **Active**.

### 4. Verifying Network Configuration
- **Security List**:
  - Navigate to the subnet used (`CI-Demo` VCN, public subnet).
  - Open the **Default Security List** for the VCN.
  - Confirm an ingress rule allows incoming traffic on **port 80** (required for WordPress HTTP access).
  - Example: Rule allowing all incoming traffic on port 80.
- **Public IP**:
  - Copy the public IP address from the **Container Instance Details** page.

### 5. Accessing and Installing WordPress
1. Open a new browser tab and enter the public IP address.
2. The WordPress installation page appears.
3. Complete the setup:
   - **Language**: Select `English` (default) and click **Continue**.
   - **Site Title**: Enter `CI-Demo`.
   - **Username**: Choose a username (e.g., `admin`).
   - **Password**: Use the auto-generated password (copy it for later use).
   - **Email**: Provide an email address.
   - Click **Install WordPress**.
4. After installation, click the **Login** link.
5. Log in with the username and password.
6. Verify the WordPress site is live and functional.

## Key Considerations
- **IAM Policies**: Ensure permissions for OCIR image pulls and networking. For private OCIR repositories, use OCI Vault secrets with appropriate policies.
- **Networking**:
  - The public IP and port 80 ingress rule enable WordPress access.
  - Use Network Security Groups (NSGs) for finer traffic control in production.
- **Resource Allocation**: Throttling ensures balanced resource use (2 OCPUs, 6 GB memory per container).
- **Security**:
  - Running as root and using `mysql_native_password` is acceptable for this demo but not recommended for production.
  - Enable non-root user, read-only filesystem, and secure authentication plugins (e.g., `caching_sha2_password`) for production.
- **Ephemeral Storage**: Ensure combined image sizes (WordPress + MySQL) are <7.5 GB to fit within the 15 GB limit. Data is lost on instance deletion; use persistent storage (e.g., OCI Block Volume) for production.
- **Monitoring**: Check CPU, memory, and storage usage in the OCI Console to ensure performance.

## Use Case for Alex
For Alex, who needs to test isolated web applications:
- **WordPress Site**: This setup provides a fully functional, publicly accessible WordPress site with MySQL, ideal for testing.
- **Minimal Management**: OCI handles infrastructure, allowing focus on application testing.
- **Security**: Public images simplify setup, but private repositories with Vault secrets and secure configurations can be used for production.
- **Scalability**: The `CI.Standard.A1.Flex` shape supports small-scale testing; extended OCPUs can be used for larger workloads.

## Summary
This guide completed the configuration of a container instance to run a WordPress site:
- Configured the MySQL container from OCIR with environment variables, resource throttling (50%), and the `mysql_native_password` plugin for compatibility.
- Reviewed and created the instance with two containers (WordPress and MySQL) sharing resources and communicating via `localhost`.
- Verified network settings (port 80 ingress rule) and accessed the WordPress site via the public IP.
- Installed and logged into WordPress to confirm functionality.

This setup demonstrates the ease and flexibility of OCI Container Instances for deploying web applications. For detailed instructions, refer to the [OCI Container Instances Documentation](https://docs.oracle.com/en-us/iaas/Content/ContEng/Concepts/contengabout.htm).

Thank you for following along!