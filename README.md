# Docmost Helm Chart

This Helm chart allows deploying ([**Docmost**](https://docmost.com/)) on a Kubernetes cluster.

## Prerequisites

- A functional Kubernetes cluster
- Helm installed ([Installation Guide](https://helm.sh/docs/intro/install/))
- Kubernetes version `>=1.22.x`

## Dependencies

| Repository | Name | Version |
|------------|------|---------|
| <https://bjw-s.github.io/helm-charts> | common | 1.5.1 |
| <oci://registry-1.docker.io/bitnamicharts> | Postgresql | 16.4.6 |
| <oci://registry-1.docker.io/bitnamicharts> | Redis | 20.6.3 |

## Chart Installation

### OCI (TODO)

## Configuration

Configuration values can be overridden via the `values.yaml` file or command line.

### Ingress
By default, ingress is not activated.
To activate and customize it:
```yaml
ingress:
  main:
    enabled: true
#    className: nginx,traefik, ...
    hosts:
      - host: docmost-app.local
        paths:
          - path: /
            pathType: Prefix
```

### Docmost secretKey
**Mandatory** for Docmost launch !

To generate it: `openssl rand -hex 32`

In future releases, it will no longer be necessary to fill in this field, it will be automatically generated ([Issue #1](../../issues/1))
```yaml
docmost:
  secretKey: "5e5accb8a565a5e1302d0d1f6933a2f95a176142b4759165e6dcc40418a1e4ae" #(example)
```

### Docmost storage
2 types of storage are available:
  - local
  - s3
  
By default this is the local mode and creates an 8GB PVC to store the media (photos, files, ...).
You can, however, customize a few parameters:
```yaml
docmost:
[...]
  storage:
    type: local
#  s3:
[...]

persistence:
  # -- Configure persistence settings for the chart under this key.
  data:
    enabled: true
    retain: true
#   storageClass: ""
    accessMode: ReadWriteOnce
    size: 5Gi
```
If you want to configure this storage with S3, you need to disable persistence and configure the following variables.

Example for MinIO configuration.
```yaml
docmost:
[...]
  storage:
    type: s3
  s3:
    accessKey: "docmost"
    secretKey: "docmost"
    region: "us-east-1"
    bucket: "docmost"
    endpoint: "https://minIO:9000"
    pathStyle: true

persistence:
  # -- Configure persistence settings for the chart under this key.
  data:
    enabled: false
[...]
```

## Postgresql and Redis bitnami Charts
Deployment of ([postgresql](https://github.com/bitnami/charts/tree/main/bitnami/postgresql)) and ([redis](https://github.com/bitnami/charts/tree/main/bitnami/redis)) is handled by Bitnami charts.
If you'd like to know more about the default values of these charts, don't hesitate to have a look at their documentation.

If you want to manage your PVCs for Bitnami Postgresql and redis charts:
```yaml
postgresql:
  enabled: true
[...]
  primary:
    persistence:
      existingClaim: docmost-postgresql

redis:
  enabled: true
[...]
  master:
    persistence:
      existingClaim: docmost-redis
```
