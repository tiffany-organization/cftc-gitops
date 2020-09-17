# Containers from the Couch: Intro to GitOps

The contents of this demo closely follows the Weaveworks + AWS [Introduction to GitOps on EKS](https://weaveworks-gitops.awsworkshop.io/22_workshop_1.html).

The EKS cluster used for this demo was created from the `cluster.yaml` file found at the root of this repository.

```sh
eksctl create cluster -f cluster.yaml
```

The `cluster.yaml` file specifies the creation of a single node EKS cluster with additional IAM policies to allow for the creation and usage of an ALB Ingress Controller.

Additionally, we enable this cluster for GitOps by using:

```sh
eksctl enable repo \
    --git-url git@github.com:tiffany-organization/cftc-gitops \
    --git-email tiffanywang3@users.noreply.github.com \
    --cluster gitops-tw \
    --region eu-central-1
```

