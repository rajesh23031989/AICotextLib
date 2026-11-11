# Kubernetes Manifest Changes

## Container Port Update

### Deployment: containerPort

#### Before
```yaml
spec:
  containers:
    - name: myapp
      ports:
        - containerPort: 80
```

#### After
```yaml
spec:
  containers:
    - name: myapp
      ports:
        - containerPort: 8080
```

### Alternative (if using 5000)

#### Before
```yaml
ports:
  - containerPort: 5000
```

#### After
```yaml
ports:
  - containerPort: 8080
```

## Service Port Update

### Service: targetPort

#### Before
```yaml
spec:
  ports:
    - port: 80
      targetPort: 5000
      protocol: TCP
```

#### After
```yaml
spec:
  ports:
    - port: 80
      targetPort: 8080
      protocol: TCP
```

**Note:** Keep external `port: 80` unchanged to maintain backward compatibility with ingress.

## Health Check Updates

If health probes use ports, update them:

### Before
```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 5000
```

### After
```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
```

## Files to Search

Common Kubernetes file locations:
- `Kubernetes/sce-deploy-app.yaml`
- `Kubernetes/deployment.yaml`
- `Kubernetes/service.yaml`
- `k8s/*.yaml`
- `deploy/*.yml`

## Complete Deployment Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: myregistry/myapp:latest
          ports:
            - containerPort: 8080
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8080
      protocol: TCP
  type: ClusterIP
```

## Ingress (No Change Usually)

Ingress typically routes to port 80 on the service, so no changes needed:

```yaml
spec:
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /
            backend:
              service:
                name: myapp-service
                port:
                  number: 80  # No change - points to service port
```
