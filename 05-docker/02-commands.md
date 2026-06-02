# Docker — Essential Commands

## Images
```bash
docker build -t my-app:v1 .          # Build image from Dockerfile
docker images                          # List images
docker pull nginx:latest               # Download image
docker push my-repo/my-app:v1         # Upload to registry
```

## Containers
```bash
docker run -d -p 8080:3000 my-app:v1  # Run container
#          │  │         │
#          │  │         └── image name
#          │  └── map host:container port
#          └── detached (background)

docker ps                              # List running containers
docker ps -a                           # List all containers
docker logs <container-id>             # View logs
docker exec -it <container-id> sh     # Shell into container
docker stop <container-id>             # Stop container
docker rm <container-id>               # Remove container
```

## Cleanup
```bash
docker system prune -a                 # Remove all unused data
```
