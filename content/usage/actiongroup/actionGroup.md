---
draft: false
title: Action Group Setup Guide
linktitle: Action Group Setup Guide
weight: 1
description: Bicep Quickstart Guidance for the Azure Verified Modules (AVM) program
---

Complete guide to setting up Azure Monitor action groups for Alert Bridge.

---

## What is an Action Group?

An **Action Group** is a collection of notification and action settings that can be reused across multiple alert rules. It defines **what happens** when an alert fires.

For Alert Bridge, the action group sends alert data to your SolarWinds integration via webhook.

---

## Prerequisites

- Azure subscription
- Alert Bridge deployed and running
- Contributor access to create action groups

---

## Step-by-Step Guide

### Step 1: Get Webhook URL

1. Go to **Azure Portal** → **Resource Groups**
2. Find your Alert Bridge resource group (e.g., `rg-alert-bridge`)
3. Click the **Logic App** resource (name: `logicapp-*`)
4. Click **"Overview"** tab
5. Find **"Workflow URL"** or **"Callback URL"**
6. Click **"Copy URL"** button

**The URL looks like:**
```
https://prod-123.eastus.logic.azure.com:443/workflows/abc123.../triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers...
```

**Keep this URL handy** - you'll need it in the next step.

---

### Step 2: Create Action Group

#### Navigate to Action Groups

1. In Azure Portal, search for **"Monitor"**
2. Click **"Alerts"** in the left navigation
3. Click **"Action groups"**
4. Click **"+ Create"** button

#### Basics Tab

**Subscription:** Select your Azure subscription  
**Resource Group:** Select same as Alert Bridge (recommended)  
**Region:** Global (default)  
**Action group name:** `alert-bridge-solarwinds`  
**Display name:** `SolarWinds`  

**Note:** Display name appears in email/SMS notifications, keep it short!

**Click "Next: Notifications >"** (skip for now)

#### Notifications Tab

**Skip this tab** - we'll use Actions instead.

**Click "Next: Actions >"**

#### Actions Tab

**Click "+ Add action"**

**Action type:** Select **"Webhook"** from dropdown

**Name:** `Send to SolarWinds`

**Details:**
- **URI:** Paste the webhook URL from Step 1
- **Enable common alert schema:** ✅ **CHECKED** (Critical!)

**Click "OK"**

**Important:** The "Enable common alert schema" checkbox MUST be checked for alerts to work properly!

#### Tags Tab (Optional)

Add tags if desired:
- `Environment: Production`
- `Purpose: ITSM Integration`

**Click "Next: Review + create >"**

#### Review and Create

1. Review your settings
2. Ensure "Enable common alert schema" shows **Yes**
3. Click **"Create"**

**Wait ~30 seconds for creation to complete.**

---

### Step 3: Test Action Group

#### Manual Test

1. After creation, click your action group name
2. Click **"Test action group"** button (top toolbar)
3. **Sample type:** Select "Metric Alert - CPU Percentage"
4. **Action type:** Webhook
5. Click **"Test"** button

#### Verify Results

**In Azure:**
- You should see "Test completed successfully"
- Check timestamp to confirm it just ran

**In SolarWinds:**
1. Go to **Incidents** list
2. Look for a new incident titled: "Azure Alert: Test"
3. Should appear within 1-2 minutes

**If incident appears:** ✅ Success! Your action group works!  
**If not:** See [Troubleshooting](#troubleshooting) below

---

### Step 4: Add to Alert Rules

Now add this action group to your existing alert rules:

#### For Each Alert Rule:

1. **Monitor** → **Alerts** → **Alert rules**
2. Click an alert rule you want to send to SolarWinds
3. Click **"Edit"** or **"Manage actions"**
4. **Action groups** section → Click **"+ Add action group"**
5. Select `alert-bridge-solarwinds`
6. Click **"Save"**

**Repeat for all relevant alerts.**

---

## Advanced Configuration

### Multiple Action Groups

You can create multiple action groups for different scenarios:

**Example 1: By Environment**
- `alert-bridge-prod` - Production alerts only
- `alert-bridge-dev` - Development alerts

**Example 2: By Severity**
- `alert-bridge-critical` - Sev0 and Sev1 only
- `alert-bridge-info` - Sev2 and Sev3

**Example 3: By Team**
- `alert-bridge-infrastructure`
- `alert-bridge-application`

### Adding Multiple Actions

You can add multiple actions to one action group:

**Example:** Email + SolarWinds
1. Add webhook action (SolarWinds)
2. Add email action (ops@company.com)
3. Both happen when alert fires

### Suppress Alerts

Configure **action group suppression** to prevent alert storms:

1. Edit alert rule
2. **Advanced options** → **Alert suppression**
3. **Suppress alerts for:** 5 minutes
4. Save

This prevents duplicate alerts within 5 minutes.

---

## Troubleshooting

### Test Failed: "Bad Request"

**Cause:** Common alert schema not enabled

**Fix:**
1. Edit action group
2. Edit webhook action
3. ✅ Check "Enable common alert schema"
4. Save and test again

### Test Succeeded But No Incident

**Possible causes:**

**1. Check Logic App Run History**
1. Go to Logic App resource
2. Click "Overview" → "Runs history"
3. Find the recent run
4. Click to see details

**If succeeded:** Issue is with SolarWinds API  
**If failed:** Check error message

**2. Check SolarWinds API Token**
1. Verify token is valid
2. Check token has create incident permission
3. See [SolarWinds Setup](../getting-started/solarwinds-setup)

**3. Check Logic App Configuration**
1. Ensure API token stored in Key Vault
2. Verify Key Vault access policy allows Logic App
3. Check SolarWinds base URL is correct

### Incidents Created But Missing Data

**Cause:** Legacy alert schema used

**Fix:**
1. Edit action group webhook action
2. ✅ Ensure "Enable common alert schema" is checked
3. Save

### Multiple Duplicate Incidents

**Cause:** Multiple action groups configured for same alert

**Fix:**
1. Check alert rule → Actions
2. Remove duplicate action groups
3. Keep only one SolarWinds action group per alert

---

## Common Alert Schema Example

When "Enable common alert schema" is checked, Azure sends:

```json
{
  "schemaId": "azureMonitorCommonAlertSchema",
  "data": {
    "essentials": {
      "alertId": "/subscriptions/.../Microsoft.Insights/...",
      "alertRule": "High CPU Usage",
      "severity": "Sev1",
      "signalType": "Metric",
      "monitorCondition": "Fired",
      "monitoringService": "Platform",
      "alertTargetIDs": ["/subscriptions/.../Microsoft.Compute/virtualMachines/vm1"],
      "configurationItems": ["vm1"],
      "originAlertId": "abc123",
      "firedDateTime": "2025-10-29T10:00:00Z",
      "description": "CPU has been above threshold",
      "essentialsVersion": "1.0",
      "alertContextVersion": "1.0"
    },
    "alertContext": {
      "properties": null,
      "conditionType": "SingleResourceMultipleMetricCriteria",
      "condition": {
        "windowSize": "PT5M",
        "allOf": [
          {
            "metricName": "Percentage CPU",
            "metricNamespace": "Microsoft.Compute/virtualMachines",
            "operator": "GreaterThan",
            "threshold": "80",
            "timeAggregation": "Average",
            "dimensions": [],
            "metricValue": 95.5
          }
        ]
      }
    }
  }
}
```

**Alert Bridge uses this structured data to create rich SolarWinds incidents!**

---

## Best Practices

### 1. Use Descriptive Names

**Good:** `alert-bridge-prod-infrastructure`  
**Bad:** `ag1`

### 2. One Action Group Per Environment

Separate prod and non-prod alerts:
- Better control
- Different SolarWinds priorities
- Easier troubleshooting

### 3. Document Your Action Groups

Add tags and descriptions:
```
Name: alert-bridge-prod
Description: Sends production alerts to SolarWinds Service Desk
Tags: Environment=Production, System=ITSM
```

### 4. Test Regularly

Test action groups:
- After creation
- After any configuration changes
- Monthly as part of DR testing

### 5. Monitor Action Group Usage

Use Azure Monitor to track:
- Action group invocations
- Failures
- Latency

---

## Security Considerations

### Webhook URL Security

The webhook URL contains:
- **API version** - Safe to expose
- **Signature parameter** - Validates request authenticity
- **Access token** - Securely embedded

**Keep the URL private** but it's safe if leaked (signature validation prevents abuse).

### Rotating URLs

If webhook URL is compromised:

1. Go to Logic App
2. **Settings** → **Access control**
3. **Regenerate access key**
4. Update all action groups with new URL

---

## Next Steps

- **[Configure Priority Mapping](./priority-mapping)**
- **[Set Up Severity Filtering](./severity-filtering)**
- **[Test Your Integration](../getting-started/quickstart.md#step-4-test-it)**

---

## Additional Resources

- [Azure Action Groups Documentation](https://docs.microsoft.com/azure/azure-monitor/alerts/action-groups)
- [Common Alert Schema Reference](https://docs.microsoft.com/azure/azure-monitor/alerts/alerts-common-schema)
- [Webhook Action Reference](https://docs.microsoft.com/azure/azure-monitor/alerts/action-groups-webhook)

---

**Need help?** Contact [support@cynteocloud.com](mailto:support@cynteocloud.com)
