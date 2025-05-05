---
layout: post
title: "Implement containerized solutions"
date: 2025-03-05
---

**Learning path link: [AZ-204: Implement containerized solutions](https://learn.microsoft.com/en-us/training/paths/az-204-implement-iaas-solutions/)**

*Methodology: For each learning objective in each module, write a short summary demonstrating knowledge matching the objective as learned in the course.*

## Post-course review

This was an extremely long course (by lines the longest yet) and covered another service that I don't use at work, though I wanted to learn more about it. To summarize, this module covered the main Azure services related to containerized solutions: 
1. Azure Container Registry: Store/manage container images and related artifacts
2. Azure Container Instances: Run container in Azure without having to manage VMs or other service infrastructure
3. Azure Container Apps: 
The only service not covered was AKS (Azure Kubernetes Service) which seems to have been outside the scope of this learning path. ACI seems like it is the choice for simpler, short running tasks while Container Apps are more suited for apps that need orchestration and have persistent workloads and more advanced deployment strategies. I see the benefits in portability, consistency and environmnet control offered by containerized solutions but for the most part in my org, PaaS solutions like App Service or Function Apps are sufficient for our needs. This is another module where I'm glad I learned the basics but I don't see it being used in my day to day.

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
- Geo replication available in premium registries. Guard against regional failure, network-close for faster push/pulls in development
- Zone redundancy in Premium tier, uses Azure AZs to replicate registry to 3 zones in each region
- Can create as many repos, images, layers, tags as needed up to registry storage limit, though large numbers of those can impact performance

### ACR Tasks to automate builds and deployments

ACR Tasks:
- Provide cloud-based container image building for Linux, Windows and ARM
- Extend early parts of app development cycle to cloud with on-demand container image builds
- Enable automatic builds triggered by source code updates, updates to base images or timers

> Each ACR Task has an associated source code context - the location of a set of source files used to build a container image or other artifact. Example contexts include a Git repository or a local filesystem.

Task scenarios:
- Quick task: build and publish a single container image to a container on demand, in Azure without needing a local Docker. (Like `docker build`, `docker push` for the cloud)
  - Verify your automated build definitions and catch potential problems before committing your code
  - Offloading your container image builds to Azure
- Automatically triggered tasks: build image on source code update, base image update or on schedule
  - Source code update: use Azure CLI command `az acr task create` and specify a Git repository and optionally a branch and Dockerfile
  - Base image update: Build application images when base image changes in the registry or a public repo
  - Schedule: Timer triggers on a defined schedule
- Multi-step task: Extend a single image build and push capability with multi step, multi container workflows. These are defined in a YAML file. Example automation: 
  1. Build a web application image
  2. Run the web application container
  3. Build a web application test image
  4. Run the web application test container, which performs tests against the running application container
  5. If the tests pass, build a Helm chart archive package
  6. Perform a `helm` upgrade using the new Helm chart archive package

Can use the platform tag to build Windows images or linux imgages for other architectures like `--platform Linux/arm64/v8`. Options:
- Linux: AMD64, Arm, Arm64, 386
- Windows: AMD64

### Elements in a Dockerfile
> A Dockerfile is a script that contains a series of instructions that are used to build a Docker image

They include the following info:
- Base or parent image to create the new image
- COmmands to update the base OS or install other software
- Build artifacts to include, like a developed application
- COmmand to run when container is launched

**Sample .NET Dockerfile |**

```Dockerfile
# Use the .NET 6 runtime as a base image
FROM mcr.microsoft.com/dotnet/runtime:6.0

# Set the working directory to /app
WORKDIR /app

# Copy the contents of the published app to the container's /app directory
COPY bin/Release/net6.0/publish/ .

# Expose port 80 to the outside world
EXPOSE 80

# Set the command to run when the container starts
CMD ["dotnet", "MyApp.dll"]
```

### Build and run an image using Azure CLI
**Create a container registry |**
Create resource group: `az group create --name az204-acr-rg --location <myLocation>`
Create Basic tier container registry: `az acr create --resource-group az204-acr-rg --name <myContainerRegistry> --sku Basic`

**Build and push image from Dockerfile |**
Create a docker file based on MS hello-world image: `echo FROM mcr.microsoft.com/hello-world > Dockerfile`
Build image and push to registry: `az acr build --image sample/hello-world:v1 --registry <myContainerRegistry> --file Dockerfile .`

**Verify results |**
List repos in registry: `az acr repository list --name <myContainerRegistry> --output table`
List tags on the repository: `az acr repository show-tags --name <myContainerRegistry> --repository sample/hello-world --output table`

**Run image in the ACR |**
Run container image from your container registry: `az acr run --registry <myContainerRegistry> --cmd '$Registry/sample/hello-world:v1' /dev/null`
`--cmd` parameter above runs container in default config, but supports other docker run params or other docker commands.

**Clean up |**
Delete rg: `az group delete --name az204-acr-rg --no-wait`


## Module 2: Run container images in Azure Container Instances

#### Learning Objectives

- Describe the benefits of Azure Container Instances and how resources are grouped
- Deploy a container instance in Azure by using the Azure CLI
- Start and stop containers using policies
- Set environment variables in your container instances
- Mount file shares in your container instances

### Benefits of Azure Container Instances 
> Azure Container Instances (ACI) offers the fastest and simplest way to run a container in Azure, without having to manage any virtual machines and without having to adopt a higher-level service.
Benefits:
- Start containers in seconds
- Expose container groups to internet with IP address and fully qualified domain name
- Isolate as completely as it would be in VM (security)
- ACI stores minimum customer data required to ensure container groups run as expected
- Persistent storage for Azure File shares mounted to container
- Use same API for both Windows and Linux

Azure Kubernetes Service is reccomended if full container orchestration across multiple containers is needed.

Top level resource in ACI is a container group, which contains sontainers scheduled on same host machine. Containers in group share lifecycle, resources, local network and storage.

Example container group with multiple containers:
![Example container group with multiple containers](assets/images/container-groups-example.png)

- Is scheduled on a single host machine.
- Is assigned a DNS name label.
- Exposes a single public IP address, with one exposed port.
- Consists of two containers. One container listens on port 80, while the other listens on port 5000.
- Includes two Azure file shares as volume mounts, and each container mounts one of the shares locally.

Deployment uis done with a Resource Manager template or YAML file. RM template reccomended when deploying more resources with the container, YAML better when its just the container instance.

ACI allocates CPUs, memory and optionally GPUs to a container group based on teh resrouces requests on instances in the group.

Groups share in IP, must expose a port for clients to reach. Port mapping is not supported because port namespace is shared. Containers in group can use localhost on exposed ports to communicate with eachother.

External volumes (Azure file share, Secret, Empty directory, Cloned git repo) can be specified to mount to specific paths within individual containers in a group.

> Multi-container groups are useful in cases where you want to divide a single functional task into a few container images. These images might be delivered by different teams and have separate resource requirements.

An example group might contain an application container and a logging container, where the logging container collects the logs and metrics output by the app and writes them to longterm storage. Or a frontend container and backend container.


### Deploy an instance with Azure CLI

1. Create a resource group using `az group create`
2. Create a container
```
DNS_NAME_LABEL=aci-example-$RANDOM
az container create --resource-group az204-aci-rg \
    --name mycontainer \
    --image mcr.microsoft.com/azuredocs/aci-helloworld \
    --ports 80 --os-type Linux --cpu 1 --memory 1 \
    --dns-name-label $DNS_NAME_LABEL --location <myLocation> 
```
3. Verify container
```
az container show --resource-group az204-aci-rg \
    --name mycontainer \
    --query "{FQDN:ipAddress.fqdn,ProvisioningState:provisioningState}" \
    --out table
```
From a browser, navigate to your container's FQDN to see it running.
4. Clean up group using `az group delete`


### Start and stop containers with policies

> With a configurable restart policy, you can specify that your containers are stopped when their processes are completed. Because container instances are billed by the second, you're charged only for the compute resources used while the container executing your task is running.

**Container restart policies |**
<table>
<thead>
<tr>
    <th>Restart policy</th>
    <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
    <td><code>Always</code></td>
    <td>Containers in the container group are always restarted. This is the <strong>default</strong> setting applied when no restart policy is specified at container creation.</td>
</tr>
<tr>
    <td><code>Never</code></td>
    <td>Containers in the container group are never restarted. The containers run at most once.</td>
</tr>
<tr>
    <td><code>OnFailure</code></td>
    <td>Containers in the container group are restarted only when the process executed in the container fails (when it terminates with a nonzero exit code). The containers are run at least once.</td>
</tr>
</tbody>
</table>

You specify one of those policies when calling `az container create` with a parameter like `--restart-policy OnFailure`.

### Set env variables in container instances
Environment variables are similar to the `--env` command-line argument to `docker run`.

> If you need to pass secrets as environment variables, Azure Container Instances supports secure values for both Windows and Linux containers.

This parameter to `az container create` will craete two named env variables: `--environment-variables "NumWords"="5" "MinLength"="8"`

**Secure values |**
- Environment variables with secure values aren't visible in your container's properties
- Values can be accessed only from within the container
- Set a secure environment variable by specifying the `secureValue` property in YAML file instead of the regular `value` for the variable's type
```YAML
apiVersion: 2018-10-01
location: eastus
name: securetest
properties:
  containers:
  - name: mycontainer
    properties:
      environmentVariables:
        - name: 'NOTSECRET'
          value: 'my-exposed-value'
        - name: 'SECRET'
          secureValue: 'my-secret-value'
      image: nginx
      ports: []
      resources:
        requests:
          cpu: 1.0
          memoryInGB: 1.5
  osType: Linux
  restartPolicy: Always
tags: null
type: Microsoft.ContainerInstance/containerGroups
```

To deploy this container group with YAML:
`az container create --resource-group myResourceGroup --file secure-env.yaml`


### Mount file shares in container instances

> By default, Azure Container Instances are stateless. If the container crashes or stops, all of its state is lost. To persist state beyond the lifetime of the container, you must mount a volume from an external store. Azure Container Instances can mount an Azure file share created with Azure Files.

Azure Files offers fully managed file shares in the cloud that are accessible via the industry standard Server Message Block (SMB) protocol. Provides file-sharing features similar to using an Azure file share with Azure VMs.

Must specify the share and volume mount point of the file share when creating the container like below:

`az container create --resource-group $ACI_PERS_RESOURCE_GROUP --name hellofiles --image mcr.microsoft.com/azuredocs/aci-hellofiles --dns-name-label aci-demo --ports 80 --azure-file-volume-account-name $ACI_PERS_STORAGE_ACCOUNT_NAME --azure-file-volume-account-key $STORAGE_KEY --azure-file-volume-share-name $ACI_PERS_SHARE_NAME --azure-file-volume-mount-path /aci/logs/`

This can also mount a volume with a YAML template.

If multiple volumes need to be mounted, you must deploy with an Azure Resource Manager template or a YAML file and the `volumes` array.

## Module 3: Implement Azure Container Apps

#### Learning Objectives

- Describe the features benefits of Azure Container Apps
- Deploy container app in Azure by using the Azure CLI
- Utilize Azure Container Apps built-in authentication and authorization
- Create revisions and implement app secrets

### Features and benefits of Azure Container Apps
Container Apps run on top of AKS. Common uses of Azure Container Apps include:

- Deploying API endpoints
- Hosting background processing applications
- Handling event-driven processing
- Running microservices

Dynamic scaling based on HTTP traffic, event-driven processing, CPU or memory load, and any KEDA-supported scaler. Container apps can handle tons of scenarios related to container revisions, autoscaling, HTTPS ingress, traffic splitting, service discovery, microservices, etc. 

Container apps are deployed to a container app environment, which is a secure boundary around groups of container apps. Apps in the same env are deployed in the same virtual network and use the same Log Analytics workspace. 

Use the same enviroment when services are related, want to use same virtual network, Instrument Dapr apps that communicate with Dapr API, apps that share Dapr config, apps that share logs. Dont use if you dont want apps to share resources and if they cant communicate with Dapr. 

Microservice architectures allow you to independently develop, upgrade, version, and scale core areas of functionality in an overall system. Azure Container Apps provides the foundation for deploying microservices featuring:

- Independent scaling, versioning, and upgrades
- Service discovery
- Native Dapr integration

> Use of Dapr provides an even richer microservices programming model. Dapr includes features like observability, pub/sub, and service-to-service invocation with mutual TLS, retries, and more.

> Azure Container Apps manages the details of Kubernetes and container orchestration for you. Containers in Azure Container Apps can use any runtime, programming language, or development stack of your choice. Azure Container Apps supports any Linux-based x86-64 (linux/amd64) container image. There's no required base container image, and if a container crashes it automatically restarts.

Containers are configured in the ARM template used to create a new container app or revision. Multiple containers can be included in a single container app using the "sidecar pattern" (advanced usecase, not often best choice). Those containers will share resources and app lifecycle. 

Can deploy containers hosted in container registry using configuration. Limitations are that Azure Container Apps can't run privileged containers that require root access and only Linux based images are supported.

### Deploy with Azure CLI
Rather than copying all of these CLI commands to here, I read the page and will just reference it with the link below:

[CLI Example Exercise](https://learn.microsoft.com/en-us/training/modules/implement-azure-container-apps/3-exercise-deploy-app)

### Utilize Azure Container Apps built-in auth

> The built-in authentication feature for Container Apps can save you time and effort by providing out-of-the-box authentication with federated identity providers, allowing you to focus on the rest of your application.

Azure Container Apps provide various built-in auth providers and works largely the same way as it does in App Service. See the relevant section of the [App Services Post](./2025-01-28-app-services.md) for more information.

The feature should only be used with HTTPS and `allowInsecure` needs to be disabled for that. Auth can be used in addition to unauthenticated access or access can be otherwise restricted. Set *Restrict access* setting to determine this, either `Require Authentication` or `Allow unauthenticated`.


### Create revisions and implement secrets

**Revision Management |**
- Revisions are immutable snapshots of a container app version, useful for versioning and rollback.
- New revisions are created with revision-scope changes when updating the application.
- Active revisions can be controlled, and traffic routing can be specified for each.
- Revision names can be customized using a revision suffix which is added to the container app’s name.
- Updates can include changes to environment variables, computing resources, scale parameters, and the deployment image.
- Revision-scope changes result in the generation of a new revision.

**Implementing Secrets |**
- Secrets allow secure storage of sensitive configuration values at the application level.
- Adding, removing, or changing secrets does not trigger the creation of new revisions.
- Secrets can be referenced by multiple revisions and are scoped to the application, not tied to specific revisions.
- Updated or deleted secrets do not automatically affect existing revisions; responses include deploying a new revision or restarting an existing one.
- Azure Container Apps does not support direct integration with Azure Key Vault.
- Managed identity can be enabled for accessing secrets through the Key Vault SDK within the application.
- Secrets are declared when creating a container app using the --secrets parameter, which takes name/value pairs.
- In container apps, secrets can be referenced in environment variables set during the creation of new revisions.