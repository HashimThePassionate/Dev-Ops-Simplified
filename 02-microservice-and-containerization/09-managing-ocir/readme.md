# **Managing Oracle Cloud Infrastructure Registry (OCIR)**

<details>
<summary><strong>📋 Table of Contents</strong></summary>

- [**Managing Oracle Cloud Infrastructure Registry (OCIR)**](#managing-oracle-cloud-infrastructure-registry-ocir)
  - [Overview](#overview)
  - [Managing Repositories](#managing-repositories)
    - [Creating a Repository](#creating-a-repository)
    - [Deleting a Repository](#deleting-a-repository)
    - [Moving a Repository](#moving-a-repository)
  - [Managing Images](#managing-images)
    - [Viewing Images](#viewing-images)
    - [Pushing an Image](#pushing-an-image)
    - [Pulling an Image](#pulling-an-image)
    - [Deleting Images](#deleting-images)
    - [Untagging Images](#untagging-images)
    - [Image Retention Policies](#image-retention-policies)
  - [Managing Security](#managing-security)
    - [Access Control](#access-control)
    - [Authentication](#authentication)
    - [Vulnerability Scanning](#vulnerability-scanning)
    - [Pulling Images in Kubernetes](#pulling-images-in-kubernetes)
  - [Preparing for OCIR](#preparing-for-ocir)
  - [Summary](#summary)

</details>

## Overview
This guide explains how to manage **Oracle Cloud Infrastructure Registry (OCIR)** through three key aspects: managing repositories, managing images within repositories, and securing repositories and images. It provides detailed steps and best practices for working with OCIR to streamline containerized workflows.

## Managing Repositories

### Creating a Repository
- Create an empty repository in a specific compartment with a unique name across all compartments in the tenancy.
- Limit: Up to 500 repositories per region per tenancy, with a cumulative storage limit of 500 GB and up to 100,000 images per repository.
- Example repository name: `hashim/acme-web-app`.

### Deleting a Repository
- Delete unneeded repositories to free up storage.
- Note: Deletion may take up to 48 hours to take effect, and storage is released after this period.

### Moving a Repository
- Repositories can be moved between compartments to:
  - Change authorized users.
  - Adjust billing allocation.

## Managing Images

### Viewing Images
- View images in OCIR using:
  - The OCI Console.
  - The `docker images` command after logging into OCIR with an authentication token.

### Pushing an Image
1. Tag the local image with the fully qualified path to the OCIR repository:
   ```bash
   docker tag <source-image> <region-key>.ocir.io/<tenancy-namespace>/<repository-name>:<tag>
   ```
   Example:
   ```bash
   docker tag my-app iad.ocir.io/ansh81vru1zp/hashim/acme-web-app:4.6.3
   ```
2. Log in to OCIR:
   ```bash
   docker login <region-key>.ocir.io
   ```
   Use your OCI username and authentication token.
3. Push the image:
   ```bash
   docker push <region-key>.ocir.io/<tenancy-namespace>/<repository-name>:<tag>
   ```

### Pulling an Image
- Log in to OCIR using an authentication token, then pull the image:
  ```bash
  docker pull <region-key>.ocir.io/<tenancy-namespace>/<repository-name>:<tag>
  ```
  Example:
  ```bash
  docker pull iad.ocir.io/ansh81vru1zp/hashim/acme-web-app:4.6.3
  ```

### Deleting Images
- Delete unneeded or old images to clean up repository tags.
- Images can be undeleted within 48 hours of deletion; after that, they are permanently removed.

### Untagging Images
- Remove tags from images without deleting them (referred to as "untagging") to clean up the repository tag list.

### Image Retention Policies
- Set up retention policies to automatically delete images based on criteria, such as:
  - Images not pulled for a specified number of days.
  - Images not tagged for a specified number of days.
  - Images without specific Docker tags (exempt tags can be specified in a comma-separated list).
- **Global Retention Policy**: Applies to all repositories in a region unless overridden by a custom policy. Default: Retains all images.
- **Custom Retention Policy**: Applies to specific repositories, but only one custom policy can be active per repository at a time.
- Note: Global policies are region-specific. Set identical criteria for consistent deletion across regions.
- An hourly process checks and deletes images meeting the criteria.

## Managing Security

### Access Control
- Use OCI Identity and Access Management (IAM) policies to control repository access at the tenancy or compartment level.
- Example IAM policies:
  - **Inspect**: View repository metadata.
  - **Read**: Pull images.
  - **Use**: Push and pull images.
  - **Manage**: Full control over repositories.
- Assign users to groups with appropriate permissions or to the tenancy administrators group (which has default access).

### Authentication
- Users must have an OCI username and authentication token to push/pull images.
- Example login command:
  ```bash
  docker login iad.ocir.io
  ```

### Vulnerability Scanning
- OCIR integrates with the OCI Vulnerability Scanning Service to scan images for vulnerabilities listed in Common Vulnerabilities and Exposures (CVE) databases.
- Required policies:
  - Allow the vulnerability-scanning service to read repositories in the tenancy or compartment.
  - Allow the service to read compartments where repositories are managed.
- Example policy:
  ```plaintext
  Allow service vulnerability-scanning-service to read repos in tenancy
  Allow service vulnerability-scanning-service to read compartments in tenancy
  ```

### Pulling Images in Kubernetes
To deploy applications to a Kubernetes cluster using OCIR images:
1. Create a Docker registry secret with OCI credentials using `kubectl`:
   ```bash
   kubectl create secret docker-registry <secret-name> \
   --docker-server=<region-key>.ocir.io \
   --docker-username=<tenancy-namespace>/<oci-username> \
   --docker-password=<oci-auth-token> \
   --docker-email=<email>
   ```
2. Specify the image and secret in the application's manifest file:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: my-app
   spec:
     containers:
     - name: my-app-container
       image: iad.ocir.io/ansh81vru1zp/hashim/acme-web-app:4.6.3
     imagePullSecrets:
     - name: <secret-name>
   ```

## Preparing for OCIR
Before pushing/pulling images, ensure:
1. **Regional Subscription**: Your tenancy is subscribed to a region where OCIR is available (check Oracle documentation).
2. **Docker CLI Access**: Install the Docker CLI on your local machine.
3. **User Permissions**: Users must belong to a group with appropriate IAM permissions or the tenancy administrators group.
4. **Authentication Token**: Users must have an OCI username and authentication token.

## Summary
Managing OCIR involves creating and organizing repositories, handling images (pushing, pulling, deleting, and untagging), and securing access through IAM policies and vulnerability scanning. By leveraging retention policies and Kubernetes integration, OCIR provides a robust solution for containerized application workflows.