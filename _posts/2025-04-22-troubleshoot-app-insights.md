---
layout: post
title: "Troubleshooting with Application Insights"
date: 2025-03-17
---

**LinkedIn Learning Course Link: [Microsoft Azure Developer Associate (AZ-204) Cert Prep by Microsoft Press](https://www.linkedin.com/learning/microsoft-azure-developer-associate-az-204-cert-prep-by-microsoft-press/)**

## Lesson 9: Troubleshoot solutions using Application insights

*Methodology: Watch each video in the lesson and take notes, summarize learning in "post-lesson review" section when done.*

## Post-lesson review
Really interesting course, particularly the third video that was convering availability tests and alerts. I think for my orgs main site that is something that we could definitely improve is our time to discovery for the service going down, I will have to suggest this as an alternative or additional layer to Pingdom which we currently are using since it seems more full featured. Also really cool how you can run workbooks in resoponse to triggered alerts.

<hr/>

### Video 1: Learning Objectives

The learning objectives for this lesson are:
- Configure app to use App Insights and set alerts
- Monitor and analyze metrics, logs and traces

<hr/>

### Video 2: Configure and app or service to use Application Insights

What is Application Insights?
- Collects metrics and telemetry data
- Describes app activity and health
    - Response times
    - Request rates
    - Page views
    - Load performance
    - User and session count
    - Custom events and metrics

Can create custom metrics and send them to application insights as the application runs.

Experiences:
- Investigate: Application dashbaords for health, applicaion map (architecture of app), live metrics of activity, performance, availability, etc
- Monitor: Live metrics of activity, performance, availability, etc. Can use KQL to query logs and metrics
- Usage: Have events for users and sessions, as well as funnels to analyze conversion rates and progression of users
- Code Analysis: Can do code optimization and refactoring with AI and get code analysis

Visualizations:
- Azure Workbooks (free)
- Grafana / Managed Grafana
- Power BI

**Demo Creating Application Insights Notes**
- Create an application insights instance, config name, sub, RG, region, log analytics workspace
- There is an instrumentation key and connection string that can be used to connect application to AppInsights
- Blades:
  - Investigate (live metrics)
  - Monitoring
  - Usage

<hr />

### Video 3: Monitor and analyze metrics, logs and traces

#### Availability tests:
- Can leverage Azure network to check availability from all over the world and check response times
- Can have up to 100 tests per resource (boil down to HTTP/S call to page/api)

4 Types:
- Standard test
- Custom TrackAvailibility test
- Deprecated:
  - Multistep web test
  - Ping test (doesnt check for certificate issues just that the service is up or down)
- Integrates with Alerts

#### Azure Alerts
- Create Alert Rule to detect outages before users see them
  - Alert Rule is definition of what we are trying to monitor
- Fired Alerts: respond to Alert Rule that is triggered
- In the fired alert, you have actions
  - Notifications (email, sms, push, etc)
  - Run azure runbook automations
  - Run azure function or logic app
  - Secure web hooks
  - Send to event hub
- Stateless/stateful alerts 
  - Stateless fire every time the check fails
  - Statefull fire only once


#### Creating a Availability test:
- Name it
- Choose URL
- Enable retry
- Check SSL cert validity
- Test frequency
- Select regions to test from
- Add custom headers
- Define success criteria

#### Alert Rules
- Can also create alert rules directly from the Monitor | Alerts section of the Azure Portal
- Create action groups and configure things like notifications as part of creating an alert rule
- Could also crate an automation runbook in the actions
