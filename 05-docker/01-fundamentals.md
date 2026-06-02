# Docker — Fundamentals

## What Problem Does Docker Solve?
```
Without Docker:                    With Docker:
┌──────────────────┐              ┌──────────────────┐
│ "Works on my     │              │ Same container    │
│  machine" 😅     │              │ runs everywhere   │
│                  │              │                   │
│ Dev: Node 18     │              │ ┌──────────────┐  │
│ Staging: Node 16 │              │ │  App + Node  │  │
│ Prod: Node 14    │              │ │  18 + deps   │  │
│                  │              │ │  (all in one) │  │
│ = BUGS           │              │ └──────────────┘  │
└──────────────────┘              └──────────────────┘
```

## Architecture
```
┌─── Your Machine ──────────────────────────────────┐
│                                                    │
│  ┌─── Docker Engine ──────────────────────────┐   │
│  │                                             │   │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐   │   │
│  │  │Container │ │Container │ │Container │   │   │
│  │  │  nginx   │ │  app     │ │  redis   │   │   │
│  │  │ :80      │ │ :3000    │ │ :6379    │   │   │
│  │  └──────────┘ └──────────┘ └──────────┘   │   │
│  │       │            │            │          │   │
│  │  ┌────┴────────────┴────────────┴──────┐   │   │
│  │  │         Docker Network              │   │   │
│  │  └─────────────────────────────────────┘   │   │
│  │                                             │   │
│  │  Images (blueprints):                       │   │
│  │  nginx:latest, node:18-alpine, redis:7      │   │
│  └─────────────────────────────────────────────┘   │
│                                                    │
└────────────────────────────────────────────────────┘

Image = blueprint (like a class)
Container = running instance (like an object)
```

## Dockerfile (How to Build an Image)
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install --production
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

## Checklist
- [ ] Install Docker Desktop
- [ ] Build a Dockerfile for a simple web app
