# Microblog Helm Chart

Kubernetes deployment chart for the Microblog app with optional PostgreSQL and Redis dependencies, ALB Ingress on 443, and GitOps-friendly configuration.

## Quick start

```bash
# Fetch/refresh dependencies
helm dependency update microblog/

# Install (adjust namespace as needed)
helm install microblog microblog/ -n microblog --create-namespace
```

## Key values

Edit `microblog/values.yaml` or create an env-specific file (e.g. `values-prod.yaml`).

```yaml
# Image
image:
  repository: ghcr.io/<org-or-user>/microblog
  tag: "v1.2.3"         # Prefer immutable tags (e.g., commit SHA)
  pullPolicy: IfNotPresent

# Private registry auth (required for GHCR private repos)
imagePullSecrets:
  - name: ghcr-secret

# Service
service:
  type: ClusterIP
  port: 80
  targetPort: 5000

# Ingress (ALB)
ingress:
  enabled: true
  className: alb
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS": 443}]'
    alb.ingress.kubernetes.io/ssl-policy: ELBSecurityPolicy-TLS-1-2-2017-01
  hosts: []              # e.g. ["blog.example.com"]
  certificateArn: ""     # ACM cert ARN for HTTPS

# App config
config:
  secretKey: "change-me" # If secrets.create=true, this will be embedded into a Secret
  logToStdout: "1"

# Secrets handling
secrets:
  create: true           # If true, create a Secret holding SECRET_KEY from config.secretKey
  name: ""               # Optional explicit Secret name (defaults to release/chart name)
  secretKeyKey: "secretKey"

# Pod Disruption Budget
pdb:
  enabled: true
  minAvailable: 1

# Optional dependencies (OCI charts)
postgres:
  enabled: true
  auth:
    password: "microblog-prod-password"
    database: microblog
  persistence:
    enabled: true
    size: 10Gi
  service:
    enabled: true
    type: ClusterIP
    port: 5432

redis:
  enabled: true
  persistence:
    enabled: true
    size: 2Gi
  service:
    enabled: true
    type: ClusterIP
    port: 6379
```

## Image pull secret for GHCR

If your image is private, create a pull secret in the target namespace:

```bash
kubectl create secret docker-registry ghcr-secret \
  --docker-server=ghcr.io \
  --docker-username="<github-username>" \
  --docker-password="<github-personal-access-token>" \
  -n microblog
```

## Install/upgrade

```bash
# Install with env-specific values
helm install microblog microblog/ -f values-prod.yaml -n microblog --create-namespace

# Upgrade
helm upgrade microblog microblog/ -f values-prod.yaml -n microblog
```

## GitOps (ArgoCD) notes

- Jenkins updates `.image.repository` and `.image.tag` in the chart repo via yq.
- ArgoCD Application is set to auto-sync; the cluster reconciles to new images after merge.
- Ingress certificate ARN and annotations should be set per environment in the values file.

## Operational notes

- Liveness/readiness probe: HTTP GET on `/` at container port 5000 (Service 80).
- PDB enabled by default (`minAvailable: 1`) to improve availability during voluntary disruptions.

## Common commands

```bash
# Inspect rendered manifests
helm template microblog microblog/ -f values-prod.yaml -n microblog

# Uninstall
helm uninstall microblog -n microblog
```
