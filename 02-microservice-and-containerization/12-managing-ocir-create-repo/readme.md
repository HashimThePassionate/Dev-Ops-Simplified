# **Managing Oracle Cloud Infrastructure Registry (OCIR): Part 1 Demo**

<details>
<summary>📋 Table of Contents</summary>

- [**Managing Oracle Cloud Infrastructure Registry (OCIR): Part 1 Demo**](#managing-oracle-cloud-infrastructure-registry-ocir-part-1-demo)
  - [Overview](#overview)
  - [Prerequisites](#prerequisites)
  - [Managing OCIR Repositories](#managing-ocir-repositories)
    - [1. Creating a Repository](#1-creating-a-repository)
    - [2. Listing Repositories](#2-listing-repositories)
    - [3. Viewing Repository Details](#3-viewing-repository-details)
    - [4. Editing Repository Properties](#4-editing-repository-properties)
    - [5. Moving a Repository Between Compartments](#5-moving-a-repository-between-compartments)
    - [6. Deleting a Repository](#6-deleting-a-repository)
  - [Example: Managing Repositories in OCIR](#example-managing-repositories-in-ocir)
    - [Step 1: Create Repositories](#step-1-create-repositories)
    - [Step 2: List Repositories](#step-2-list-repositories)
    - [Step 3: View and Edit Repository Details](#step-3-view-and-edit-repository-details)
    - [Step 4: Move a Repository](#step-4-move-a-repository)
    - [Step 5: Delete a Repository](#step-5-delete-a-repository)
  - [Summary](#summary)

</details>

## Overview

This guide demonstrates how to manage repositories in **Oracle Cloud Infrastructure Registry (OCIR)**. It covers creating, listing, viewing, editing, moving, and deleting repositories. These operations are essential for organizing container images in OCIR. An example is included to illustrate these tasks in a practical scenario.

## Prerequisites

- **OCI Account**: The user must be part of the tenancy's Administrators group or have appropriate IAM policies (e.g., `REPOSITORY_MANAGE`) to manage OCIR resources.
- **OCI Console Access**: For managing repositories via the OCI Console.
- **Region and Compartment**: Ensure the correct region (e.g., US West Phoenix, `phx`) and compartment (e.g., `OCIR-OKE-DEMO`) are selected.

## Managing OCIR Repositories

### 1. Creating a Repository

1. **Navigate to Container Registry**:

   - In the OCI Console, go to **Navigation Menu** &gt; **Developer Services** &gt; **Containers & Artifacts** &gt; **Container Registry**.
   - Select the region (e.g., US West Phoenix) and compartment (e.g., `OCIR-OKE-DEMO`).

2. **Create a Repository**:

   - Click **Create Repository**.
   - **Name**: Provide a unique name across all compartments in the tenancy (e.g., `oci-repo-private/nginx`). Avoid confidential information.
   - **Access Type**:
     - **Private**: Only authorized users (e.g., tenancy administrators) can access. Default option.
     - **Public**: Anyone with the URL can pull images (requires Administrator group membership or `REPOSITORY_MANAGE` permission).
   - **Tags** (Optional): Add freeform or defined tags for organization.
   - Click **Create**.

3. **Notes**:

   - Repositories are limited to 500 per region per tenancy.
   - Public repositories require specific permissions to create.

### 2. Listing Repositories

- View repositories in the OCI Console under **Container Registry** &gt; **Repositories and Images** dropdown.
- Permissions determine which repositories you can list:
  - **Administrators Group**: Can list all repositories in the tenancy.
  - **Non-Administrators**: Limited by IAM policies defining accessible repositories.
- Example: In the `OCIR-OKE-DEMO` compartment, two repositories are listed:
  - `oci-repo-private/nginx` (Private)
  - `oci-repo-public/mysql` (Public)

### 3. Viewing Repository Details

- Select a repository (e.g., `oci-repo-private/nginx`) to view:

  - **Created By**: User who created the repository.
  - **Created Date**: Timestamp of creation.
  - **Access Type**: Private or Public.
  - **Unique Identifier**: Repository ID.
  - **Namespace**: Tenancy namespace (e.g., `ansh81vru1zp`).
  - **Total Size**: Size of container images (0 MB for empty repositories).
  - **Billable Size**: Storage usage in GB for billing.
  - **Last Push**: Timestamp of the last image push (N/A for empty repositories).
  - **Readme**: Customizable description of the repository.

- Edit the **Readme** section to add details:

  - Click **Edit Readme**, enter information (e.g., "This repository stores NGINX images for web applications"), and save.

### 4. Editing Repository Properties

- **Change Access Type**:
  - Select a repository (e.g., `oci-repo-public/mysql`).
  - Click **Change to Private** to make it private or **Change to Public** to revert.
  - Example: Switch `oci-repo-public/mysql` from public to private and back.
- **Add Scanner**:
  - Enable image scanning for vulnerabilities (covered in later demos).
- Requires appropriate permissions for public access changes.

### 5. Moving a Repository Between Compartments

1. Select the repository (e.g., `oci-repo-public/mysql`).
2. Click **Move Compartment**.
3. Choose a new compartment (e.g., `CTDOKE`) and click **Submit**.
4. Verify the move:
   - The repository is removed from the original compartment (`OCIR-OKE-DEMO`).
   - It appears in the new compartment (`CTDOKE`).
5. Move it back by repeating the process, selecting `OCIR-OKE-DEMO`.

### 6. Deleting a Repository

- **Why Delete**: Free up the repository limit (500 per region) and storage when no longer needed.
- **Process**:
  - Select a repository (e.g., `mysql` in the `CTDOKE` compartment with an image `mysql:8.0.31`).
  - Click **Delete Repository**.
  - Confirm deletion (this also deletes all images in the repository, e.g., `mysql:8.0.31`).
- **Note**: Deletion may take up to 48 hours to take effect, and storage is released afterward.

## Example: Managing Repositories in OCIR

This example demonstrates creating, editing, moving, and deleting a repository in OCIR.

### Step 1: Create Repositories

1. **Private Repository**:

   - Name: `oci-repo-private/nginx`
   - Access: Private
   - Compartment: `OCIR-OKE-DEMO`
   - Action: Click **Create Repository** in the OCI Console.
   - Result: Repository created with 0 images.

2. **Public Repository**:

   - Name: `oci-repo-public/mysql`
   - Access: Public
   - Compartment: `OCIR-OKE-DEMO`
   - Action: Click **Create Repository**.
   - Result: Repository created with 0 images.

### Step 2: List Repositories

- In the OCI Console, under **Container Registry** &gt; **Repositories and Images**:
  - See `oci-repo-private/nginx` (Private) and `oci-repo-public/mysql` (Public).

### Step 3: View and Edit Repository Details

1. Select `oci-repo-private/nginx`.
2. Add a Readme:

   ```markdown
   This repository stores NGINX images for web applications.
   ```
   - Click **Edit Readme** &gt; Save.
3. Change `oci-repo-public/mysql` to private:
   - Select the repository &gt; Click **Change to Private**.
   - Verify the access type updates to Private.

### Step 4: Move a Repository

- Move `oci-repo-public/mysql` to `CTDOKE`:
  - Select the repository &gt; **Move Compartment** &gt; Choose `CTDOKE` &gt; **Submit**.
  - Verify in `CTDOKE` under **Repositories and Images**.
- Move it back to `OCIR-OKE-DEMO`:
  - Repeat the process, selecting `OCIR-OKE-DEMO`.

### Step 5: Delete a Repository

- In the `CTDOKE` compartment, select a repository (e.g., `mysql` with image `mysql:8.0.31`).
- Click **Delete Repository** &gt; Confirm.
- Result: Repository and its image are deleted (may take up to 48 hours).

## Summary

This demo covered essential OCIR repository management tasks:

- **Creating**: Private (`oci-repo-private/nginx`) and public (`oci-repo-public/mysql`) repositories.
- **Listing**: Viewing repositories based on permissions.
- **Viewing Details**: Accessing metadata and adding Readme content.
- **Editing**: Changing access types (public to private or vice versa).
- **Moving**: Transferring repositories between compartments.
- **Deleting**: Removing repositories and their images.

These operations ensure efficient organization of container images in OCIR. Stay tuned for future demos on managing images and security.