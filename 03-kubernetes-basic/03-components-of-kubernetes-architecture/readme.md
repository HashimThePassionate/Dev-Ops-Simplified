# **Kubernetes Cluster Components and Features**

<details>
<summary>Table of Contents</summary>

- [**Kubernetes Cluster Components and Features**](#kubernetes-cluster-components-and-features)
  - [Kubernetes Cluster Structure](#kubernetes-cluster-structure)
  - [Control Plane Components](#control-plane-components)
    - [Kubernetes API Server](#kubernetes-api-server)
    - [etcd](#etcd)
    - [Kubernetes Scheduler (kube-scheduler)](#kubernetes-scheduler-kube-scheduler)
    - [Kubernetes Controller Manager](#kubernetes-controller-manager)
    - [Cloud Controller Manager](#cloud-controller-manager)
  - [Worker Node Components](#worker-node-components)
    - [Kubelet Service](#kubelet-service)
    - [Kube-Proxy](#kube-proxy)
    - [Container Runtime Engine](#container-runtime-engine)
  - [Key Kubernetes Features](#key-kubernetes-features)
    - [Health Checks](#health-checks)
    - [Networking](#networking)
    - [Service Discovery](#service-discovery)
    - [Load Balancing](#load-balancing)
    - [Logging](#logging)
    - [Rolling Updates](#rolling-updates)
    - [Automatic Bin Packing](#automatic-bin-packing)
    - [Container Replication](#container-replication)
    - [Container Autoscaling](#container-autoscaling)
    - [Volume Management](#volume-management)
    - [Resource Usage Monitoring](#resource-usage-monitoring)
  - [Summary](#summary)

</details>

## Kubernetes Cluster Structure
A Kubernetes system consists of multiple **clusters**, each with:
- One **master node** (part of the control plane) that handles scheduling, state changes, and updates.
- Multiple **worker nodes** that host application workloads.
- Clusters can be deployed in the **cloud** or **on-premises**.
- Nodes within a cluster are routinely replaced to maintain system health.

## Control Plane Components
The **control plane** is the nerve center of a Kubernetes cluster, managing its operations. It includes the following components:

### Kubernetes API Server
- Acts as the primary interface for communication between cluster components.
- Serves as the entry point for external requests via its **HTTP API**.
- Manages access, authentication, and operations like pod deployments using **kubectl** commands or **REST API** calls.

### etcd
- A distributed key-value store that holds all cluster data, including:
  - Metrics
  - Configuration
  - Metadata about pods, services, and deployments

### Kubernetes Scheduler (kube-scheduler)
- Assigns pods to nodes based on:
  - Compute resource requests and limits defined in the manifest.
  - Available node resources.
- Ensures workloads are placed on suitable nodes for optimal performance.

### Kubernetes Controller Manager
- A daemon that runs multiple controller functions to maintain the cluster’s desired state, including:
  - **Replication Controller**: Ensures the desired number of pod replicas are running.
  - **Endpoint Controller**: Manages service endpoints.
  - **Namespace Controller**: Handles namespace-related operations.
  - **Service Account Controller**: Manages service accounts.
  - **DaemonSet Controller**: Ensures DaemonSets run on specific nodes.
  - **Job Controller**: Manages batch jobs.
- Monitors objects via the API server, comparing their current state to the desired state and taking corrective actions if needed.

### Cloud Controller Manager
- Runs cloud-specific controller loops (introduced as an alpha feature in Kubernetes 1.6).
- Interacts with underlying cloud providers to manage cloud-specific resources.

## Worker Node Components
Worker nodes run the application workloads and include the following components:

### Kubelet Service
- Manages node-level pod operations.
- Receives pod specifications from the API server via HTTP requests.
- Ensures pods on the node are healthy and match their defined specifications.

### Kube-Proxy
- A networking proxy and load balancer.
- Facilitates communication between:
  - The node and the API server.
  - Pods within the cluster.
- Enables seamless pod-to-pod communication.

### Container Runtime Engine
- Manages container lifecycles on each node.
- Kubernetes supports **Open Container Initiative (OCI)**-compliant runtimes, such as:
  - Docker
  - CRI-O
  - rkt

## Key Kubernetes Features

### Health Checks
- **Readiness Probes**: Ensure an application is ready to serve traffic before a service routes requests to it.
- **Liveness Probes**: Check if an application is still running. If it fails, Kubernetes replaces the pod with a new one.

### Networking
- Facilitates container orchestration by:
  - Isolating independent containers.
  - Connecting coupled containers.
  - Providing external client access to containers.

### Service Discovery
- Enables containers to discover and connect to other containers within the cluster.

### Load Balancing
- Works with container replication, scaling, and service discovery.
- Provides a stable endpoint for clients to access replicated containers.

### Logging
- Monitors application behavior through logs generated by pods.
- Essential for troubleshooting and observability.

### Rolling Updates
- Updates containerized applications with minimal downtime.
- Typically involves providing new container images for deployment.

### Automatic Bin Packing
- Uses algorithms to place containers on nodes based on:
  - Current host load.
  - Co-location constraints.
  - Availability constraints.
- Ensures efficient resource utilization without compromising availability.

### Container Replication
- Maintains a specified number of pod replicas.
- Stops surplus replicas or starts new ones as needed.

### Container Autoscaling
- Automatically adjusts the number of running containers based on:
  - CPU utilization.
  - Custom application metrics.

### Volume Management
- Provides persistent storage for containers.
- Ensures data remains available even if containers are ephemeral or crash.
- Aligns with best practices for stateless containers with ephemeral files.

### Resource Usage Monitoring
- Tracks CPU and RAM usage for pods and containers.
- Critical for cost management, as higher resource usage increases costs.

## Summary
This section covered the structure and components of a Kubernetes cluster, including the **control plane** (API server, etcd, scheduler, controller manager, cloud controller manager) and **worker node** components (kubelet, kube-proxy, container runtime). Key features like health checks, networking, service discovery, load balancing, logging, rolling updates, bin packing, replication, autoscaling, volume management, and resource monitoring enable Kubernetes to efficiently manage containerized workloads at scale.