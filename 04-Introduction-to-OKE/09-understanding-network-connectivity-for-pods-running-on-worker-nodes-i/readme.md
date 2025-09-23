# **Understanding Networking Connectivity for Pods in Kubernetes**

<details>
<summary>📋 Table of Contents</summary>

- [**Understanding Networking Connectivity for Pods in Kubernetes**](#understanding-networking-connectivity-for-pods-in-kubernetes)
  - [Container Network Interface (CNI) Specification](#container-network-interface-cni-specification)
    - [Role of CNI Plugins in Kubernetes](#role-of-cni-plugins-in-kubernetes)
    - [Consistency in CNI Plugins](#consistency-in-cni-plugins)
    - [Managing CNI Plugin Versions](#managing-cni-plugin-versions)

</details>

</br>

In Kubernetes, pods rely on unique and routable IP addresses to communicate with each other, with the cluster's control plane node, with pods on other clusters, with other services such as storage services, and sometimes with the broader internet. This document explores the vital networking model in Kubernetes, focusing on the Container Network Interface (CNI) specification, a pivotal framework for network resource management.

## Container Network Interface (CNI) Specification

CNI is a versatile framework that encompasses a specification and libraries, allowing for the creation of various plugins designed for Linux container network setup. These plugins come in a variety of flavors tailored to meet diverse network configuration requirements.

### Role of CNI Plugins in Kubernetes

CNI plugins streamline the setup of network interfaces for pods running on worker nodes. They are responsible for:
- Assigning IP addresses to pods.
- Maintaining network connections.

Depending on the specific network type chosen, Kubernetes utilizes different CNI plugins to facilitate these processes. The choice of network type directly impacts the CNI plugin used. For example:
- **VCN-native pod networking**: Aligns with the OCI VCN-native pod networking CNI plugin.
- **Flannel overlay network**: Works with the Flannel CNI plugin.

### Consistency in CNI Plugins

It is critical to maintain consistency when using CNI plugins within a Kubernetes cluster. All node pools within a single cluster must utilize the same CNI plugin. When creating a cluster, the choice of CNI plugin is definitive and cannot be modified later, making it a significant decision.

### Managing CNI Plugin Versions

When a cluster is initiated, it employs the most recent version of the CNI plugin. You have the flexibility to choose between:
- Allowing Oracle to manage updates automatically.
- Manually selecting versions.

If you opt for manual updates, you are responsible for keeping the CNI plugin add-on up to date and aligned with your cluster's needs. The CNI plugin for pod networking is an essential cluster add-on, and the