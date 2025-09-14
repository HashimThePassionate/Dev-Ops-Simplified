# **Docker Commands Demo**

<details>
<summary>📋 Table of Contents</summary>

- [**Docker Commands Demo**](#docker-commands-demo)
  - [Introduction](#introduction)
  - [Setting Up Cloud Shell](#setting-up-cloud-shell)
  - [Verifying Docker Installation](#verifying-docker-installation)
  - [Docker Commands Demo](#docker-commands-demo-1)
    - [1. Running a Container: `docker run`](#1-running-a-container-docker-run)
    - [2. Listing Containers: `docker ps`](#2-listing-containers-docker-ps)
    - [3. Removing a Container: `docker rm`](#3-removing-a-container-docker-rm)
    - [4. Running a Container with Interactive Shell: `docker run -it`](#4-running-a-container-with-interactive-shell-docker-run--it)
    - [5. Running a Container in Detached Mode with Port Mapping: `docker run -d -p`](#5-running-a-container-in-detached-mode-with-port-mapping-docker-run--d--p)
    - [6. Stopping a Container: `docker stop`](#6-stopping-a-container-docker-stop)
    - [7. Starting a Container: `docker start`](#7-starting-a-container-docker-start)
    - [8. Restarting a Container: `docker restart`](#8-restarting-a-container-docker-restart)
    - [9. Viewing Port Mappings: `docker port`](#9-viewing-port-mappings-docker-port)
    - [10. Viewing Container Logs: `docker logs`](#10-viewing-container-logs-docker-logs)
    - [11. Inspecting a Container: `docker inspect`](#11-inspecting-a-container-docker-inspect)
    - [12. Viewing Top Processes: `docker top`](#12-viewing-top-processes-docker-top)
    - [13. Executing Commands in a Running Container: `docker exec`](#13-executing-commands-in-a-running-container-docker-exec)
  - [Conclusion](#conclusion)

</details>

## Introduction

To work with Docker commands, you need Docker installed on your local terminal. For Oracle Cloud Infrastructure (OCI) users, Cloud Shell provides a pre-configured environment with Docker installed. This guide demonstrates key Docker commands using Cloud Shell, showcasing their functionality and practical applications.

## Setting Up Cloud Shell

To access Cloud Shell in OCI:
1. Click the Cloud Shell icon in the top right corner of the OCI console.
2. This spins up a Cloud Shell machine and connects you to it.

## Verifying Docker Installation

To check the Docker version installed in Cloud Shell:
- **Command**: `docker -v`
- **Output Example**: `Docker version 19.03.11-ol`

## Docker Commands Demo

Below is a walkthrough of key Docker commands demonstrated in Cloud Shell.

### 1. Running a Container: `docker run`
- **Command**: `docker run hello-world`
- **Description**: This command checks if the `hello-world` image is available locally. If not, it pulls the image from Docker Hub (a public Docker registry) and runs it in a container.
- **Output**:
  - If the image is not found locally, Docker pulls it from the registry.
  - The container runs and outputs: `Hello from Docker!`
- **Verification**:
  - Use `docker images` to confirm the `hello-world` image was downloaded.
  - **Output Example**:
    ```
    REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
    hello-world   latest    bf756fb1ae65   5 months ago   13.3kB
    ```
  - The output shows the image name, tag, ID, creation date, and size.

### 2. Listing Containers: `docker ps`
- **Command**: `docker ps`
- **Description**: Lists all running containers.
- **Output**: If no containers are running, the output is empty.
- **Variation**: `docker ps -a`
  - Lists all containers (running and stopped).
  - **Output Example**: Shows a `hello-world` container in the `Exited` state with its container ID and status (e.g., `Exited (0) 1 minute ago`).

### 3. Removing a Container: `docker rm`
- **Command**: `docker rm [container_id]`
- **Description**: Deletes a specified container using its container ID or name.
- **Example**:
  - Copy the container ID from `docker ps -a` output (e.g., first column).
  - Run: `docker rm bf756fb1ae65`
  - **Output**: Returns the container ID upon successful deletion.
- **Verification**: Run `docker ps -a` to confirm the container is removed (output is empty).

### 4. Running a Container with Interactive Shell: `docker run -it`
- **Command**: `docker run -it busybox sh`
- **Description**:
  - Pulls the `busybox` image from Docker Hub if not available locally.
  - The `-it` flag runs the container in interactive mode, providing shell access.
- **Output**:
  - If the image is not found locally, Docker pulls it.
  - The container starts, and you get a shell prompt (e.g., `/ #`).
- **Example Commands in the Container**:
  - `pwd`: Displays the present working directory (e.g., `/`).
  - `ls`: Lists the container’s filesystem contents.
  - `exit`: Exits the shell and stops the container.
- **Verification**:
  - Run `docker ps`: No running containers (since `exit` stopped the container).
  - Run `docker ps -a`: Shows the `busybox` container in the `Exited` state with its container ID and name (auto-assigned by Docker).

### 5. Running a Container in Detached Mode with Port Mapping: `docker run -d -p`
- **Command**: `docker run -d -p 80:80 [image_name]`
- **Description**:
  - The `-d` flag runs the container in detached mode (in the background).
  - The `-p 80:80` flag maps port 80 of the host to port 80 of the container.
- **Example**:
  - Run: `docker run -d -p 80:80 nginx`
  - Docker pulls the `nginx` image if not available locally, containerizes it, and returns the container ID.
- **Verification**:
  - Run `docker ps`: Shows the running container with details like container ID, image (`nginx`), creation time, and port mapping (`0.0.0.0:80->80/tcp`).
  - Use `curl localhost:80` to test the connection to the container’s port 80.
    - **Output**: Displays the HTML content of the NGINX web server running in the container.
  - **Note**: Cloud Shell lacks a public IP, so `curl` is used to demonstrate connectivity.

### 6. Stopping a Container: `docker stop`
- **Command**: `docker stop [container_id]`
- **Description**: Stops a running container.
- **Example**:
  - Run: `docker stop [container_id]`
  - The container stops, and the command returns the container ID.
- **Verification**:
  - Run `curl localhost:80`: Returns `Connection refused` since the container is stopped.
  - Run `docker ps`: No running containers.

### 7. Starting a Container: `docker start`
- **Command**: `docker start [container_id]`
- **Description**: Starts a stopped container.
- **Example**:
  - Run: `docker start [container_id]`
  - The container restarts, and the command returns the container ID.
- **Verification**:
  - Run `curl localhost:80`: Confirms the connection to port 80 is restored.
  - Run `docker ps`: Shows the container as running.

### 8. Restarting a Container: `docker restart`
- **Command**: `docker restart [container_id]`
- **Description**: Restarts a running container.
- **Output**: Returns the container ID upon successful restart.

### 9. Viewing Port Mappings: `docker port`
- **Command**: `docker port [container_id]`
- **Description**: Displays the port mappings for a specified container.
- **Output Example**: `80/tcp -> 0.0.0.0:80`, indicating the container’s port 80 is mapped to the host’s port 80.

### 10. Viewing Container Logs: `docker logs`
- **Command**: `docker logs [container_id]`
- **Description**: Displays the logs of a container, useful for troubleshooting.
- **Output**: Shows log entries generated by the container (e.g., NGINX access logs).

### 11. Inspecting a Container: `docker inspect`
- **Command**: `docker inspect [container_id]`
- **Description**: Provides detailed information about a container, including:
  - State (e.g., running, stopped)
  - Image used
  - CPU and memory details
  - Storage attachments
  - Running commands
  - Network settings
- **Use Case**: Useful for debugging and understanding container configurations.

### 12. Viewing Top Processes: `docker top`
- **Command**: `docker top [container_id]`
- **Description**: Lists the top processes running in a container.
- **Output Example**: Shows processes like the NGINX server running in the container.

### 13. Executing Commands in a Running Container: `docker exec`
- **Command**: `docker exec -it [container_id] [command]`
- **Description**: Runs a command inside a running container.
- **Example**:
  - Run: `docker exec -it [container_id] sh`
  - Provides shell access to the running container.
- **Exiting the Shell**:
  - `exit`: Stops the container and exits the shell.
  - `Ctrl+D`: Exits the shell without stopping the container.
- **Verification**:
  - Run `docker ps` after `Ctrl+D`: Confirms the container is still running.

## Conclusion

This demo showcases essential Docker commands for managing containers, from running and stopping containers to inspecting their details and accessing their shells. Using Cloud Shell’s pre-configured Docker environment, these commands demonstrate the power and flexibility of containerization. By mastering these commands, developers can efficiently create, manage, and troubleshoot containers, paving the way for scalable and portable applications.