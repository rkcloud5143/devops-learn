# Docker — Deep Dive 🐳

Beyond the basics: Dockerfile best practices, multi-stage builds, networking, security, and optimization.

---

## Dockerfile Best Practices

### Bad Dockerfile ❌
```dockerfile
FROM ubuntu:latest
RUN apt-get update
RUN apt-get install -y nodejs npm
RUN apt-get install -y curl
COPY . /app
WORKDIR /app
RUN npm install
EXPOSE 3000
CMD ["node", "server.js"]
```

Problems: `latest` tag (unpredictable), too many layers, runs as root, huge image (~900MB), no .dockerignore.

### Good Dockerfile ✅
```dockerfile
# Stage 1: Build
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .

# Stage 2: Run
FROM node:20-alpine
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
WORKDIR /app
COPY --from=builder /app .
USER appuser
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s CMD curl -f http://localhost:3000/health || exit 1
CMD ["node", "server.js"]
```

Why it's better: pinned version, alpine (small), multi-stage, non-root user, health check, ~150MB.

---

## Multi-Stage Builds

Build dependencies stay in the build stage. Final image is tiny.

### Go Example
```dockerfile
# Stage 1: Build (has Go compiler — 1.2GB)
FROM golang:1.22-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o myapp .

# Stage 2: Run (just the binary — 15MB)
FROM alpine:3.19
RUN apk --no-cache add ca-certificates
COPY --from=builder /app/myapp /myapp
USER nobody
ENTRYPOINT ["/myapp"]
```

### Python Example
```dockerfile
FROM python:3.12-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

FROM python:3.12-slim
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY . .
ENV PATH=/root/.local/bin:$PATH
USER nobody
CMD ["python", "app.py"]
```

---

## Image Optimization

```
┌─── Image Size Comparison ──────────────────────┐
│                                                  │
│  ubuntu:latest          ~  77 MB                │
│  node:20                ~ 350 MB                │
│  node:20-slim           ~ 200 MB                │
│  node:20-alpine         ~  50 MB                │
│  alpine:3.19            ~   7 MB                │
│  scratch (empty)        ~   0 MB                │
│  distroless             ~  20 MB                │
│                                                  │
│  Rule: alpine > slim > full                     │
│  For Go/Rust: scratch or distroless             │
└──────────────────────────────────────────────────┘
```

### Tips to Reduce Size
```dockerfile
# 1. Combine RUN commands (fewer layers)
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    rm -rf /var/lib/apt/lists/*

# 2. Use .dockerignore
# .dockerignore
node_modules
.git
*.md
.env
dist
coverage

# 3. Copy dependency files first (cache layers)
COPY package*.json ./
RUN npm ci
COPY . .              # Code changes don't bust npm cache

# 4. Use --no-cache for pip/apk
RUN pip install --no-cache-dir -r requirements.txt
RUN apk add --no-cache curl
```

---

## Docker Networking

```
┌─── Network Types ──────────────────────────────────────────┐
│                                                             │
│  bridge (default)                                          │
│  ├── Containers get private IPs                            │
│  ├── Can communicate by container name                     │
│  └── Isolated from host network                            │
│                                                             │
│  host                                                      │
│  ├── Container uses host's network directly                │
│  ├── No port mapping needed                                │
│  └── No network isolation                                  │
│                                                             │
│  none                                                      │
│  └── No networking at all                                  │
│                                                             │
│  overlay                                                   │
│  ├── Multi-host networking                                 │
│  └── Used in Swarm / Kubernetes                            │
└─────────────────────────────────────────────────────────────┘
```

```bash
# Create custom network
docker network create my-network

# Run containers on same network (can reach each other by name)
docker run -d --name db --network my-network postgres
docker run -d --name app --network my-network -e DB_HOST=db my-app
# app can reach db at hostname "db"

# Inspect network
docker network inspect my-network

# List networks
docker network ls
```

---

## Docker Compose (Deep Dive)

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
      target: production          # Multi-stage target
    ports:
      - "3000:3000"
    environment:
      - DB_HOST=db
      - REDIS_HOST=redis
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    volumes:
      - ./src:/app/src            # Dev: hot reload
    networks:
      - backend
    restart: unless-stopped
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: secret
    volumes:
      - db-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - backend

  redis:
    image: redis:7-alpine
    networks:
      - backend

volumes:
  db-data:

networks:
  backend:
```

```bash
docker-compose up -d              # Start all
docker-compose up -d --build      # Rebuild and start
docker-compose logs -f app        # Follow app logs
docker-compose exec app sh        # Shell into app
docker-compose down               # Stop all
docker-compose down -v            # Stop + remove volumes
docker-compose ps                 # Status
```

---

## Docker Security

```
┌─── Security Checklist ─────────────────────────────────────┐
│                                                             │
│  ✅ Run as non-root user (USER instruction)                │
│  ✅ Use minimal base images (alpine, distroless)           │
│  ✅ Scan images for vulnerabilities (Trivy, Snyk)          │
│  ✅ Don't store secrets in images                          │
│  ✅ Use read-only filesystem where possible                │
│  ✅ Pin image versions (not :latest)                       │
│  ✅ Use .dockerignore                                      │
│  ✅ Don't run privileged containers                        │
│  ✅ Limit resources (CPU, memory)                          │
│  ✅ Sign images (Docker Content Trust)                     │
└─────────────────────────────────────────────────────────────┘
```

```bash
# Scan image for vulnerabilities
docker scout cves my-app:latest
trivy image my-app:latest

# Run with security options
docker run \
  --read-only \
  --tmpfs /tmp \
  --cap-drop ALL \
  --security-opt no-new-privileges \
  --memory 512m \
  --cpus 0.5 \
  --user 1000:1000 \
  my-app
```

---

## ECR (Elastic Container Registry)

```bash
# Create repository
aws ecr create-repository --repository-name my-app

# Login to ECR
aws ecr get-login-password --region ca-central-1 | \
  docker login --username AWS --password-stdin 123456789012.dkr.ecr.ca-central-1.amazonaws.com

# Tag and push
docker tag my-app:latest 123456789012.dkr.ecr.ca-central-1.amazonaws.com/my-app:v1.0
docker push 123456789012.dkr.ecr.ca-central-1.amazonaws.com/my-app:v1.0

# Enable image scanning
aws ecr put-image-scanning-configuration \
  --repository-name my-app \
  --image-scanning-configuration scanOnPush=true

# Lifecycle policy (auto-delete old images)
aws ecr put-lifecycle-policy \
  --repository-name my-app \
  --lifecycle-policy-text '{
    "rules": [{
      "rulePriority": 1,
      "description": "Keep last 10 images",
      "selection": {
        "tagStatus": "any",
        "countType": "imageCountMoreThan",
        "countNumber": 10
      },
      "action": { "type": "expire" }
    }]
  }'
```

---

*Docker is the foundation of modern DevOps. Master it! 🐳*
