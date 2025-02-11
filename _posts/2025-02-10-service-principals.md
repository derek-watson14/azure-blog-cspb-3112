---
layout: post
title: "[Bonus] Authenticate with Service Principals"
date: 2025-02-10
---

**Learning path link: [Authenticate your Azure deployment pipeline by using service principals](https://learn.microsoft.com/en-us/training/modules/authenticate-azure-deployment-pipeline-service-principals/)**

*Methodology: For each learning objective in each module, write a short summary demonstrating knowledge matching the objective as learned in the course.*

## Post-course review
This was a "bonus" course not part of the main certification curriculum that I decided to take to better understand how Application Registrations and Service Principals work in the context of Azure DevOps development pipelines. It was a super interesting course and I learned a ton about the topic included how tightly coupled the two are, the difference between SPs and MIs as well as the reasons for using service principals and their benefits. The course also covered role assignments which I knew very little about.

Overall 10/10 course that took a fairly short amount of time to work through but will provide huge benefit since I run a bunch of Azure pipelines at work and we will be creating more in the coming months. I want to explore the learning path further actually to learn about Bicep templates. I think I will delve into those courses on other short weeks when the main curriculum is not so long.

*Course notes by module start below*

## Module: Authenticate your Azure deployment pipeline by using service principals

#### Learning Objectives

- Explain what a service principal is, how it works, and how it compares to a managed identity
- Create a service principal and manage its keys
- Configure the appropriate authorization for a service principal to deploy Azure resources

### Service Principal overview

- Service principals are ways to authenticate pipelines, applications and software in Azure.

When deploying a Bicep template, you ask ARM to create or modify azure resources. When you deploy the template, ARM checks if the resources exist. If not, it created them and ensures they are configured correctly.

All of these operations require permissions because they access and modify Azure resources. The specific permissions needed will depend on the the template. Some examples include: creating deployments, create and modifying services. 

When you use your own account to do this, your own *identity* is used for the permissions. If you move to a pipeline, you need to use a different type of identity because the pipeline itself is running deployments.

**Types of security principals |** Entra ID is the service that manages identities for Azure. There are multiple types on indentities (or security principals):

- User: Represents a human who logs in and uses MFA
- Group: A collection of users. A way to assign permissions to set of users not a single entity with a login.
- Service Principal: An automated process or system that usually isnt directly run by a human.
- Managed Identity: Special type of service principal that is designed for situations where humans arent involved in auth.

A service principal is a type of account. It is not a human but can sign into Entra ID and interact with the auth process. No MFA or other human protections for SPs. In Entra, a service principal is identified by an *application ID*  and credential. The applicaiton id is a GUID , and the credential for pipelines is usually a strong password called a key. Certificates are another option for a credential.

Managed identities are a special type of service principal that dont require you to know or maintain its credentials. It is assocaited with an Azure resource so Azure manages the credentials automatically and Azure automatically signs in when the resource needs access to something. 

Managed identities are generally used for App Service apps or VMs, and are useful for automating resource management, DB connection, or accessing Key Vault. Usually NOT used with pipelines. Managed identities require that you own and manage the Azure resources that run your deployments. When you work with Azure pipelines, you usually rely on shared infrastructure ie. the VMs and other resources used to run your jobs are not yours but pooled resources. You can self host a VM (self-hosted agent) to run piplines but usually it will be a hosted agent provided by Microsoft. 

Note: Outside of pipelines, managed identities are usually more secure and easier to work with than service principals and are preferred.

**Why not just use an account? |** User accounts aren't intended for unattended use. User accounts are for humans and often use MFA or CAPTCHA to ensure it is a human. The main benefit of a pipeline is that it is automated and doesnt require a human at all.


**How do service principals work? |** 

> "Microsoft Entra ID has a concept of an application, which represents a system, piece of software, process, or some other non-human agent. **You can think of a deployment pipeline as an application.**"

- They are a feature of Microsoft Entra ID, which is a global service. A *tenant* on entra represents a company or org
- Entra has the concept of an *application*, representing a system, piece of software, process or other non-human agent. In this case **the deployment pipeline is the application**
- An object called an "application registration" represents the application in Entra
- Service pricipals and applications are tightly linked. **Every time you create an app registration in an Entra tenant, a service principal is created in that same tenant**
- Service principals have a lot of additional config and function not relevant to pipelines
- Managed identities are NOT assocaited with an application, unlike other service principals

Summary: when you create a service principal, you first create an app registration. Most tools do this for you. You might not use all of the features of the application with a deployment pipeline, even though they are available.


### Creating service principal & key
When SP needs to communicate with Azure, it signs into Entra and is issued a token that the application stores and uses when making Azure requests. The credential is either a key or certificate. Managed identities do not require management of credentials.

**Keys |**
- Similar to passwords but longer and more complex
- Cryptographically random, hard to guess
- Generally only need to handle the key when first configuring the pipeline and SP
- That is the only time you can see it, must be kept secure
- SPs can have multiple keys
- Keys expire
- Stored on app registration object
- Keys can be created and deleted there too
- If a key is lost a new one must be created

**Certificates |**
- Harder to manage, but more secure
- Harder to steal. Harder to intercept or modify requres using certs
- More maintenence overhead

When creating a key for a pipelines service principal, it is good to immediately copy the key into the pipelines config. In a pipeline you can specify your service principal and key, and can use service connections instead of storing credentials. 

> Creating and modifying service principals requires that you have the related permissions in Microsoft Entra ID. In some organizations, you might need an administrator to perform these steps for you.

Can create a service principal and key in the portal or with the `New-AzADServicePrincipal` command. An appplication registration is always created for the service principal.

A service principal can be identified by:
- Application ID/Client ID: unique identifier similar to a username
- Object ID: App registration and SPs have separate Object IDs, also unique
- Display Name: Human readable, descriptive, **not unique**

**Handling expired keys |**
Default expiration time is set to one year for service principals, after which time the key no longer works in Entra. Thus keys must be rotated and should be recreated if a leak is suspected. Can also be done in CLI. Can gen a new key and swap it before deleting the old one for 0 downtime.

Service principals are not removed automatically so need to audit and remove old ones. Do not leave inactive credentials around. Document service principals for audit purposes, including the following info:
- Identifying information
- Purpose of SP
- Who created it, is responsible for it
- Permissions it needs & justifications
- Expected lifetime of SP

Regular audits ensure they are being used and permissions are still correct.

### Giving service principal authorization to deploy
Previous sections were about **authentication**. Next section will be about **authorization**. 

A service principal does not have any authorization by itself. That is the responsibility of Azure's role-based access system, sometimes called IAM, which can be used to grant a service principal access to a resource group, subscription or management group. 

> By using Azure role-based access control (RBAC), you can grant a service principal access to a specific resource group, subscription, or management group.

Everything in this section uses RBAC to grant access to create and manage Azure resources. Entra has its own role system, called "directory roles", which also can be referred to as roles.

**Components of role assignments |**
1. Assignee: Who is the role assigned to. This is the service principal, identified by the applicaiton id.
2. Role: Like Reader, Contributor, Owner. Each give different permissions. Minimum permissions for a SP is ideal. There are many specific roles with just a subset of functionality, or even custom roles.
3. Scope: How broadly is the role assigned?
    - Single resource: Not typical for pipelines because it is *creating the resources* and using multiple resources
    - Resource group: Good option for many development pipelines, grants permissions to all resources within the group
    - Subscription: All resources within a subscription. Usually too permissive for a deployment pipeline unless the deployment creates a resource group

**Role assignment best practices |**
- Use the least permissive role possible
- Use the narrowst scope possible
- Contributer role is often a good default role assignment
- Built-in roles are often better than custom roles because if project grows, the custom role may need to be continually redefined
- Test pipeline to ensure role assignment works
- Service principals can have different roles at different scopes
- For different enviroments (dev, test, prod) deploy resources to different resource groups or subscriptions
- Create a service principal for each environment, do not mix environment permissions
- Role assignments can be done using CLI or portal (adding a description for role assignments is a good idea for auditing purposes)
- Can use Bicep to create role assignments, if you want to create resource groups and service principals as well as deploy them with IaC