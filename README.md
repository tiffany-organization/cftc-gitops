# Containers from the Couch: Intro to GitOps

The contents of this demo closely follows the Weaveworks + AWS [Introduction to GitOps on EKS](https://weaveworks-gitops.awsworkshop.io/22_workshop_1.html).

## Cluster Creation

The EKS cluster used for this demo was created from the `cluster.yaml` file found at the root of this repository.

```sh
eksctl create cluster -f cluster.yaml
```

The `cluster.yaml` file specifies the creation of a single node EKS cluster with additional IAM policies to allow for the creation and usage of an ALB Ingress Controller.

## Enable GitOps

Additionally, we enable GitOps in this cluster by using:

```sh
eksctl enable repo \
    --git-url git@github.com:tiffany-organization/cftc-gitops \
    --git-email tiffanywang3@users.noreply.github.com \
    --cluster gitops-tw \
    --region eu-central-1
```

`eksctl enable repo` performs a few actions on your behalf:

* Generates Flux & Helm Operator manifests
* Adds those manifests to your specified repository in the `flux` folder
* Applies those manifests to deploy them to your cluster in the `flux` namespace
* Prints Flux's public SSH key for you to add to your repository as a Deploy Key

## ALB Ingress Controller

For this demo, we'll be deploying an ALB Ingress Controller for our Sock Shop demo Ingress resource.

## Application Demos

In order to demonstrate some sample microservices, we look to open source examples. Today we'll be using Weaveworks' Sock Shop microservice and Stefan Prodan's Podinfo.
