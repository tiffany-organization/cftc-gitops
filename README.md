# Intro to GitOps

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

For this demo, we will be updating Flux's poll interval to 1 minute, by adding an additional argument in `flux/flux-deployment.yaml`

```yaml
- --git-poll-interval=1m
```

## ALB Ingress Controller

For this demo, we'll be deploying an ALB Ingress Controller for our Sock Shop demo Ingress resource.

## Application Demos

In order to demonstrate some sample microservices, we look to open source examples. Today we'll be using Weaveworks's Sock Shop microservice and Stefan Prodan's Podinfo.

### Sock Shop

The manifests for the Sock Shop microservice are included under `demo-apps/sockshop`.

We can navigate to the Sock Shop in our browser by grabbing the [Ingress address](http://c5d56887-sockshop-frontend-c5a2-567432663.eu-central-1.elb.amazonaws.com):

```sh
kubectl get ingress -n sock-shop
```

You'll notice that we can see a catalogue of socks, displayed both on the home page and on the Catalogue page.

While we did deploy these manifests ahead of our demo to save on some time, all we had to do was add the resource manifests to our Git Repository.

The Flux agent is monitoring this repository, and it deployed the Sock Shop manifests on our behalf.

Flux uses its git tag in this repository (flux) to keep track of what components it has already deployed, and what resources were updated or added that it needs to act upon.

What you see in the Git Repository is what is reflected in our EKS cluster.

If, for example, we were to manually scale down our `catalogue` deployment, we might accomplish this by running:

```sh
kubectl scale --replicas=0 deployment/catalogue -n sock-shop
```

We'd expect that this stops our `catalogue` pod from running.

Let's verify that this happened:

```sh
kubectl get pods -n sock-shop
```

While we see our `catalogue-db` pod, we don't see our `catalogue` pod. This change is reflected in our Sock Shop site -- we no longer have our catalogue of socks!

However, since this change was made manually, and the current state of our cluster is now out of sync with the desired state we've defined in Git, Flux will reconcile this change on our behalf.

Since we've set Flux's poll interval to 1m, we should expect to see the `catalogue` pod come back shortly.

### Podinfo

For our podinfo deployment, we will be demonstrating a very simple HelmRelease.

```yaml
---
apiVersion: helm.fluxcd.io/v1
kind: HelmRelease
metadata:
  name: podinfo
  namespace: default
spec:
  chart:
    repository: https://stefanprodan.github.io/podinfo
    name: podinfo
    version: 3.2.0
```

The HelmRelease above specifies usage of the Podinfo Helm Chart, and will deploy our podinfo pod in the default namespace.

We can take a look at the podinfo HelmRelease that was deployed by running:

```sh
kubectl get hr
```

And as with Kubernetes resources, you can describe it as well:

```sh
kubectl describe hr
```

The HelmRelease is a Custom Resource Definition (CRD) in `flux/crds.yaml`.
