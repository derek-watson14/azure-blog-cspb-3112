---
layout: post
title: "Event Based Solutions"
date: 2025-04-10
---

**LinkedIn Learning Course Link: [Microsoft Azure Developer Associate (AZ-204) Cert Prep by Microsoft Press](https://www.linkedin.com/learning/microsoft-azure-developer-associate-az-204-cert-prep-by-microsoft-press/)**

## Lesson 11: Develop event-based solutions

*Methodology: Watch each video in the lesson and take notes, summarize learning in "post-lesson review" section when done.*

## Post-lesson review
Really good course I thought. This is the first one that I have completed fully with the video learning instead of the Azure Learning Module and I found it to be much more engaging than the alternative. I also liked the small insights from the speaker like managed identites being far preferred for auth vs shared access sigs. I can see myself using Event Grid in particular in the future for building serverless solutions with Azure Functions at work.

<hr/>

### Video 1: Learning Objectives

The learning objectives for this lesson are:
- Implement solutions that use Azure Event Grid
- Implement solutions that use Azure Event Hub

<hr/>

### Video 2: Implement Solutions that use Azure Event Grid

What is event grid about?
- Fully managed Event Routing Service. Connects publishers to subscribers
    - Simplifies event based communication by allowing management of events across services and applications
    - Deeply integrated with Azure Services and third party services
    - Simplifies event consumption and lowers cost of services similar to this one
    - Reliable, routes events from azure and non-azure resources to subscribers
    - Uses HTTP and MQTTP (Message Queue Text Transfer Pro) protocols among others
- Real Time Processing to multiple destinations
- Facilicates loosely coupled systems, improving scalability and flexibility

Clients and devices would publish/subscribe using MQTT to Azure Event Grid. So it is a two way communication. This way clients/devices dont need to be directly connected to eachother, AEG acts as middleware.

#### Supports two schemas:
- Event Grid schema
- Cloud event schema
- JSON-based
- 1MB max size
- Billed in 64KB blocks
- Sends event to subscribers
Object has properties (all strings except data, all standard except data):
> topic, subject, id, eventType, eventTime, data: object, dataVersion, metadataVersion

Could be used if a key in Key Vault is going to expire, fire an event to subs 30 days before it occurs. Or when a new blob is added to a container, fire an event, etc.

RBAC:
- Event Grid Sub Reader
- EG Sub Contributer
- EG Contributer
- EG Data Sender

(Idea of least privledges applies here)

Event type filtering is also possible:
```JSON
"filter": {
    "includedEventTypes": [
        "Microsoft.Resources.ResourceWriteFailure",
    ],
    "subjectBeginsWith": "/blobServices/default/containers/mycontainer/log",
    "subjectEndsWith": ".jpg"
}
```

#### Use Cases:
- Serverless Architectures
- Real Time Event Processing
- Event-Driven Microservices
- Data Integration
- Audit and Monitoring (say certificates for example, Azure will AUTOMATICALLY fire events for that)

#### Creating in portal:
1. Create "event grid namespace", select RG and location, networking, security settings etc
2. Within the namespace, can scale, add custom domains, change/view access keys, create managed identities, etc. 
3. Write some code to interact with the namespace using Azure SDK

<hr/>

### Video 3: Implement solutions that use Azure Event Hub

**What is Azure Event Hub?**
- Big data streaming platform for event ingestion
- Can recieve and process **millions** of events per second
- Key points:
    - Big data streaming platform
    - Fully managed
    - Real-time data processing
    - Supports multiple producers (sources)

> Different usecase than Event Grid. EG is push based and designed for discrete events like a blob upload. Event Hub is for high throughput and streaming large volumes of event data and is "pull based", think IoT or analytics

**Workflow:**
Event Producers send data to event hub => event hub ingests stores and buffers data => event consumers (like a data platform) stream data from event hub.

**Key Features**
- High throughput and scalability
- Partitioned consumer model (distribution of data across partitions)
- Capture and retention (in azure storage or azure data lake for batch processing)
- Event Replay (alllows consumers to replay and process events from a point in time)
- Integration with Azure Services (seamless, lots, like Azure Functions or Stream Analytics)
- Geo-Replication
- Advanced security

**Uses RBAC with built in roles**
- Azure Event Hubs Data Owner
- Azure Event Hubs Data Sender
- Azure Event Hubs Data Reveiver
*Consider principal of least privlege*

SHOULD authorize access with managed identities and one of the RBAC roles. 
CAN authorize access to publishers and consumers with shared access signatures (not as secure and good as using managed identities). Course teacher would NOT reccomend that. 

**Use Cases**
- Telemetry and Log Ingestion
- Real-Time Analytics
- Fraud Detection and Monitoring
- Data Streaming Pipelines
- Event Driven Architectures


#### Service Comparison:

| Feature              | Azure Event Hubs                        | Azure Service Bus                           | Azure Event Grid                          |
|----------------------|-----------------------------------------|---------------------------------------------|-------------------------------------------|
| **Purpose**          | Real-Time Data Ingestion                | Enterprise Messaging                        | Event Routing                              |
| **Message Pattern**  | High-Throughput Stream Ingestion        | Queue-Based Messaging                       | Pub/Sub with Push                          |
| **Event Delivery**   | At-Least-Once Delivery                  | At-Least-Once Delivery                      | Near Real-Time Delivery                    |
| **Message Size Limit** | Up to 1 MB                            | Up to 256 KB (Standard), 1 MB (Premium)     | Up to 1 MB                                 |
| **Partitioning**     | Supports Partitioning for Parallelism   | Limited Support                             | No Partitioning                            |
| **Use Cases**        | Big Data Analytics, Telemetry, IoT      | Complex Messaging Patterns, Transactions    | Real-Time Event Processing, Serverless     |

