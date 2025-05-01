---
layout: post
title: "Implement secure Azure solutions"
date: 2025-04-30
---

**LinkedIn Learning Course Link: [Microsoft Azure Developer Associate (AZ-204) Cert Prep by Microsoft Press](https://www.linkedin.com/learning/microsoft-azure-developer-associate-az-204-cert-prep-by-microsoft-press/)**

## Lesson 7: Implement Azure secure solutions

*Methodology: Watch each video in the lesson and take notes, summarize learning in "post-lesson review" section when done.*

### Post-lesson review
*Completed after course notes*

<hr/>

### Video 1: Learning Objectives
- Secure app configuration data by using App Configuration or Azure Key Vault
- Develop code that uses keys, secrets and certificates stored in Key Vault
- Implement managed identities for Azure resources

### Video 2: Secure app configuration data by using App Configuration or Azure Key Vault

#### Azure Key Vault
- Service for storing and accessing: 
  - Secrets, keys and certificates (TLS, SSL)
- Has standard and premium tiers
  - Standard uses software to encrypt
  - Premium uses hardware to encrypt/guard (higher standard)
- Can be used as key management solution (like for encryption keys to encrypt data at rest)

Example: Use this to store cert for custom domain, updating cert in key vault propogates out easily.

**Why use it?**
- Centralize app secrets
- Monitor access and use
- Very secure
- Simplified administraction of secrets
- Easy integration with other services like app services

Authenticating an app to key vault:
- Use Entra ID with service principal (managed identity or app registration)

#### App Configuration
- Central management of app settings and feature flags
- Store all settings in App Configuration and secure access in one place
- Fully managed
- Supports managed identites
- Fully encrypted
- Native integration with popular frameworks like .NET (there is a provider for this, or can use REST API if there is no SDK)

#### Demo
**Key Vault**
- `dotnet add package`: `Azure.Security.KeyVault.Secrets`, `Azure.Identity`
- Create a string with the name of the key vault, the URI
- Create a SecretClient using those strings plus a DefaultAzureCredential to connect
- use GetSecretSync(<name>) to get secrets with the client

**App Configuration**
- Also a resource that you create, an "App Configuration"
- Config sub, rg, location, name, tier, replication behavior, access settings, recovery options, etc
- Encrypted at rest by default using Azure managed encryption key
- Just add keys and values through the portal in the resource
- Can add KEY VAULT REFERENCE instead of just key value to add secrets
- The Key Vault secret will be exposed as an ENV variable in the app once the config is integrated with the applicaiton, so you dont need to code the key vault integration at all

### Video 3: Develop code that uses keys, secrets and certificates stored in Key Vault

Packages:
- Azure.Security.KeyVault.Secrets
- Azure.Identity
- System

Create a secret client with new SecretClient(vaultUri, new DefaultAzureCredential())

Then can use SetSecretAsync(name, val) and GetSecretAsync(name)

This is the Key Vault SDK.

### Video 4: Implement managed identities for Azure resources

Managed identities:
- Provides an identity for applications in MS Entra ID
- Applications use this identity when connecting to resources that support RBAC
  - Ex. App Service that needs some access to Blob Storage

Using managed identities allows you to avoid having to manage credentials!

You can authenticate to ANY resource that supports MI.

No cost.

**Types of Managed Identites**

- System Assigned
  - One per resource
  - Lifecycle associated with the resource
- User-assigned
  - Can be used by multiple resources (created by the user)

Can use managed identites to access secrets in key vault or other resources and use the secret to access external APIs or whatever. This can allow us to store NO secrets in the application code.

**Demo**

- Settings > Identity > System/User assigned
- Must enable the system assigned, then status will be on
- Can then see it in Entra Application Type > Managed Identites (sort by date if just created)
- It will be a Service Principal of type Enterprise Application
- Then on the resource the MI needs access to, go to Access Control blade > Add > Select Desired Role > Select managed identity that needs access > Review and assign
- Now can do the UseDefaultCredentials instead of the access key when using that resource!

**Key Vault - App Services Integration Demo**
- In Web App, go to Env Variables, set a KVP (could be an API key for example)
- BETTER: 
  - Add it as a secret to key vault
  - Enable system assigned managed id on the resource that needs access to KV
  - Go back to key vault, and give IAM role Key Vault Secrets User (read contents of secrets) to the managed identity for the resource
  - Go once more to resource (web app) and under the value for the key use @Microsoft.KeyVault(SecretUri=<secretUri>)
    - Once done, it will show the source as Key Vault!
  - Now you can access this as you would any other environmental variable in code but it is being stored in key vault

