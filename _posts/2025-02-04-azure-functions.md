---
layout: post
title: "Implement Azure Functions"
date: 2025-02-04
---

**Course Link: [AZ-204: Implement Azure Functions](https://learn.microsoft.com/en-us/training/paths/implement-azure-functions/)**

*Methodology: For each learning objective in each module, write a short summary demonstrating knowledge matching the objective as learned in the course.*

<hr/>

## Module 1: Explore Azure Functions

### Learning Objectives

- Explain functional differences between Azure Functions, Azure Logic Apps, and WebJobs
- Describe Azure Functions hosting plan options
- Describe how Azure Functions scale to meet business needs

#### Functional differences between Azure Functions, Logic Apps and WebJobs
Azure Functions are a serverless solution that allow deveopers to build systems to react to critical events. Web APIs, database changes, IoT data streams and message queues are all examples of services/events that can be handled by functions. 

Functions support *triggers* which are ways to start code execution and *bindings* which are ways to simplify coding for input and output data. Logic Apps and WebJobs are other integration and automation services available on Azure. All of them define input, actions, conditions and output.

**Azure Functions vs Logic Apps |**
- Both enable serverless workloads
- Functions: Serverless compute service
- Logic App: Serverless workflow integration platform
- Both can create complex orchestrations, which is a collection of functions or steps that accomplish a complex task
- Functions use code, Logic Apps use GUI or config files

**Key Differences**

| Topic       | Functions                          | Logic Apps                      |
|-------------|------------------------------------|---------------------------------|
| Development | Code first, imperative             | Designer first, declarative     |
| Connectivity| ~12 built-in bindings, can code custom | Many connectors, can build custom |
| Actions     | Activities are functions, code each one| Ready made actions in GUI |
| Monitoring  | Application Insights               | Azure Portal, Azure Monitor logs |
| Management  | REST API, Visual Studio       | Azure Portal, REST API, powershell, VS |
| Execution   | Run in azure or locally      | Azure, locally or on prem            |

**Azure Functions vs WebJobs |**
WebJobs is also a code first integration service designed for developers, both are built on Azure App Service and support features like source control integration, auth and monitoring with app insights. 

Functions is actually build on the WebJobs SDK so it shares many of the same triggers and connections to other Azure services.

Functions have the following, WebJobs do not:
- Serverless app model with autoscaling
- Can be developed and tested in browser
- Pay-per-use pricing
- Integration with Logic Apps
- Additional trigger events (beyond shared events Timer, Storage queues and blobs, Bus queues and topics, Cosmos DB, Event Hubs): HTTP/Webhook, Azure Event Grid

Functions offer more dev productivity vs WebJobs, more programming languages, dev environments, Azure integration and pricing. **Generally Functions is the better choice.**

#### Azure Functions hosting plan options
- Consumption plan (generally available)
    - Default hosting plan. Pay as you go with autoscale. Instances added dynamically based on usage.
    - TIMEOUT Duration: 5 default, 10 max
- Flex consumption (preview)
    - High scalability with compute choices, virtual networking and pay-as-you-go. Can configure per-instance concurrency and reduce cold starts with pre-provisioned instances.
    - TIMEOUT Duration: 30 default, unlimited max
- Premium plan (GA, container on Linux)
    - Automatically scales based on demand using prewarmed workers w/o delay. More powerful instances and connection to VNs possible
    - Choose when: functions run near continuously, want more control of instances, high number of executions and high execution bill, but low GB-seconds when using Comsumption Plan, need more CPU or memory, code needs to run longer, need VN connectivity, want to user custom Linux image to run functions
    - TIMEOUT Duration: 30 default, unlimited max
- Dedicated plan (GA, container on Linux)
    - Good for long running scenarios where Durable Functions cant be used
    - Choose when: predictable billing/manual scaling ok, want multiple apps/function apps on same plan, need access to larger compute, need isolation and secure network access using App Service Environment
    - High memory usage and scale
    - TIMEOUT Duration: 30 default, unlimited max
- Container apps (GA, container on Linux)
    - Containerized funtion apps in a fully managed environment
    - Use Functions programming model to buiold event-driven, serverless cloud native function apps.
    - Choose when: Want to package custom libraries with function code, need to move code execution from on-prem or legacy to cloud native microservices, avoid overhead of managing K8 clusters and compute, need high end processing power and dedicated CPU
    - TIMEOUT Duration: 30 default, unlimited max

*Unlimited duration can be inturrupted by scale actins, deployments, updates, patching, etc.*
*HTTP limit is always 230 seconds max (3m 50s)*

Can be hosted on both Linux and Winows VMs. Hosting option dictates, scaling, resource availability, support for advanced functionality, support for linux containers.

#### Scaling Azure Functions
- Consumtion Plan:
    - Event driven autoscaling. Adds more instances based on number of trigger events. Max instaces: Windows 200, Linux 100
- Flex Consumption Plan:
    - Per function scaling calculated per function basis. Max instances limited only by total memory usage of all instances accross a region
- Premium:
    - Event driven autoscaling based on trigger events. Max: Windows 100, Linux 20-100
- Dedicated:
    - Manual/autoscale. Max: 10-30 instances, 100 on App Service Environment
- Container Apps:
    - Event driven autoscaling based on trigger events. Max: 10-300

<hr />

## Module 2: Develop Azure Functions

### Learning Objectives

- Explain the key components of a function
- Create triggers and bindings to control when a function runs and where the output is directed
- Connect a function to services in Azure
- Create a function by using Visual Studio Code and the Azure Functions Core Tools

#### Key components of functions
**Function App |**
- Is composed of one or more functions that are managed, deployed, and scaled together
- Provides exection context for the functions, way to organize and manage them collectively
- All functions in the app share the same pricing plan, deployment method and runtime version

**Local Development |**
Easy to test functions on local machine, even if they are connected to live services. You can also debug them on your local computer using the Functions runtime. This is the preferred way to develop functions vs in the Azure Portal.

Local project always contain:
- host.json: Contains config options that affect all functions in an app. These settings can be overridden per environment using application settings. 
- local.settings.json: Manages other settings for the app when running locally, otherwise settings from Azure portal are used (do not store in remote)

Settings required by the app from local.settings.json must also be present in Azure for the app to run. Settings can be sync'd with CLI or editor extension.

#### Creating triggers and bindings
**Triggers |** Define how a function is invoked. A function can have exactly one trigger, triggers have assocaited data that usually acts as the payload of the function. 

**Bindings |** Declaratively connect another resource to the function. Can be connected as input or output bindings. Binding data is provided to the function as a parameter. Bindings are optional and a function can have multiple.

Triggers and bindings avoid needing to hardcode access to other services within the function body. You just use params and the return statment.

In C# and Java, function decorators and parameters are used to define triggers and bindings. In other langauges (JS, PowerShell, Python, TS), you use function.json schema. For those langauges, the portal provides a UI for adding bindings in the *Integration* tab or can directly edit the file. These definitions set up the data type of the input data (stream, string and binary are the types).

All triggers and bindings have a direction property. Triggers are always `in`, while bindings can be `in` or `out`, some bindings have `inout`. 

**Example |** You want to write a new row to Azure Table storage whenever a new message appears in Azure Queue storage. Use Azure Queue storage trigger and Azure Table storage output below:

```json
// function.json
{
  "disabled": false,
    "bindings": [
        {
            "type": "queueTrigger",
            "direction": "in",
            "name": "myQueueItem",
            "queueName": "myqueue-items",
            "connection":"MyStorageConnectionAppSetting"
        },
        {
          "tableName": "Person",
          "connection": "MyStorageConnectionAppSetting",
          "name": "tableBinding",
          "type": "table",
          "direction": "out"
        }
  ]
}
```

- `type` and `direction` identify the trigger/bindings
- `name` names the parameter or defines the return method
- `<service>Name` and `connection` define the resource and connection string

```C#
// Same thing using C# decorators to define return and params to define trigger
public static class QueueTriggerTableOutput
{
    [FunctionName("QueueTriggerTableOutput")]
    [return: Table("outTable", Connection = "MY_TABLE_STORAGE_ACCT_APP_SETTING")]
    public static Person Run(
        [QueueTrigger("myqueue-items", Connection = "MY_STORAGE_ACCT_APP_SETTING")]JObject order,
        ILogger log)
    {
        return new Person() {
                PartitionKey = "Orders",
                RowKey = Guid.NewGuid().ToString(),
                Name = order["Name"].ToString(),
                MobileNumber = order["MobileNumber"].ToString() };
    }
}

public class Person
{
    public string PartitionKey { get; set; }
    public string RowKey { get; set; }
    public string Name { get; set; }
    public string MobileNumber { get; set; }
}
```

#### Connecting to Azure Services

### Using Visual Studio Code and Azure Functions Core Tools