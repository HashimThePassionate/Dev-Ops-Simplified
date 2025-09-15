# **Managing OCIR: Image Management and Retention Policies**

This guide is the third part of a series on managing Oracle Cloud Infrastructure Registry (OCIR). It focuses on managing images within OCIR repositories, including listing images, viewing image details, deleting and undeleting images, unversioning images, and configuring retention policies to automate image deletion. The tasks are performed using the OCI Console and the OCI CLI in Oracle CloudShell.

<details>
<summary><strong>📚 Table of Contents</strong></summary>

- [**Managing OCIR: Image Management and Retention Policies**](#managing-ocir-image-management-and-retention-policies)
  - [Introduction](#introduction)
  - [Prerequisites](#prerequisites)
  - [Step-by-Step Instructions](#step-by-step-instructions)
    - [1. Listing Images in OCIR Repositories](#1-listing-images-in-ocir-repositories)
    - [2. Viewing Image Details](#2-viewing-image-details)
    - [3. Deleting an Image](#3-deleting-an-image)
    - [4. Undeleting an Image](#4-undeleting-an-image)
    - [5. Unversioning an Image](#5-unversioning-an-image)
    - [6. Managing Images with Retention Policies](#6-managing-images-with-retention-policies)
      - [6.1 Global Image Retention Policy](#61-global-image-retention-policy)
      - [6.2 Custom Image Retention Policy](#62-custom-image-retention-policy)
  - [Summary](#summary)

</details>

## Introduction

In this guide, we will cover the following tasks:
- Listing images in OCIR repositories.
- Viewing image details and associated metadata.
- Deleting an image from a repository.
- Undeleting a previously deleted image.
- Unversioning an image to remove its version identifier.
- Setting up global and custom image retention policies to manage image deletion.

We assume you have two repositories in the `OCIR-OKE-DEMO` compartment:
- `oci-repo-private/nginx` with the `nginx:latest` image.
- `oci-repo-public/mysql` with the `mysql:latest` image.

## Prerequisites
- **Permissions**: You need appropriate permissions to manage images and repositories. If you belong to the administrators group, you can perform all actions (e.g., deleting images, managing retention policies) across any repository in the tenancy.
- **OCI Console Access**: Access to the OCI Console to navigate to the Container Registry.
- **OCI CLI**: Available in Oracle CloudShell for tasks like undeleting images.
- **Auth Token**: Required for CLI operations (if not already configured from previous demos).

## Step-by-Step Instructions

### 1. Listing Images in OCIR Repositories
1. Open the OCI Console and navigate to **Developer Services > Container Registry**.
2. Select the `OCIR-OKE-DEMO` compartment. You should see two repositories listed: `oci-repo-private/nginx` and `oci-repo-public/mysql`, each containing one image.
3. To list images in a repository:
   - Click the dropdown menu next to a repository (e.g., `oci-repo-private/nginx`).
   - Select an image (e.g., `nginx:latest`) to view its details.

### 2. Viewing Image Details
1. Select the `nginx:latest` image from the `oci-repo-private/nginx` repository.
2. The **Information** tab displays:
   - Publisher of the image.
   - OCID (Oracle Cloud Identifier) of the image.
   - Image digest (SHA256).
   - Creation timestamp.
   - Total size of the image (in MB).
   - Pull count and last pulled timestamp.
3. The **Layers** tab shows the SHA256 message digest for each layer in the image.
4. The **Versions** tab displays the fully qualified path of the image (e.g., `phx.ocir.io/<tenancy-namespace>/oci-repo-private/nginx:latest`) and its version identifier (`latest`). If multiple versions of the image are pushed, they will be listed here.
5. The **Tags** tab shows any assigned tags (none in this case, as no tags were assigned).
6. The **Signatures** and **Scan Results** tabs provide information on whether the image was signed or scanned for vulnerabilities. These topics will be covered in a future demo.
7. The **Copy Pull Command** button provides the command to pull the image to a Docker client (e.g., `docker pull phx.ocir.io/<tenancy-namespace>/oci-repo-private/nginx:latest`).

### 3. Deleting an Image
To delete an image from a repository:
1. Select the `mysql:latest` image from the `oci-repo-public/mysql` repository.
2. Copy the **Image OCID** (found in the **Information** tab) for use in the undelete operation later.
3. Click the **Delete Image** button and confirm the deletion.
4. After deletion:
   - The image count in the repository decreases by 1.
   - The dropdown menu for `oci-repo-public/mysql` will show no images.
   - The repository may still show a size (e.g., 158.44 MB) because deleted images remain in storage for up to 48 hours, allowing for undeletion during this period.

### 4. Undeleting an Image
Deleted images can be restored within 48 hours using the OCI CLI:
1. Open CloudShell and clear the screen:
   ```
   clear
   ```
2. Run the OCI CLI command to restore the deleted image:
   ```
   oci artifacts container image restore --image-id <image-ocid>
   ```
   - Replace `<image-ocid>` with the OCID of the `mysql:latest` image copied earlier.
   - Example:
     ```
     oci artifacts container image restore --image-id ocid1.containerimage.oc1.phx.example
     ```
3. Press **Enter**. The command should complete successfully, restoring the image.
4. Return to the OCI Console, refresh the `OCIR-OKE-DEMO` compartment page, and verify that the `mysql:latest` image is restored in the `oci-repo-public/mysql` repository.
5. Note: Restored images lose their version identifier (e.g., `latest`) and are assigned `unknown` followed by the SHA256 digest as the version.

### 5. Unversioning an Image
Unversioning removes the version identifier from an image without deleting it, helping to clean up the repository’s image list:
1. Select the `nginx:latest` image from the `oci-repo-private/nginx` repository.
2. Go to the **Versions** tab and click **Remove Version**.
3. Confirm the action by clicking **Remove**.
4. After unversioning:
   - The version identifier changes to `unknown` (similar to restored images).
   - The image is only visible in the dropdown menu if the **Include Unversioned Images** option is checked.
5. To hide unversioned images:
   - Uncheck **Include Unversioned Images** in the repository view.
   - The dropdown will only show versioned images (none in this case, as both images are now unversioned).
6. Re-check **Include Unversioned Images** to view both versioned and unversioned images.

### 6. Managing Images with Retention Policies
Retention policies automate image deletion based on specified criteria. They can be global (applying to all repositories in a region) or custom (overriding global policies for specific repositories).

#### 6.1 Global Image Retention Policy
1. In the OCI Console, go to **Container Registry > Settings**.
2. Under **Global Image Retention Policy**, click **Edit Global Policy**.
3. By default, the policy retains all images (no automatic deletion).
4. Available deletion criteria:
   - **Delete images not pulled in X days**: Specify the number of days (e.g., images not pulled in 30 days are deleted).
   - **Delete images not versioned in X days**: Specify the number of days (e.g., unversioned images not versioned in 15 days are deleted).
   - **Exempt Versions**: Optionally specify versions to exempt from deletion (e.g., `latest` or `3.0.1`).
5. An hourly process checks images against the criteria and deletes those that match.
6. Note: The global policy applies to all repositories in the region unless overridden by a custom policy.

#### 6.2 Custom Image Retention Policy
1. In the **Settings** page, under **Image Retention Override Policies**, click **Add Override Policy**.
2. Provide a policy name (e.g., `custom-ocir-policy`).
3. Specify deletion criteria, e.g.:
   - Delete images not pulled in 2 days.
   - Delete images not versioned in 15 days.
   - Optionally, specify exempt versions (e.g., `latest`).
4. Click **Save Changes** to create the policy.
5. Edit the policy to associate repositories:
   - Click **Edit Override Policy**.
   - Under **Associated Repositories**, click **Add Another Repository**.
   - Select repositories (e.g., `oci-repo-private/nginx` and `oci-repo-public/mysql`).
   - Optionally, change compartments to select repositories from other compartments (e.g., `CTDOKE` compartment).
   - Example: Select a repository from the `CTDOKE` compartment and save changes.
6. Verify the policy in the **Settings** page. The selected repositories will follow the custom policy, overriding the global policy.
7. Note: New or updated retention policies may take several hours to be enforced due to the hourly process that checks images for deletion.

## Summary
In this guide, we successfully:
- Listed images in the `OCIR-OKE-DEMO` compartment and viewed details for the `nginx:latest` image.
- Deleted the `mysql:latest` image and noted the 48-hour window for undeletion.
- Restored the deleted `mysql:latest` image using the OCI CLI, observing that it was assigned an `unknown` version.
- Unversioned the `nginx:latest` image to clean up the repository’s image list.
- Configured global and custom image retention policies to automate image deletion based on specified criteria.

This concludes the three-part series on managing OCIR, covering repository creation, image management (push/pull), and advanced image management tasks like deletion, undeletion, unversioning, and retention policies. For further details, refer to the OCI Container Registry documentation.

Thank you for following along!