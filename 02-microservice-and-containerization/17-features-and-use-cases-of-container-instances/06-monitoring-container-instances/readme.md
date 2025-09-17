# **Administering and Monitoring OCI Container Instances**

<details>
<summary>Table of Contents</summary>

- [Administering and Monitoring OCI Container Instances](#administering-and-monitoring-oci-container-instances)
  - [Introduction](#introduction)
  - [Prerequisites](#prerequisites)
  - [Step-by-Step Instructions](#step-by-step-instructions)
    - [1. Accessing the Container Instance](#1-accessing-the-container-instance)
    - [2. Administrative Tasks](#2-administrative-tasks)
    - [3. Monitoring Health, Capacity, and Performance](#3-monitoring-health-capacity-and-performance)
    - [4. Viewing Container Details](#4-viewing-container-details)
    - [5. Viewing Container Logs](#5-viewing-container-logs)
    - [6. Container-Specific Metrics](#6-container-specific-metrics)
    - [7. Stopping the Container Instance](#7-stopping-the-container-instance)
    - [8. Key Consideration: Image Architecture Compatibility](#8-key-consideration-image-architecture-compatibility)
  - [Use Case for Alex](#use-case-for-alex)
  - [Summary](#summary)

</details>

## Introduction

OCI Container Instances provide a serverless platform for running containerized workloads, with built-in tools for administration and monitoring. This demo covers:
- Performing administrative tasks such as editing, stopping, restarting, and deleting container instances.
- Monitoring health, capacity, and performance using metrics.
- Viewing container details and logs for troubleshooting.

## Prerequisites
- **OCI Console Access**: Signed in with permissions to manage container instances in the `CTDOKE` compartment.
- **Container Instance**: The `wordpress-demo` instance from the previous demo, configured with:
  - Shape: `CI.Standard.A1.Flex`, 4 OCPUs, 12 GB memory.
  - Networking: Public subnet in `CI-Demo` VCN, public IP, port 80 open.
  - Containers:
    - WordPress: `docker.io/library/wordpress:latest`, 50% resources (2 OCPUs, 6 GB memory).
    - MySQL: `phx.ocir.io/<tenancy-namespace>/oci-repo-public/mysql:8.0.31`, 50% resources, `--default-authentication-plugin=mysql_native_password`.
- **IAM Policies**:
  - `manage compute-container-family` in `CTDOKE`.
  - `use virtual-network-family` for networking.
  - `read repos` in tenancy for OCIR image pulls.

## Step-by-Step Instructions

### 1. Accessing the Container Instance
1. Sign in to the OCI Console.
2. Click the **Hamburger Menu** (top-left corner).
3. Navigate to **Developer Services > Container Instances**.
4. Select the `CTDOKE` compartment.
5. Click on the `wordpress-demo` container instance from the list.

### 2. Administrative Tasks
The **Container Instance Details** page provides options for managing the instance:
- **Edit Container Instance**:
  - Update the instance name (e.g., change `wordpress-demo` to a new name).
- **Stop/Start/Restart**:
  - **Stop**: Stops the instance and all containers gracefully, based on the 30-second graceful shutdown timeout configured earlier. If containers exceed this timeout, they are forcibly terminated, and ephemeral storage data is lost.
  - **Start**: Starts a stopped instance.
  - **Restart**: Restarts the instance and its containers.
- **More Actions Dropdown**:
  - **Edit VNIC**: Modify the Virtual Network Interface Card (VNIC) settings, such as IP address or Network Security Groups (NSGs).
  - **Move Resource**: Move the container instance to another compartment.
  - **Add Tags**: Add free-form or defined tags for resource management.
  - **Delete**: Delete the container instance. Note: Ephemeral storage data is lost upon deletion.
- **Billing Note**:
  - Container instances use standard shapes, which pause billing when stopped.
  - Stopped or failed instances still count toward service limits.

### 3. Monitoring Health, Capacity, and Performance
Under the **Resources** section, select the **Metrics** tab to monitor the container instance:
- **Filter Options**:
  - Use **Start Time** and **End Time** to filter metrics by date range.
  - Use **Quick Select** options (e.g., last hour, last 24 hours) for predefined ranges.
- **Available Metrics**:
  - **CPU Utilization (%)**: Percentage of CPU resources in use.
  - **Memory Utilization (%)**: Percentage of memory resources in use.
  - **CPU Used**: Cumulative CPU usage since instance creation.
  - **Memory Used**: Cumulative memory usage since instance creation.
  - **Ephemeral Storage Used**: Amount of ephemeral storage in use (critical for ensuring images, logs, and data fit within the 15 GB limit).

### 4. Viewing Container Details
1. Under the **Resources** section, select the **Containers** link.
2. View the list of containers: `wordpress` and `mysql`.
3. Select the `mysql` container to view details:
   - **Edit Name**: Update the container name if needed.
   - **Entrypoint Arguments**: Displays the configured argument `--default-authentication-plugin=mysql_native_password`.
   - **Commands**: No custom commands were set (uses image defaults).
   - **Environment Variables**: Lists configured variables:
     - `MYSQL_ROOT_PASSWORD=wordpress`
     - `MYSQL_DATABASE=wordpress`
     - `MYSQL_USER=wordpress`
     - `MYSQL_PASSWORD=wordpress`
   - Click **Close** to return.

### 5. Viewing Container Logs
1. In the `mysql` container details, click **View Logs**.
2. A window displays the container’s logs, useful for troubleshooting issues.
3. Click **Refresh** to update logs periodically.
4. Click **Close** to exit the log view.
5. Repeat for the `wordpress` container if needed.

### 6. Container-Specific Metrics
1. Scroll down in the `mysql` container details to view metrics:
   - **Container CPU Used**: CPU usage specific to the MySQL container (limited to 2 OCPUs due to 50% throttling).
   - **Container Memory Used**: Memory usage (limited to 6 GB).
   - **Container Ephemeral Storage Used**: Storage usage for the container’s image, logs, and data.
2. Return to the **Container Instance Details** page.

### 7. Stopping the Container Instance
- Click the **Stop** button to stop the instance.
- **Behavior**:
  - Containers are stopped gracefully within the 30-second timeout.
  - If containers exceed the timeout, they are forcibly terminated, and ephemeral storage data is lost.
- **Billing**: Paused when stopped (standard shapes).
- **Service Limits**: Stopped instances still count toward limits.

### 8. Key Consideration: Image Architecture Compatibility
- **Architecture Mismatch**: Container instance creation may fail if the container image architecture (e.g., AMD64, ARM64) is incompatible with the chosen shape (`CI.Standard.A1.Flex` is ARM-based).
- **Verification**:
  - Ensure the WordPress (`docker.io/library/wordpress:latest`) and MySQL (`phx.ocir.io/<tenancy-namespace>/oci-repo-public/mysql:8.0.31`) images are compatible with ARM64.
  - Check image architecture in Docker Hub or OCIR documentation or by inspecting the image locally (e.g., `docker inspect`).
- **Best Practice**: Cross-check image architecture with the shape before deployment to ensure a smooth process.

## Use Case for Alex
For Alex, who needs to test isolated web applications:
- **Administration**: Easily manage the `wordpress-demo` instance by editing, stopping, or restarting it without server management.
- **Monitoring**: Use metrics (CPU, memory, storage) and logs to troubleshoot performance or issues with the WordPress site.
- **Compatibility**: Ensure image compatibility with the ARM-based `CI.Standard.A1.Flex` shape for reliable deployment.
- **Data Management**: Be aware that ephemeral storage data (e.g., WordPress configurations, MySQL data) is lost on deletion; use persistent storage for production.

## Summary
This guide covered:
- **Administration**: Editing, stopping, restarting, and deleting the `wordpress-demo` container instance, with options to modify VNIC, move compartments, and add tags.
- **Monitoring**: Viewing instance-level metrics (CPU, memory, ephemeral storage) and container-specific metrics/logs for the `wordpress` and `mysql` containers.
- **Key Consideration**: Ensuring image architecture compatibility with the chosen shape to avoid deployment failures.
- **Data and Billing**: Understanding the impact of stopping/deleting instances on ephemeral storage and billing.

These tasks enable efficient management and monitoring of containerized workloads. For detailed instructions, refer to the [OCI Container Instances Documentation](https://docs.oracle.com/en-us/iaas/Content/ContEng/Concepts/contengabout.htm).

Thank you for following along!