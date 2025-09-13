# **DevOps case Study 1**

<details>
<summary><strong>📑 Table of Contents</strong></summary>

- [**DevOps case Study 1**](#devops-case-study-1)
  - [**Introduction: Maryam and Hashim’s Conversation**](#introduction-maryam-and-hashims-conversation)
  - [**What is Cloud Native?**](#what-is-cloud-native)
  - [**Vision Stays’ Requirements**](#vision-stays-requirements)
  - [**Step-by-Step Plan**](#step-by-step-plan)
    - [**Step 1: Migrating the Static Website to OCI Compute Instance**](#step-1-migrating-the-static-website-to-oci-compute-instance)
    - [**Step 2: Adding an OCI Load Balancer**](#step-2-adding-an-oci-load-balancer)
    - [**Step 3: Setting Up IAM Policies**](#step-3-setting-up-iam-policies)
    - [**Step 4: Implementing Microservices Architecture**](#step-4-implementing-microservices-architecture)
    - [**Step 5: Using OCI Services**](#step-5-using-oci-services)
      - [**a. OKE (Oracle Kubernetes Engine)**](#a-oke-oracle-kubernetes-engine)
      - [**b. OCI Functions**](#b-oci-functions)
      - [**c. OCI Container Registry \& Artifact Registry**](#c-oci-container-registry--artifact-registry)
      - [**d. Oracle API Gateway**](#d-oracle-api-gateway)
      - [**e. Autonomous Database**](#e-autonomous-database)
      - [**f. Web Application Firewall (WAF)**](#f-web-application-firewall-waf)
      - [**g. DevOps Project \& Resource Manager**](#g-devops-project--resource-manager)
      - [**h. Service Mesh**](#h-service-mesh)
      - [**i. Core OCI Services**](#i-core-oci-services)
  - [**Summary**](#summary)

</details>

## **Introduction: Maryam and Hashim’s Conversation**
This lesson, presented by Mahendra Mehra, a senior OCI instructor, dives into Oracle Cloud Infrastructure (OCI) cloud native solutions through a case study of Vision Stays, a hotel chain. 

**The Story Begins:**
- **Maryam** (the client) talks to **Hashim** (an Oracle consultant) about her concerns regarding migrating her static website to the cloud and adding new features, specifically on OCI.
- Hashim reassures her that moving to OCI is highly beneficial and can enhance the website’s capabilities.
- Maryam knows a bit about the cloud native approach but is unclear on the details. Hashim offers to explain.

## **What is Cloud Native?**
Hashim explains that cloud native is a way of building and running applications that leverages the distributed computing capabilities of the cloud. In simple terms:

- **Cloud Native Apps** make full use of the cloud’s scale, elasticity (flexibility), resiliency (reliability), and flexibility.
- These apps are loosely-coupled systems, which are:
  - **Resilient**: If one part fails, the whole system doesn’t crash.
  - **Manageable**: Easy to maintain.
  - **Observable**: Easy to track performance and issues.
- Technologies enabling cloud native development include:
  - **Microservices and Containers**: Small, independent services.
  - **Kubernetes**: For managing containers.
  - **Service Mesh**: For communication between microservices.
  - **Serverless Functions and APIs**: Event-driven code execution.
  - **Kafka**: For data streaming.
  - **DevOps Practices and CI/CD Pipelines**: For fast, automated development and deployment.

OCI provides tools and services to streamline development, reduce operational tasks, and enable faster app deployment.

## **Vision Stays’ Requirements**
**Vision Stays** is a hotel chain with a static website (HTML, CSS, JavaScript, media files). They want to enhance it by:
- Adding a **Hotel Booking Facility** so customers can book rooms online.
- Enabling **Integration Capabilities** for third-party hotel aggregators (e.g., Booking.com) to check room availability and make bookings.
- Using OCI’s cloud native solutions to host and enhance the website.

## **Step-by-Step Plan**

### **Step 1: Migrating the Static Website to OCI Compute Instance**
- First, the static website’s files (HTML, CSS, JavaScript, media assets) will be moved to an **OCI Compute Instance** (a virtual machine).
- OCI’s compute capabilities ensure reliable performance and scalability, meaning the website can handle increased traffic efficiently.

### **Step 2: Adding an OCI Load Balancer**
- An **OCI Load Balancer** will be used as the website’s front end.
- It acts as a traffic distributor, intelligently distributing incoming requests across multiple backend VM instances hosting the website. Benefits include:
  - **Optimal Resource Use**: Efficiently utilizes resources.
  - **High Availability**: Ensures the website is always accessible.
  - **Responsiveness**: Provides fast response times.
- **Advanced Features**:
  - **SSL Termination**: Enables secure connections.
  - **Session Persistence**: Maintains user sessions.
  - **Content-Based Routing**: Directs requests to the right place.

### **Step 3: Setting Up IAM Policies**
- **IAM (Identity and Access Management) Policies** will be created to control access for teams deploying the website.
- These policies ensure:
  - Only authorized individuals or teams can access specific resources (e.g., VMs, databases).
  - **Data Confidentiality, Integrity, Availability**: Data stays secure, unchanged, and always accessible.
- This strengthens the environment’s security posture.

### **Step 4: Implementing Microservices Architecture**
- After deploying the static website, a **microservices architecture** will be implemented to add new features.
- **Monolithic vs. Microservices**:
  - Monolithic: Everything (code, database, features) is in one large chunk, making scaling and maintenance difficult.
  - Microservices: Small, independent services that improve scalability, agility, and maintainability.

### **Step 5: Using OCI Services**
The following OCI services will support the microservices architecture:

#### **a. OKE (Oracle Kubernetes Engine)**
- Some microservices will be deployed as **containers** on **OKE**, a Kubernetes-based service.
- OKE offers:
  - **Automated Provisioning**: Automatically sets up containers.
  - **Scaling**: Scales containers based on load.
  - **Orchestration**: Manages multiple containers to work together smoothly.

#### **b. OCI Functions**
- Some microservices will be implemented using **OCI Functions**.
- These are serverless functions that run code in response to events (e.g., a button click), offering a lightweight and scalable solution.

#### **c. OCI Container Registry & Artifact Registry**
- Container images and artifacts (e.g., code packages) will be stored in **OCI Container Registry** and **Artifact Registry**.
- These provide secure and scalable storage, making it easy to manage and distribute images/artifacts across development and deployment environments.

#### **d. Oracle API Gateway**
- The **Oracle API Gateway** will enable communication between microservices and external clients (e.g., browsers, apps).
- Features:
  - **Authentication/Authorization**: Only authorized users can access.
  - **Rate Limiting**: Controls API call volume to prevent overload.
  - **Traffic Routing**: Sends requests to the correct microservice.
- This ensures secure and seamless integration.

#### **e. Autonomous Database**
- Persistent data for microservices (e.g., booking details, user info) will be stored in **OCI Autonomous Database**.
- This fully managed database:
  - Automatically handles provisioning, scaling, and patching.
  - Reduces operational overhead.
  - Ensures high availability and performance.

#### **f. Web Application Firewall (WAF)**
- A **WAF** policy will be implemented to enhance website security.
- WAF protects against common web threats (e.g., SQL injection, cross-site scripting), safeguarding the website from vulnerabilities and breaches.

#### **g. DevOps Project & Resource Manager**
- Automation from development to deployment will use:
  - **DevOps Project**: Streamlines collaboration and automation across the software development lifecycle.
  - **Resource Manager**: Manages infrastructure via code and templates (Infrastructure as Code), ensuring consistent and repeatable deployments.

#### **h. Service Mesh**
- **OCI Service Mesh** will facilitate reliable communication between microservices.
- It improves:
  - **Reliability**: Prevents system crashes if one service fails.
  - **Observability**: Makes it easy to track performance and errors.

#### **i. Core OCI Services**
- Additional services include:
  - **Logging**: Tracks app performance and issues.
  - **Notifications**: Sends alerts and updates.
  - **OCI Vault**: Securely stores and manages sensitive data (e.g., passwords, API keys) with encryption.

## **Summary**
This case study shows how Vision Stays’ static website can be migrated and enhanced on OCI:
- **Compute Instance and Load Balancer** for reliable, scalable hosting.
- **IAM Policies** for security.
- **Microservices Architecture** for agility and new features (booking, third-party integration).
- **OCI Services** (OKE, Functions, API Gateway, Autonomous DB, etc.) for efficient management.
- **WAF** for protection.
- **DevOps Project and Resource Manager** for automation.
- **Service Mesh** for reliable communication.
- **Logging, Notifications, and Vault** for observability and secure data management.

