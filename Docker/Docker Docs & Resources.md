## Official Documentation
- [Docker Docs](https://docs.docker.com/)
- [Docker Compose Docs](https://docs.docker.com/compose/)
- [Docker Hub](https://hub.docker.com/)
- [Docker Engine](https://docs.docker.com/engine/)
- [Docker Swarm](https://docs.docker.com/engine/swarm/)

## Video Resources
### YouTube Channels & Tutorials
- [Docker Tutorial for Beginners](https://www.youtube.com/watch?v=pTFZFxd4hOI) - Programming with Mosh
- [Docker Crash Course](https://www.youtube.com/watch?v=Wf2eSG3owoA) - Traversy Media
- [Docker Compose Tutorial](https://www.youtube.com/watch?v=HG6yIjZapSA) - TechWorld with Nana
- [Docker Deep Dive](https://www.youtube.com/watch?v=i7ABlHngi1Q) - Fireship
- [Docker Swarm Tutorial](https://www.youtube.com/watch?v=Tm0Q5zr3FL4) - TechWorld with Nana

## Docker Commands
### Basic Commands
```bash
# Images
docker pull [image]
docker build -t [name] .
docker images
docker rmi [image]

# Containers
docker run [options] [image]
docker ps
docker stop [container]
docker rm [container]
docker exec -it [container] [command]

# System
docker system prune
docker system df
```

### Docker Compose Commands
```bash
docker-compose up
docker-compose down
docker-compose build
docker-compose logs
docker-compose ps
docker-compose exec [service] [command]
```

## Dockerfile Reference
```dockerfile
# Common instructions
FROM
WORKDIR
COPY
ADD
RUN
ENV
EXPOSE
CMD
ENTRYPOINT
```

## Docker Compose File Reference
```yaml
version: "3.9"
services:
  web:
    build: .
    ports:
      - "8000:8000"
  db:
    image: postgres
    volumes:
      - db-data:/var/lib/postgresql/data
volumes:
  db-data:
```

## Best Practices
### Dockerfile Best Practices
- [Official Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Container Security](https://docs.docker.com/engine/security/)
- [Multi-stage Builds](https://docs.docker.com/develop/develop-images/multistage-build/)

### Docker Compose Best Practices
- [Compose File Version 3 Reference](https://docs.docker.com/compose/compose-file/compose-file-v3/)
- [Production Considerations](https://docs.docker.com/compose/production/)
- [Environment Variables](https://docs.docker.com/compose/environment-variables/)

## Tools & Extensions
- [Portainer](https://www.portainer.io/) - Container Management UI
- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- [VS Code Docker Extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-docker)
- [Dive](https://github.com/wagoodman/dive) - Image Layer Explorer

## Networking
- [Docker Network Types](https://docs.docker.com/network/)
- [Network Configuration](https://docs.docker.com/config/containers/container-networking/)
- [Docker DNS](https://docs.docker.com/config/containers/container-networking/#dns-services)

## Storage & Volumes
- [Volumes](https://docs.docker.com/storage/volumes/)
- [Bind Mounts](https://docs.docker.com/storage/bind-mounts/)
- [tmpfs Mounts](https://docs.docker.com/storage/tmpfs/)

## Docker Swarm
- [Swarm Mode Overview](https://docs.docker.com/engine/swarm/)
- [Swarm Services](https://docs.docker.com/engine/swarm/services/)
- [Stack Deploy](https://docs.docker.com/engine/swarm/stack-deploy/)

## Security
- [Security Best Practices](https://docs.docker.com/engine/security/)
- [Content Trust](https://docs.docker.com/engine/security/trust/)
- [AppArmor Security](https://docs.docker.com/engine/security/apparmor/)
- [Seccomp Security](https://docs.docker.com/engine/security/seccomp/)

## Monitoring & Logging
- [Container Logs](https://docs.docker.com/config/containers/logging/)
- [Docker Stats](https://docs.docker.com/engine/reference/commandline/stats/)
- [Prometheus Integration](https://docs.docker.com/config/daemon/prometheus/)

## Registry & Distribution
- [Docker Registry](https://docs.docker.com/registry/)
- [Docker Hub](https://hub.docker.com/)
- [Private Registry](https://docs.docker.com/registry/deploying/)

## Development Workflows
- [Docker Development Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [Docker Compose Development](https://docs.docker.com/compose/development/)
- [Live Reload with Docker](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#add-metadata-to-images)

## Common Use Cases
### Web Development
```yaml
version: "3.9"
services:
  web:
    build: .
    ports:
      - "3000:3000"
    volumes:
      - .:/app
    environment:
      - NODE_ENV=development
  db:
    image: postgres:13
    environment:
      - POSTGRES_PASSWORD=password
```

### Database Services
```yaml
version: "3.9"
services:
  postgres:
    image: postgres:13
    environment:
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres-data:/var/lib/postgresql/data
  redis:
    image: redis:6
    volumes:
      - redis-data:/data
volumes:
  postgres-data:
  redis-data:
```

## Troubleshooting
- [Debug Containers](https://docs.docker.com/config/containers/debugging/)
- [Common Problems](https://docs.docker.com/config/daemon/#troubleshoot-the-daemon)
- [Docker Logs](https://docs.docker.com/config/containers/logging/)

## Learning Resources
- [Docker Labs](https://github.com/docker/labs)
- [Play with Docker](https://labs.play-with-docker.com/)
- [Docker Curriculum](https://docker-curriculum.com/)

## Community Resources
- [Docker Forums](https://forums.docker.com/)
- [Stack Overflow - Docker](https://stackoverflow.com/questions/tagged/docker)
- [Docker Community](https://www.docker.com/community/)

Remember to:
- Use official images when possible
- Keep images small
- Use multi-stage builds
- Manage secrets properly
- Use proper networking
- Implement health checks
- Monitor container resources
- Keep Docker and images updated
- Follow security best practices
- Use proper logging
- Document your configurations