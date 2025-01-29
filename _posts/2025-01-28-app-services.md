---
layout: post
title: "Implement Azure App Service web apps"
date: 2025-01-28
---

**Course Link: [AZ-204: Implement Azure App Service web apps](https://learn.microsoft.com/en-us/training/paths/create-azure-app-service-web-apps/)**

*Methodology: For each learning objective in each module, write a short summary demonstrating knowledge matching the objective as learned in the course.*

## Post-course review
The main Azure resource I work on at work is an App Service, so this course was really interesting. A lot of the information I knew already to some extent, but some was new and interesting. My favorite new information to me was:

- We dont use this feature, but I love that App Service provides a no-code solution to authentication with federated identity providers, and you can use the providers SDK or just a login page.
- I learned that Portal application settings override those in a appsettings.json or web.config file. This might be a preferable way to set some application settings.
- We use custom logging with our CMS, but it might be worth using regular application logging since it works natively with App Service
- I didnt really know about autoscaling options, we definitely have a defined traffic pattern because users are in just a few timezones. I will have to make some estimates to see if using autoscaling could save money, because that could result in cost savings for the org.
- Learning more about deployment slots has made me think about how we could add a "test" slot to run tests
- Interesting that explicitly setting traffic % to 0 for a slot makes it hidden without using a QSP. I might make that change for our stage slot so that it is not accessible to the public.

*Course notes by module below*
<hr/>

## Module 1: Explore Azure App Service

### Learning Objectives

- Describe Azure App Service key components and value
- Explain how Azure App service manages authentication and authorization
- Identify methods to control inbound and outbound traffic to web app
- Deploy an app to App Service using Azure CLI commands

#### App Service Key Components and Value
**Intro**
Azure App Service is an HTTP-based service that allows users to host web-applications and REST APIs built in any programming language or framework on both Windows and Linux based environments. The compute backing the app service can be easily scaled horizontally or vertically and you can even deploy containerized apps. The service has out of the box CI/CD connection to Azure DevOps, Github and other version control sources. Deployment slots are another feature that allow for A/B testing and staging. 

**Plans**
Every App Service app runs in an App Service plan. This plan defines the compute resources for the application, and multiple apps can be configured to run on the same plan (using the same resources). Plan pricing tiers are: Free & Shared (share VM  w/ other customers), Dedicated (dedicated VMs for your app), Isolated (dedicated VMs on dedicated Azure Virtual Networks, network and compute isolation). Typically you want multiple apps on one plan to save cost, but isolate an app into a new plan if it is resource intensive or needs to scale independently or live in a different geographic region.

**Deployment Basics**
Code can be deployed manually or via an automated deployment tool like Azure DevOps or Github. With automated tools, build and tests are run in the cloud and the putput is pushed to the App automatically. With manual deployment this is done on a local machine and pushed using FTP/S, Git, the Azure CLI or a ZIP deploy with a cURl command. Using deployment slots allows for code to go to a staging enviromnet, then be swapped warm to production. This eliminates downtime. You could also add slots for testing, QA, etc that could be connected to branches and be part of a CD pipeline. Slots can also be used with containerized applications, but reqiire a few more steps to tag the container image.

#### Authentication and Authorization
Azure App Service provides built in auth support that requires minimal additional code in the application. Not required that you use it, but can be a time saver by working out of the box with federated identiy providers like Google, Facebook, Microsoft, etc. Can integrate with one or multiple providors and doesnt require any particular language. All you have to do is enable one or more providors when configuring the app service, and a special sign-in endpoint will be created at `/.auth/login/<provider>`.

When enabled, every HTTP request passes through it before reaching your application code. It handles authenticating users, OAuth token management, session management, injecting identity information into HTTP request headers. Runs seperately from application code. Token store is built into App Service. 

You can either present the providers login page (no provider SDK) to the user or use the providers SDK to sign users in manually and submit to the App Service for validation (more for browserless apps like REST APIs or Azure Functions).

Handling unauthenticated requests, App Service can either defer authentication to application code (allowing for more flexibility) or require authentication, in which case traffic will be redirected to one configured identity providor. 

**Authentication Flow:**
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
**Application Settings**
In App Service, app settings are passed as environment variables to the application code or with a flag to a container. Settings can be accessed in the management console at **Environment variables > Application Settings**. For ASP.NET developers, setting app settings in the App Service is equivalent to setting them in appSettings in the web.config file, but the values in the App Service override those in the config files! That way development settings can be kept in config files and production settings safe in the App Service. 

When editing settings, you can check a box to stick the setting to a certain deployment slot. You can also edit them as a JSON file if needed. Connection strings can similarly be set up on an App Service and will override `connectionStrings` in a web.config file. Environment variables for containers can be passed via the cloud shell.

**General Settings**
In the management console **Configuration > General settings** you can configure common settings for an app. These include things like stack settings (language/SDK version), platform settings (bitness, HTTP version...), Debugging, and incoming client certificates.

#### Installing SSL/TLS certs on an app
To secure information transmitted between an app and client, certs can be installe. You can create, upload or import private or public certs to App Service. You can buy or use free certificates from App Service, import certs from Key vault or upload private/public certs. There are requirements for private uploaded certs in regard to key size and password protection. Free/managed certs are easiest as a turn-key solution and is fully managed renewal by APp Service but has some limitations regarding wildcard certs, private DNS, exportability and character use/domain name length.

#### Enabling diagnostic logging for monitoring/debugging
App services come with built-in diagnostics to assist with debugging. The following types of logging are available:
- Application logging: Log messages generated by application code. Messages are generated by the web framework chosen or by application code using the standard logging pattern of the language. 
- Web server logging: Raw HTTP request data including HTTP method, URI, client IP, user agent, response code, etc. (No linux)
- Detailed error messages: Copies of .html error pages that would have been sent to the client browser in development mode (No linux)
- Failed request tracking: Detailed tracing information on failed requests, including a trace of IIS components used to process the request. One folder ofr each failed request containing XML logs. (No linux)
- Deployment logging: Used to determine why a deployment failed. Happens automatically, not configurable.

Some types of logging are not available on Linux, like Web server logging and failed request logging. 

To enable application logging, go to **App Service logs** in management console, select **On** for Application logging and set the level of logging (Disabled, Error, Warning, Information, Verbose).

Other logging can be enabled and configured similarly in this section. To log messages in code, use the usual logging facilities for a framework. ASP.NET for example uses `System.Diagnostics.Trace.<TraceError>("Message here")`. 

Streaming logs in real time is possible when the log type is set, and are stored in the `/LogFiles` directory. If you use Azure blob storage for the log type, a client tool that can connect to Azure Storage.

#### Create virtual app to directory mappings
The **Path mappings** page displays path mapping options based on OS type. For windows apps, for example, you can customize IIS handler mappings to handle requests for specific file extensions. Adding the handler you configure: the file extension, absolute path of the script processor (D:\home\site\wwwroot is the apps root directory) and command line arguments for the script processor. You can also edit or add virtual applications and directories here.

For linux and containerized apps, this occurs in an Azure Storage account, which must also be configured.

## Module 3: Scale apps in Azure App Service

### Learning Objectives
- Identify scenarios for which autoscaling is an appropriate solution
- Create autoscaling rules for a web app
- Monitor the effects of autoscaling

#### When is autoscaling appropriate?
- Autoscaling makes scaling decisions based on parameters set by user
- Can be according to schedule or system resource consumption
- Autoscaling monitors resource metrics of the web app and detects situations where additional resources are required to handle increased workload
- Adds or removes web servers and balances load between them. Does not change CPU power, memory or storage capacity of the web servers. Instead it just changes the number of web servers.
- Autoscale events are triggered when predefined thresholds are crossed (allocating/deallocating resources)
- Must be careful. Ex. a DoS attack could cause needless and expensive scaling. Need to add request filtering before such requests reach a service

**When to use?**
If the app performs resource-intensive processing with each request, autoscaling might be ineffective because small instances can be overloaded by large requests, so scaling out will not provide relief. Also for long term growth, autoscaling has monitoring overhead so manual scaling may be preferable if growth can be anticipated. Apps with small number of instances are susceptible to downtime even with autoscaling becasue it takes time to spin up additional instances.

Changes in application load that are predictable are good candidates for autoscaling.

**"Automatic Autoscaling Scaling"**
Automates the autoscaling so that you do not have to define rules. Scale out instances are prewarmed for smooth performance transitions. Apps within the same Plan can scale differently and independently. Allows you to set max instances the plan can scale to as to not overwhelm a DB or backend that does not scale as easily.

**Autoscale factors**
- Based on a metric (like length of disk queue, number of HTTP requests awaiting processing)
- According to schedule (if you know at night time you have less traffic)
- Can limit max instance count
- Can create multiple conditions for different schedules and metrics, any condition can trigger scaling in this scenario
Metrics include: CPU percentage, Memory percentage, Disk Queue length (outstanding I/O requests across all instances), HTTP queue length (client requests awaiting processing), Data In (bytes recieved), Data out (bytes sent). Can also scale based on metrics from other services.

**How it works**
Autoscale rule aggregates values retrieved for a metric for all instances across a persiod known as a *time grain* (1 minute usually). Aggregated value is the time aggregation. A second, longer interval called a *Duration* is aggregated next (5-10 minutes usually) using each time grain in that interval. Can use different aggregations for each interval (Avg, Min, Max, Sum, Last, Count). Could use CPU %, average for time grain then maximum for duration and autoscaling would be based on max average CPU % over the duration. Based on that duration aggregation and a rule, instances are scaled out or in. There is a cooldown period between scale events to allow the system to stabalize with the new number of instances. 

**Example**
You could define the following four rules in the same autoscale condition:

- If the HTTP queue length exceeds 10, scale out by 1
- If the CPU utilization exceeds 70%, scale out by 1
- If the HTTP queue length is zero, scale in by 1
- If the CPU utilization drops below 50%, scale in by 1

Scale out occurs if ANY condition is met. Scale in occurs only when ALL conditions are met.

**Best Practices**
- Ensure max and min instance values are different and have an adequate margin between the two values (which are inclusive)
- Choose appropriate statistic for your metric (average is most common)
- Choose thresholds carefully for all metrics, using different values for scale out and scale in to avoid "flapping" (see example).
- Remember that scale-in occurs only when all rules are met. 
- Select a safe default instance count. That count is used if metrics become unavailable
- Configure notifications for successful/failed scale actions to ensure you are monitoring changes

#### Creating autoscaling rules
Autoscaling can be enabled in the App Service Plan, under the **Scale out** configuration. Choose manual, automatic or rules based scaling. Manual is the default. Scale conditions and scale rules can be configured using the management console GUI. Can specify max instances in scale conditions. A condition contains one or more rules and instructions to scale in or out. 

#### Monitoring effects of autoscaling
Azure portal enables you to track when autoscaling occurs in the **Run history** chart. Shows how number of instances varies over time and what conditions are triggering scaling. Run history chart and metrics in the Overview page can be used to correlate autoscaling events with resource utilizaiton.


## Module 4: Explore Azure App Service deployment slots

### Learning Objectives
- Describe the benefits of using deployment slots.
- Understand how slot swapping operates in App Service.
- Perform manual swaps and enable auto swap.
- Route traffic manually and automatically.

#### Benefits of using deployment slots
- You can validate changes in a staging environment before swapping it with a production slot
- Deploying to a stage slot then swapping into production ensures that all instances of the slot are warm before going into production, this eliminates downtime without dropped requests
- Slot swapping can be automated if human validation isnt needed
- After a swap, the previous production deployment is in the staging slot. If there is a failure in prod, swapping back to a known working deployment is fast

The number of slots available to an app service varies by tier.

#### How slot swapping works
App Service completes the following process on slot swap to ensure no downtime:
1. Apply settings from target slot to all instances of source slot, including slot-specific app settings and connection strings, CD settings, App Service auth settings
2. Wait for every instance in the slot source to complete restart
3. Trigger local cache initialization on each instance in source slot (another restart)
4. Trigger application initiation by making HTTP request to app root of each instance of source slot. If any response is returned, its considered warmed up
5. If ALL instances are warmed up, perform swap by switching routing rules for the two slots. Now the taget slot contains the app from the source.
6. Perform same operations again on the source slot since it now contains app from the target.

Note that all work during this process happens in the source slot, the target is online and unchanged regardless of success or failure until the operation completes. Some settings are not swapped during this process like:
- Publishing endpoints
- custom domain names
- private certs and TLS/SSL settings
- scale settings
- WebJobs schedulers
- IP restrictions
- Always One
- Diagnostic log settings
- CORS
- Virtual network integration
- Manged identities
- Settings ending with _EXTENSION_VERSION 

Can be overridden with app setting `WEBSITE_OVERRIDE_PRESERVE_DEFAULT_STICKY_SLOT_SETTINGS`

#### How to perform manual slot swapping and auto swap
Slots can be manually swapped in the UI of the azure portal with some button clicks. Can also *swap with preview* there to apply target settings change to source without completing swap. Can then preview the source before completing or cancelling the swap. In the **Activity log** you can get information on a swap operation or its suboperations.

Enabling autoswap means that everytime code is pushed to that slot, that slot is automatically swapped into production when its warmed up. That setting is in Configuration > General settings. 

You can also define custom warm-up actions in the applicationInitialization configuration element in a web.config file. Can use this to configure custom ping path to verify restarts, and acceptable response codes. 

#### Routing traffic between slots manually and automatically
By default all traffic goes to production slot for an app. You can also use the  Traffic % column of a slot to automatically route a percentage of traffic to that slot randomly. In the client the `x-ms-routing-name` cookie will tell you the slot your session is pinned to. You can also route production traffic manually with an <a> tag in your code pointing to a certain slot with the same x-... syntax as in the cookie.

By default, new slots are given a routing rule of 0%, a default value is displayed in grey. When you explicitly set the routing rule value to 0% it's displayed in black, your users can access the staging slot manually by using the x-ms-routing-name query parameter.