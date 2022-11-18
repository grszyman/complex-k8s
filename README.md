# Docker and Kubernetes: The Complete Guide

The project was created based on a course [Docker and Kubernetes: The Complete Guide](https://www.udemy.com/course/docker-and-kubernetes-the-complete-guide)

# Table Of Contents

1. [Kubernetes Dashboard](#kubernetes-dashboard)
2. [Ingress Nginx](#ingress-nginx)
3. [Google Cloud](#google-cloud)

# Deployment Diagram

![](./docs/deployment.svg)

# Kubernetes Dashboard

## Configuration

* [Deploy and Access the Kubernetes Dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)

```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.6.1/aio/deploy/recommended.yaml
kubectl apply -f k8s-dashboard
```

## Access Token

```shell
kubectl -n kubernetes-dashboard create token admin-user
```

For macOS you can use:
```shell
kubectl -n kubernetes-dashboard create token admin-user | pbcopy
```

# Ingress Nginx

* [Studying the Kubernetes Ingress system](https://www.joyfulbikeshedding.com/blog/2018-03-26-studying-the-kubernetes-ingress-system.html)
* [Kubernetes Documentation /../ Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)

```shell
brew install helm
```

```shell
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```

# Google Cloud

* [Deploying to Google Kubernetes Engine](https://docs.github.com/en/actions/deployment/deploying-to-your-cloud-provider/deploying-to-google-kubernetes-engine)

## Google Cloud CLI

* [Install the Google Cloud CLI](https://cloud.google.com/sdk/docs/install-sdk)

```shell
brew install google-cloud-sdk
```

## Pricing Calculator

* [Google Cloud Pricing Calculator](https://cloud.google.com/products/calculator)

## Configuration

```shell
gcloud iam service-accounts create github-deployer
```
```shell
gcloud iam service-accounts list
```

```shell

GKE_PROJECT='...' && \
SA_EMAIL='...' && \

gcloud projects add-iam-policy-binding $GKE_PROJECT \
	--member=serviceAccount:$SA_EMAIL \
	--role=roles/container.admin && \
gcloud projects add-iam-policy-binding $GKE_PROJECT \
	--member=serviceAccount:$SA_EMAIL \
	--role=roles/storage.admin && \
gcloud projects add-iam-policy-binding $GKE_PROJECT \
	--member=serviceAccount:$SA_EMAIL \
	--role=roles/container.clusterViewer
```

```shell
gcloud iam service-accounts keys create key.json --iam-account=$SA_EMAIL
```

```shell
gcloud config set compute/zone us-central1-c && \
gcloud container clusters get-credentials complex-cluster
```

## Static IP and Domain Name

> Ingress Nginx doesn't support global IPs

```shell
gcloud compute addresses create complex-ip --global && \
gcloud compute addresses describe complex-ip --global
```

```shell
gcloud compute addresses create complex-regional-ip --region us-central1 && \
gcloud compute addresses describe complex-regional-ip --region us-central1
```

## Setup Helm Charts

* [NGINX Ingress or GKE Ingress?](https://medium.com/@glen.yu/nginx-ingress-or-gke-ingress-d87dd9db504c)
* [ingress-nginx](https://artifacthub.io/packages/helm/ingress-nginx/ingress-nginx#configuration) - Ingress controller
for Kubernetes using NGINX as a reverse proxy and load balancer
* [Configure domain names with static IP addresses](https://cloud.google.com/kubernetes-engine/docs/tutorials/configuring-domain-name-static-ip)
* [Installing with Helm](https://cert-manager.io/docs/installation/helm/#installing-with-helm)

```shell
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx && \
helm repo update && \
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --set controller.service.loadBalancerIP=`gcloud compute addresses describe complex-regional-ip --region us-central1 --format="value(address)"` && \
helm repo add jetstack https://charts.jetstack.io && \
helm repo update && \
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --set installCRDs=true && \
helm repo add mittwald https://helm.mittwald.de && \
helm repo update && \
helm upgrade --install kubernetes-secret-generator mittwald/kubernetes-secret-generator --namespace utils --create-namespace
```