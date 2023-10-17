This lab tutorials are intended to lead you through the process how you can build different infrastructure (DevOps/GitOps platform, etc.) on Kubernetes.

# Pre-requisites

To successfully achieve the goal you need a LAB environment. You can buy/rent one kubernetes solution from the cloud providers (AWS, GCP, Azure, DigitalOcean, etc) or build your own using virtualization.
In this tutorials we use Linux KVM and minikube to have a kubernetes cluster and build different lab environments, see [Preparation](docs/preparation/ReadMe.md)

# Required basic knowledge

* Linux command line

# Basic Infrastructure

The basic services are installed manually using Helm charts 
Basic services are:
* MetalLB
* Traefik
* Cert-manager
* Keycloak
* ArgoCD

The installation of these components is on [Preparation](docs/preparation/ReadMe.md) page

# GitOps service - ArgoCD

All other workloads are installed through ArgoCD. All the charts are basic charts that uses the original chart as dependencies, eg.: my grafana chart is just a chart that points to the official grafana chart as dependencies and we specified the grafana chart values under the dependency "grafana:" chart (see the code in [Custom Applications](custom-apps/) )

## Lab environments

* [Building Kubernetes Monitoring](docs/01_monitoring_lab/ReadMe.md)
* [Building Storage System](docs/02_storage_lab/ReadMe.md)

# ToDos

* Try to add a storage provider as basic service (rook, minio or something that could be run outside also)