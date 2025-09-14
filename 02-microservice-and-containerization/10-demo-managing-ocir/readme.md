# **Managing Oracle Cloud Infrastructure Registry (OCIR): Demo Guide**

<details>
<summary><strong>📋 Table of Contents</strong></summary>

- [**Managing Oracle Cloud Infrastructure Registry (OCIR): Demo Guide**](#managing-oracle-cloud-infrastructure-registry-ocir-demo-guide)
  - [Overview](#overview)
  - [Prerequisites](#prerequisites)
  - [Step 1: Creating a Repository in OCIR](#step-1-creating-a-repository-in-ocir)
  - [Step 2: Generating an Authentication Token](#step-2-generating-an-authentication-token)
  - [Step 3: Logging into OCIR with Docker Client](#step-3-logging-into-ocir-with-docker-client)
  - [Step 4: Pushing an Image to OCIR](#step-4-pushing-an-image-to-ocir)
  - [Step 5: Pulling an Image from OCIR](#step-5-pulling-an-image-from-ocir)
  - [Step 6: Managing Images in OCIR Dashboard](#step-6-managing-images-in-ocir-dashboard)
  - [Summary](#summary)

</details>

## Overview
This guide demonstrates how to create and manage a container registry in **Oracle Cloud Infrastructure Registry (OCIR)**. It covers creating a repository, pushing and pulling images using a Docker client, browsing the OCIR dashboard, and managing images with retention policies. The demo assumes the user is part of the tenancy's Administrators group or has the necessary IAM policies.

## Prerequisites
- **OCI Account**: The user must belong to the Administrators group or have permissions to manage OCIR resources (e.g., `REPOSITORY_MANAGE`).
- **Docker CLI**: Installed on the local machine or Cloud Shell for Docker operations.
- **OCI Console Access**: For managing repositories and viewing the OCIR dashboard.
- **Authentication Token**: Generated in the OCI Console for Docker login.

## Step 1: Creating a Repository in OCIR

1. **Navigate to Container Registry**:
   - In the OCI Console, click the **Hamburger Menu** > **Developer Services** > **Containers & Artifacts** > **Container Registry**.
   - Select the desired compartment (e.g., `OCIR-OKE-DEMO`).

2. **Create a Repository**:
   - Click **Create Repository**.
   - Enter a unique repository name (e.g., `oci-demo`). Avoid confidential information in the name.
   - Choose **Access Type**:
     - **Public**: Allows anyone with the URL to pull images (requires Administrator group membership or `REPOSITORY_MANAGE` permission).
     - **Private**: Restricts access to authorized users (e.g., tenancy administrators).
   - Select **Private** (default) and click **Create Repository**.
   - The repository is created with a unique identifier and initially contains no images (size: 0 MB).

3. **Automatic Repository Creation** (Optional):
   - In the **Settings** menu, enable the option to automatically create private repositories in the tenancy's root compartment when running `docker push` without specifying an existing repository.
   - Disable this to push images only to existing repositories.

## Step 2: Generating an Authentication Token

1. **Access User Settings**:
   - In the OCI Console, click the **Profile Menu** > **User Settings** > **Auth Tokens** (under Resources).

2. **Create a Token**:
   - Click **Generate Token**.
   - Provide a description (e.g., `ocir-token`).
   - Click **Generate Token**, copy the token, and save it securely (it cannot be retrieved later).

## Step 3: Logging into OCIR with Docker Client

1. **Open Cloud Shell or Local Terminal**:
   - Use a terminal with Docker CLI installed (e.g., OCI Cloud Shell).

2. **Log in to OCIR**:
```bash
   oci os ns get --query data --raw-output # get Object storage namespace
  docker login bom.ocir.io -u "abcd1234xyz/username@gmail.com"
```
   - Replace `<region-key>` with the region code (e.g., `phx` for US West Phoenix; refer to Oracle documentation for region codes).
   - **Username**: `<tenancy-namespace>/<oci-username>` (e.g., `ansh81vru1zp/mahendra`).
   - **Password**: The authentication token generated in Step 2.
   - Example:
     ```bash
     docker login phx.ocir.io
     ```
     Output: `Login Succeeded`.

## Step 4: Pushing an Image to OCIR

1. **Check Local Images**:
   - List available images:
     ```bash
     docker images
     ```
   - Example image: `hashimdocker/oci-demo:v1`.

2. **Tag the Image**:
   - Create a new tag for the image with the fully qualified OCIR path:
     ```bash
     docker tag <source-image> <region-key>.ocir.io/<tenancy-namespace>/<repository-name>:<tag>
     ```
     Example:
     ```bash
     docker tag mahidocker2018/oci-demo:v1 phx.ocir.io/ansh81vru1zp/oci-demo:v1
     ```
   - Verify the new tag:
     ```bash
     docker images
     ```
     The new tag references the same image ID as the source image.

3. **Push the Image**:
   - Push the tagged image to OCIR:
     ```bash
     docker push <region-key>.ocir.io/<tenancy-namespace>/<repository-name>:<tag>
     ```
     Example:
     ```bash
     docker push phx.ocir.io/ansh81vru1zp/oci-demo:v1
     ```

4. **Verify in OCIR Dashboard**:
   - Return to the OCI Console > **Container Registry** > **oci-demo** repository.
   - Refresh the page to see the pushed image (e.g., `oci-demo:v1`, size: 52.75 MB).
   - Click the image to view details:
     - **Date Created**: Timestamp of the push.
     - **Repository**: `oci-demo`.
     - **Size**: Image size.
     - **Pushed By**: User who pushed the image.
     - **Layers**: View image layers under the **Layers** tab.
     - **Tags**: View associated tags (e.g., `v1`).
   - Use the **Image Filter** to toggle between tagged and untagged images.

## Step 5: Pulling an Image from OCIR

1. **Remove Local Images** (Optional):
   - Delete existing local images to test pulling:
     ```bash
     docker rmi -f <image-name-or-id>
     ```
     Example:
     ```bash
     docker rmi -f phx.ocir.io/ansh81vru1zp/oci-demo:v1
     ```
   - Verify removal:
     ```bash
     docker images
     ```

2. **Pull the Image**:
   - Pull the image from OCIR:
     ```bash
     docker pull <region-key>.ocir.io/<tenancy-namespace>/<repository-name>:<tag>
     ```
     Example:
     ```bash
     docker pull phx.ocir.io/ansh81vru1zp/oci-demo:v1
     ```
   - Verify the image:
     ```bash
     docker images
     ```

## Step 6: Managing Images in OCIR Dashboard

1. **Deleting an Image**:
   - In the OCIR dashboard, select the image (e.g., `oci-demo:v1`).
   - Click the **Actions** menu > **Delete Image**.
   - Confirm deletion. Note: Images can be undeleted within 48 hours; after that, they are permanently removed.

2. **Untagging an Image**:
   - To remove a tag without deleting the image:
     - Select the image > **Actions** > **Remove Tag**.
     - Confirm the action. This keeps the image but removes the tag.

3. **Setting Image Retention Policies**:
   - Navigate to **Container Registry** > **Settings** > **Image Retention Policies** (under Resources).
   - **Global Image Retention Policy**:
     - Default: Retains all images.
     - Edit to set criteria (e.g., delete images not pulled for X days or untagged for X days).
     - Specify exempt tags in a comma-separated list to prevent deletion.
   - **Custom Image Retention Policy**:
     - Click **Create Policy**.
     - Provide a name (e.g., `custom-ocir`).
     - Set criteria (e.g., delete images not pulled for 10 days).
     - Save and add repositories (e.g., `oci-demo`).
     - Only one custom policy can apply to a repository at a time.
   - An hourly process checks images against criteria and deletes those that match.

## Summary
This demo covered:
- Creating a private repository (`oci-demo`) in OCIR.
- Generating an authentication token for Docker login.
- Logging into OCIR and pushing a tagged image (`oci-demo:v1`) from a Docker client.
- Pulling the image back to the local machine.
- Browsing the OCIR dashboard to view image details, layers, and tags.
- Managing images by deleting or untagging them.
- Setting up global and custom image retention policies to automate image cleanup.

This guide provides a practical approach to managing OCIR repositories and images for containerized workflows.