# **Managing OCIR: Pulling Images and Automatic Repository Creation**

This guide continues from a previous demonstration on pushing images to Oracle Cloud Infrastructure Registry (OCIR) repositories. In this part, we focus on pulling images from OCIR repositories and explore the feature of automatic repository creation when pushing to a non-existent repository. The steps are performed using the Docker CLI within Oracle CloudShell.

<details>
<summary>📋 Table of Contents</summary>

- [**Managing OCIR: Pulling Images and Automatic Repository Creation**](#managing-ocir-pulling-images-and-automatic-repository-creation)
  - [Introduction](#introduction)
  - [Prerequisites](#prerequisites)
  - [Step-by-Step Instructions](#step-by-step-instructions)
    - [1. Pulling an Image from a Public Repository](#1-pulling-an-image-from-a-public-repository)
    - [2. Pulling an Image from a Private Repository](#2-pulling-an-image-from-a-private-repository)
    - [3. Automatic Repository Creation on Push](#3-automatic-repository-creation-on-push)
  - [Summary](#summary)

</details>

## Introduction

In this guide, we will:

- Pull images from both public and private OCIR repositories.
- Demonstrate the effect of pulling from a private repository without authentication.
- Explore the OCIR setting that allows automatic creation of a private repository in the root compartment when pushing an image to a non-existent repository.

We assume you have already pushed the `nginx:latest` image to the private repository `oci-repo-private/nginx` and the `mysql:latest` image to the public repository `oci-repo-public/mysql`, as covered in the previous guide.

## Prerequisites

- **Permissions**: You need appropriate permissions to pull images from OCIR repositories. For private repositories, authentication is required.
- **Docker CLI**: Available in Oracle CloudShell.
- **Auth Token**: Required for pulling images from private repositories.
- **OCIR Setting**: The "Create repository on first push in root compartment" setting should be enabled to demonstrate automatic repository creation (if applicable).

## Step-by-Step Instructions

### 1. Pulling an Image from a Public Repository

1. Navigate to the OCIR repository page in the OCI Console.

2. Select the public repository `oci-repo-public/mysql` and click on the `mysql:latest` image.

3. Click the **Copy Pull Command** button to copy the command for pulling the image.

4. Open the CloudShell terminal and maximize it for better visibility.

5. Clear the local Docker repository to ensure no images are present:

   ```
   docker rmi $(docker images -q)
   ```

   This command deletes all images in the local repository. Confirm the deletion by running:

   ```
   docker images
   ```

   The output should show no images in the local repository.

6. Paste the copied pull command for the `mysql:latest` image:

   ```
   docker pull phx.ocir.io/<tenancy-namespace>/oci-repo-public/mysql:latest
   ```

   Replace `<tenancy-namespace>` with your tenancy namespace (e.g., `intoraclerohit`).

7. Run the command. Since the repository is public, no authentication is required, and the image will be pulled successfully. Anyone with the URL to the public repository can pull the image.

8. Verify the image is downloaded by running:

   ```
   docker images
   ```

   You should see the `mysql:latest` image in the local repository.

### 2. Pulling an Image from a Private Repository

1. Navigate to the private repository `oci-repo-private/nginx` in the OCI Console and select the `nginx:latest` image.

2. Copy the pull command for the `nginx:latest` image.

3. In the CloudShell terminal, attempt to pull the image without authentication to demonstrate access restrictions:

   - First, log out from the OCIR session:

     ```
     docker logout phx.ocir.io
     ```

     This removes login credentials, and you should see a message confirming logout.
   - Paste and run the pull command:

     ```
     docker pull phx.ocir.io/<tenancy-namespace>/oci-repo-private/nginx:latest
     ```

     This will fail with an error message: `anonymous users are only allowed read access on public repos`, as the `nginx:latest` image is in a private repository.

4. Log back into OCIR to authenticate:

   ```
   docker login phx.ocir.io
   ```

   - For the username, enter:

     ```
     <tenancy-namespace>/<username>
     ```

     Replace `<tenancy-namespace>` with your tenancy namespace and `<username>` with your OCI username. If your tenancy is federated with Oracle Identity Cloud Service, use:

     ```
     <tenancy-namespace>/oracleidentitycloudservice/<username>
     ```
   - For the password, paste the auth token generated previously (from the OCI Console under **My Profile &gt; Auth Tokens**). The password will not be visible when pasted.
   - Press **Enter**. You should see `Login Succeeded`.

5. Re-run the pull command for the `nginx:latest` image:

   ```
   docker pull phx.ocir.io/<tenancy-namespace>/oci-repo-private/nginx:latest
   ```

   This time, the pull should succeed.

6. Verify both images are in the local repository:

   ```
   docker images
   ```

   You should see both `mysql:latest` (from the public repository) and `nginx:latest` (from the private repository).

### 3. Automatic Repository Creation on Push

OCIR allows automatic creation of a private repository in the tenancy’s root compartment if the "Create repository on first push in root compartment" setting is enabled and you have sufficient permissions (e.g., belonging to the tenancy’s administrator group or having `REPOSITORY_MANAGE` permissions).

To demonstrate this:

1. In the OCI Console, go to **Settings** and confirm that the **Create repository on first push in root compartment** option is enabled.

2. In the CloudShell terminal, clear the screen:

   ```
   clear
   ```

3. Tag the existing `nginx:latest` image to a new, non-existent repository:

   ```
   docker tag <image-id> phx.ocir.io/<tenancy-namespace>/oci-repo-doesnotexist/nginx:latest
   ```

   - Replace `<image-id>` with the first four characters of the `nginx:latest` image ID (e.g., `bc64`), obtained from `docker images`.
   - Replace `<tenancy-namespace>` with your tenancy namespace.
   - Example command:

     ```
     docker tag bc64 phx.ocir.io/intoraclerohit/oci-repo-doesnotexist/nginx:latest
     ```

4. Verify the tagging by running:

   ```
   docker images
   ```

   You should see the new tagged image with the path `phx.ocir.io/<tenancy-namespace>/oci-repo-doesnotexist/nginx:latest`.

5. Push the tagged image to the non-existent repository:

   ```
   docker push phx.ocir.io/<tenancy-namespace>/oci-repo-doesnotexist/nginx:latest
   ```

   Example:

   ```
   docker push phx.ocir.io/intoraclerohit/oci-repo-doesnotexist/nginx:latest
   ```

6. During the push, OCIR may optimize storage by mounting layers from an existing repository (e.g., `oci-repo-private/nginx`) if the image shares identical layers. This is known as **layer mounting**, where the registry links to common layers to save storage space.

7. Return to the OCI Console, navigate to the **Container Registry**, and select the root compartment. Refresh the page to confirm that a new private repository `oci-repo-doesnotexist/nginx` has been created with the `nginx:latest` image.

8. Note: Automatic repository creation requires you to be in the tenancy’s administrator group or have `REPOSITORY_MANAGE` permissions on the tenancy. Without these permissions, you must push to an existing repository.

## Summary

In this guide, we successfully:

- Pulled the `mysql:latest` image from a public OCIR repository without authentication.
- Demonstrated the need for authentication when pulling the `nginx:latest` image from a private repository.
- Showed how to log in to OCIR to pull images from private repositories.
- Demonstrated automatic creation of a private repository (`oci-repo-doesnotexist/nginx`) in the root compartment when pushing an image to a non-existent repository, leveraging the OCIR setting and layer mounting for storage optimization.
