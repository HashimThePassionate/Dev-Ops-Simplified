
# **Kubernetes Components**

<details>
<summary>Table of Contents</summary>

- [**Kubernetes Components**](#kubernetes-components)
  - [Kubernetes Cluster Overview](#kubernetes-cluster-overview)
    - [Control Plane](#control-plane)
    - [Data Plane](#data-plane)
  - [Nodes](#nodes)
  - [Pods](#pods)
  - [Services](#services)
    - [Types of Services](#types-of-services)
  - [Ingress](#ingress)
  - [Deployments](#deployments)
  - [ConfigMaps](#configmaps)
  - [Secrets](#secrets)
  - [Volumes](#volumes)
  - [Stateful Applications and StatefulSets](#stateful-applications-and-statefulsets)
  - [Summary](#summary)

</details>

## Kubernetes Cluster Overview

A Kubernetes cluster consists of two main components:
- **Control Plane**: Manages the Kubernetes cluster.
- **Data Plane**: Hosts worker nodes, which can be virtual or physical machines.

### Control Plane
The control plane hosts components that manage the Kubernetes cluster, such as scheduling, scaling, and maintaining the desired state.

### Data Plane
The data plane consists of worker nodes that host **pods**, which run one or more **containers**. These containers are built from **Docker images** stored in an image registry (e.g., OCI container registry).

## Nodes
- A **node** is the smallest unit of computing hardware in Kubernetes.
- It encapsulates one or more applications as containers.
- Nodes are worker machines with a container runtime environment (e.g., Docker).
- Nodes host pods and manage their execution.

## Pods
- A **pod** is the basic building block in Kubernetes, encapsulating:
  - One or more containers
  - Storage resources
  - A unique network IP
- Pods represent a single instance of an application and run on nodes.
- Typically, a pod runs one main application container, but it can include tightly coupled helper or sidecar containers.
- Pods are **ephemeral**:
  - They can die and be recreated with a new private IP address.
  - Kubernetes scales pods (creates or deletes them) based on traffic demands.
- Pods communicate using their unique private IP addresses, but their ephemeral nature makes direct IP-based communication unreliable.

## Services
- **Services** provide a stable endpoint for communication with pods, solving the issue of changing pod IP addresses.
- Each service is assigned a **virtual IP address** that persists until explicitly destroyed.
- Requests to a service are redirected to the appropriate pods.
- The lifecycle of a service is independent of the pods it routes traffic to, ensuring stable communication.

### Types of Services
1. **ClusterIP** (default):
   - Exposes the service on an internal IP within the cluster.
   - Only reachable from within the cluster.
2. **NodePort**:
   - Exposes the service on the same port of each selected node using network address translation (NAT).
   - Accessible externally using the node’s IP and port (superset of ClusterIP).
3. **LoadBalancer**:
   - Creates an external load balancer (supported in clouds like OCI).
   - Assigns a fixed external IP to the service (superset of NodePort).

## Ingress
- **Ingress** is not a service type but an entry point that routes external traffic to multiple services within the cluster based on rules.
- Useful for exposing multiple services under a single IP address using the same Layer 7 protocol (e.g., HTTP).
- Example: For an e-commerce application with services like `Place Order` and `Cancel Order`:
  - `www.xyz.com/placeorder` routes to the Place Order service.
  - `www.xyz.com/cancelorder` routes to the Cancel Order service.
- Ingress enables secure communication using domain names and protocols.

## Deployments
- A **deployment** manages a set of identical pods, automating their creation, updates, and deletion.
- Defined in a YAML file, a deployment:
  - Creates and maintains a replica set of pods.
  - Updates pods and replica sets.
  - Supports rollback to previous versions.
  - Enables autoscaling.
  - Allows pausing or resuming deployments.

## ConfigMaps
- **ConfigMaps** store configuration data as key-value pairs, used by pods, deployments, or services.
- They allow external configuration of environment variables or URLs without rebuilding application images.
- Example: If a database URL changes, update the ConfigMap instead of rebuilding the application.

## Secrets
- **Secrets** store sensitive data (e.g., passwords, OAuth tokens, SSH keys) in an encrypted, base64-encoded format.
- Managed by the kubelet service in a temporary file system.
- Secrets provide flexibility in pod lifecycle management and reduce the risk of exposing sensitive data.
- Unlike ConfigMaps, Secrets are designed for secure credential storage.

## Volumes
- **Volumes** are directories accessible to containers in a pod, providing persistent storage.
- Data in volumes outlasts ephemeral containers, ensuring data persistence even if a container crashes or restarts.
- Volumes connect containers to persistent data stores, maintaining data integrity.

## Stateful Applications and StatefulSets
- **Stateful applications** (e.g., MySQL, Oracle, PostgreSQL) store and track data, unlike stateless applications (e.g., Node.js) that process new data per request.
- **StatefulSets** manage stateful applications by providing:
  - Unique network identifiers
  - Stable persistent storage
  - Ordered deployment, scaling, and rolling updates
- Example: A MySQL database in a Kubernetes cluster with multiple replicas:
  - Read requests are distributed across pods.
  - Write requests go to the primary pod, with data synchronized to other replicas.
- Deleting or scaling down a StatefulSet does not delete associated volumes, ensuring data safety.

## Summary
This lesson covered key Kubernetes components:
- **Nodes**: Worker machines hosting pods.
- **Pods**: Ephemeral units running containers.
- **Services**: Stable endpoints for pod communication.
- **Ingress**: Routes external traffic to services.
- **Deployments**: Manage sets of identical pods.
- **ConfigMaps**: Store configuration data.
- **Secrets**: Securely store sensitive data.
- **Volumes**: Provide persistent storage.
- **StatefulSets**: Manage stateful applications with stable storage and ordered operations.