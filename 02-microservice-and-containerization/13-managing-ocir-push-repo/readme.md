# **Managing OCIR: Pushing Images to OCIR Repositories**

<details>
<summary>📋 Table of Contents</summary>

- [**Managing OCIR: Pushing Images to OCIR Repositories**](#managing-ocir-pushing-images-to-ocir-repositories)
  - [Introduction](#introduction)
  - [Prerequisites](#prerequisites)
  - [Step-by-Step Instructions](#step-by-step-instructions)
    - [1. Launching CloudShell](#1-launching-cloudshell)
    - [2. Creating an Auth Token](#2-creating-an-auth-token)
    - [3. Logging into OCIR](#3-logging-into-ocir)
    - [4. Checking for Existing Docker Images](#4-checking-for-existing-docker-images)
    - [5. Pulling Images from Docker Hub](#5-pulling-images-from-docker-hub)
    - [6. Tagging and Pushing the nginx Image to the Private Repository](#6-tagging-and-pushing-the-nginx-image-to-the-private-repository)
    - [7. Tagging and Pushing the MySQL Image to the Public Repository](#7-tagging-and-pushing-the-mysql-image-to-the-public-repository)
  - [Summary](#summary)

</details>

---

This guide is a detailed walkthrough based on a demonstration for managing Oracle Cloud Infrastructure Registry (OCIR) repositories. It covers the process of pushing images to OCIR repositories using the Docker CLI in Oracle CloudShell. Below is a step-by-step explanation of the tasks performed.

## Introduction
In this guide, we will perform tasks to push images to OCIR repositories. We assume you have already created two repositories from a previous setup:
- **oci-repo-private/nginx**: A private repository for nginx images.
- **oci-repo-public/mysql**: A public repository for MySQL images.

We will use the Docker CLI within Oracle CloudShell to push images to these repositories.

## Prerequisites
- **Permissions**: Your permissions determine which images you can push to the container registry. If you belong to the administrators group, you can push images to any repository in the tenancy.
- **Docker CLI**: Available in Oracle CloudShell, which we will use for this demonstration.
- **Auth Token**: Required for authenticating with OCIR.

## Step-by-Step Instructions

### 1. Launching CloudShell
1. Access the Oracle Cloud Infrastructure (OCI) Console.
2. Navigate to the OCIR repository page, where you can see the previously created repositories (`oci-repo-private/nginx` and `oci-repo-public/mysql`).
3. Launch CloudShell from the OCI Console. This may take some time to spin up.
4. Once CloudShell is ready, you will see a terminal prompt. Clear the screen for better visibility using the `clear` command.

### 2. Creating an Auth Token
To push images to OCIR, you need an authentication token. Follow these steps to create one:
1. Minimize the CloudShell window.
2. Click on the **Profile Menu** in the top-right corner of the OCI Console and select **My Profile**.
3. Under the **Resources** section, click on **Auth Tokens**.
4. If no tokens are listed, click **Generate Token**.
5. Provide a description for the token (e.g., `OCIR token`) and click **Generate Token**.
6. Copy the generated token and save it in a secure location (e.g., a notepad file) for later use. This token will act as the password for Docker login.
7. Click **Close** to return to the OCI Console.
8. Restore the CloudShell window.

### 3. Logging into OCIR
To authenticate with the OCIR, use the Docker CLI:
1. Run the following command in CloudShell:
   ```
   docker login <regionkey>.ocir.io
   ```
   Replace `<regionkey>` with the appropriate region key for your OCI region. For example:
   - For **India West Mumbai**, use `bom`. (i.e., `bom.ocir.io`)
   - To find the region key for your region, refer to the [OCI Container Registry documentation](https://docs.oracle.com/en-us/iaas/Content/Registry/Concepts/registryprerequisites.htm) for a list of region keys.

2. When prompted for a username, enter it in the format:
   ```
   <tenancy-namespace>/<username>
   ```
   - **Tenancy Namespace**: This is the auto-generated object storage namespace for your tenancy. To find it:
     - Go to the **Profile Menu** and select **Tenancy Information**.
     - Copy the **Object Storage Namespace** displayed.
   - If your tenancy is federated with Oracle Identity Cloud Service, use the format:
     ```
     <tenancy-namespace>/oracleidentitycloudservice/<username>
     ```

3. For the password, paste the auth token you copied earlier. Note that the password will not be visible on the screen when pasted.
4. Press **Enter**. If successful, you will see the message `Login Succeeded`.

### 4. Checking for Existing Docker Images
1. Maximize the CloudShell terminal for better visibility.
2. Run the following command to check for existing Docker images:
   ```
   docker images
   ```
   If no images are present, the output will be empty.

### 5. Pulling Images from Docker Hub
Since there are no images in the local repository, download the required images:
1. Pull the latest nginx image:
   ```
   docker pull nginx
   ```
   This will download the `nginx:latest` image from Docker Hub. Verify the download by running:
   ```
   docker images
   ```
   You should see the `nginx` image with the tag `latest` and a size of approximately 187 MB.

2. Pull the latest MySQL image:
   ```
   docker pull mysql
   ```
   Verify the download by running `docker images` again. You should see both `nginx:latest` and `mysql:latest` in the local repository.

### 6. Tagging and Pushing the nginx Image to the Private Repository
To push the nginx image to the `oci-repo-private/nginx` repository:
1. Minimize the CloudShell terminal and navigate to **Developer Services > Container Registry** in the OCI Console.
2. Select the `oci-repo-private/nginx` repository.
3. Return to CloudShell and tag the nginx image using the `docker tag` command:
   ```
   docker tag <image-id> <regionkey>.ocir.io/<tenancy-namespace>/oci-repo-private/nginx:latest
   ```
   - Replace `<image-id>` with the first four characters of the nginx image ID (e.g., `bc64`). You can find the image ID by running `docker images`.
   - Replace `<regionkey>` with your region key (e.g., `bom`).
   - Replace `<tenancy-namespace>` with your tenancy namespace (e.g., `intoraclerohit`).
   - Example command:
     ```
     docker tag bc64 bom.ocir.io/intoraclerohit/oci-repo-private/nginx:latest
     ```

4. Run `docker images` to verify the tagged image. You should see a new entry with the fully qualified path (`bom.ocir.io/<tenancy-namespace>/oci-repo-private/nginx:latest`) and the same image ID as the original `nginx:latest`.

5. Push the tagged image to the OCIR repository:
   ```
   docker push bom.ocir.io/<tenancy-namespace>/oci-repo-private/nginx:latest
   ```
   Example:
   ```
   docker push bom.ocir.io/intoraclerohit/oci-repo-private/nginx:latest
   ```

6. The push operation will display progress and return a SHA256 digest upon completion.
7. Return to the OCI Console, refresh the `oci-repo-private/nginx` repository page, and verify that the `latest` tagged image appears.

### 7. Tagging and Pushing the MySQL Image to the Public Repository
To push the MySQL image to the `oci-repo-public/mysql` repository:
1. Run `docker images` to confirm the presence of the `mysql:latest` image.
2. Tag the MySQL image using the `docker tag` command:
   ```
   docker tag <image-id> <regionkey>.ocir.io/<tenancy-namespace>/oci-repo-public/mysql:latest
   ```
   - Replace `<image-id>` with the first four characters of the MySQL image ID (e.g., `2d9a`).
   - Example command:
     ```
     docker tag 2d9a bom.ocir.io/abchaaha36a/oci-repo-public/mysql:latest
     ```

3. Run `docker images` to verify the tagged image. You should see a new entry with the fully qualified path (`bom.ocir.io/<tenancy-namespace>/oci-repo-public/mysql:latest`) and the same image ID as the original `mysql:latest`.

4. Push the tagged image to the OCIR repository:
   ```
   docker push bom.ocir.io/<tenancy-namespace>/oci-repo-public/mysql:latest
   ```
   Example:
   ```
   docker push bom.ocir.io/intoraclerohit/oci-repo-public/mysql:latest
   ```

5. The push operation will display progress and return a SHA256 digest upon completion.
6. Return to the OCI Console, refresh the `oci-repo-public/mysql` repository page, and verify that the `latest` tagged image appears.

## Summary
In this guide, we successfully:
- Created an auth token for OCIR authentication.
- Logged into OCIR using the Docker CLI in CloudShell.
- Pulled the `nginx:latest` and `mysql:latest` images from Docker Hub.
- Tagged and pushed the nginx image to the private `oci-repo-private/nginx` repository.
- Tagged and pushed the MySQL image to the public `oci-repo-public/mysql` repository.
- Verified the pushed images in the OCIR Console.

This concludes the process of pushing images to OCIR repositories. For further steps, such as pulling images from OCIR, refer to additional documentation or follow-up guides.