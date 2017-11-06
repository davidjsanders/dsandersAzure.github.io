---
layout: post
title: Why Kubernetes is the right choice for containers
date: 2017-11-06 11:00:00 +0000
categories:
- K8S
- Kubernetes
- Google
- Microsoft
- Azure
- GKS
- AKS
- Hobby App
---

#### tl;dr: The flexibility and portability of Kubernetes makes it *incredibly* simple to run containerized workloads on-premises or on any cloud; managed Kubernetes services like AKS and GKE abstract the complexity from devs and ops while enabling the right to switch service providers on a dime! While other clustering solutions are available (e.g. Docker Swarm, DC/OS, etc.), K8S makes it *so simple* it requires much less cognitive load to learn and optimize.

![Hard disk image](/uploads/2017/11/06/k8s_blue.png)

### Why Kubernetes (K8S) is winning container scheduling and orchestration
So, you may (or may not) know that Microsoft joined the K8S first world with a new managed Azure Kubernetes Services (AKS) which aims to abstract the management layer and make it “…easier to manage and operate your Kubernetes environments, all without sacrificing portability”([1](https://azure.microsoft.com/en-us/blog/introducing-azure-container-service-aks-managed-kubernetes-and-azure-container-registry-geo-replication/)). 
Like the great techie I am, I’ve been investigating and researching the new service and my first take is it’s a great effort impacted by (some unforeseen?) high demand, leading to performance issues which realistically prevent me using it in anger. The solution isn't really ready
for primetime yet (it is still tagged beta, although I think alpha may be more realistic.)
So, why does this make Kubernetes the right choice for containerization? Let me explain:
To spin up AKS it's a really simple set of commands:
```
az group create --name groupname --location westus2

az aks create \
  --resource-group groupname \
  --name groupnameCluster \
  --generate-ssh-keys \
  --dns-name-prefix myk8s \
  --kubernetes-version 1.8.1 \
  --agent-count 2 \
  --agent-vm-size Standard_A2

az aks get-credentials --resource-group groupname --name groupnameCluster
```
I can then spin up my applications and services with a simple set of commands that works on *any* K8S installation:
```
kubectl create configmap cowbull-config --from-file=cowbull.cfg
kubectl create -f scripts/redis-service.yml
kubectl create -f scripts/redis-deploy.yml
kubectl create -f scripts/cowbull-service.yml
kubectl create -f scripts/cowbull-deploy.yml
kubectl create -f scripts/webapp-deploy.yml
```
And that's it. Really simple, right? **BUT**, as I said, AKS is not *really* working well for me just now (see my GitHub issues), so what if I want to move it to another managed K8S service, like Google Container Engine (GKE):
```
gcloud container clusters create clustername \
      --disk-size 100 \
      --zone us-east-4a \
      --enable-cloud-logging \
      --enable-cloud-monitoring \
      --machine-type n1-standard-1 \
      --num-nodes 3
gcloud container clusters get-credentials clustername --zone=us-east-4a
```
Thats it. All of my ``kubectl`` commands work exactly the same against GKE as AKS. Nothing changes in the app - the configuration map, pods, services, and external IPs are created in Kubernetes without any other vendor specific commands.

Those who know me, know I'm careful with cash and I don't really like to spin up cloud architectures for continuous usage (a.k.a. charges) when my linux laptop is readily available at my side, so what about locally running my K8S app?
```
minikube start --memory 4096 --cpus 2
kubectl config use-context minikube
```
Everything now works exactly the same, but on my laptop; I have no load balancer so won't get an external IP, but it's the same K8S. This flexibility to take my Docker containerized apps and run them on any cluster on any cloud provider is why I think Kubernetes is going to win the container orchestration and scheduling contest. It's just **so** easy to use.

David

david@davidjsanders.com 

#### References
1. Monroy, G (24 Oct 2017), *Introducing AKS (managed Kubernetes) and Azure Container Registry improvements*.
Available at: [https://azure.microsoft.com/en-us/blog/introducing-azure-container-service-aks-managed-kubernetes-and-azure-container-registry-geo-replication/](https://azure.microsoft.com/en-us/blog/introducing-azure-container-service-aks-managed-kubernetes-and-azure-container-registry-geo-replication/)
(Accessed: 6 November 2017)
2. Kubernetes (12 Oct 2016), *Kubernetes Logo*. Available at: [https://github.com/kubernetes/kubernetes/tree/master/logo](https://github.com/kubernetes/kubernetes/tree/master/logo) (Accessed: 6 November 2017)