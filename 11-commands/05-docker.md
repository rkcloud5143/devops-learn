# Docker — Command Reference 🐳

Every Docker command you'll use daily.

---

## Images

```bash
# ─── Build ───
docker build -t my-app:v1 .                       # Build from Dockerfile
docker build -t my-app:v1 -f Dockerfile.prod .     # Custom Dockerfile
docker build --no-cache -t my-app:v1 .             # No layer cache
docker build --target builder -t my-app:build .    # Multi-stage target
docker build --build-arg VERSION=2.0 -t my-app .   # Build argument
docker build --platform linux/amd64 -t my-app .    # Specific platform

# ─── List / Inspect ───
docker images                                      # List images
docker images -a                                   # Include intermediate
docker image ls --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
docker inspect my-app:v1                           # Full details
docker history my-app:v1                           # Layer history

# ─── Tag / Push / Pull ───
docker tag my-app:v1 registry.com/my-app:v1        # Tag for registry
docker push registry.com/my-app:v1                 # Push to registry
docker pull nginx:1.25-alpine                      # Pull from registry

# ─── Remove ───
docker rmi my-app:v1                               # Remove image
docker rmi -f my-app:v1                            # Force remove
docker image prune                                 # Remove dangling images
docker image prune -a                              # Remove ALL unused images

# ─── Save / Load ───
docker save my-app:v1 > my-app.tar                 # Export to file
docker load < my-app.tar                           # Import from file
```

---

## Containers

```bash
# ─── Run ───
docker run nginx                                   # Run (foreground)
docker run -d nginx                                # Detached (background)
docker run -d --name web nginx                     # Named container
docker run -d -p 8080:80 nginx                     # Port mapping
docker run -d -p 8080:80 -p 8443:443 nginx         # Multiple ports
docker run -d -P nginx                             # Random host ports
docker run -d --restart unless-stopped nginx        # Auto-restart
docker run -d --restart always nginx                # Always restart
docker run --rm nginx echo "hello"                 # Remove after exit
docker run -it ubuntu /bin/bash                    # Interactive terminal
docker run -d --name web --hostname webserver nginx # Custom hostname

# ─── Environment Variables ───
docker run -e DB_HOST=localhost nginx               # Single var
docker run --env-file .env nginx                   # From file

# ─── Volumes ───
docker run -v my-vol:/data nginx                   # Named volume
docker run -v /host/path:/container/path nginx     # Bind mount
docker run -v /host/path:/container/path:ro nginx  # Read-only

# ─── Resources ───
docker run --memory=512m --cpus=1 nginx            # Limit resources
docker run --memory=512m --memory-swap=1g nginx    # Memory + swap

# ─── Network ───
docker run --network my-net nginx                  # Custom network
docker run --network host nginx                    # Host network
docker run --network none nginx                    # No network

# ─── Security ───
docker run --user 1000:1000 nginx                  # Run as user
docker run --read-only nginx                       # Read-only filesystem
docker run --cap-drop ALL nginx                    # Drop all capabilities
docker run --security-opt no-new-privileges nginx  # No privilege escalation

# ─── List ───
docker ps                                          # Running containers
docker ps -a                                       # All (including stopped)
docker ps -q                                       # IDs only
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# ─── Lifecycle ───
docker start container-name                        # Start stopped
docker stop container-name                         # Graceful stop
docker restart container-name                      # Restart
docker kill container-name                         # Force stop
docker pause container-name                        # Pause
docker unpause container-name                      # Unpause

# ─── Interact ───
docker exec -it container-name /bin/sh             # Shell into container
docker exec -it container-name bash                # Bash (if available)
docker exec container-name ls /app                 # Run command
docker exec -u root container-name whoami          # Run as root

# ─── Logs ───
docker logs container-name                         # All logs
docker logs -f container-name                      # Follow (tail -f)
docker logs --tail 100 container-name              # Last 100 lines
docker logs --since 1h container-name              # Last hour
docker logs --timestamps container-name            # With timestamps

# ─── Inspect ───
docker inspect container-name                      # Full details
docker inspect -f '{{.NetworkSettings.IPAddress}}' container-name  # IP
docker inspect -f '{{.State.Status}}' container-name               # Status
docker stats                                       # Live resource usage
docker stats --no-stream                           # Snapshot
docker top container-name                          # Running processes
docker diff container-name                         # Changed files

# ─── Copy ───
docker cp container-name:/path/file ./local        # From container
docker cp ./local container-name:/path/file        # To container

# ─── Remove ───
docker rm container-name                           # Remove stopped
docker rm -f container-name                        # Force remove running
docker container prune                             # Remove all stopped
```

---

## Volumes

```bash
docker volume create my-vol                        # Create
docker volume ls                                   # List
docker volume inspect my-vol                       # Details
docker volume rm my-vol                            # Remove
docker volume prune                                # Remove unused
```

---

## Networks

```bash
docker network create my-net                       # Create bridge network
docker network create --driver overlay my-net      # Overlay (Swarm)
docker network ls                                  # List
docker network inspect my-net                      # Details
docker network connect my-net container-name       # Connect container
docker network disconnect my-net container-name    # Disconnect
docker network rm my-net                           # Remove
docker network prune                               # Remove unused
```

---

## Docker Compose

```bash
docker compose up                                  # Start all services
docker compose up -d                               # Detached
docker compose up -d --build                       # Rebuild images
docker compose up -d --force-recreate              # Force recreate
docker compose up -d service-name                  # Specific service

docker compose down                                # Stop and remove
docker compose down -v                             # Also remove volumes
docker compose down --rmi all                      # Also remove images

docker compose ps                                  # List services
docker compose logs                                # All logs
docker compose logs -f service-name                # Follow specific
docker compose exec service-name sh                # Shell into service
docker compose run service-name command            # Run one-off command

docker compose build                               # Build all images
docker compose pull                                # Pull all images
docker compose restart                             # Restart all
docker compose stop                                # Stop (don't remove)
docker compose start                               # Start stopped

docker compose config                              # Validate and show config
docker compose top                                 # Running processes

# Multiple compose files
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

---

## System Cleanup

```bash
docker system df                                   # Disk usage
docker system prune                                # Remove unused (containers, networks, images)
docker system prune -a                             # Remove ALL unused
docker system prune -a --volumes                   # Include volumes
docker system info                                 # System-wide info
```

---

## ECR (AWS Container Registry)

```bash
# Login
aws ecr get-login-password --region ca-central-1 | \
  docker login --username AWS --password-stdin 123456789012.dkr.ecr.ca-central-1.amazonaws.com

# Tag and push
docker tag my-app:v1 123456789012.dkr.ecr.ca-central-1.amazonaws.com/my-app:v1
docker push 123456789012.dkr.ecr.ca-central-1.amazonaws.com/my-app:v1

# Pull
docker pull 123456789012.dkr.ecr.ca-central-1.amazonaws.com/my-app:v1
```

---

*Docker commands = your daily bread and butter! 🐳*
