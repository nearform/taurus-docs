# Taurus on Azure

Deploy your web application to Azure with ease using __Taurus__.

[__Source Repository__][taurus-azure-repo]

## High-Level Architecture
The diagram below displays Taurus high-level architecture. The backend consists of a Virtual Private Cloud (VPC) containing an Application Load Balancer (ALB) and Amazon Elastic Kubernetes Service (EKS). The frontend consists of CloudFront and Amazon Simple Storage Service (S3).

![High level architecture][high-level-architecture]
Fig.1 Taurus High-Level Architecture

Taurus provides you with the following capabilities:
- Provision Azure.
- Install Kubernetes add-ons.

## Azure Provisioning
Taurus focuses on the main infrastructure components and we expect you will extend it. That's why it's called a boilerplate.

The main Taurus components are:
- Networking (VPC)
- Kubernetes (EKS, add-ons, and Identity and Access Management (IAM) roles)
- Database (Amazon Relational Database Service (RDS))
- Static website (S3, CloudFront and Secure Sockets Layer(SSL))

## Kubernetes Add-Ons
As with infrastructure, Taurus focuses on the necessary Kubernetes add-ons. You need to install the following Kubernetes add-ons:
- Helm
- AAD Pod Identity
- Metrics server
- Application Gateway Ingress Controller
- ExternalDNS

Each add-on is described in more detail below. Details on how to install these add-ons are available in the following section [Install Kubernetes Add-Ons].

### Helm 
Helm is a tool that streamlines installing and managing Kubernetes applications.

For more information on Helm charts, refer to [The Chart Template Developer's Guide](https://docs.helm.sh/chart_template_guide/#the-chart-template-developer-s-guide).

### AAD Pod Identity

AAD Pod Identity provides AD credentials to containers running inside a Kubernetes cluster based on annotations.

This enables fine-grained pod permissions instead of having to amalgamate them
into a role with all the permissions of all the apps deployed in the cluster.

For more information and usage refer to [AAD Pod Identity] on GitHub.

### Metrics Server

The metrics server enables cluster-wide metrics collection from Kubernetes resources, for example, container CPU and memory usage. You also need the metrics server to enable Horizontal Pod Autoscaling (HPA).

For more information, refer to [Kubernetes Metrics Server] on GitHub.

Note: Metrics server replaces the deprecated Heapster metrics collector.

### Application Gateway Ingress Controller
An Ingress is configured to give services externally-reachable URLs, load balance traffic, terminate SSL or Transport Layer Security (TLS), and offer name-based virtual hosting. The Ingress controller is responsible for fulfilling the Kubernetes Ingress by provisioning the required Application Gateway machinery.

For more information, refer to [Application Gateway Ingress Controller] documentation.

### ExternalDNS

ExternalDNS auto-synchronises exposed Kubernetes Services and Ingresses with DNS providers.

ExternalDNS is not a DNS server itself, but instead configures other DNS providers, for example, Azure Route 53. It allows you to control DNS records dynamically via Kubernetes resources in a DNS provider-agnostic way.

For more information refer to [Kubernetes ExternalDNS] on GitHub.

# Explore Taurus
The quickest way to explore Taurus is to view our Quick Start Guide. It covers cloning and pulling Taurus locally. It describes the prerequisites, Azure provisioning and how to install Kubernetes add-ons.

- Go to the [Quick Start Guide].

<!-- Internal Links -->
[high-level-architecture]: img/high-level-architecture.jpg
[Install Kubernetes Add-Ons]:/azure/helm/
[Quick Start Guide]:/azure/quick-start/

<!-- External Links -->
[taurus-azure-repo]: https://github.com/nearform/taurus-azure

[AAD Pod Identity]: https://github.com/Azure/aad-pod-identity
[Kubernetes Metrics Server]:https://github.com/kubernetes-sigs/metrics-server
[Application Gateway Ingress Controller]: https://docs.microsoft.com/en-us/azure/application-gateway/ingress-controller-overview
[Kubernetes ExternalDNS]: https://github.com/kubernetes-sigs/external-dns
