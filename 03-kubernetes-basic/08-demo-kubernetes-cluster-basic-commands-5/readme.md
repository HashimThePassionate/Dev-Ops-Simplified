# **Kubernetes Service Management Commands**

<details>
<summary>Table of Contents</summary>

- [**Kubernetes Service Management Commands**](#kubernetes-service-management-commands)
  - [Prerequisites](#prerequisites)
  - [Service Management Commands](#service-management-commands)
    - [1. `kubectl get service`](#1-kubectl-get-service)
    - [2. `kubectl expose deployment <name> --name=<service-name> --port=<port> --target-port=<target-port> --type=<type>`](#2-kubectl-expose-deployment-name---nameservice-name---portport---target-porttarget-port---typetype)
      - [a. Creating a ClusterIP Service](#a-creating-a-clusterip-service)
      - [b. Creating a NodePort Service](#b-creating-a-nodeport-service)
      - [c. Creating a LoadBalancer Service](#c-creating-a-loadbalancer-service)
  - [Running Commands in OCI Cloud Shell](#running-commands-in-oci-cloud-shell)
  - [Summary](#summary)

</details>

<br/>

This guide covers `kubectl` commands for managing services in a Kubernetes cluster, demonstrated using the OCI Cloud Shell with an Oracle Kubernetes Engine (OKE) cluster (e.g., OKE-DEMO-1). These commands focus on creating and accessing different types of services to manage internal and external traffic to pods.


## Prerequisites
- A configured Kubernetes cluster (e.g., OKE-DEMO-1 in OCI).
- Access to OCI Cloud Shell with `kubectl` pre-configured.
- Existing deployments (e.g., `my-deployment` with pods running `nginx:latest` exposing port 80, as created in previous lessons).
- Necessary security list rules configured in the OCI Virtual Cloud Network (VCN) to allow external traffic (e.g., for NodePort or LoadBalancer services).

## Service Management Commands

### 1. `kubectl get service`
- **Purpose**: Lists all services in the current namespace.
- **Command**:
  ```bash
  kubectl get service
  ```
- **Output**: Displays services, including:
  - The default `kubernetes` service (type: `ClusterIP`).
  - User-created services (e.g., `myapp-cip-service`, `myapp-np-service`, `myapp-lb-service`).
- **Example Output**:
  - Initially shows only the `kubernetes` service of type `ClusterIP`.
  - After creating additional services, lists all services with their types, cluster IPs, and ports.

### 2. `kubectl expose deployment <name> --name=<service-name> --port=<port> --target-port=<target-port> --type=<type>`
- **Purpose**: Creates a service to expose a deployment’s pods to internal or external traffic.
- **Details**: Supports different service types (`ClusterIP`, `NodePort`, `LoadBalancer`) to route traffic to pods.

#### a. Creating a ClusterIP Service
- **Purpose**: Exposes pods for internal cluster communication.
- **Command**:
  ```bash
  kubectl expose deployment my-deployment --name=myapp-cip-service --port=80 --target-port=80 --type=ClusterIP
  ```
- **Output**: `service/myapp-cip-service exposed`
- **Details**:
  - Creates a `ClusterIP` service named `myapp-cip-service` for the `my-deployment` deployment.
  - Routes traffic to pods on port 80 (as defined in the deployment’s container spec).
  - Accessible only within the cluster.
- **Verification**:
  - List services:
    ```bash
    kubectl get service
    ```
    - Output: Shows `myapp-cip-service` with a cluster IP and port 80.
  - Access the service from within a pod:
    - Get pod names:
      ```bash
      kubectl get pods
      ```
    - Access a pod’s shell (e.g., pod ending with `zc76d`):
      ```bash
      kubectl exec -it my-deployment-zc76d -- /bin/bash
      ```
    - Inside the pod, test the service:
      ```bash
      curl <cluster-ip>:80
      ```
      - Output: Returns the NGINX welcome page (`Welcome to nginx!`), confirming internal access to the `ClusterIP` service.
    - Exit the pod:
      ```bash
      exit
      ```

#### b. Creating a NodePort Service
- **Purpose**: Exposes pods to external traffic via a static port on each node’s IP.
- **Command**:
  ```bash
  kubectl expose deployment my-deployment --name=myapp-np-service --port=80 --target-port=80 --type=NodePort
  ```
- **Output**: `service/myapp-np-service exposed`
- **Details**:
  - Creates a `NodePort` service named `myapp-np-service` for the `my-deployment` deployment.
  - Maps port 80 to a node port (e.g., 31614) for external access.
- **Verification**:
  - List services:
    ```bash
    kubectl get service
    ```
    - Output: Shows `myapp-np-service` with type `NodePort`, no external IP, and a node port (e.g., 31614).
  - Get node IPs:
    ```bash
    kubectl get nodes -o wide
    ```
    - Output: Lists external IPs of nodes.
  - Access the service externally:
    - Use the node IP and node port (e.g., `<node-ip>:31614`) in a browser.
    - Initially, access may fail due to security list restrictions in the OCI VCN.
  - Update OCI security list:
    - Navigate to the VCN in the OCI console, select the node security list, and add an ingress rule:
      - Source: `0.0.0.0/0`
      - Protocol: TCP
      - Destination Port: 31614
    - Save the rule.
  - Retry accessing `<node-ip>:31614` in the browser.
    - Output: Displays the NGINX welcome page, confirming external access via the `NodePort` service.

#### c. Creating a LoadBalancer Service
- **Purpose**: Exposes pods to external traffic via a cloud provider’s load balancer with a unique external IP.
- **Command**:
  ```bash
  kubectl expose deployment my-deployment --name=myapp-lb-service --port=80 --target-port=80 --type=LoadBalancer
  ```
- **Output**: `service/myapp-lb-service exposed`
- **Details**:
  - Creates a `LoadBalancer` service named `myapp-lb-service` for the `my-deployment` deployment.
  - Allocates a unique external IP from the cloud provider’s pool (e.g., OCI Load Balancer).
- **Verification**:
  - List services:
    ```bash
    kubectl get service
    ```
    - Output: Shows `myapp-lb-service` with type `LoadBalancer` and an external IP (initially `pending`).
  - Wait for the external IP to be assigned (may take a few moments).
  - Re-run:
    ```bash
    kubectl get service
    ```
    - Output: Displays the assigned external IP for `myapp-lb-service`.
  - Access the service:
    - Copy the external IP and open it in a browser (e.g., `http://<external-ip>`).
    - Output: Displays the NGINX welcome page, confirming external access.
  - Verify in OCI console:
    - Navigate to the Networking section and check the Load Balancers.
    - Confirm a new load balancer was created for `myapp-lb-service` with the assigned external IP.

## Running Commands in OCI Cloud Shell
1. **Access Cloud Shell**: Open the OCI Cloud Shell, pre-configured with `kubectl`.
2. **Execute Commands**:
   - Run the commands above to create and verify services.
   - Clear the screen (`clear`) between commands for readability.
3. **Troubleshooting**:
   - For `NodePort` access issues, ensure the OCI VCN security list allows traffic on the assigned node port (e.g., 31614).
   - For `LoadBalancer`, wait for the external IP to be assigned and verify the load balancer in the OCI console.
   - Use `kubectl get pods` and `kubectl exec` to test internal access to `ClusterIP` services.
4. **Verification**:
   - Use `kubectl get service` to list services and check their types, IPs, and ports.
   - Test external access with a browser or `curl` for `NodePort` and `LoadBalancer` services.
   - Test internal access from within a pod for `ClusterIP` services.

## Summary
These `kubectl` commands (`get service`, `expose`) enable you to manage services in a Kubernetes cluster. They allow listing services, creating `ClusterIP` services for internal communication, `NodePort` services for external access via node IPs, and `LoadBalancer` services for external access via a cloud provider’s load balancer. Demonstrated in the OCI Cloud Shell, these commands facilitate traffic routing to pods in an OKE cluster, with proper security list configuration for external access.