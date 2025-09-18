# 🎯 Easter Egg Hunter & Bonus Challenge Solutions Guide

## 🥚 Easter Egg Solutions

### Easter Egg #1: The Secret Endpoint

**Challenge**: Find the hidden endpoint in the backend service
**Solution**:

```bash
# Check the backend API routes
curl http://localhost:3001/bootcamp-secret
# or
curl http://localhost:3001/tech4dev
# or
curl http://localhost:3001/api/bootcamp/secret'
# Expected response: Special bootcamp message
```

### Easter Egg #2: The Konami Code

**Challenge**: Activate hidden frontend feature with gaming sequence
**Solution**:

1. Open frontend at `http://localhost:8080`
2. Press keys in sequence: `↑↑↓↓←→←→BA`
3. Developer console will show extra metrics panel

### Easter Egg #3: The Metrics Detective

**Challenge**: Find "coffee consumption" metric in ML service
**Solution**:

```bash
# Check ML service metrics
curl http://localhost:3002/metrics | grep coffee
# Look for: ml_service_coffee_consumption_total
```

### Easter Egg #4: The Pod Whisperer

**Challenge**: Name pod with Kubernetes mascot to get special logs
**Solution**:

```yaml
# Add to any deployment metadata
metadata:
  name: phippy-backend  # or gopher-ml-service
# Check logs for ASCII art:
kubectl logs phippy-backend -n shopmicro
```

### Easter Egg #5: The Time Traveler

**Challenge**: Add retro mode annotation to namespace
**Solution**:

```bash
kubectl annotate namespace shopmicro retro.mode="1985"
# All service timestamps will display in retro format
```

## 🏆 Bonus Challenge Solutions

### 1. Horizontal Pod Autoscaling

```yaml
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: backend-hpa
  namespace: shopmicro
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: backend
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

### 2. Persistent Storage

```yaml
# persistent-volumes.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: postgres-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data/postgres
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
  namespace: shopmicro
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

### 3. Ingress Configuration

```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: shopmicro-ingress
  namespace: shopmicro
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: shopmicro.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend
                port:
                  number: 3000
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: backend
                port:
                  number: 3001
```

### 4. Resource Limits

```yaml
# Add to all deployments
resources:
  requests:
    memory: '128Mi'
    cpu: '100m'
  limits:
    memory: '512Mi'
    cpu: '500m'
```

### 5. Comprehensive Health Checks

```yaml
# Enhanced probes
livenessProbe:
  httpGet:
    path: /health
    port: 3001
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 3
readinessProbe:
  httpGet:
    path: /ready
    port: 3001
  initialDelaySeconds: 5
  periodSeconds: 5
  timeoutSeconds: 3
  failureThreshold: 3
```

### 6. Chaos Engineering

```bash
# Install chaos-mesh or use simple script
#!/bin/bash
while true; do
  POD=$(kubectl get pods -n shopmicro -o name | shuf -n 1)
  echo "Killing $POD"
  kubectl delete $POD -n shopmicro
  sleep 60
done
```

### 7. Custom Grafana Dashboards

```json
{
  "dashboard": {
    "title": "🎪 ShopMicro Circus Dashboard",
    "panels": [
      {
        "title": "🎠 Request Carousel",
        "type": "stat",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])"
          }
        ]
      }
    ]
  }
}
```

## 🎮 Achievement Unlock Commands

### Deployment Master

```bash
# Deploy everything in one go
kubectl apply -f k8s/ --recursive
# Check all pods are running within 5 minutes
kubectl wait --for=condition=Ready pod --all -n shopmicro --timeout=300s
```

### Troubleshoot Hero

```bash
# Common debugging commands
kubectl describe pods -n shopmicro
kubectl logs -f deployment/backend -n shopmicro
kubectl get events -n shopmicro --sort-by='.lastTimestamp'
```

### Metrics Guru

```bash
# Access Grafana and create custom dashboard
kubectl port-forward -n shopmicro svc/grafana 3000:3000
# Login: admin/admin
# Import dashboard ID: 12345 or create custom
```

### Performance Optimizer

```bash
# Monitor resource usage
kubectl top pods -n shopmicro
kubectl top nodes
# Adjust resource limits based on actual usage
```

### Security Sentinel

```yaml
# Add security context to deployments
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  fsGroup: 2000
  capabilities:
    drop:
      - ALL
```

## 🏅 Verification Commands

```bash
# Verify all easter eggs found
echo "🥚 Easter Egg Checklist:"
echo "1. Secret endpoint: $(curl -s http://localhost:3001/bootcamp-secret | wc -l) lines"
echo "2. Konami code: Check browser console"
echo "3. Coffee metric: $(curl -s http://localhost:3002/metrics | grep -c coffee) found"
echo "4. Pod whisperer: $(kubectl logs -n shopmicro --selector=app=backend | grep -c ASCII) ASCII arts"
echo "5. Time traveler: $(kubectl get namespace shopmicro -o jsonpath='{.metadata.annotations.retro\.mode}')"

# Verify bonus challenges
echo "🏆 Bonus Challenge Status:"
echo "HPA: $(kubectl get hpa -n shopmicro | wc -l) configured"
echo "PV: $(kubectl get pv | wc -l) volumes"
echo "Ingress: $(kubectl get ingress -n shopmicro | wc -l) rules"
echo "Resources: All pods have limits set"
```

## 🎊 Final Achievement

Once all easter eggs and bonus challenges are complete, you'll unlock the **Kubernetes Bootcamp Master** achievement!

**Proof of completion**: Take screenshots of:

1. All pods running successfully
2. Grafana dashboard with custom metrics
3. Each easter egg discovery
4. Bonus challenges implemented

**Reward**: Digital certificate and bragging rights! 🏆
