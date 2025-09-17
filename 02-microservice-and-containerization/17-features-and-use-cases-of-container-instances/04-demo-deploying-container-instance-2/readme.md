# **Configuring WordPress and MySQL Containers in OCI Container Instances**

This guide continues the process of creating an Oracle Cloud Infrastructure (OCI) Container Instance to run a fully functional WordPress site, focusing on configuring the WordPress container and adding a MySQL container. The WordPress image is pulled from Docker Hub, while the MySQL image is pulled from OCI Container Registry (OCIR). This setup is ideal for users like Alex who want to test isolated web applications without managing servers.

<details>
<summary>Table of Contents</summary>

- [**Configuring WordPress and MySQL Containers in OCI Container Instances**](#configuring-wordpress-and-mysql-containers-in-oci-container-instances)
  - [Introduction](#introduction)
  - [Prerequisites](#prerequisites)
  - [Step-by-Step Instructions](#step-by-step-instructions)
    - [1. Configuring the WordPress Container](#1-configuring-the-wordpress-container)
      - [Container Name](#container-name)
      - [Image Selection](#image-selection)
      - [Environment Variables](#environment-variables)
      - [Advanced Options](#advanced-options)
        - [Resources Tab](#resources-tab)
        - [Startup Options Tab](#startup-options-tab)
        - [Security Tab](#security-tab)
      - [Finalize WordPress Container](#finalize-wordpress-container)
    - [2. Adding the MySQL Container](#2-adding-the-mysql-container)
    - [3. Container Instance Resource Sharing](#3-container-instance-resource-sharing)
    - [4. Completing the Setup](#4-completing-the-setup)
  - [Key Considerations](#key-considerations)
  - [Use Case for Alex](#use-case-for-alex)
  - [Summary](#summary)

</details>

## Introduction

In this demo, we configure the WordPress container within the container instance created in the previous session (named `wordpress-demo` in the `CTDOKE` compartment, using `CI.Standard.A1.Flex` with 4 OCPUs and 12 GB memory). We will:
- Configure the WordPress container with environment variables, resource limits, and security settings.
- Add a MySQL container to complete the WordPress setup.
- Ensure the containers communicate over `localhost` within the same networking namespace.

## Prerequisites
- **OCI Console Access**: Signed in with permissions to manage container instances in the `CTDOKE` compartment.
- **Container Instance Setup**: Basic details, placement, shape (`CI.Standard.A1.Flex`, 4 OCPUs, 12 GB memory), and networking (public subnet in `CI-Demo` VCN with public IP) configured as per the previous guide.
- **IAM Policies**:
  - `manage compute-container-family` in `CTDOKE`.
  - `use virtual-network-family` for networking.
  - `read repos` in tenancy for OCIR image pulls.
  - Optional: `read secret-bundles` for private OCIR repository authentication via OCI Vault.
- **Container Images**:
  - WordPress: `docker.io/library/wordpress:latest` (public, Docker Hub).
  - MySQL: `phx.ocir.io/<tenancy-namespace>/oci-repo-public/mysql:latest` (public OCIR repository from previous demos).

## Step-by-Step Instructions

### 1. Configuring the WordPress Container
In the **Container Configuration** section of the OCI Container Instance creation wizard:

#### Container Name
- **Name**: Set to `wordpress`.
- **Note**: The name does not need to be unique (the OCID is the unique identifier). Avoid including confidential information for security.

#### Image Selection
1. Click **Select Image**.
2. Choose **External Registry** for the WordPress image.
3. Specify:
   - **Registry Hostname**: `docker.io`
   - **Repository Name**: `library/wordpress`
   - **Tag**: Optional; defaults to `latest` (pulls `wordpress:latest`).
4. **Registry Credentials**:
   - Select **None** (default) since `wordpress:latest` is a public image on Docker Hub.
   - For private registries, options include:
     - **Basic Authentication**: Provide username and password.
     - **OCI Vault Secret**:
       - Create a secret in OCI Vault to store credentials securely.
       - Click **Create New Secret**, specify a name, description, master encryption key, and registry credentials (username/password).
       - Ensure IAM policies allow the container instance to read secrets (e.g., `Allow dynamic-group ContainerInstanceDynamicGroup to read secret-bundles in tenancy`).
     - For this demo, since the image is public, click **Cancel** on the Create Secret window and select **None**.
5. Click **Select Image** to confirm.

#### Environment Variables
Set environment variables to configure the WordPress container (based on [Docker Hub WordPress documentation](https://hub.docker.com/_/wordpress)):
1. Add variables:
   - **WORDPRESS_DB_HOST**: `127.0.0.1` (MySQL container runs on the same instance, accessible via localhost).
   - **WORDPRESS_DB_USER**: `wordpress`.
   - **WORDPRESS_DB_PASSWORD**: `wordpress`.
   - **WORDPRESS_DB_NAME**: `wordpress`.
2. Click **Add Another Variable** for each variable and enter the key-value pairs.
3. These variables enable WordPress to connect to the MySQL container.

#### Advanced Options
Click **Show Advanced Options** to configure additional settings.

##### Resources Tab
- **Default Behavior**: Containers can use all available resources (4 OCPUs, 12 GB memory) of the container instance.
- **Resource Throttling**:
  - Enable **Resource Throttling**.
  - Select **Percentage** and set to `50%` for both vCPUs and memory.
  - Result: The WordPress container is limited to 2 OCPUs and 6 GB memory, reserving resources for the MySQL container.
- **Purpose**: Prevents the WordPress container from consuming all resources and starving the MySQL container.

##### Startup Options Tab
- Options to override image defaults:
  - **Working Directory**: Set the container’s working directory.
  - **Command**: Specify the entrypoint command.
  - **Command Arguments**: Provide additional arguments.
- For this demo, leave these as default (no changes needed for WordPress).

##### Security Tab
- **Read-Only Root Filesystem**:
  - Option to enable to restrict write access to the root filesystem, enhancing security.
  - For this demo, leave unchecked (default).
- **Run as Non-Root User**:
  - Option to run the container without root privileges.
  - If enabled, specify **User ID** and **Group ID** (1–65535; 0 is reserved for root).
  - For this demo, leave unchecked to run as root (default User ID/Group ID = 0).
- **Linux Capabilities**:
  - Configure capabilities to add or drop.
  - Options:
    - **Add Capabilities**: Enter `ALL` to include all capabilities (except those dropped).
    - **Drop Capabilities**: Enter `ALL` to drop all default capabilities, retaining only those listed in Add Capabilities.
    - If both fields are blank, default capabilities are retained.
  - For this demo, leave both fields blank to use default capabilities.

#### Finalize WordPress Container
- With environment variables, resource throttling (50%), and default startup/security settings configured, the WordPress container is ready.

### 2. Adding the MySQL Container
1. Click **Add Container** to configure the second container.
2. **Container Name**: Set to `mysql`.
3. **Image Selection**:
   - Click **Select Image**.
   - Choose **OCI Container Registry**.
   - Specify the image path: `phx.ocir.io/<tenancy-namespace>/oci-repo-public/mysql:latest`.
     - Replace `<tenancy-namespace>` with your tenancy namespace (e.g., `intoraclerohit`).
     - Example: `phx.ocir.io/intoraclerohit/oci-repo-public/mysql:latest`.
   - **Registry Credentials**:
     - Since `oci-repo-public/mysql:latest` is a public repository (from previous demos), select **None**.
     - For private OCIR repositories, use **OCI Vault Secret** and ensure IAM policies allow access (e.g., `Allow dynamic-group ContainerInstanceDynamicGroup to read repos in tenancy`).
   - Click **Select Image**.
4. **Environment Variables** (based on [Docker Hub MySQL documentation](https://hub.docker.com/_/mysql)):
   - Add:
     - **MYSQL_ROOT_PASSWORD**: `wordpress` (matches WordPress configuration).
     - **MYSQL_DATABASE**: `wordpress` (creates the database for WordPress).
     - **MYSQL_USER**: `wordpress`.
     - **MYSQL_PASSWORD**: `wordpress`.
   - Click **Add Another Variable** for each variable.
5. **Advanced Options**:
   - **Resources**:
     - Enable **Resource Throttling**.
     - Set to `50%` for vCPUs and memory (2 OCPUs, 6 GB memory).
   - **Startup Options**: Leave as default.
   - **Security**:
     - Leave read-only root filesystem and non-root user unchecked (run as root).
     - Leave Linux capabilities blank (use default capabilities).
6. **Finalize**: The MySQL container is now configured to work with the WordPress container.

### 3. Container Instance Resource Sharing
- **Compute Resources**: Both containers share the container instance’s 4 OCPUs and 12 GB memory. Resource throttling ensures each container uses up to 2 OCPUs and 6 GB memory.
- **Networking Namespace**: Containers communicate over `localhost` or `127.0.0.1`, allowing WordPress to connect to MySQL using the `WORDPRESS_DB_HOST=127.0.0.1` variable.
- **Ephemeral Storage**: Ensure combined image sizes (WordPress + MySQL) are <7.5 GB to fit within the 15 GB ephemeral storage limit.

### 4. Completing the Setup
- Review all settings:
  - Container instance: `wordpress-demo`, `CI.Standard.A1.Flex`, 4 OCPUs, 12 GB memory, public subnet, public IP, `On Failure` restart policy, 30-second graceful shutdown.
  - WordPress container: `wordpress`, `docker.io/library/wordpress:latest`, environment variables, 50% resource limits, default startup/security.
  - MySQL container: `mysql`, `phx.ocir.io/<tenancy-namespace>/oci-repo-public/mysql:latest`, environment variables, 50% resource limits, default startup/security.
- Click **Create** to launch the container instance.
- Monitor the instance creation and container status via the **Container Instance Details** page in the OCI Console.

## Key Considerations
- **IAM Policies**: Ensure permissions for OCIR image pulls and Vault secrets (if using private repositories).
- **Networking**: The public IP allows access to the WordPress site (e.g., via HTTP on port 80). Configure NSGs to restrict traffic to HTTP/HTTPS.
- **Security**: Running as root is acceptable for this demo but consider enabling non-root user and read-only filesystem for production.
- **Monitoring**: Check CPU, memory, and ephemeral storage usage in the OCI Console to ensure performance and avoid storage issues.
- **Data Persistence**: Ephemeral storage is lost on instance deletion; for production, configure persistent storage (e.g., OCI Block Volume).

## Use Case for Alex
For Alex, who needs to test isolated web applications:
- **WordPress Site**: This setup provides a fully functional WordPress site with MySQL, accessible publicly.
- **Minimal Management**: OCI handles infrastructure, allowing Alex to focus on application testing.
- **Resource Optimization**: Throttling ensures balanced resource use between containers.
- **Security**: Public images simplify setup, but private repositories with Vault secrets can be used for secure production environments.

## Summary
This guide configured:
- A WordPress container from Docker Hub with environment variables for MySQL connectivity, resource throttling (50%), and default startup/security settings.
- A MySQL container from OCIR with matching environment variables and resource limits.
- Containers sharing the same networking namespace for `localhost` communication.

The setup leverages the container instance’s resources efficiently and ensures accessibility via a public IP. For detailed instructions, refer to the [OCI Container Instances Documentation](https://docs.oracle.com/en-us/iaas/Content/ContEng/Concepts/contengabout.htm).

Thank you for following along!