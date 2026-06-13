# Installation

## Prerequisites
- A Kubernetes cluster
- Helm v3
- kubectl access to the cluster
## Installation Steps
1. Create a repository for the cluster apps, including the ArgoCD, e.g. [name]-stage-cluster-apps.
2. Create a directory for the ArgoCD installation, e.g. [name]-stage-cluster-apps/argocd.
3. Create a file named Chart.yaml in the argocd directory with the following content:
```
apiVersion: v2
name: "argocd"
description: "An umbrella chart for installing ArgoCD"
type: application
version: 0.1.0
dependencies:
  - name: argo-cd
    version: 5.x
    repository: "https://argoproj.github.io/argo-helm"
```
4. Create a file named values.yaml in the argocd directory with the following content:
```
argo-cd:
  configs:
    params:
      server.insecure: true
    cm:
      url: https://[argocd-domain]
      exec.enabled: true
      params:
        server.enable.gzip: true
    rbac:
      policy.csv: |
        p, role:readonly, applications, get, */*, allow
        p, role:readonly, certificates, get, *, allow
        p, role:readonly, clusters, get, *, allow
        p, role:readonly, repositories, get, *, allow
        p, role:readonly, projects, get, *, allow
        p, role:readonly, accounts, get, *, allow
        p, role:readonly, gpgkeys, get, *, allow
        p, role:readonly, logs, get, */*, allow

        p, role:admin, applications, create, */*, allow
        p, role:admin, applications, update, */*, allow
        p, role:admin, applications, delete, */*, allow
        p, role:admin, applications, sync, */*, allow
        p, role:admin, applications, override, */*, allow
        p, role:admin, applications, action/*, */*, allow
        p, role:admin, applicationsets, get, */*, allow
        p, role:admin, applicationsets, create, */*, allow
        p, role:admin, applicationsets, update, */*, allow
        p, role:admin, applicationsets, delete, */*, allow
        p, role:admin, certificates, create, *, allow
        p, role:admin, certificates, update, *, allow
        p, role:admin, certificates, delete, *, allow
        p, role:admin, clusters, create, *, allow
        p, role:admin, clusters, update, *, allow
        p, role:admin, clusters, delete, *, allow
        p, role:admin, repositories, create, *, allow
        p, role:admin, repositories, update, *, allow
        p, role:admin, repositories, delete, *, allow
        p, role:admin, projects, create, *, allow
        p, role:admin, projects, update, *, allow
        p, role:admin, projects, delete, *, allow
        p, role:admin, accounts, update, *, allow
        p, role:admin, gpgkeys, create, *, allow
        p, role:admin, gpgkeys, delete, *, allow
        p, role:admin, exec, create, */*, allow

        p, role:api-cicd, applications, get, */*, allow
        p, role:api-cicd, applications, sync, */*, allow
        p, role:api-cicd, applications, action/batch/CronJob/create-job, */*, allow
        p, role:api-cicd, applicationsets, get, */*, allow
        p, role:api-cicd, applicationsets, create, */*, allow

        p, role:dev-admin, applications, *, [name]*/*, allow
        p, role:dev-admin, applicationsets, *, [name]*/*, allow
        p, role:dev-admin, repositories, *, [name]*/*, allow 
        p, role:dev-admin, projects, *, [name]*, allow
        p, role:dev-admin, accounts, get, [name]*, allow
        p, role:dev-admin, exec, create, [name]*/*, allow

        p, role:devs, applications, get, [name]*/*, allow
        p, role:devs, applications, sync, [name]*/*, allow
        p, role:devs, applications, action/batch/CronJob/create-job, [name]*/*, allow
        p, role:devs, applicationsets, get, [name]*/*, allow
        p, role:devs, certificates, get, [name]*/*, allow
        p, role:devs, clusters, get, [name]*/*, allow
        p, role:devs, repositories, get, [name]*/*, allow
        p, role:devs, projects, get, [name]*/*, allow
        p, role:devs, logs, get, [name]*/*, allow
        p, role:devs, applications, *, [name]*/*, allow

        g, role:admin, role:readonly
        g, admin, role:admin
        g, demo, role:readonly
        g, SuperAdmins, role:admin
        g, Developers, role:devs
        g, Admins, role:dev-admin

  server:
    replicas: 1
    ingress:
      # -- Enable an ingress resource for the Argo CD server
      enabled: true
      # -- Additional ingress annotations
      annotations:
        cert-manager.io/cluster-issuer: letsencrypt-prd

      # -- Additional ingress labels
      labels: {}
      # -- Defines which ingress controller will implement the resource
      # -- If empty, if any ingress controller is marked as default, it will be used
      ingressClassName: "traefik"

      hosts:
      - [argocd-domain]

      paths:
      - /
      pathType: ImplementationSpecific
      # -- Additional ingress paths
      extraPaths: []
      # - path: /*
      #   pathType: Prefix
      #   backend:
      #     service:
      #       name: ssl-redirect
      #       port:
      #         name: use-annotation

      # -- Ingress TLS configuration
      tls:
      - secretName: argocd-tls
        hosts:
        - [argocd-domain]

      # -- Uses `server.service.servicePortHttps` instead `server.service.servicePortHttp`
      https: false

  global:
    env:
      - name: HTTP_PROXY
        value: http://proxy.sample:12889
      - name: HTTPS_PROXY
        value: http://proxy.sample:12889
      - name: http_proxy
        value: http://proxy.sample:12889
      - name: https_proxy
        value: http://proxy.sample:12889
      - name: NO_PROXY
        value: argocd-repo-server,argocd-application-controller,argocd-metrics,argocd-server,argocd-server-metrics,argocd-redis,argocd-redis-ha-haproxy,argocd-dex-server,localhost,10.0.0.0/8,*.neor.space,*.ir

  redis-ha:
    enabled: false
    image:
      repository: docker.io/redis
    haproxy:
      image:
        repository: docker.io/haproxy

  controller:
    replicas: 1

  repoServer:
    replicas: 1

  applicationSet:
    replicas: 1
```
5. Install the ArgoCD using the following command:
```
cd [name]-stage-cluster-apps/argocd
helm dependency build
helm install argocd . -n argocd --create-namespace
```
6. Find the ArgoCD server password using the following command:
```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```
7. Access the ArgoCD server using the following command:
```
kubectl port-forward svc/argocd-server -n argocd 8080:80
```
8. Login to the ArgoCD server using ```admin``` as the username and the password from step 6.
