---
layout: post
title: "Message Based Solutions"
date: 2025-03-17
---

**LinkedIn Learning Course Link: [Microsoft Azure Developer Associate (AZ-204) Cert Prep by Microsoft Press](https://www.linkedin.com/learning/microsoft-azure-developer-associate-az-204-cert-prep-by-microsoft-press/)**

## Lesson 12: Develop message-based solutions

*Methodology: Watch each video in the lesson and take notes, summarize learning in "post-lesson review" section when done.*

## Post-lesson review
Another good lesson from LinkedIn learning, enjoyed seeing both the demo in the Azure Portal to create these resources in a click-ops kind of way and the simple console apps he created demonstrating how to interact with the resources using the .NET SDK. One thing I learned that I did not know before was that Azure Storage Queues are just build into Azure Storage Accounts which I have worked with many times. We use a queue based solution (better suited for Service Bus) for our remote deposit capture process so it was interesting to learn more about some of the building blocks for this solution.

<hr/>

### Video 1: Learning Objectives

The learning objectives for this lesson are:
- Implement solutions that use Azure Service Bus
- Implement solutions that use Azure Queue Storage queues

<hr/>

### Video 2: Implement solutions that use Azure Service Bus

Azure Service Bus and Azure Storage Queues are both azure messaging services. Both services are designed to handle async communication between applications.

**Messaging Pattern:**
- If requests are sent at a variable speed but processed at a consistent speed a worker might get overloaded, with a queue you eliminate this risk by storing the queue so the worker can work at its own speed. 
- Also if a service is unavailable/offline messages can wait in the queue until the worker is available later on. 
- Can still post messages to a queue and get an immediate response even if the worker isnt actually processing the message right now. 
- Can horizontally scale worker instances to meet average load rather than max load
- Kind of like throttling (load levelling pattern) to a level a worker can handle

**Azure Service Bus**
- Fully managed enterprise service
- Reliable and secure using queues, topics and subscriptions
- Uses patterns like FIFO
- Guarantees delivery at least once
- Advanced features like: dead letter, sessions, message deferral
- Integrates security features like Entra ID and RBAC

**Scenarios**
- Transfer business data (like purchase orders z.B.)
- Decouple applications
- Topics and subscriptions 1-N relationship
- Can use things like message ordering or message deferral

**Azure Service Bus Tiers**
- Basic: For dev/test workloads
- Standard: Variable throughput and latency, pay-as-you-go pricing, message size to 256KB
- Premium: Higher and predictable throughput and latency, fixed pricing, message size to 100MB

Typically standard is sufficient (particularly message size is sufficient).

**Concepts**
Queues: FIFO, guaranteed delivery, duplicate detection
Topics: Pub-Sub pattern, topics and subs, filters and rules, isolation between subscribers, good for broadcasting messages to multiple recievers

- Messages that cant be processed can go to dead letter queue. 
- Topics have 1-n relationship, good for broadcasting

**SDK** 
There is a SDK for .NET and many other major langauges.

```C#
// Reference
using Azure.Messaging.ServiceBus;

ServiceBusClient client;
ServiceBusSender sender;

// Create sender
client = new ServiceBusCLient(connStr);
sender = client.CreateSender(queueName);

// Create a batch of messages
using ServiceBusMessageBatch messageBatch = await sender.CreateMessageBatchAsync();

for (...)
{
    if (!messageBatch.TryAddMessage(new ServiceBusMessage(...)))
    {
        throw new Exception
    }
}

// Send
await sender.SendMessagesAsync(messageBatch);
```

Can create a service bus through the Azure Portal like with all other services. Create queues and topics there, choose features, even view/send messages in the queue in the web with the service bus explorer web experience.

<hr/>

### Video 3: Implement solutions that use Azure Queue Storage queues

Azure Storage Queues are a simpled messaging service designed for essential communication. Low cost, simple, and scalable. (Simpler means simpler than storage bus). Much cheaper than storage bus but also far fewer features. 

Managed service, stores messages, queues can be accessed with HTTP/S, components include:
- URL Format
- Storage account
- Queue
- Message

64KB in size for the messages in Azure Queue Storage. Number of messages stored is only limited by the size of the storage account. Need a storage account attatched. Names must be all lowercase for the queue. Messages must be <64KB but also include some metadata, like TTL (7 days is default).

There is also an SDK to simplify interaction with storage queue. 

Similar to the example in Service Bus. Need a using, create a client with a connection string, use SendMessage method on the client to send a message, RecieveMessages to recieve messages, DeleteMessage to delete messages.

Plenty of SDK docs from Microsoft online. When dequeing a message withing 30s you need to either delete the message or renew the lease you have on the message or it will appear in the queue again. 

#### How to choose between the two:
- Service Bus
    - Need to handle larger messages than 64KB
    - Require FIFO ordering delivery
    - Support for duplicate detection
    - Control transactions and adomicity on send/recieve
- Storage Queues
    - Need to store more than 80GB of message (because using storage account)
    - Dont need extra feautres
    - Good example would be processing images to thumbnails, order doesnt matter
    - CHEAPER

**Creating a Storage Queue in the Azure Portal**
1. Create a storage account (must be standard tier)
2. Go to the Data Storage > Queues blade (same place you look at containers)
3. Create a new queue with a name

After that is complete, you can use the Azure SDK to interact with the queue. Must install the package Azure.Storage.Queues and Azure.Identity (to handle auth).

Go to Program.cs and add namespaces of the packages just installed. 

Then:

- Create a connection string (found in Access Keys blade)
- Create a newx QueueClient passing the connection string and name as a parameter
- Can use CreateIfNotExists with the client
- Add a message to the queue with SendMessageAsync
- Read the message with RecieveMessageAsync