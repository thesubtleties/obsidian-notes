# Comprehensive Docker Reference Guide

## Container Management

```bash
# Run a container
docker run [OPTIONS] IMAGE [COMMAND]
docker run -d --name my-app -p 8080:80 nginx

# List containers
docker ps                  # Running containers
docker ps -a               # All containers
docker ps --size           # Show container disk usage

# Container lifecycle
docker start CONTAINER     # Start a stopped container
docker stop CONTAINER      # Stop a running container
docker restart CONTAINER   # Restart a container
docker rm CONTAINER        # Remove a container
docker rm -f CONTAINER     # Force remove a running container
docker rename OLD_NAME NEW_NAME  # Rename a container

# Execute commands in container
docker exec -it CONTAINER bash
docker exec -it CONTAINER sh
```

## Image Management

```bash
# List images
docker images
docker images -a           # Show all images (including intermediates)

# Pull images
docker pull IMAGE[:TAG]

# Build image from Dockerfile
docker build -t NAME:TAG .
docker build --no-cache -t NAME:TAG .  # Build without using cache

# Remove image
docker rmi IMAGE
docker rmi -f IMAGE        # Force remove image

# Tag images
docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]

# Save & load images (for transfer without registry)
docker save -o filename.tar IMAGE
docker load -i filename.tar
```

## Network Management

```bash
# List networks
docker network ls

# Create network
docker network create NETWORK
docker network create --driver overlay --attachable my-network  # Swarm mode
docker network create --subnet=192.168.0.0/16 my-network  # Custom subnet

# Connect/disconnect containers
docker network connect NETWORK CONTAINER
docker network disconnect NETWORK CONTAINER

# Inspect network
docker network inspect NETWORK

# Remove network
docker network rm NETWORK
```

## Volume Management

```bash
# List volumes
docker volume ls

# Create volume
docker volume create VOLUME

# Remove volume
docker volume rm VOLUME

# Inspect volume
docker volume inspect VOLUME

# Mount host directory
docker run -v /host/path:/container/path IMAGE
```

## Docker Compose

```bash
# Start services
docker-compose up -d
docker-compose -f custom-file.yml up -d  # Use custom file

# Stop services
docker-compose down
docker-compose down --volumes  # Remove volumes too
docker-compose down --rmi all  # Remove images too

# View logs
docker-compose logs -f
docker-compose logs -f SERVICE

# Scale service
docker-compose up -d --scale SERVICE=NUM

# Rebuild services
docker-compose up -d --build

# View running services
docker-compose ps
```

## System Cleanup (In-Depth)

```bash
# Display disk usage
docker system df           # Show space used by Docker
docker system df -v        # Detailed breakdown of space usage

# Remove dangling resources
docker image prune         # Remove dangling images
docker container prune     # Remove stopped containers
docker volume prune        # Remove unused volumes
docker network prune       # Remove unused networks

# Remove all unused resources
docker system prune        # Remove unused data
docker system prune -a     # Remove all unused images too
docker system prune --volumes -a  # Remove all unused data including volumes

# Remove build cache
docker builder prune       # Remove build cache
docker builder prune -a    # Remove all build cache

# Targeted cleanup
docker images -f "dangling=true" -q | xargs docker rmi  # Remove dangling images
docker ps -a -f status=exited -q | xargs docker rm      # Remove stopped containers

# Truncate container logs
truncate -s 0 $(docker inspect --format='{{.LogPath}}' CONTAINER_NAME)

# Limit log size (in container run command)
docker run --log-driver=json-file --log-opt max-size=10m --log-opt max-file=3 IMAGE
```

## Inspecting & Debugging

```bash
# View container logs
docker logs CONTAINER     # View container logs
docker logs -f CONTAINER  # Follow log output
docker logs --tail 100 CONTAINER  # Last 100 lines
docker logs --since 2h CONTAINER  # Logs from last 2 hours

# Inspect resources
docker inspect CONTAINER  # View container details
docker inspect IMAGE      # View image details
docker inspect VOLUME     # View volume details
docker inspect NETWORK    # View network details

# Resource usage
docker stats              # Live resource usage for all containers
docker stats CONTAINER    # Live resource usage for specific container
docker top CONTAINER      # Running processes in container

# Copy files between host and container
docker cp CONTAINER:SRC_PATH DEST_PATH  # Container to host
docker cp SRC_PATH CONTAINER:DEST_PATH  # Host to container

# Check container changes
docker diff CONTAINER     # Show changed files in container
```

## Docker Registry & Image Management

```bash
# Login to registry
docker login [SERVER]
docker logout [SERVER]

# Push/pull images
docker push IMAGE[:TAG]
docker pull IMAGE[:TAG]

# Search images
docker search TERM

# Image history
docker history IMAGE      # Show image layer history
```

## Resource Limits & Constraints

```bash
# Memory limits
docker run --memory=1g --memory-swap=2g IMAGE

# CPU limits
docker run --cpus=1.5 --cpu-shares=1024 IMAGE

# Storage limits
docker run --storage-opt size=10G IMAGE  # Requires specific storage driver
```

## Docker Context

```bash
# List contexts
docker context ls

# Create context
docker context create CONTEXT --docker "host=ssh://user@host"

# Use context
docker context use CONTEXT

# Inspect context
docker context inspect CONTEXT
```

## Docker Disk Management (Advanced)

```bash
# Find largest containers
docker ps -a --size | sort -k 7 -h

# Find largest images
docker images --format "{{.Size}}\t{{.Repository}}:{{.Tag}}" | sort -h

# Export container filesystem (for analysis)
docker export CONTAINER > container.tar

# Reclaim space in thin-provisioned storage
docker run --privileged --pid=host alpine nsenter -t 1 -m -u -n -i -- bash -c "echo 1 > /proc/sys/vm/drop_caches"

# WSL2 specific (Windows)
wsl --shutdown  # From PowerShell
optimize-vhd -Path "path\to\docker-desktop-data.vhdx" -Mode Full  # From Admin PowerShell
```

## Docker Swarm

```bash
# Initialize swarm
docker swarm init --advertise-addr IP

# Join swarm
docker swarm join --token TOKEN IP:PORT

# List nodes
docker node ls

# Deploy stack
docker stack deploy -c docker-compose.yml STACK

# List services
docker service ls

# Scale service
docker service scale SERVICE=NUM

# Update service
docker service update --image NEW_IMAGE SERVICE

# Remove stack
docker stack rm STACK
```

## Docker Health Checks

```bash
# Add health check to container
docker run --health-cmd="curl -f http://localhost/ || exit 1" \
           --health-interval=5s \
           --health-retries=3 \
           --health-timeout=2s \
           --health-start-period=15s \
           IMAGE

# Check container health
docker inspect --format='{{.State.Health.Status}}' CONTAINER
```

## Environment Variables & Secrets

```bash
# Environment variables
docker run -e VAR=value IMAGE
docker run --env-file=.env IMAGE

# Docker secrets (swarm mode)
docker secret create SECRET_NAME FILE
docker service create --secret SECRET_NAME IMAGE
```

