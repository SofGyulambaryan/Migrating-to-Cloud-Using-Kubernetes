---

copyright:
  years: 2023
lastupdated: "2023-06-03"

keywords: MigrationToCloud
content-type: tutorial
completion-time: 30m

---

# Migrating an app to Cloud

## Overview

Nordlys is migrating its application to a cloud infrastructure using Kubernetes. This document will provide short description on how to prepare a sample application, working environment and then deploy the image.  

## Objectives

- Create a Dockerfile for your app to build a Docker image.
- Deploy your app image into the Kubernetes cluster.

## Audience

Nordlys app developers.

## Prerequisites

- Access to working docker image registry.
- Access to a Kubernates cluster.
- Target your CLI to the cluster.
- Learn about Docker and Kubernetes terminology.

## Running a container with an app in a Kubernetes cluster

The goal is to run a container with our app on Kubernetes. Let's say our ExampleApp will accept requests on 8800 port.

1. Create a directory and subdirectories.

    ```sh
    mkdir quickstart_docker
    mkdir quickstart_docker/application
    mkdir quickstart_docker/docker
    mkdir quickstart_docker/docker/application
    ```

    ```sh
    quickstart_docker/ # directory for the entire project
        ├──application/ # app codes
        └── docker/ # docker related files
            └── application/  # Dockerfile for the app
    ```

2. Create `application.py` file in `quickstart_docker/application` directory with the following content:
    ```python
    import http.server
    import socketserver
    
    PORT = 8000
    
    Handler = http.server.SimpleHTTPRequestHandler
    
    httpd = socketserver.TCPServer(("", PORT), Handler)
    
    print("serving at port", PORT)
    httpd.serve_forever()
    ```

3. Create a file `Dockerfile` in `quickstart_docker/docker/application`.

    ```dockerfile
    # Use base image from the registry
    FROM python:3.5
    # Set the working directory to /app
    WORKDIR /app
    # Copy the 'application' directory contents into the container at /app
    COPY ./application /app

    # Make port 8000 available to the world outside this container
    EXPOSE 8000

    # Execute 'python /app/application.py' when container launches
    CMD ["python", "/app/application.py"]
    ```

4. Run the following command in a terminal to build the image.

    ```sh
    docker build . -f docker/application/Dockerfile -t exampleapp
    ```
    
    |Parameter            | Description      |
    |--------------------|------------------|
    | `.` | Working directory, the build context |
    | `-f docker/application/Dockerfile` | Relative path to the Dockerfile |
    | `-t exampleapp` | The target image tag |

    Details on how to build Docker images can be found here: 
    https://docs.docker.com/engine/reference/builder/

5. Then, we check the list of images;
    ```sh
    docker images
    ```

    ```sh
    REPOSITORY            TAG      IMAGE ID         CREATED         SIZE       
    exampleapp          latest   83wse0edc28a    2 seconds ago     153 MB   
    python               3.6     05sob8636w3f    6 weeks ago       153 MB         
   ```

6. Next, we will put the image into the registry.
