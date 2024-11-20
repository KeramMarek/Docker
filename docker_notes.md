# Docker Notes

## Table of Contents

1. [Installing Docker](#installing-docker)
2. [Docker Container Run](#docker-container-run)
3. [Docker Container Manipulation](#docker-container-manipulation)
4. [Docker Logs, Inspect, and Stats](#docker-logs-inspect-and-stats)
5. [Docker Volumes](#docker-volumes)
6. [Docker Networks](#docker-networks)
7. [Docker Images](#docker-images)
8. [Dockerfile](#dockerfile)
9. [Docker Context](#docker-context)
10. [Docker Compose](#docker-compose)

---

## Installing Docker

Steps to install Docker:
```bash
sudo apt update                          # Update libraries
sudo apt install docker.io               # Download and install Docker
sudo usermod -aG docker $USER            # Allows running Docker without 'sudo' (restart required)
docker --version                         # Check the installed version
docker container run --rm hello-world    # Simple test container to verify Docker works
```

---

## Docker Container Run

Commands to run Docker containers:
```bash
docker container run
-it                                        # Interactive terminal
-d                                         # Run in background
-v {volume name}:/{image path}             # Links volume to the image
-v {host path}:/{image path}               # Bind host path to the image
-m                                         # Limits memory used by the container
--rm {container name}                      # Remove after stop
--name {container name}                    # Name the container
--user userID:groupID                      # Run container with local user privileges
--restart always                           # Restart always when stopped
--restart on-failure:INTEGER               # Restart on failure INTEGER times
--restart unless-stopped                   # Restart always unless stopped by the user
--network {network name}                   # Set the type of network
--env VARIABLE=VALUE                       # Set an environment variable
--env-file {.env file name}                # Set environment variables from a file
--publish {host port}:{container port}     # Publish on specific ports
--memory 100M                              # Limit memory to 100 MB
--cpus 2                                   # Limit to 2 CPUs
```

---

## Docker Container Manipulation

Commands to manipulate Docker containers:
```bash
docker container ls                                   # List running containers
docker container ls -all                              # List all containers, including stopped
docker container create {container name}              # Create a container
docker container stop {container name}                # Stop a container
docker container start {container name}               # Start a container
docker container attach {container name}              # Attach to a running container
docker container prune                                # Remove all non-running containers
docker container exec -it {container name} {command}  # Execute a command in a container
```

---

## Docker Logs, Inspect, and Stats

Commands to view container logs, inspect details, and monitor stats:
```bash
docker container logs {container name}             # Show container logs
docker container logs --follow {container name}    # Show live container logs

docker container inspect {container name} | jq --raw-output .[0].NetworkSettings.IPAddress    # Start inspection to view .NetworkSettings.IPAddress
docker container inspect {container name} | jq --raw-output .[0]                              # Start inspection to view full output

docker container stats                              # Show live container stats
```

---

## Docker Volumes

Commands to manage Docker volumes:
```bash
docker volume ls                     # List all volumes
docker volume prune                  # Delete unused volumes
docker volume create {volume name}   # Create a volume
docker volume rm {volume name}       # Remove a volume
```

---

## Docker Networks

Commands to manage Docker networks:
```bash
docker network create                                      # Create a new network
docker network disconnect {network name} {container name}  # Disconnect a container from a network
docker network connect {network name} {container name}     # Connect a container to a network
docker network rm {network name}                           # Remove a network
docker network prune                                       # Remove unused networks
```

---

## Docker Images

Commands to manage Docker images:
```bash
docker login -u {user}                                     # Log in to Docker Hub
docker image tag {image name} marvy936/{image name}:{tag}  # Tag an image
docker image push marvy936/{image name} --all-tags         # Push all image tags to Docker Hub
docker image pull marvy936/{image name}                    # Pull an image from Docker Hub
docker image ls                                            # List all images
docker image rm {image name}                               # Remove an image
docker image inspect {image name}                          # Inspect image details
docker image build --tag {image name} .                    # Build image from the current directory
docker image history {image name}                          # View all layers of an image
```

---

## Dockerfile

Key Dockerfile instructions:
- **FROM**: Specifies the base image for building your image.
- **LABEL**: Adds metadata to the image, like author, version, and description.
- **RUN**: Executes commands for installing packages or configuring the environment.
- **COPY**: Copies files or directories from the host system to the image.
- **ADD**: Similar to `COPY`, but also supports extracting tar files and downloading from URLs.
- **CMD**: Defines the default command to run when the container starts.
- **ENTRYPOINT**: Defines a command to execute every time the container starts (can be combined with `CMD`).
- **ENV**: Sets environment variables available inside the container.
- **EXPOSE**: Documents which ports the application listens on (does not expose them automatically).
- **VOLUME**: Creates a mount point for persistent data storage.
- **USER**: Specifies the user to execute commands as.
- **WORKDIR**: Sets the working directory for commands like `RUN`, `CMD`, `ENTRYPOINT`, `COPY`, and `ADD`.

---

## Docker Context

Commands to manage Docker contexts:
```bash
docker context ls                                        # List all contexts
docker context inspect {context name}                    # Inspect context details
docker context create {context name} \                   # Create a new context
    --description "{description}" \                      # Add a description
    --docker "host=ssh://{remote user}@{remote host}"    # Specify Docker endpoint
docker context use {context name}                        # Switch to the desired context
docker context rm {context name}                         # Remove a context
```

Additional commands for SSH keys:
```bash
ssh-copy-id -i ~/.ssh/id_rsa {user}@{remote host}         # Copy SSH key to remote host
```

---

## Docker Compose

Commands to manage Docker Compose setups:
```bash
docker compose up -d              # Start containers in detached mode
docker compose down               # Stop and remove containers
docker compose ls                 # List Docker Compose services
```

### Using Environment Files
- **`.env`**: Default file for variables, automatically loaded by Docker Compose.
- **Custom Variable Files**: Use `env_file` in your Docker Compose file to load variables from a different file.

---
