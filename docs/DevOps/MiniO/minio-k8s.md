# How to Install MinIO with Helm Chart in Kubernetes

This guide explains how to install MinIO in your Kubernetes cluster using Helm charts.

## Prerequisites

Before you begin, make sure you have:

- A running Kubernetes cluster (v1.21+)
- Helm installed on your local machine ([Helm Installation Guide](https://helm.sh/docs/intro/install/))
- kubectl configured to interact with your Kubernetes cluster

# Install MinIO Operator

1. Create a directory for the Minio Operator installation, e.g. [name]-stage-cluster/minio-operator.
2. Create a file named Chart.yaml in the minio-operator directory with the following content:

```
apiVersion: v2
name: minio-operator
description: An umbrella chart for the minio-operator application
type: application
version: 0.1.0
dependencies:
- name: operator
  alias: minio-operator
  version: 5.x.x
  repository: https://operator.min.io
  condition: minio-operator.enabled
```

3. Create a file named values.yaml in the minio-operator directory with the following content:
```
minio-operator:
  enabled: true
  operator:
    env:
    - name: OPERATOR_STS_ENABLED
      value: "on"
    - name: CLUSTER_DOMAIN
      value: "k8s.stage.[name]-it.com"
  console:
    enabled: true
    ingress:
      enabled: true
      ingressClassName: traefik
      annotations:
        cert-manager.io/cluster-issuer: letsencrypt-prd
      host: minio-console.stage.[name]-it.com
      tls:
      - secretName: minio-console-tls
        hosts:
        - minio-console.stage.[name]-it.com
```

# Install MinIO Server

1. Create a directory for the Minio Server installation, e.g. [name]-stage-cluster/minio-server.
2. Create a file named Chart.yaml in the minio-server directory with the following content:
```
apiVersion: v2
name: minio-tenant
description: minio Operator
version: 0.1.0
type: application
dependencies:
- name: tenant
  version: 5.x.x
  repository: https://operator.min.io/
```

3. Create a file named values.yaml in the minio-server directory with the following content:

```
tenant:
  tenant:
    name: [name]-stage
    configuration:
      name: [name]-stage-minio-env-conf
    pools:
    - servers: 4
      name: [name]-stage-pool-0
      volumesPerServer: 1
      size: 200Gi
      storageClassName: topolvm-provisioner
      tolerations:
      - key: node-role.[name]-it.com/stage-minio
        operator: Exists
        effect: NoSchedule
      nodeSelector:
        node-role.[name]-it.com/stage-minio: "true"
    metrics:
      enabled: true
    buckets:
    - name: [name]-b2b-stage
    prometheusOperator: true
  ingress:
    api:
      enabled: true
      ingressClassName: traefik
      # annotations:
      #   cert-manager.io/cluster-issuer: letsencrypt-prd
      host: minio.stage.[name]-it.com
      tls:
      - hosts:
        - minio.stage.[name]-it.com
        secretName: minio-tls
    console:
      enabled: true
      ingressClassName: traefik
      annotations:
        cert-manager.io/cluster-issuer: letsencrypt-prd
      host: minio-console.stage.[name]-it.com
      tls:
      - hosts:
        - minio-console.stage.[name]-it.com
        secretName: minio-console-tls
```
---
### Add minio-server.yaml and minio-operator.yaml files in the apps directory


## Minio Operator
```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: minio-server
  namespace: argocd
spec:
  destination:
    namespace: minio-server
    server: "https://kubernetes.default.svc"
  project: default
  source:
    path: minio-server
    repoURL: [apps-repo-url]
    targetRevision: main
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
    - Validate=false
    - Prune=true
    - SelfHeal=true
    - ServerSideApply=true
```
## Minio Server
```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: minio-server
  namespace: argocd
spec:
  destination:
    namespace: minio-server
    server: "https://kubernetes.default.svc"
  project: default
  source:
    path: minio-server
    repoURL: [apps-repo-url]
    targetRevision: main
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
    - Validate=false
    - Prune=true
    - SelfHeal=true
    - ServerSideApply=true
```