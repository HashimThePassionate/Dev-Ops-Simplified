# **Open Container Initiative (OCI) and Oracle Cloud Infrastructure Registry (OCIR)**

<details>
<summary><strong>📋 Table of Contents</strong></summary>

- [**Open Container Initiative (OCI) and Oracle Cloud Infrastructure Registry (OCIR)**](#open-container-initiative-oci-and-oracle-cloud-infrastructure-registry-ocir)
  - [Overview](#overview)
  - [What is OCI?](#what-is-oci)
  - [OCI Specifications](#oci-specifications)
    - [Image Specification](#image-specification)
    - [Runtime Specification](#runtime-specification)
  - [Popular Container Runtimes](#popular-container-runtimes)
  - [OCI Image Layout Specification](#oci-image-layout-specification)
    - [Layers in an OCI Image](#layers-in-an-oci-image)
  - [OCIR and OCI](#ocir-and-oci)
  - [High-Level Workflow](#high-level-workflow)
  - [Example: Creating, Pushing, and Running an Image with OCIR](#example-creating-pushing-and-running-an-image-with-ocir)
    - [Step 1: Create a Dockerfile](#step-1-create-a-dockerfile)
    - [Step 2: Create the Application](#step-2-create-the-application)
    - [Step 3: Build the Image](#step-3-build-the-image)
    - [Step 4: Log in to OCIR](#step-4-log-in-to-ocir)
    - [Step 5: Tag and Push the Image](#step-5-tag-and-push-the-image)
    - [Step 6: Pull and Run the Image](#step-6-pull-and-run-the-image)
    - [Step 7: View Image in OCIR](#step-7-view-image-in-ocir)
  - [Summary](#summary)

</details>

## Overview

This guide introduces the **Open Container Initiative (OCI)**, its specifications for container formats and runtimes, and its implementation in the **Oracle Cloud Infrastructure Registry (OCIR)**. It explains the OCI image and runtime specifications, popular container runtimes, and the OCI image layout. An example is provided to illustrate how to create, push, and run a container image using OCIR.

## What is OCI?

The **Open Container Initiative (OCI)** is an open governance structure established in June 2015 by Docker and other container industry leaders to create vendor-neutral, portable, and open standards for container formats and runtimes. OCI was formed to address the diverse technical and business needs of engineering teams, as Docker's container runtime alone couldn't meet all requirements.

OCI promotes an open-source technical community to define standards for:

- **Container Image Formats**: Ensuring compatibility across build tools.
- **Container Runtimes**: Allowing any OCI-compliant runtime to run images.

**Oracle** is a member of OCI, contributing to its goal of fostering portable container-based solutions.

## OCI Specifications

### Image Specification

The **OCI Image Specification** defines how to create an OCI-compliant container image, consisting of:

- **Image Manifest**: A configuration file listing layers and metadata for a specific architecture (e.g., ARM, AMD64) or operating system.
- **File System Serialization**: Describes how filesystem changes (e.g., file additions or deletions) are packaged into a "layer" (a blob).
- **Image Configuration**: A JSON file describing the image for use by container runtimes, detailing its relationship to filesystem layers.

### Runtime Specification

The **OCI Runtime Specification** outlines how to run an OCI image bundle as a container. Key tasks include:

- Container image management.
- Container lifecycle management (creation, start, stop).
- Container resource management.

An OCI implementation typically:

1. Downloads an OCI image.
2. Unpacks it into an OCI runtime filesystem bundle.
3. Runs the bundle using an OCI-compliant runtime.

**OCIR** is an implementation of the OCI runtime specification, enabling storage and management of OCI-compliant images.

## Popular Container Runtimes

- **containerd**: A CNCF project for complete container lifecycle management.
- **rkt**: An application container engine for cloud-native environments.
- **cri-o**: A lightweight Kubernetes container runtime interface for OCI-compliant runtimes.
- **runc**: A CLI tool for spawning and running containers per OCI specifications, initially developed by Docker.

Some runtimes (e.g., rkt) handle all tasks independently, while others (e.g., containerd) delegate low-level tasks to runtimes like runc.

## OCI Image Layout Specification

The **OCI Image Layout** organizes content-addressable blobs and location-addressable references in a slash-separated structure, supporting transport mechanisms like tar/zip archives, NFS, or HTTP/FTP. Key components include:

- **oci-layout File**: A JSON marker indicating the image layout version.
- **index.json**: The entry point for references and descriptors, acting as a multi-descriptor index.
- **Blobs Directory**: Organized by hash algorithm, containing actual content (e.g., layers).

To create an OCI runtime bundle:

1. Use the image layout and references to locate the manifest via `index.json`.
2. Apply filesystem layers in the specified order.
3. Convert the image configuration to a `config.json` for the OCI runtime.

### Layers in an OCI Image

An **OCI Image** consists of:

- **Image Manifest**: Metadata and layer references.
- **Image Configuration**: JSON configuration for the runtime.
- **Layers**: Read-only filesystem changes (e.g., file additions/deletions).
- **Container Layer**: A writable layer created during container runtime, capturing changes like file modifications. This layer is deleted when the container is removed.

Multiple containers can share the same read-only image layers, with each having its own writable container layer, improving distribution efficiency. Changed containers can be repackaged into new images by incorporating the writable layer.

## OCIR and OCI

**Oracle Cloud Infrastructure Registry (OCIR)** stores and manages OCI-compliant images, including Docker images, manifest lists, and Helm charts. In the OCIR dashboard, you can view detailed image information, including layers, by selecting an image.

## High-Level Workflow

1. **Create a Dockerfile**: Defines steps to build and run the image.
2. **Build the Image**: Use `docker build` to create an OCI-compliant image.
3. **Push to OCIR**: Store the image in OCIR.
4. **Run the Image**: Deploy the image as a container in runtimes like Oracle Container Engine for Kubernetes (OKE), Oracle Functions, or compute VMs.

## Example: Creating, Pushing, and Running an Image with OCIR

This example demonstrates building a simple Python Flask application, pushing it to OCIR, and running it as a container.

### Step 1: Create a Dockerfile

Create a file named `Dockerfile` for a Flask application.

```dockerfile
# Use an official Python runtime as a base image
FROM python:3.12-slim

# Set working directory
WORKDIR /app

# Copy application files
COPY . /app

# Install dependencies
RUN pip install --trusted-host pypi.python.org Flask

# Set environment variable
ENV NAME World

# Run the application
CMD ["python", "app.py"]
```

### Step 2: Create the Application

Create a file named `app.py`.

```python
from flask import Flask
import os
import socket

app = Flask(__name__)

@app.route("/")
def hello():
    html = "<h3>Hello {name}!</h3> <b>Hostname:</b> {hostname}<br/>"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname())

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=4000)
```

### Step 3: Build the Image

Build the Docker image locally:

```bash
docker build -t my-app:latest .
```

### Step 4: Log in to OCIR

Log in to OCIR using an authentication token:

```bash
docker login phx.ocir.io
```

- **Username**: `<tenancy-namespace>/<oci-username>` (e.g., `ansh81vru1zp/mahendra`).
- **Password**: OCI authentication token.
- **Region**: Replace `phx` with your region key (e.g., `iad` for US East Ashburn).

### Step 5: Tag and Push the Image

Tag the image for OCIR:

```bash
docker tag my-app:latest phx.ocir.io/ansh81vru1zp/oci-demo:v1
```

Push the image to OCIR:

```bash
docker push phx.ocir.io/ansh81vru1zp/oci-demo:v1
```

### Step 6: Pull and Run the Image

Pull the image from OCIR:

```bash
docker pull phx.ocir.io/ansh81vru1zp/oci-demo:v1
```

Run the container:

```bash
docker run -p 8080:4000 phx.ocir.io/ansh81vru1zp/oci-demo:v1
```

Test the application:

```bash
curl http://localhost:8080
```

Output:

```
<h3>Hello World!</h3> <b>Hostname:</b> <container-hostname><br/>
```

### Step 7: View Image in OCIR

- In the OCI Console, navigate to **Container Registry** &gt; Select the `oci-demo` repository.
- View the image (`v1`) and its layers under the **Layers** tab.

## Summary

The **Open Container Initiative (OCI)** standardizes container formats and runtimes, enabling interoperability across tools and runtimes like containerd, rkt, cri-o, and runc. **OCIR** implements these standards, providing a robust registry for managing OCI-compliant images. The example demonstrates building, pushing, and running a containerized Flask application, showcasing OCIR's role in the container lifecycle.