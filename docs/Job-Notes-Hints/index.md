> ⚠️ Note: These are internal notes primarily for personal use and may include shorthand, assumptions, or environment-specific details.

## Table of Contents
- [Monitoring](#monitoring)
- [Deployment](#deployment)
- [Kubernetes](#kubernetes)
- [Networking](#networking)
- [Troubleshooting](#troubleshooting)
- [Common Commands](#common-commands)
- [Miscellaneous](#miscellaneous)

# Monitoring

**Problem:** Alertmanager shows cluster status as "disabled"  
**Solution:** For fixing alertmanager cluster status disable problem in alertmanager UI, you should increase the replica to upper than 2

**Problem:** Some Dashboards showing no data or anything else  
**Solution:** You should check json of the dashboard to check expression, maybe some expressions dont't match with ours, check with prometheus (Query that)

# Deployment

**Title:** Connecting apps (projects) in the gitlab to argocd  
**Solution:** For connecting apps (projects) in the gitlab to argocd, we should at first create a app in argocd (apps of apps) an apps of project, apps of project is for creating the project for the apps in the argocd, and apps of apps is for creating the group of apps (front-end ann back-end cluster apps).

# Kubernetes

**Problem:** K3S Says k8s not up and running  
**Solution:** export KUBECONFIG=/etc/rancher/k3s/k3s.yaml

**Problem:** Nodelocaldns will be going into loop  
**Solution:** Don't forget to set upstream DNS (8.8.8.8, 1.1.1.1) in the all.yaml for kubespray to prevent the nodelocaldns loop problem.

**Problem:** Reinstall cluster  
**Solution:** If you fucked up and you want to install the cluster again on that nodes, at first you have to delete the lvm volumes and second you have to delete the namespaces. (At first, you should delete the apps that have a connection to pv)

**Problem:** Packaging helm charts  
**Solution:** If a helm chart have a problem with ip of iran, we helm pull it to generate .tgz (helm dependency update . and helm package .). after that push that to harbor, with helm push command. (Be carefull about your helm dependency.)

**Problem:** Increase size of a pvc that connected to a statefullset  
**Solution:** If you want to increase size of a pvc that connected to a statefullset, you can edit the value of that and fix it, at first you have to edit manually in pvc editor (in argocd or kubernetes), then after updating that you should delete the statefullset in 'non cascade way'.

**Problem:** Tls getting problem in argocd (for apps)  
**Solution:** You should open port 80 and 443 in firewall for tls (if you have pfsense or anything else)

**Problem:** Sentry's redis problem  
**Solution:** If the redis of sentry degraded and want resource tells us OOMKILLED, and if give resource but it doesnt't work, one way is to delete the redis-replica-pvc and it will be fixed  

# Common Commands

**Problem:** Set a policy for harbor to delete image  
**Solution:** If we want to set a policy for harbor to delete old images, at first we should go to that project, then on the policy section, we should add (retain the most recently pushed #artifacts) and (retain the most recently pulled #artifacts) with the count of 5 and uncheck untagged artifacts (and dryrun, then run)
with this, the harbor put the images in the trash if we want to delete them from the disk, we go to clean up and set the garbage collection schedule to GC to hourly, and then GC now.

# Miscellaneous

**Title:** Get backup from jira  
**Solution:** If you want to get backup from jira, at first get backup from the jira system panel, the get a full backup from the PVC


**Problem:** Harbor Docker Cache problem  
**Solution:** If we have problem with harbor docker cache, (The ci or anything couldn't pull the image and say not found) use the full image tag, for example use python 3.9.1-slim instead of python:3.9


**Title:** Creating user for argocd  
**Solution:** If you want to create a readonly user in argocd, at first you should change the cm section in the helm chart of argocd, then you should exec to the pod of argocd (argocd-server) and after that change the password of the cluster.

**Title:** Creating access-token for gitlab for argocd  
**Solution** For creating access-token for gitlab to connecting in the argocd, for the cicdbot user, we should create a access-token in a way that we go to admin area and the cicdbot user and after that, impersonate the user and go to edit profile to make an access-token