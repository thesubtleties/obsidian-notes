### 1. Basic Commands
```bash
# Container Management
docker run [OPTIONS] IMAGE [COMMAND]
docker start/stop/restart CONTAINER
docker ps [-a]         # List containers ([-a] = all including stopped)
docker logs CONTAINER  # View logs
docker exec -it CONTAINER bash  # Enter container

# Image Management
docker pull IMAGE[:TAG]
docker build -t NAME:TAG .
docker images         # List images
docker rmi IMAGE     # Remove image
```

### 2. Docker Compose
```bash
# Basic Operations
docker-compose up -d            # Start services
docker-compose down            # Stop and remove
docker-compose logs -f         # Follow logs
docker-compose ps              # List services

# Build/Update
docker-compose build           # Build/rebuild services
docker-compose pull           # Pull service images
docker-compose up -d --build  # Build and start
```

### 3. Container Networking
```bash
# Network Management
docker network create NETWORK
docker network ls
docker network connect NETWORK CONTAINER
docker network disconnect NETWORK CONTAINER

# Network Debugging
docker network inspect NETWORK
docker container port CONTAINER
```

### 4. Volume Management
```bash
# Volume Operations
docker volume create VOLUME
docker volume ls
docker volume inspect VOLUME
docker volume rm VOLUME

# Mounting
-v /host/path:/container/path     # Bind mount
-v VOLUME:/container/path         # Named volume
--mount type=bind,source=,target= # Modern syntax
```

### 5. Docker Compose File Structure
```yaml
version: '3.8'
services:
  service_name:
    image: image_name
    build: 
      context: .
      dockerfile: Dockerfile
    environment:
      - KEY=value
    env_file: .env
    volumes:
      - ./local:/container
    ports:
      - "host:container"
    networks:
      - network_name
    depends_on:
      - other_service

networks:
  network_name:
    external: true

volumes:
  volume_name:
    external: true
```

### 6. Dockerfile Basics
```dockerfile
FROM base_image
WORKDIR /app
COPY . .
RUN command
ENV KEY=value
EXPOSE port
CMD ["command", "arg"]
```

### 7. Resource Management
```bash
# Resource Limits
--memory="1g"
--cpus="1.5"
--pids-limit=100

# In docker-compose.yml
services:
  service:
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
```

### 8. Security
```bash
# Security Options
--security-opt no-new-privileges
--read-only
--cap-drop ALL
--cap-add specific_capability

# User/Group
-u "user:group"
USER in Dockerfile
```

### 9. Health Checks
```dockerfile
# In Dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost/ || exit 1

# In docker-compose.yml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost"]
  interval: 30s
  timeout: 3s
  retries: 3
```

### 10. Logging
```bash
# View Logs
docker logs [OPTIONS] CONTAINER
  --tail=100
  --since=1h
  -f (follow)

# Logging Drivers
--log-driver json-file
--log-driver syslog
```

### 11. Environment Variables
```bash
# Command Line
-e KEY=value
--env-file=.env

# docker-compose.yml
environment:
  - KEY=value
env_file:
  - .env
```

### 12. Common Patterns
```bash
# Development vs Production
docker-compose.yml
docker-compose.override.yml
docker-compose.prod.yml

# Multi-stage Builds
FROM node AS builder
# build steps
FROM nginx
COPY --from=builder /app/dist /usr/share/nginx/html
```

### 13. Useful Commands
```bash
# Cleanup
docker system prune
docker container prune
docker volume prune
docker image prune

# Inspection
docker inspect CONTAINER
docker stats [CONTAINER]
docker top CONTAINER

# Image Management
docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
docker push IMAGE
docker save/load
```

### 14. Best Practices
- Use specific image tags
- Minimize layer count
- Use .dockerignore
- Implement health checks
- Set resource limits
- Use multi-stage builds
- Keep images small
- Use docker-compose for multi-container
- Regular cleanup
- Monitor resource usage

### 15. Debugging
```bash
# Debug Containers
docker stats
docker events
docker wait CONTAINER
docker diff CONTAINER

# Network Debugging
docker network inspect
docker port CONTAINER
```

Remember:
- Always use version control
- Document your configurations
- Keep security in mind
- Regular maintenance
- Monitor resource usage
- Use CI/CD pipelines
- Test in similar environments