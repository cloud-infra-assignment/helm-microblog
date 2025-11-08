# Microblog Helm Chart

Deploys Microblog with PostgreSQL and Redis.

## Quick Start

```bash
helm dependency update microblog/
helm install microblog microblog/ -n microblog --create-namespace
```

## Configuration

Edit `values.yaml` or create `values-prod.yaml`:

```yaml
postgresql:
  auth:
    password: "your-password"
    database: microblog
  primary:
    persistence:
      size: 50Gi

redis:
  auth:
    password: "your-password"
  master:
    persistence:
      size: 10Gi

image:
  tag: "your-tag"

ingress:
  annotations:
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:region:account:certificate/id
```

Deploy with custom values:

```bash
helm install microblog microblog/ -f values-prod.yaml -n microblog --create-namespace
```

## Common Commands

```bash
# Upgrade
helm upgrade microblog microblog/ -f values-prod.yaml -n microblog

# Update image
helm upgrade microblog microblog/ --set image.tag=v1.0.0 -n microblog

# Database migrations
kubectl exec -it deployment/microblog -n microblog -- python manage.py migrate

# Uninstall
helm uninstall microblog -n microblog
```

## Ingress

Uses AWS ALB Ingress Controller with HTTPS on port 443. Requires ACM certificate ARN in values.
