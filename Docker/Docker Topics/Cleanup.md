### 1. Check System Status
```bash
# Check disk space usage by Docker
docker system df

# Show detailed space usage
docker system df -v

# See all containers (including stopped)
docker ps -a

# List all volumes
docker volume ls
```

### 2. Volume Cleanup
```bash
# Inspect specific volume
docker run --rm -v VOLUME_NAME:/vol alpine sh -c "du -sh /vol; ls -la /vol"

# Check all volumes (save as script)
for volume in $(docker volume ls -q); do
    echo "Checking volume: $volume"
    docker run --rm -v $volume:/vol alpine sh -c "du -sh /vol; ls -la /vol"
done

# Remove specific volumes
docker volume rm volume_name1 volume_name2

# Remove all unused volumes (CAREFUL!)
docker volume prune

# Remove unused volumes with filter
docker volume prune --filter "label!=keep"
```

### 3. Image Cleanup
```bash
# List images with details
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}\t{{.CreatedAt}}"

# Remove specific image
docker rmi image_id

# Remove dangling images (untagged)
docker image prune

# Remove unused images (CAREFUL!)
docker image prune -a

# Remove images older than 24h
docker image prune -a --filter "until=24h"
```

### 4. Container Cleanup
```bash
# Remove stopped containers
docker container prune

# Remove container and its volume
docker rm -v container_name

# Stop and remove all containers
docker stop $(docker ps -aq)
docker rm $(docker ps -aq)
```

### 5. Complete System Cleanup
```bash
# Remove all unused containers, networks, images (both dangling and unreferenced), and optionally, volumes
docker system prune

# Include volumes in cleanup (VERY CAREFUL!)
docker system prune -a --volumes
```

### 6. Safety Checks
```bash
# Check what would be removed (dry run)
docker system prune --dry-run

# List containers using a volume
docker ps -a --filter volume=VOLUME_NAME

# Check container logs before removal
docker logs container_name

# Backup volume before removal
docker run --rm -v VOLUME_NAME:/source -v $(pwd):/backup alpine tar czf /backup/volume_backup.tar.gz -C /source .
```

### 7. Best Practices
- Always verify what's running: `docker ps`
- Check volume contents before removal
- Use --dry-run when available
- Backup important data before cleanup
- Start with specific removals before using prune
- Check container dependencies
- Consider using labels for better management

### 8. Regular Maintenance
```bash
# Add to crontab for regular cleanup of dangling resources
0 0 * * 0 docker system prune -f

# Monitor docker disk usage
0 * * * * docker system df >> /var/log/docker-disk-usage.log
```

### 9. Debugging
```bash
# Check why volume won't delete
docker ps -a --filter volume=VOLUME_NAME

# Find containers with specific mounts
docker ps -a --format '{{.Names}}' --filter volume=VOLUME_NAME

# Inspect volume details
docker volume inspect VOLUME_NAME
```

Remember:
- Always backup important data
- Verify running containers before cleanup
- Check dependencies between services
- Use filters to target specific resources
- Consider using docker-compose down for project cleanup
- Monitor system regularly for space issues