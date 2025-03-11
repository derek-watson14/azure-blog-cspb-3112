---
layout: post
title: "Implement API Management"
date: 2025-03-10
---

**Learning path link: [AZ-204: Implement API Management](https://learn.microsoft.com/en-us/training/paths/az-204-implement-api-management/)**

*Methodology: For each learning objective in each module, write a short summary demonstrating knowledge matching the objective as learned in the course.*

## Post-course review
I decided to take this course earlier in semester than originally planned because it is relevant to what I am working on at work right now so it made sense to dive in. I knew the basics about API management but perhaps not just how full featured it is. The sections about policy expressions using C# and Advanced Policies was new to me and gave me some ideas of new way to handle various problems I have encountered in the past. I also was not really aware of the development portal and groups, which are potentially something to be utilized in the future.

*Course notes by module start below*

<hr/>

## Module 1: Explore API Management

#### Learning Objectives

- Describe the components, and their function, of the API Management service.
- Explain how API gateways can help manage calls to your APIs.
- Secure access to APIs by using subscriptions and certificates.
- Create a backend API.

### Describe components of API Management Service
API Management is made of the following Azure-hosted and managed components:

- API Gateway
  - Accepts API calls and routes them to appropriate backends
  - Verifies API keys and other credentials presented with requests
  - Enfoces usage quotas and rate limits
  - Transforms requests and responses specified in policy statements
  - Caches responses to improve latency and minimize load on backend services
  - Emits logs, metrics and traces for monitoring, reporting and troubleshooting
- Management Plane (admin interface)
  - Provision and configure API Management service settings
  - Define or import API schema
  - Set up policies like quotas or transformations on the APIs
  - Get analytics
  - Manage users
- Developer Portal (automatically generated, fully customizable website with API documentation)
  - Store API documentation
  - Call API with interactive console
  - Create an account and subscribe to get API keys
  - Access analytics on usage (for users)
  - Download API definitions
  - Manage API keys

Other components:

**Products |**

APIs are surfaced to developers through products. Products have 1+ APIs and a title, description and terms of use. Can be **Open** (usable w/o subscription) or **Protected** (usable only with subscription). Subscription approval is configured at product level and can either require admin approval or be autoapproved.

**Groups |**

Groups manage visibility of products to developers. Immutable groups:

- Administrators: Can manage service instances and create APIs, operations and products that are used by developers. Subsciption admins are members of this group
- Developers: Authenticated developer portal users that build applications using your APIs. Developers are granted access to the developer portal and build applications that call the operations of an API
- Guests: Unauthenticated developer portal users. Can be granted some read-only access, like viewing docs.

Custom groups can be made as well.

**Developers |**

Represent user accounts in an API Management service instance. Can be created or invited to join by admins, or can sign up from Developer portal. All are members of some groups, can subscribe to the products that grant visibility to those groups.

**Policies |**

Collection of statemnets that are executed sequentially on the request or response of the API. Popular statement actions include converting XML to JSON, rate limiting, and many others. Policy expressions can be used as attribute values or text values in any of the API Management policies. Some policies such as the Control flow and Set variable policies are based on policy expressions.

Policies can be applied at global, product, API or operation scope.


### Explain how API Gateways can manage calls to APIs

> The API Management gateway (also called data plane or runtime) is the service component that's responsible for proxying API requests, applying policies, and collecting telemetry.

An API Gateway sits between clients and services, acts as a reverse proxy routing requests from clients to services and performs other cross-cutting tasks.

Problems of exposing services directly to client (no API Gateway):

- Complex client code. Client must keep track of multiple endpoints and handle failures in a resilient way
- Couples client and backend
- One operation may require calls to multiple services
- Each service must handle auth, SSL, rate limiting, etc
- Services must expose client-friendly protocol like HTTP or WebSocket, this limits choice of communication protocols
- Services with public endpoints must be hardened as potential attack services

**Managed vs Self-Hosted |**

- Managed: default gateway component that is deployed in Azure of every service tier. All traffic flows throguh azure w/ managed gateway.
- Self-hosted: Optional, containerized version of default managed version. Useful when gateways must be run in same env as backends. Allows for on-prem deployment.


**Policies |**
Publisher can change behavior of API through configuration. Sit between API consumer and API, applied in gateway. Policies can apply changes to both inbound and outbound requests/responses.

Policies are configured in a simple XML document with the following high level structure:
```XML
<policies>
  <inbound>
    <!-- statements to be applied to the request go here -->
  </inbound>
  <backend>
    <!-- statements to be applied before the request is forwarded to 
         the backend service go here -->
  </backend>
  <outbound>
    <!-- statements to be applied to the response go here -->
  </outbound>
  <on-error>
    <!-- statements to be applied if there is an error condition go here -->
  </on-error>
</policies>
```

> If there's an error during the processing of a request, any remaining steps in the inbound, backend, or outbound sections are skipped and execution jumps to the statements in the on-error section. 
> By placing policy statements in the on-error section you can review the error by using the context.LastError property, inspect and customize the error response using the set-body policy, and configure what happens if an error occurs.


**Policy Expressions |**

Policy expressions can be used as attribute values or text values in any APIM policy. Policy expression is either a C# statement enclosed in @() or a multi-statement code block enclosed in @{}.

- Control traffic and modify behavior without needing to write specialized code to modify backend services
- Policies applied at different scopes can be applied with deterministic ordering via the "base" element, like below:

```XML
<policies>
    <inbound>
        <cross-domain />
        <base />
        <find-and-replace from="xyz" to="abc" />
    </inbound>
</policies>
```
Above, The cross-domain statement would execute first. The find-and-replace policy would execute after any policies at a broader scope.

- Can even filter response content based on product associated with request:
```XML
<outbound>
    <base />
    <choose>
        <when condition="@(context.Response.StatusCode == 200 && context.Product.Name.Equals("Starter"))">
            <!-- NOTE that we are not using preserveContent=true when deserializing response body stream into a JSON object since we don't intend to access it again. See details on /azure/api-management/api-management-transformation-policies#SetBody -->
            <set-body>
                @{
                var response = context.Response.Body.As<JObject>();
                foreach (var key in new [] {"minutely", "hourly", "daily", "flags"}) {
                response.Property (key).Remove ();
                }
                return response.ToString();
                }
            </set-body>
        </when>
    </choose>    
</outbound>
```

**Advanced Policies |**
- Control Flow: Conditionally apply policy statements based on results of eval of boolean expression
```XML
<choose>
    <when condition="Boolean expression | Boolean constant">
        <!— one or more policy statements to be applied if the above condition is true  -->
    </when>
    <!-- Can contain multiple whens -->
    <otherwise>
        <!— one or more policy statements to be applied if none of the above conditions are true  -->
</otherwise>
</choose>
```
- Forward request: Forward request to backend service specified in request context
- Limit concurrency: Prevents enclosed policies from executing by more than the specified number of requests at a time, retuurns 429 otherwise
- Log to Event Hub: Sends messages in specified formare to Event Hub defined by Logger entity
- Mock response: Aborts pipeline execution and returns mock reponse directly to caller. Tries to use responses with highest fidelity
- Retry: Executes its child policies once and then retries their execution until the retry condition becomes false or retry count is exhausted
```XML 
<retry
    condition="boolean expression or literal"
    count="number of retry attempts"
    interval="retry interval in seconds"
    max-interval="maximum retry interval in seconds"
    delta="retry interval delta in seconds"
    first-fast-retry="boolean expression or literal">
        <!-- One or more child policies. No restrictions -->
</retry>
```
- Return response: Aborts pipeline execution and returns default (200 OK) or custom response to caller with no body. Custom specified via context variable or policy statements.

### Secure APIs with subscriptions and certificates

**Using Subscriptions |**
- Subscriptions keys are an easy and common way to secure access to APIs in APIM
- APIM also supports OAuth2.0, Client certificates, and IP allow listing
- Subscription key is a unique autogenerated key that can be passed through in the headers of the client request or as a query string parameter
- Subscription scopes:
  - All APIS
  - Single API
  - Product (collection of 1+ APIs configured via APIM, can have different access rules, usage quotas, and terms of use)
- Each sub has two keys, can regen them at any time
- Devs can obtain a key by submitting subscription request
- Can be passed in request header or in query string in URL
- Testing can be done in developer portal or with CURL

**Using Certificates |**
- Certificates can be used to provide TLS (Transport Security Layer) mutual authentication between the client and API Gateway
- Can configure API Management to only allow requests with certs containing specific thumbprint
- Can check for properties like:
  - Certificate Authority (CA)
  - Thumbprint
  - Subject
  - Expiration Date
- Can also verify the cert comes from a partner by checking certificate issuer is correct and that they are self signed

In the Consumption tier, you must explicitly enable client certificates which can be done from the Custom domains page. 

Certificate authorization policies are added in the inbound processing section. For example, this policy snippet which checks the thumbprint of a client certificate passed in a request, checks the thumbprint against certs uploaded to API M, or checks the issuer and subject of a client cert, respectively:

```XML
<choose>
    <when condition="@(context.Request.Certificate == null || context.Request.Certificate.Thumbprint != "desired-thumbprint")" >
    <!-- <when condition="@(context.Request.Certificate == null || !context.Request.Certificate.Verify()  || !context.Deployment.Certificates.Any(c => c.Value.Thumbprint == context.Request.Certificate.Thumbprint))" > -->
    <!-- <when condition="@(context.Request.Certificate == null || context.Request.Certificate.Issuer != "trusted-issuer" || context.Request.Certificate.SubjectName.Name != "expected-subject-name")" > -->
        <return-response>
            <set-status code="403" reason="Invalid client certificate" />
        </return-response>
    </when>
</choose>
```

### Create a backend API

I read through this exercise but won't reproduce the code in this post. Visit [this link](https://learn.microsoft.com/en-us/training/modules/explore-api-management/8-exercise-import-api) to review.