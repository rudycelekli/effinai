---
name: k8s-deploy
description: "Kubernetes deployment configuration generation"
version: "1.0.0"
tags: [devops, kubernetes, deployment, infrastructure]
createdBy: "built-in"
status: "active"
---

# Kubernetes Deployment

## Activation
When the user asks to deploy to Kubernetes, create K8s manifests, or set up a K8s configuration.

## Workflow

### Step 1: Gather Requirements
1. Application name and image
2. Replicas needed
3. Resource requirements (CPU, memory)
4. Environment variables and secrets
5. Ports to expose
6. Persistent storage needs
7. Health check endpoints

### Step 2: Generate Core Manifests

**Deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-name
  labels:
    app: app-name
spec:
  replicas: 3
  selector:
    matchLabels:
      app: app-name
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  template:
    metadata:
      labels:
        app: app-name
    spec:
      containers:
        - name: app-name
          image: registry/app-name:latest
          ports:
            - containerPort: 3000
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 500m
              memory: 512Mi
          readinessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 10
          livenessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 15
            periodSeconds: 20
          envFrom:
            - configMapRef:
                name: app-name-config
            - secretRef:
                name: app-name-secrets
```

**Service:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: app-name
spec:
  selector:
    app: app-name
  ports:
    - port: 80
      targetPort: 3000
  type: ClusterIP
```

**Ingress:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-name
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
    - hosts: [app.example.com]
      secretName: app-tls
  rules:
    - host: app.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: app-name
                port:
                  number: 80
```

**ConfigMap & Secret:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-name-config
data:
  NODE_ENV: production
  LOG_LEVEL: info
---
apiVersion: v1
kind: Secret
metadata:
  name: app-name-secrets
type: Opaque
stringData:
  DATABASE_URL: placeholder-replace-with-actual
```

### Step 3: Add Production Hardening
- Pod Disruption Budget (PDB)
- Horizontal Pod Autoscaler (HPA)
- Network Policies
- Security Context (non-root, read-only filesystem)
- Resource quotas

### Step 4: Organize Files
```
k8s/
  base/
    deployment.yaml
    service.yaml
    configmap.yaml
  overlays/
    staging/
      kustomization.yaml
    production/
      kustomization.yaml
```

## Rules
- Always set resource requests AND limits
- Always include health checks (readiness + liveness)
- Never use `latest` tag in production — pin image versions
- Separate config from secrets
- Use namespaces to isolate environments
- Include PDB for high-availability deployments
- Run containers as non-root
- Use Kustomize or Helm for environment-specific config
