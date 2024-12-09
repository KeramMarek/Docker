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
11. [Dokcerfile to install Kubectl and Terraform example.](#dokcerfile-to-install-kubectl-and-terraform-example)



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
date                                       # You can run commands in docker container [docker container run {image} {command}]

```
Variables in container:
```bash
docker container inspect {container_name} | jq ".[0].Config.Env"
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

.[0] - needs to be always there 
     - the first dot represent the object being proccesed so docker inspect.
     - the [0] represents the first element of array, cause docker inspect return array.

Run full inspection with only .[0] and look which element you need and then use it like .[0].NetworkSettings.IPAddress .

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
docker image build --tag {image name} .                    # Build image from the current directory - needs Dockerfile
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

### Dockerfile Example

#### Multistage Build Example
```dockerfile
# Stage 1: Build the Python application
FROM python:3.9-slim AS builder

# Set meta data
LABEL maintainer="martin martin.vyhonsky@gmail.com"
LABEL description="Matrix screen saver in console"

# Set environment variables
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

# Create a working directory
WORKDIR /app

# Install build dependencies
RUN apt-get update &&     apt-get install -y --no-install-recommends gcc libpq-dev

# Install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir --upgrade pip &&     pip install --no-cache-dir -r requirements.txt

# Copy the application source code
COPY . .

# Stage 2: Create the final runtime image
FROM python:3.9-slim

# Set environment variables for production
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

# Create a user to run the application
RUN adduser --disabled-password appuser

# Set working directory
WORKDIR /app

# Copy only necessary files from the builder stage
COPY --from=builder /usr/local/lib/python3.9/site-packages /usr/local/lib/python3.9/site-packages
COPY --from=builder /usr/local/bin /usr/local/bin
COPY --from=builder /app /app

# Change ownership to the appuser
RUN chown -R appuser:appuser /app

# Switch to the appuser
USER appuser

# Expose the application port
EXPOSE 8000

# Run the application
CMD ["python", "app.py"]
```

**Before building:** Create `requirements.txt`:
```bash
vi requirements.txt
```
```plaintext
Flask==2.0.3
gunicorn==20.1.0
requests==2.26.0
psycopg2==2.9.3
```

Healthcheck.

```bash
FROM python:3.10-slim
ADD app.tgz /
RUN apt update \
&& apt install -y curl \
&& pip3 install uvicorn fastapi wheel jinja2 python-multipart
RUN adduser --no-create-home --disabled-password --gecos AppUser
appuser
EXPOSE 5000
WORKDIR /app
VOLUME [ "/app/data" ]
USER appuser
HEALTHCHECK --interval=1m --timeout=3s \
CMD curl -f http://localhost:5000/ || exit 1
CMD [ "/usr/bin/env", "python3", "/app/dashboard.py" ]
```

---

## Docker Context

Commands to manage Docker contexts:
```bash
docker context ls                                         # List all contexts
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
Check your environment files and directory structure before composing.

#### Example Compose File -> docker-compose.yml
```yaml
services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    env_file:
      - .env.db 
    networks:
      - my_network

  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    volumes:
      - wordpress_data:/var/www/html
    ports:
      - 80:80
    restart: always
    env_file:
      - .env.wordpress
    networks:
      - my_network

  adminer:
    depends_on:
      - db
      - wordpress
    image: adminer
    ports:
      - 8080:8080
    networks:
      - my_network

volumes:
  db_data:
  wordpress_data:

networks:
  my_network:
```

#### Environment Files

**.env.wordpress**
```plaintext
WORDPRESS_DB_HOST=db:3306
WORDPRESS_DB_USER=wp_user
WORDPRESS_DB_PASSWORD=wp_password
WORDPRESS_DB_NAME=wordpress_db
WORDPRESS_PORT=8000
```

**.env.db**
```plaintext
MYSQL_ROOT_PASSWORD=example_root_password
MYSQL_DATABASE=wordpress_db
MYSQL_USER=wp_user
MYSQL_PASSWORD=wp_password
```

---

## Comprehensive Docker Compose Example

Here’s an example `docker-compose.yml` that showcases most of the options you can use in a Compose file.

```yaml
version: '3.8'

services:
  app:
    image: my-app:latest
    build:
      context: ./app
      dockerfile: Dockerfile
      args:
        - APP_ENV=production   # Build arguments
    container_name: my_app_container
    restart: unless-stopped
    environment:
      - ENV_VAR1=value1        # Inline environment variable
    env_file:
      - app.env               # Load environment variables from file
    ports:
      - "8080:80"             # Map host port 8080 to container port 80
    volumes:
      - ./app_data:/data       # Mount a host directory
      - app_logs:/var/logs     # Named volume
    networks:
      - app_network
    depends_on:
      - db
    command: ["npm", "start"]
    deploy:
      replicas: 3              # Use in Swarm mode for service replication
      resources:
        limits:
          cpus: "0.5"
          memory: "512M"
        reservations:
          cpus: "0.25"
          memory: "256M"
    logging:
      driver: json-file        # Logging options
      options:
        max-size: "10m"
        max-file: "3"
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost/ || exit 1"]
      interval: 1m30s
      timeout: 10s
      retries: 3
      start_period: 40s

  db:
    image: postgres:latest
    container_name: postgres_db
    restart: always
    environment:
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mypassword
      POSTGRES_DB: mydatabase
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - app_network

  cache:
    image: redis:latest
    container_name: redis_cache
    restart: on-failure:5
    ports:
      - "6379:6379"
    networks:
      - app_network

volumes:
  app_logs:                     # Named volume for app logs
  db_data:                      # Named volume for database

networks:
  app_network:                  # User-defined network
    driver: bridge
```

### Explanation of Options

- **`services`**:
  - Defines the containers to be deployed.
  - Each service can have its own configurations like `image`, `build`, `environment`, and more.

- **`build`**:
  - Specifies build options for creating images from Dockerfiles.
  - Example:
    - `context`: The directory where the Dockerfile resides.
    - `dockerfile`: Path to the Dockerfile.
    - `args`: Build-time arguments.

- **`container_name`**:
  - Custom name for the container.

- **`restart`**:
  - Policies for restarting containers:
    - `no`: Never restart.
    - `always`: Always restart.
    - `on-failure`: Restart only on failure.

- **`environment`**:
  - Inline or file-based environment variables.

- **`ports`**:
  - Maps host ports to container ports.

- **`volumes`**:
  - Mount host directories or named volumes inside containers.

- **`networks`**:
  - Define custom Docker networks to allow communication between services.

- **`depends_on`**:
  - Specifies dependencies between services. Ensures a service starts only after its dependencies.

- **`deploy`**:
  - Used in Docker Swarm mode for scaling and resource reservations.

- **`logging`**:
  - Configures logging drivers and options.

- **`healthcheck`**:
  - Defines health checks for services to verify their availability.

---

# Dokcerfile to install Kubectl and Terraform example.
```Dockerfile
FROM alpine:3.16

LABEL org.opencontainers.image.authors="martin.vyhonsky"

ENV TERRAFORM_VERSION=1.10.1
ENV KUBECTL_VERSION=v1.27.1

RUN apk add --no-cache \
    openssh-client \
    ca-certificates \
    bash \
    wget \
    curl \
    unzip

RUN wget https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_amd64.zip -O terraform.zip \
    && unzip terraform.zip -d /usr/local/bin \
    && chmod +x /usr/local/bin/terraform \
    && rm terraform.zip

RUN curl -LO https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl \
    && install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl \
    && rm kubectl
```
