---
layout: post
title: "Implement containerized solutions"
date: 2025-01-28
---

**Learning path link: [AZ-204: Implement containerized solutions](https://learn.microsoft.com/en-us/training/paths/az-204-implement-iaas-solutions/)**

*Methodology: For each learning objective in each module, write a short summary demonstrating knowledge matching the objective as learned in the course.*

## Post-course review

*Course notes by module start below*

<hr/>

## Module 1: Manage container images in Azure Container Registry

#### Learning Objectives

- Explain the features and benefits Azure Container Registry offers
- Describe how to use ACR Tasks to automate builds and deployments
- Explain the elements in a Dockerfile
- Build and run an image in the ACR by using Azure CLI

### Container Registry Features and Benefits
- Managed container registry service based on open source Docker Registry 2.0
- Used to manage and store container images and related artifacts
- Can be used weith existing pipelines, or can use Container Registry Tasks to uild container images in Azure
- On demand or full automation with triggers available

#### Use Cases:
- Scalable orchestration systems that manage containerized applications across clusters of hosts including K8s, DC/OS and Docket Swarm
- Azure services that support building and running apps at scale inlcuding Azure K8s Service (AKS), App Service, Batch, Service Fabric

Devs can push to a container registry as part of a container development workflow using a CI tool like Azure Pipelines.

> Configure ACR Tasks to automatically rebuild application images when their base images are updated, or automate image builds when your team commits code to a Git repository. Create multi-step tasks to automate building, testing, and patching multiple container images in parallel in the cloud.

Service tiers:
- Basic: Cost optimized entrypoint for learning. Same capabilities as bigger plans, with lower includede storage and image throughput. 
- Standard: Same capabilities as Basic, with increased included storage and image throughput. Good for most.
- Premium: Highest storage, image throughput and concurrent operations. 

When images are grouped in repo, each image is a read-only snapshot of a Docker-compatible containery. Can be Windows or Linux. Stores related content formats like Helm charts as well.

ACR tasks can be used to streamline buidling, testing, publishing and deploying to Azure. 

> Configure build tasks to automate your container OS and framework patching pipeline, and build images automatically when your team commits code to source control.

**Capabilities |**
- Encryption at rest
- Regional storage (helps meet data residency and compliance requirements), geo replication available
- Geo replication available in premium registries

### ACR Tasks to automate builds and deployments

### Elements in a Dockerfile

### Build and run an image using Azure CLI