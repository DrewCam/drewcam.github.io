---
title: "Troubleshooting Framework: The 5Ws + How"
draft: true
date: 2024-12-07
categories:
  - Cybersecurity
  - Investigations
tags:
  - Troubleshooting
  - Incident Response
  - Framework
---

# Troubleshooting Framework - The 5Ws + How

When I find myself wading through logs, stuck on a strange alert, or fighting with complexity in an investigation, I rely on a simple but powerful approach: the 5Ws + How. It's the same set of questions reporters use, but turned into a security troubleshooting framework. Walking through these questions helps ensure I'm not missing critical details and keeps me from getting lost in the weeds.

<!-- more -->

## Who

Which account or entity triggered the alert? Which IP address did it originate from, and what device is relevant here? Understanding these entities ensures I know the scope and potential impact.

**Key Points:**

- User: `johndoe@example.com`
- IP Address: `192.168.x.x`
- Device: `WIN-12345`
- Affected Stakeholders: Consider the business impact and affected clients.

## What

What exactly happened? Did a security product detect anomalous behavior, a suspicious login event, or known malicious activity?

**Key Points:**

- Alert Details:

  - Name: e.g., "Impossible Travel Detected"
  - Source: e.g., Sentinel, Defender for Endpoint
  - Severity: e.g., High, Medium, Low
- Type of Threat/Activity: e.g., Phishing, Ransomware, Unauthorized Access

## Why

Why did the alert trigger? Maybe it's an unusual login location or an unpatched system. Why does it matter? Understanding relevance helps determine if this is a critical issue or a minor anomaly.

**Key Points:**

- Root Cause or Trigger: e.g., Unusual login location, unpatched system
- Relevance: Potential lateral movement, chain-of-events indicating a bigger threat

## Where

Where is this happening? Is it in Azure AD logs, a file share, or endpoint telemetry? Where geographically did the activity originate?

**Key Points:**

- Systems/Platforms: e.g., Azure AD, Defender for Endpoint, File Share
- Affected Regions: e.g., Simultaneous logins from the US and India

## When

When did it start, and what is the timeline of events? Identifying the sequence helps correlate with other incidents or changes in the environment.

**Key Points:**

- Initial Alert Time: e.g., 2024-11-25 14:00 UTC
- Document timestamps for each investigation step

## How

How do I investigate further? Which tools and queries will help confirm or rule out suspicions?

**Key Points:**

- Sentinel: Use KQL queries to identify related events

  ```kql
  SecurityEvent
  | where TimeGenerated > ago(1d)
  | where Account == "johndoe@example.com"
  ```

- Defender for Endpoint: Review device threat history
- Azure Portal: Examine sign-in logs, resource health logs for anomalies

By stepping through these questions, I keep on track and maintain structure in my investigative process. Each "W" plus "How" ties the pieces together, helping me quickly identify where to focus my efforts.
