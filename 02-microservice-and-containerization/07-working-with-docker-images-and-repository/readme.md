# **Working with Docker Images and Docker Repository**

<details>
<summary>Table of Contents</summary>

- [**Working with Docker Images and Docker Repository**](#working-with-docker-images-and-docker-repository)
  - [Overview](#overview)
  - [Sample Python Application](#sample-python-application)
  - [creating python file](#creating-python-file)
  - [creating Dockerfile file](#creating-dockerfile-file)
  - [Dockerfile](#dockerfile)
    - [Explanation of Dockerfile Instructions:](#explanation-of-dockerfile-instructions)
  - [Building the Docker Image](#building-the-docker-image)
    - [Build Process:](#build-process)
  - [Running the Container](#running-the-container)
    - [Testing the Application](#testing-the-application)
  - [Managing Docker Repositories](#managing-docker-repositories)
    - [Pulling an Image from Docker Hub](#pulling-an-image-from-docker-hub)
    - [Pushing an Image to Docker Hub](#pushing-an-image-to-docker-hub)
    - [Deleting a Local Image](#deleting-a-local-image)
  - [Summary](#summary)

</details>

## Overview
This guide demonstrates how to work with Docker images and Docker repositories. It covers creating a Docker image from a Dockerfile, containerizing a Python application, and managing Docker images by pulling from and pushing to Docker Hub. The demo includes a sample Python Flask application and a corresponding Dockerfile.

## Sample Python Application
The following is a simple Flask application (`app.py`) that displays a "Hello, World" message along with the container's hostname.

## creating python file
```sh
cd ~
nano app.py
```

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

paste python code and hit ctrl + O and then enter and then x

This application:
- Uses Flask to create a web server.
- Responds to the root URL (`/`) with an HTML response containing the environment variable `NAME` (default: "world") and the container's hostname.
- Runs on port `4000`.

## creating Dockerfile file
```sh
cd ~
nano Dockerfile
```

## Dockerfile
The Dockerfile defines the environment for the Python application.

```dockerfile
# Use an official Python runtime as a parent image
FROM python:3.12-slim

WORKDIR /app
ADD . /app

RUN pip install --trusted-host pypi.python.org Flask

ENV NAME World

CMD ["python", "app.py"]
```

Paste Dockerfile and hit ctrl + O and then enter and then x


### Explanation of Dockerfile Instructions:
- **FROM python:3.12-slim**: Specifies the base image (a lightweight Python 3.12 image).
- **WORKDIR /app**: Sets the working directory inside the container to `/app`.
- **ADD . /app**: Copies all files from the current directory to the `/app` directory in the image.
- **RUN pip install ...**: Installs Flask as a dependency.
- **ENV NAME World**: Sets an environment variable `NAME` with the default value "World".
- **CMD ["python", "app.py"]**: Specifies the command to run the application when the container starts.

## Building the Docker Image
To build the Docker image from the Dockerfile, use the following command:

```bash
mkdir -p ~/flask-hello
mv app.py Dockerfile ~/flask-hello/
cd ~/flask-hello
docker build -t hashim-demo .
```

- `-t hashim-demo`: Tags the image with the name `hashim-demo`.
- `.`: Indicates the Dockerfile is in the current directory.

### Build Process:
1. Pulls the base `python:3.12-slim` image from Docker Hub.
2. Creates the `/app` working directory.
3. Copies the current directory's contents to `/app`.
4. Installs Flask using `pip`.
5. Sets the `NAME` environment variable.
6. Configures the `CMD` to run `app.py`.

To verify the image was created, list all images:

```bash
docker images
```

This will show the `hashim-demo` image with the tag `latest` and the base `python:3.12-slim` image.

## Running the Container
To containerize the application, use the `docker run` command:

```bash
docker run --name my-container -p 8080:4000 hashim-demo
```

- `--name my-container`: Names the container `my-container`.
- `-p 8080:4000`: Maps port `8080` on the host to port `4000` in the container.

To verify the container is running:

```bash
docker ps
```

### Testing the Application
Open new shell of oci and test the application, use `curl` to access the mapped port:

```bash
curl http://localhost:8080
```

This will output:
```
<h3>Hello World!</h3> <b>Hostname:</b> <container-hostname><br/>
```

The hostname corresponds to the container's ID or name, visible in the `docker ps` output.

## Managing Docker Repositories

### Pulling an Image from Docker Hub
To pull an image from Docker Hub:

```bash
docker pull <repository-name>/<image-name>
```

Example:
```bash
docker pull hashimdocker/my_website_nginx
```

This downloads the specified image to your local repository, visible with `docker images`.

### Pushing an Image to Docker Hub
1. **Create a Repository**:  
   On Docker Hub, create a new repository (e.g., `oci-demo`) with public or private visibility.

2. **Tag the Local Image**:  
   Tag the local image to match the Docker Hub repository:

   ```bash
   docker tag hashim-demo hashimdocker/oci-demo:v1
   ```

   This creates a new image tag `hashimdocker/oci-demo:v1`.

3. **Log in to Docker Hub**:

   ```bash
   docker login
   ```

   Enter your Docker Hub username and password.

4. **Push the Image**:

   ```bash
   docker push hashimdocker/oci-demo:v1
   ```

   This uploads the image to the `oci-demo` repository with the tag `v1`.

5. **Verify on Docker Hub**:  
   Refresh the Docker Hub repository page to confirm the image appears with the specified tag.

### Deleting a Local Image
To remove an image from your local repository:

```bash
docker rmi <image-name-or-id>
```

Example:
```bash
docker rmi hello-world
```

**Note**: If a container is using the image, stop and remove the container first:

```bash
docker stop <container-name>
docker rm <container-name>
```

Then, delete the image.

## Summary
In this demo, we:
- Created a Docker image for a Python Flask application using a Dockerfile.
- Built the image with `docker build`.
- Ran the container with `docker run` and tested the application.
- Demonstrated pulling an image from Docker Hub using `docker pull`.
- Pushed a local image to a Docker Hub repository using `docker tag`, `docker login`, and `docker push`.
- Showed how to delete a local image with `docker rmi`.

This guide provides a practical approach to working with Docker images and managing Docker Hub repositories.