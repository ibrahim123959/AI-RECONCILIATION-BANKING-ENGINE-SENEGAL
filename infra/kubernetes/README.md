# Kubernetes

Manifests Kubernetes pour déploiement production.

## Structure
```
kubernetes/
├── backend-deployment.yaml         # Backend pods + service
├── ml-engine-deployment.yaml       # ML engine pods + service
├── frontend-deployment.yaml        # Frontend pods + service
├── postgres-deployment.yaml        # PostgreSQL statefulset
└── ingress.yaml                    # Load balancer + routing
```

## Apply Manifests
```bash
# Apply all
kubectl apply -f infra/kubernetes/

# Apply specific
kubectl apply -f infra/kubernetes/backend-deployment.yaml

# Verify
kubectl get pods
kubectl get services
kubectl get ingress
```

## Deployments

### Backend
- **Replicas:** 3 (high availability)
- **Resources:** 512Mi RAM, 500m CPU
- **Health checks:** /health endpoint
- **Auto-scaling:** HPA (2-10 pods)

### ML Engine
- **Replicas:** 2
- **Resources:** 2Gi RAM, 1 CPU
- **Health checks:** /health endpoint
- **GPU:** Optional (for production)

### Frontend
- **Replicas:** 2
- **Resources:** 256Mi RAM, 250m CPU
- **CDN:** CloudFlare (recommended)

## Services
```yaml
# backend-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
    - port: 8000
      targetPort: 8000
```

## Ingress
```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: reconciliation-ingress
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
    - hosts:
        - reconciliation.example.com
      secretName: tls-secret
  rules:
    - host: reconciliation.example.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: backend-service
                port:
                  number: 8000
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80
```

## Secrets
```bash
# Create secret from file
kubectl create secret generic db-credentials \
  --from-literal=DATABASE_URL=postgresql://...

# Use in deployment
env:
  - name: DATABASE_URL
    valueFrom:
      secretKeyRef:
        name: db-credentials
        key: DATABASE_URL
```

## Scaling

### Manual
```bash
kubectl scale deployment backend-deployment --replicas=5
```

### Auto (HPA)
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: backend-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: backend-deployment
  minReplicas: 3
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

## Monitoring
```bash
# Logs
kubectl logs -f deployment/backend-deployment

# Describe pod
kubectl describe pod <pod-name>

# Events
kubectl get events --sort-by=.metadata.creationTimestamp
```

## Rollout
```bash
# Update image
kubectl set image deployment/backend-deployment \
  backend=reconciliation-backend:v2.0.0

# Check status
kubectl rollout status deployment/backend-deployment

# Rollback if needed
kubectl rollout undo deployment/backend-deployment
```