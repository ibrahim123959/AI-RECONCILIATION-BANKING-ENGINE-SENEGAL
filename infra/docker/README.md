# Docker

Dockerfiles optimisés pour production.

## Structure
```
docker/
├── backend.Dockerfile      # Backend API image
├── ml-engine.Dockerfile    # ML service image
└── frontend.Dockerfile     # Frontend image
```

## Images

### Backend
- **Base:** python:3.11-slim
- **Size:** ~500MB
- **Layers:** Multi-stage build

### ML Engine
- **Base:** python:3.11-slim
- **Size:** ~2.5GB (with PyTorch)
- **Layers:** Multi-stage build

### Frontend
- **Base:** node:18-alpine
- **Size:** ~150MB
- **Output:** Static files (nginx)

## Build Images
```bash
# Backend
docker build -f infra/docker/backend.Dockerfile -t reconciliation-backend:latest .

# ML Engine
docker build -f infra/docker/ml-engine.Dockerfile -t reconciliation-ml:latest .

# Frontend
docker build -f infra/docker/frontend.Dockerfile -t reconciliation-frontend:latest .
```

## Run Containers
```bash
# Backend
docker run -d -p 8000:8000 \
  -e DATABASE_URL=postgresql://... \
  reconciliation-backend:latest

# ML Engine
docker run -d -p 8001:8001 reconciliation-ml:latest

# Frontend
docker run -d -p 80:80 reconciliation-frontend:latest
```

## Optimization

### Multi-Stage Build
```dockerfile
# Stage 1: Build
FROM python:3.11-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Stage 2: Runtime
FROM python:3.11-slim
COPY --from=builder /root/.local /root/.local
COPY . .
CMD ["uvicorn", "app.main:app"]
```

### Layer Caching
- Dependencies installed first (cached)
- Code copied last (changes frequently)

## Registry

### Push to Docker Hub
```bash
docker tag reconciliation-backend:latest username/reconciliation-backend:v1.0.0
docker push username/reconciliation-backend:v1.0.0
```

### Push to GCR (Google)
```bash
docker tag reconciliation-backend:latest gcr.io/project-id/reconciliation-backend:v1.0.0
docker push gcr.io/project-id/reconciliation-backend:v1.0.0
```