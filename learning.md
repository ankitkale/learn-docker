# Docker Tutorial
[Tutorial from which i learnt docker, click here ](https://youtu.be/exmSJpJvIPs?si=eMlwUn4OzLrIvlNA)

## Introduction to Docker

Docker is a platform that facilitates the building of containers.

A **container** consists of an application and its dependencies.

### Properties of Containers

- **Portable**: Can be shared across different machines.
- **Lightweight**: Minimal resource overhead.
- **Isolated Environments**: Allows creation of separate environments on the same machine using containers.

## Docker Images

A Docker image is an executable file containing commands to build a Docker container. It serves as a blueprint for creating containers (similar to the relationship between a class and an object in programming).

Docker images are shared, and containers are created from these images.

## Docker Commands

1. `docker pull IMAGE_NAME`  
   Pulls the image if it does not exist on the local machine.

2. `docker images`  
   Displays all images available on the system.

3. `docker run IMAGE_NAME`  
   Builds and runs a container using the specified image.

4. `docker run -it IMAGE_NAME`  
   Runs the container in interactive mode, providing access to its terminal.

5. `docker ps -a`  
   Shows all containers (running and stopped).

6. `docker ps`  
   Shows only active (running) containers.

7. `docker start CONT_NAME or CONT_ID`  
   Restarts an existing container.

8. `docker stop CONT_NAME or CONT_ID`  
   Stops a running container.

9. `docker rm CONT_ID/NAME`  
   Removes a container.

10. `docker rmi IMG_ID/NAME`  
    Removes a Docker image.

11. `docker pull IMAGE_NAME:VERSION`  
    Pulls a specific version of an image.

12. `docker run -d IMAGE_NAME`  
    Runs the container in detached mode (background).

13. `docker run --name CONT_NAME -d IMAGE_NAME`  
    Runs the container in detached mode with a custom name.

## Docker Image Layers

Docker images are composed of layers:

```
Container (mutable layer)
|
Layer 2 (immutable)
|
Layer 1 (immutable)
|
Base Layer (small-sized Linux-based image, immutable)
```

All layers are immutable except the container layer, which is mutable.

## Port Binding

By default, containers operate with their own isolated ports, separate from the host machine's ports.

To bind a host machine port to a container port, use port binding.

```bash
docker run -p HOST_PORT:CONT_PORT IMAGE_NAME
```

**Example**:
```bash
docker run -p 8080:3306 IMAGE_NAME
```

### Troubleshooting Commands

14. `docker logs CONT_ID`  
    Displays the logs of a container.

15. `docker exec -it CONT_ID /bin/bash` or `docker exec -it CONT_ID /bin/sh`  
    Provides access to the container's shell to inspect files, environment, etc.

## Docker vs. Virtual Machine

| Aspect              | Docker                                      | Virtual Machine (VM)                          |
|---------------------|---------------------------------------------|-----------------------------------------------|
| **Virtualization Level** | Uses the host machine's kernel; virtualizes only the application layer. | Uses its own kernel; virtualizes both the host OS and application layer. |
| **Layers**          | Application Layer<br>Host OS Kernel<br>Hardware | Application Layer<br>Guest OS<br>Hypervisor<br>Host OS Kernel<br>Hardware |
| **Performance**     | Lightweight due to shared kernel.           | Heavier due to full OS emulation.             |

**Disadvantages**: Docker was initially designed for Linux-based systems. Docker Desktop introduces a lightweight hypervisor layer that utilizes a minimal Linux distribution internally. In essence, Docker Desktop runs containers within a small Linux-based VM.

## Developing with Docker

### Docker Networks

Docker networks enable internal connections between containers on the same network.

16. `docker network ls`  
    Lists all Docker networks on the system.

17. `docker network create NetworkName`  
    Creates a new Docker network.

#### Example Practice Project (Node.js + HTML/CSS + MongoDB)

**Setup MongoDB**:
```bash
docker run -d \
  -p 27017:27017 \
  --name mongo \
  --network mongo-network \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=qwerty \
  mongo
```

**Setup Mongo-Express**:
```bash
docker run -d \
  -p 8081:8081 \
  --name mongo-express \
  --network mongo-network \
  -e ME_CONFIG_MONGODB_AUTH_USERNAME=admin \
  -e ME_CONFIG_MONGODB_AUTH_PASSWORD=qwerty \
  -e ME_CONFIG_MONGODB_URL="mongodb://admin:qwerty@mongo:27017" \
  mongo-express
```

Default credentials for Mongo-Express: Username - `admin`, Password - `qwerty`.

### Docker Compose

Docker Compose is a tool for defining and running multi-container Docker applications. It uses a YAML (Yet Another Markup Language) file to specify configurations.

The YAML file includes instructions for required containers, such as:

```
version: ""  # Optional

services:
  service_name:
    image: IMAGE_NAME
    ports:
      - "27017:27017"
    environment:
      - MONGO_INITDB_ROOT_PASSWORD=...
```

#### Docker Compose Commands

18. `docker compose -f fileName.yaml up -d`  
    Creates and runs containers defined in the YAML file in detached mode.

19. `docker compose -f fileName.yaml down`  
    Permanently deletes/removes the containers defined in the YAML file.

**Note**: For containers defined in the YAML file, Docker Compose automatically creates a dedicated network and runs the containers within it, eliminating the need for manual network creation.

**Important**: Docker does not provide data persistence by default. Restarting Docker or a container will result in loss of changes.

## Dockerizing an Application

The process for dockerizing an app (e.g., TestApp) is:

```
TestApp
|
Docker Image
|
Container
```

Jenkins can be used to automate the conversion of the application into a Docker image. A `Dockerfile` contains the instructions for building the image.

### Important Dockerfile Instructions

- **FROM base_image**: Specifies the base image (e.g., Node.js for a Node.js app) with required dependencies.
- **WORKDIR path**: Sets the working directory in the image where files are copied and commands are executed.
- **COPY host_path image_path**: Copies files from the host machine to the image.
- **RUN command**: Executes a command during the build process (multiple RUN commands are allowed).
- **CMD command**: Specifies the command to execute when the container starts (only one CMD is allowed).
- **EXPOSE port**: Exposes a port from the image.
- **ENV key=value**: Defines environment variables.

After creating the `Dockerfile`, build the image with:

20. `docker build -t APP_NAME:VERSION .`

```markdown
## Publishing a Docker Image to Docker Hub

1. Log in to Docker Hub and create a new public (or private) repository. Note the repository name (e.g., `username/my-app`).
2. Log in from the terminal:
   ```bash
   docker login
   ```
3. Build the image using the exact repository name and a tag:
   ```bash
   docker build -t username/my-app:latest .
   ```
4. Push the image to Docker Hub:
   ```bash
   docker push username/my-app:latest
   ```

## Volumes in Docker

Volumes provide persistent data storage for containers. Data stored in a volume exists outside the container lifecycle and can be shared among multiple containers.

### Volume Types & Usage

| Type              | Syntax Example                                    | Description                                                                 |
|-------------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| **Named Volume** (recommended) | `docker run -v VOL_NAME:/cont_dir IMAGE`         | Docker-managed volume with a custom name. Most commonly used.               |
| **Anonymous Volume**           | `docker run -v /cont_dir IMAGE`                  | Docker creates a temporary unnamed volume.                                  |
| **Bind Mount**                 | `docker run -v /host/path:/cont_dir IMAGE`       | Host filesystem directory is directly mounted; managed by the host, not Docker. |

### Volume Management Commands

```bash
docker volume ls          # List all volumes
docker volume create VOL_NAME   # Create a named volume
docker volume rm VOL_NAME       # Remove a specific volume
docker volume prune             # Remove all unused volumes
```

### Defining Volumes in Docker Compose (`docker-compose.yml`)

```yaml
services:
  my-service:
    image: my-image
    volumes:
      - my-data:/app/data          # Named volume
      - ./host-folder:/app/host    # Bind mount

volumes:
  my-data:                       # Declares the named volume
```

## Docker Networks (Detailed)

Docker networks control how containers communicate with each other and with the outside world.

| Network Type | Driver   | Key Characteristics                                                                 | Common Use Case                                      |
|--------------|----------|-------------------------------------------------------------------------------------|------------------------------------------------------|
| **Bridge**   | bridge   | Default network. Containers on the same bridge network can communicate via container names/IPs. Isolated from host network. | Multiple containers interacting on the same host (most common). |
| **Host**     | host     | Container uses the host’s network stack directly. No port mapping needed; container has no separate IP. | Performance-critical apps needing direct host network access. |
| **None**     | none     | Container has no network interface except loopback. Completely isolated.           | Tasks requiring full network isolation (e.g., security-sensitive processes). |

### Network Commands Recap

```bash
docker network ls                  # List networks
docker network create my-network   # Create a custom bridge network
docker run --network my-network ...   # Run container on specific network
```

**End of Docker Notes**
```