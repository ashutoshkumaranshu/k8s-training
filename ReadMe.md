This lab tutorial is intended to lead you through the process how you can build a DevOps/GitOps platform on Kubernetes.
The goal is to have a cluster with TLS secured (cert-manager) cluster where services could be reached with their FQDNs (MetalLB + Traefik). Services could be installed in a GitOps way (Helm, ArgoCD). All the services could be reached through SSO (Keycloak OIDC) 

# Pre-requisites

To successfully achieve the goal you need a LAB environment. You can buy/rent one kubernetes solution from the cloud providers (AWS, GCP, Azure, DigitalOcean, etc) or build your own using virtualization.
In this tutorial I use Linux KVM and minikube to have a kubernetes cluster to build the whole environment see [Preparation](docs/preparation/ReadMe.md)

# Basic services

The basic services are installed manually using Helm charts see [Basic Services](docs/basicservices/ReadMe.md)
Basic services are running on minikube:
* MetalLB
* Traefik
* Cert-manager
* Keycloak

# GitOps service - ArgoCD

All other workloads are installed through ArgoCD using Helm based applications. All the charts are basic charts that uses the original chart as dependencies, eg.: my grafana chart is just a chart that points to the official grafana chart as dependencies and I specified the grafana chart values under the dependency "grafana:" chart (see the code in [Custom Helm Charts](custom-helm-charts/) )

## Other service installed by ArgoCD

* TBD

# ToDos

* Try to add a storage provider as basic service
* Add monitoring for all the basic services and ArgoCD herded services also