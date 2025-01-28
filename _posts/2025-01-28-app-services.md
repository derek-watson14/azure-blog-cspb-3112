---
layout: post
title: "Implement Azure App Service web apps"
date: 2025-01-28
---

**Course Link: [AZ-204: Implement Azure App Service web apps](https://learn.microsoft.com/en-us/training/paths/create-azure-app-service-web-apps/)**

*Methodology: For each learning objective in each module, write a short summary demonstrating knowledge matching the objective as learned in the course.*

<hr/>

## Module 1: Explore Azure App Service

### Learning Objectives

- Describe Azure App Service key components and value
- Explain how Azure App service manages authentication and authorization
- Identify methods to control inbound and outbound traffic to web app
- Deploy an app to App Service using Azure CLI commands

#### App Service Key Components and Value
##### Intro
Azure App Service is an HTTP-based service that allows users to host web-applications and REST APIs built in any programming language or framework on both Windows and Linux based environments. The compute backing the app service can be easily scaled horizontally or vertically and you can even deploy containerized apps. The service has out of the box CI/CD connection to Azure DevOps, Github and other version control sources. Deployment slots are another feature that allow for A/B testing and staging. 

##### Plans
Every App Service app runs in an App Service plan. This plan defines the compute resources for the application, and multiple apps can be configured to run on the same plan (using the same resources). Plan pricing tiers are: Free & Shared (share VM  w/ other customers), Dedicated (dedicated VMs for your app), Isolated (dedicated VMs on dedicated Azure Virtual Networks, network and compute isolation). Typically you want multiple apps on one plan to save cost, but isolate an app into a new plan if it is resource intensive or needs to scale independently or live in a different geographic region.

##### Deployment Basics
Code can be deployed manually or via an automated deployment tool like Azure DevOps or Github. With automated tools, build and tests are run in the cloud and the putput is pushed to the App automatically. With manual deployment this is done on a local machine and pushed using FTP/S, Git, the Azure CLI or a ZIP deploy with a cURl command. Using deployment slots allows for code to go to a staging enviromnet, then be swapped warm to production. This eliminates downtime. You could also add slots for testing, QA, etc that could be connected to branches and be part of a CD pipeline. Slots can also be used with containerized applications, but reqiire a few more steps to tag the container image.

#### Authentication and Authorization
Azure App Service provides built in auth support that requires minimal additional code in the application. Not required that you use it, but can be a time saver by working out of the box with federated identiy providers like Google, Facebook, Microsoft, etc. Can integrate with one or multiple providors and doesnt require any particular language. All you have to do is enable one or more providors when configuring the app service, and a special sign-in endpoint will be created at `/.auth/login/<provider>`.

When enabled, every HTTP request passes through it before reaching your application code. It handles authenticating users, OAuth token management, session management, injecting identity information into HTTP request headers. Runs seperately from application code. Token store is built into App Service. 

You can either present the providers login page (no provider SDK) to the user or use the providers SDK to sign users in manually and submit to the App Service for validation (more for browserless apps like REST APIs or Azure Functions).

Handling unauthenticated requests, App Service can either defer authentication to application code (allowing for more flexibility) or require authentication, in which case traffic will be redirected to one configured identity providor. 

##### Authentication Flow:
No SDK:
Sign user in with login page > redirect to callback > add authenticated cookie > include cookie in subsequent requests

With SDK:
Sign in with SDK to get token > post token to app service for validation > return App Service token to client > present token in header on subsequent requests

#### Inbound and Outbound Traffic Controls
Various inbound and outbound traffic controls like (app-assigned address, access restrictions, private endpoints and virtual network integration etc) can be mixed/combined to control inbound traffic to the app. You can use them to do things like restrict access to the app to certain IPs, or support IP-based SSL needs of an application. 

App service is a *distributed system*. Roles that handle HTTP/HTTPS requests are called "frontends", roles that host customer workloads are called "workers". Higher end plans have dedicated workers, lower plans share workers. When you change VM family (Premium-PremiumV2-PremiumV3) you get a new set of outbound addresses, which can be found in the properties of the app. The `possibleOutboundIpAddresses` lists all possible addresses an app might use within its scale unit (shared VM except on isolated). 

#### Deployment Using Azure CLI
If you have the CLI, you can use the command `az webapp up` to create an update web apps. It:
- Creates a default resource group
- Creates a default app service plan
- Creates an app with the specified name
- Zip deploys files from the current working dir to the web app

Using the CLI, you can configure the name and setup for your app.

<hr/>

## Module 2: Configure web app settings

### Learning Objectives

- Create application settings that are bound to deployment slots.
- Explain the options for installing SSL/TLS certificates for your app.
- Enable diagnostic logging for your app to aid in monitoring and debugging.
- Create virtual app to directory mappings.

#### Configuring settings bound to deployment slots
##### Application Settings
In App Service, app settings are passed as environment variables to the application code or with a flag to a container. Settings can be accessed in the management console at **Environment variables > Application Settings**. For ASP.NET developers, setting app settings in the App Service is equivalent to setting them in appSettings in the web.config file, but the values in the App Service override those in the config files! That way development settings can be kept in config files and production settings safe in the App Service. 

When editing settings, you can check a box to stick the setting to a certain deployment slot. You can also edit them as a JSON file if needed. Connection strings can similarly be set up on an App Service and will override `connectionStrings` in a web.config file. Environment variables for containers can be passed via the cloud shell.

##### General Settings
In the management console **Configuration > General settings** you can configure common settings for an app. These include things like stack settings (language/SDK version), platform settings (bitness, HTTP version...), Debugging, and incoming client certificates.

#### Installing SSL/TLS certs on an app
