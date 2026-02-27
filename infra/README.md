# Infrastructure

Configuration infrastructure pour déploiement du système.

## Structure
```
infra/
├── docker/                 # Dockerfiles production
│   ├── backend.Dockerfile
│   ├── ml-engine.Dockerfile
│   └── frontend.Dockerfile
│
├── kubernetes/             # Manifests K8s
│   ├── backend-deployment.yaml
│   ├── ml-engine-deployment.yaml
│   ├── frontend-deployment.yaml
│   ├── postgres-deployment.yaml
│   └── ingress.yaml
│
└── terraform/              # Infrastructure as Code
    ├── main.tf
    └── variables.tf
```

## Environnements

### Development (Local)
- **Tool:** Docker Compose
- **Config:** `docker-compose.yml` (racine projet)
- **Database:** PostgreSQL container
- **Cache:** Redis container

### Staging
- **Platform:** Render.com
- **Config:** `render.yaml`
- **Database:** Supabase (free tier)
- **Cache:** Upstash Redis (free tier)

### Production
- **Platform:** Kubernetes (GKE/EKS/AKS)
- **Config:** `kubernetes/*.yaml`
- **Database:** Managed PostgreSQL
- **Cache:** Managed Redis

## Quick Deploy

### Local (Docker Compose)
```bash
docker-compose up -d
```

### Staging (Render)
```bash
# Push to develop branch triggers auto-deploy
git push origin develop
```

### Production (Kubernetes)
```bash
# Apply manifests
kubectl apply -f infra/kubernetes/

# Verify deployments
kubectl get pods
kubectl get services
```

## Stack Infrastructure

| Component | Dev | Staging | Production |
|-----------|-----|---------|------------|
| **Compute** | Docker | Render | K8s (GKE) |
| **Database** | PostgreSQL 15 | Supabase | Cloud SQL |
| **Cache** | Redis 7 | Upstash | Cloud Memorystore |
| **Storage** | Local | Render Disk | Cloud Storage |

## Monitoring

- **Logs:** Structured JSON logs
- **Metrics:** Prometheus + Grafana
- **Errors:** Sentry
- **Uptime:** UptimeRobot

## Cost Estimate

### Staging (Render + Supabase)
- Backend: $7/month
- ML Engine: $7/month
- Frontend: $0 (static)
- Database: $0 (free tier)
- Cache: $0 (free tier)
- **Total: $14/month**

### Production (GKE)
- Compute (3 nodes): ~$150/month
- Database (managed): ~$50/month
- Cache (managed): ~$30/month
- Storage: ~$10/month
- **Total: ~$240/month**

## Security

- ✅ HTTPS enforced (TLS 1.3)
- ✅ Secrets management (K8s secrets / Render env vars)
- ✅ Network policies (K8s)
- ✅ Database encryption at rest
- ✅ Regular backups (automated)