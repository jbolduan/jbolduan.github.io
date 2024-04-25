---
title: Setting Up Azure Diagnostics for Intune Logging to Splunk
date: 2024-04-25 10:14:00 -0500
categories: Intune Azure Splunk
---

When it comes to analyzing logs from Intune and pulling them into Splunk the process is very simple.  I've only set up the Azure/Intune side of things but the Splunk side should be fairly simple based on my reading.

## Azure Setup

1. Log into the [Azure Portal](https://portal.azure.com/).
2. Create a resource group for the Intune/Splunk integration.
    - This can be done separately or when you create the Log Analytics Workspace.
3. Create a Log Analytics Workspace.
4. Create an Event Hubs Namespace.
    - All left as default aside from naming.
5. Create an Event Hub.

## Intune Setup

1. Log into the [Intune Portal](https://endpoint.microsoft.com/).
2. Navigate to **Tenant Administration** > **Diagnostic settings**.
3. Click **Add diagnostic setting**.
4. Name the setting.
5. Select the **Send to Log Analytics** option and choose the Log Analytics Workspace created earlier.
6. Select the **Stream to an event hub** option and choose the Event Hub created earlier.
7. Select the categories of logs to send to the Event Hub.
8. Click **Save**.

## Splunk Permission Setup

1. Log into the [Azure Portal](https://portal.azure.com/).
2. Navigate to Microsoft Entra ID.
3. Create a **New registration**.
    - Give the application a name.
    - Select the **Accounts in this organizational directory only** option.
    - Click **Register**.
4. Create a **Client Secret**.
    - Click **Certificates & secrets**.
    - Click **New client secret**.
    - Give the secret a description.
    - Set the expiration.
    - Click **Add**.
    - **Note**: The secret will only be displayed once.
5. Navigate to the **Event Hub Namespace** created earlier and go to the **Access control (IAM)** section.
6. Assign the **Azure Event Hubs Data Receiver** permission.

## Splunk Setup

This is the portion I haven't done.  It should be fairly simple using the [Microsoft Azure Add-on for Splunk](https://splunkbase.splunk.com/app/3757).
