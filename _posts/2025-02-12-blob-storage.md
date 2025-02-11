---
layout: post
title: "Develop solutions using blob storage"
date: 2025-02-12
---

**Learning path link: [AZ-204: Develop solutions that use Blob storage](https://learn.microsoft.com/en-us/training/paths/develop-solutions-that-use-blob-storage/)**

*Methodology: For each learning objective in each module, write a short summary demonstrating knowledge matching the objective as learned in the course.*

*Course notes by module start below*

<hr/>

## Module 1: Explore Azure Blob storage

#### Learning Objectives

- Understanding Azure Blob Storage: Features, Types of Storage Accounts, and Access Tiers
- Understanding Azure Blob Storage: Storage Accounts, Containers, and Blobs
- Understanding Azure Storage Security and Encryption Features

### Features, types of storage accounts, access tiers

Microsoft's blob storage solution on the cloud, optimized for massive amounts of unstructured data like binary or text data. Designed for:

- Serving images or documents to the browser
- Storing files for distributed access
- Audio and video streaming
- Writing log files
- Storing data for backup, restore, disaster recovery, archiving
- Storing data for analysis

Blob storage can be accessed with HTTP/S anywhere in the world by client applications or users. Objects accessible via REST API, Azure CLI, or the client library.

**Types of storage accounts |**
- Standard: General purpose recommended for most scenarios
    - Blob storage, Data Lake, Queue Storage, Table Storage, Azure Files. 
    - LRS, GRS, RA-GRS, ZRS, GZRS, RA-GZRS
- Premium: More performant with SSDs. Choose between block blobs, page blobs or file shares

**Access tiers |**
- Hot: Optimized for frequent access, highest storage cost, lowest access cost
- Cool: For large amounts of data infrequently accessed but stored for 30 days minimum. Lower storage costs, higher access cost compared to Hot
- Cold: Infrequently accessed & stored for minimum of 90 days. Low storage cost, higher access cost compared to Cool
- Archive: Available only for individual block blobs. Ok for data that can tolerate hours of retrieval latency and remains in this tier for 180 days. Most cost effective by access cost is more expensive and slower than all other tiers.

**Resource types |** 
- Storage Account
- Container in an account
- Blob in a container

*Storage accounts* provide a unique namespace, every object that is stored within has an address including the unique account name. THe base address will be like:

`http://mystorageaccount.blob.core.windows.net`

A *container* organizes a set of blobs, like a directory. Accounts can have unlimited containers, and containers can have unlimited blobs. Container name must be a valid DNS name and is also included in the URI (3-63 chars, start with letter/number, only lowercase letters, numbers, and dash without consecutive dashes).

`https://myaccount.blob.core.windows.net/mycontainer`

There are three types of *blobs*:
- Block blobs: Text and binary data. Made of blocks of data that can be managed individually (max 190.7 TiB)
- Append blobs: Made of block blobs, but optimized for append operations. Good for logging.
- Page blobs: Store random access files up to 8TB. Store virtual hard drive files and act as disks for Azure VMs

`https://myaccount.blob.core.windows.net/mycontainer/myblob`

**Security features |**
- Uses Service-Side Encryption automatically
- Client side encryption also available for Blob Storage and Queue Storage
- USes 256-bit Advances Encryption Standard, one of strongest available block ciphers
- FIPS 140-2 compliant
- Cannot be disabled
- Automatic on all performance and access tiers for all blobs
- No extra cost

Encrypted with Microsoft managed keys by default. Optionally can manage encryption with your own keys using either/both:
- Customer managed keys which must be stored in Azure Key Vault
- Customer provided keys which can be stored elsewhere and must be provided on read/write requests

Client libraries for .NET, Java, Python support encrypting data within client applications before upload, and decryption when downloading. Queue Storage client libraries for .NET/Python also support **client-side encryption**. These libs use AES to encrpyt user data. Can use Version 2 (Galois/COunter Mode with AES) or Version 1 (Cipher Block Chaining with AES)

<hr/>

## Module 2: Manage the Azure Blob storage lifecycle

#### Learning objectives

- Describe how each of the access tiers is optimized.
- Create and implement a lifecycle policy.
- Rehydrate blob data stored in an archive tier.

### Access tier optimization
Over time data access frequency tends to change and some data even expires. This differs between data sets. The different access tiers (see "Access tiers" above) allow you to optimize for those differnt access patterns. 

### Lifecycle policies

You can create rule-based policies to transition blob data across access tiers or even expire data (delete at end of lifecycle). These rules can be applied to entire storage accounts, containers or to a subset of blobs using name prefixes or index tags. Allows developer to design cheaper solution based on needs.

Policies include a filter set and action set. Filter set defines which ojects to act on, action set defines what to do. A collection of rules is called a **policy**. One rule required per policy. Sample:

```JSON
{
  "rules": [
    {
      "name": "rule1", // 256 chars, case sensitive, unique
      "enabled": true, // To temp disable, default true, optional
      "type": "Lifecycle", // enum, Lifecycle is valid type
      "definition": { // Filter set and action set go here
        "actions": {
            "snapshot": {}, // Perform action on snapshot versions
            "version": { // Perform action on previous versions of blob
                "delete": {
                    "daysAfterCreationGreaterThan": 90
                }
            },
            "baseBlob": { // Perform action on current version
                "tierToCool": {
                    "daysAfterModificationGreaterThan": 30
                },
                "tierToArchive": {
                    "daysAfterModificationGreaterThan": 90,
                    "daysAfterLastTierChangeGreaterThan": 7
                },
                "delete": {
                    "daysAfterModificationGreaterThan": 2555
                },
                "enableAutoTierToHotFromCool" // Nor supported for snapshot or versions
            }
        },
        "filters": {
            "blobTypes": [ // Predefined enum values
                "blockBlob"
            ],
            "prefixMatch": [ // (optional) Array of strings for prefixes to match, must start with container name
                "sample-container/blob1"
            ], 
            "blobIndexMatch": [] // (optional) Array of dict values with blob index tag key and value conditions to be matched, up to 10
        }
        
      } 
    },
    {
      "name": "rule2",
      ...
    }
  ]
}
```

Actions (all supported for blockBlob):
- tierToCool
- tierToCold
- enableAutoTierToHotFromCool
- tierToArchive
- delete (also supported appendBlob)

> If you define more than one action on the same blob, lifecycle management applies the least expensive action to the blob. For example, action delete is cheaper than action tierToArchive. Action tierToArchive is cheaper than action tierToCool.

Run Conditions (all integers)
- daysAfterModificationGreaterThan (base blob)
- daysAfterCreationGreaterThan (snapshot)
- daysAfterLastAccessTimeGreaterThan (current version with access tracking)
- daysAfterLastTierChangeGreaterThan (only applies to tierToArchive action)

Policies can be set up using Azure Portal, Azure PowerShell, CLI, REST APIs.

### Rehydrating blob data from archive
Cant read or modify archived blobs. Considered to be offline. Need to first rehydrate the blob. To do so you can either:

- Copy it to an online tier with copy operation. (Reccomended for most scenarios)
    - Must copy the archived blob to a new blob with a different name or to a different container
    - v. 2021-02-12 supports cross-account copies if in same region
- Change access tier back to online tier
    - Cannot be cancelled
    - Does not effect last modified time, so lifecycle policy could move it back to archive after rehydration

This operation can take several hours. Standard priority rehydrates in order recieved taking up to 15 hours. High priority can complete in under 1 hour for objects <10GiB in size. 

<hr/>

## Module 3: Work with Azure Blob Storage

#### Learning objectives
- Create an application to create and manipulate data by using the Azure Storage client library for Blob storage.
- Manage container properties and metadata by using .NET and REST.


