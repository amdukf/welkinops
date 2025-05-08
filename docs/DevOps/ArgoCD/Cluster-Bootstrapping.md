# Cluster Bootstrapping
## Overview
This document provides the steps to bootstrap the cluster for ArgoCD installation. We will use argocd app of apps pattern to install the required components for the cluster like ingress-controller.

# Prerequisites
- All the steps in the Installation document should be completed.
- The cluster should be up and running.
- If topolvm is going to use as the storage class, worker nodes must have a topolvm volume group, first check with ```lsblk``` and then ```vgcreate topolvm /dev/sdX``` that sdX is that partition that you want to assign, you should change it, its better to run this commands before helm upgrade **and be aware that the size of the PVs of the apps you make should be proportionate.**
## Steps
1. Clone the apps repository
2. Change the directory to the argocd directory
3. Create these directories and files:
- ```templates```
   - ```apps``` directory
     - argocd.yaml
     - ```cluster-apps.yaml```

4. Create the argocd.yaml file with the following content:
```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: argocd
  namespace: argocd
spec:
  destination:
    namespace: argocd
    server: "https://kubernetes.default.svc"
  project: default
  source:
    path: argocd
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
```
1. Create the cluster-apps.yaml file with the following content:
```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: cluster-apps
  namespace: argocd
spec:
  destination:
    namespace: argocd
    server: "https://kubernetes.default.svc"
  project: default
  source:
    path: apps
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
```
1. Before applying the argocd.yaml and cluster-apps.yaml files, replace the [apps-repo-url] with the actual apps repository URL.

2. Now we should use helm upgrade to create new applications in the ArgoCD but before that, we need to create a secret for the apps repository, so ArgoCD can access the repository. Create a file in secret main directory with the following content (you should ignore that directory with gitignore):
```
apiVersion: v1
kind: Secret
metadata:
  name: org-repo-creds
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: repo-creds
stringData:
  url: https://[your-org-repo-url]
  type: git
  password: [access-token]
  username: ouath2
```
For the access token, if you are using github as repository, you have to generate a ```fine-grained token``` and add your team in Resource owner. choose the Only select repositories in the Repository access and select the repositories that you want to use, then in the Permissions, just select Contents and give the read permission.

In your git provider create an access token with the repository access scope for all the repositories under the organization that you want to use. Replace the [your-org-repo-url] with the actual organization git URL and [access-token] with the actual access token. for example:
```
stringData:
  url: https://gitlab.name.example.space/devops
  type: git
  password: glpat-1234567890
  username: ouath2
```
Apply the secret file:
```
kubectl apply -f [path-to-secret-file]
```
if you are using harbor, you should make 2 secrets:
```
apiVersion: v1
kind: Secret
metadata:
  name: harbor-psql-auth
  namespace: harbor
stringData:
  postgres-password: xxxxxxxxxxxxxx 
  password: xxxxxxxxxxxxxx
  replication-password: xxxxxxxxxxxxxxxx
```
```
apiVersion: v1
kind: Secret
metadata:
    name: harbor-secret-key
    namespace: harbor
stringData:
    secretKey: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```
actually you can generate the password and secretkeys with this (you can change the length if you want):
```
openssl rand -base64 56 | tr -d '+/=' | cut -c 1-42
```
Now we can use helm upgrade to create the applications:
```
  helm upgrade argocd . -n argocd
```
and check the applications that made in the cluster:
```
kubectl get applications -A
```
After the installation is completed, every app manifest in the apps directory will be installed in the cluster by the ArgoCD.
Note: Then you can install what is needed for the cluster like the ingress-controller, cert-manager, etc. in the apps directory and commit the changes to the apps repository. ArgoCD will install the new applications in the cluster automatically.

## Conclusion
After completing the steps in this document, the cluster will be bootstrapped with the required components that you choose and the ArgoCD will manage the applications and itself in the cluster.