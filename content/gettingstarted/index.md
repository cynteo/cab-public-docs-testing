+++
title = "Getting Started"
type = "default"
weight = 2
+++

# Azure Monitor Configuration

Configure Azure Monitor alerts to work with Alert Bridge.

---

## Overview

Cynteo Alert Bridge integrates with Azure Monitor using **Action Groups** and **Common Alert Schema**. This guide shows you how to configure your alerts properly.

---

## Prerequisites

- Azure subscription with Monitor alerts configured
- Contributor access to create/modify alert rules
- Alert Bridge deployed ([Quick Start](./quickstart))

---

## Understanding Alert Flow

```
Azure Resource → Azure Monitor → Alert Rule → Action Group → Alert Bridge → SolarWinds
```

---

## Step 1: Create Action Group

### 1.1 Navigate to Action Groups

1. Open [Azure Portal](https://portal.azure.com)
2. Search for **"Monitor"**
3. Click **"Alerts"** in left menu
4. Click **"Action groups"**
5. Click **"+ Create"**

### 1.2 Configure Basics

**Subscription:** Your Azure subscription  
**Resource Group:** Same as Alert Bridge (recommended)  
**Region:** Global (default)  
**Action group name:** `alert-bridge-solarwinds`  
**Display name:** `SolarWinds` (shows in alert emails)

### 1.3 Add Webhook Action

Click **"Next: Actions"**

**Action type:** Webhook  
**Name:** `Send to SolarWinds`  
**URI:** Paste your Alert Bridge webhook URL

**To get webhook URL:**
1. Go to your Alert Bridge resource group
2. Click the Logic App (name: `logicapp-*`)
3. Click "Overview"
4. Copy "Callback URL"

**✅ CRITICAL:** Check **"Enable the common alert schema"**

This ensures alerts are sent in the correct format!

### 1.4 Review and Create

Click **"Review + create"** → **"Create"**

---

## Step 2: Configure Existing Alerts

### Add Action Group to Alert Rule

For each alert you want sent to SolarWinds:

1. **Monitor** → **Alerts** → **Alert rules**
2. Select an alert rule
3. Click **"Edit"** (or **"Manage actions"**)
4. Under **"Action groups"**, click **"+ Add action group"**
5. Select `alert-bridge-solarwinds`
6. Click **"Save"**

**Repeat for all relevant alert rules.**

---

## Step 3: Create New Alert Rule (Example)

### Example: CPU Alert for VM

#### 3.1 Create Alert Rule

1. Navigate to a **Virtual Machine**
2. Click **"Alerts"** in left menu
3. Click **"+ Create"** → **"Alert rule"**

#### 3.2 Configure Condition

**Signal:** Percentage CPU  
**Threshold:** Static  
**Operator:** Greater than  
**Threshold value:** 80  
**Aggregation type:** Average  
**Evaluation period:**
- Aggregation granularity: 5 minutes
- Frequency: 5 minutes

#### 3.3 Configure Actions

**Action group:** Select `alert-bridge-solarwinds`

#### 3.4 Configure Details

**Severity:** Select appropriate severity (Sev0-Sev3)  
**Alert rule name:** `High CPU Usage`  
**Description:** `Alert when CPU exceeds 80%`  
**Resource group:** Your resource group  
**Enable rule:** ✅ Yes

**Click "Review + create" → "Create"**

---

## Alert Severity Mapping

Azure Monitor severity maps to SolarWinds priority:

| Azure Severity | SolarWinds Priority | Use Case |
|----------------|---------------------|----------|
| Sev0 | High | Critical outages |
| Sev1 | High | Major issues |
| Sev2 | Medium | Performance degradation |
| Sev3 | Low | Informational |

**Configure this mapping during Alert Bridge deployment.**

---

## Common Alert Schema

### Why Common Alert Schema is Required

Alert Bridge requires **Common Alert Schema** to properly parse alerts.

**✅ Correct (Common Alert Schema):**
```json
{
  "schemaId": "azureMonitorCommonAlertSchema",
  "data": {
    "essentials": {
      "alertId": "...",
      "severity": "Sev1",
      "monitorCondition": "Fired"
    }
  }
}
```

**❌ Incorrect (Legacy Schema):**
```json
{
  "status": "Activated",
  "context": {
    "severity": "High"
  }
}
```

### How to Enable

**When creating action group:**
- ✅ Check **"Enable the common alert schema"**

**For existing action groups:**
1. Edit action group
2. Edit webhook action
3. ✅ Check **"Enable the common alert schema"**
4. Save

---

## Alert Types Supported

Alert Bridge supports all Azure Monitor alert types:

### Metric Alerts
- Virtual machine metrics (CPU, memory, disk)
- App Service metrics (response time, errors)
- Storage metrics (capacity, transactions)
- **ANY** Azure resource metric

### Log Alerts
- Log Analytics queries
- Application Insights queries
- Custom log searches

### Activity Log Alerts
- Resource health events
- Service health notifications
- Administrative operations

### Resource Health Alerts
- Resource availability changes
- Platform-initiated events

---

## Best Practices

### 1. Use Descriptive Alert Names

**Good:**
```
"Production Web App - High Response Time"
"Database Server - Low Memory"
"Storage Account - High Transactions"
```

**Bad:**
```
"Alert 1"
"Test"
"CPU"
```

Alert name becomes the SolarWinds incident title!

### 2. Set Appropriate Severity

- **Sev0:** Service down, data loss, security breach
- **Sev1:** Major functionality impaired
- **Sev2:** Degraded performance, non-critical issues
- **Sev3:** Informational, capacity planning

### 3. Add Helpful Descriptions

Descriptions appear in SolarWinds incident details:

```
"Alert triggers when average CPU exceeds 80% for 5 minutes. 
Indicates potential capacity issues. Check recent deployments 
and scale up if needed."
```

### 4. Use Dynamic Thresholds (when applicable)

For variable workloads, use **dynamic thresholds** instead of static:

- Automatically adapts to patterns
- Reduces false positives
- Better for seasonal/cyclical workloads

### 5. Configure Alert Suppression

Prevent alert storms:

1. Edit alert rule
2. **Advanced options** → **Alert suppression**
3. Enable for appropriate duration (e.g., 5 minutes)

This prevents multiple incidents for rapid-fire alerts.

---

## Testing Your Alerts

### Method 1: Lower Threshold Temporarily

1. Edit alert rule
2. Lower threshold to trigger immediately (e.g., CPU > 1%)
3. Save
4. Wait 1-2 minutes
5. Verify incident in SolarWinds
6. Restore original threshold

### Method 2: Use Test Action Group

1. Action group → **"Test"** button
2. Select sample alert type
3. Click **"Test"**
4. Check SolarWinds for test incident

---

## Troubleshooting

### Alert Fires But No Incident

**Check:**
1. Action group uses **common alert schema** ✅
2. Webhook URL is correct
3. Logic App run history shows success
4. [Troubleshooting Guide](../troubleshooting/alert-not-creating-tickets)

### Incident Missing Information

**Cause:** Legacy alert schema used

**Fix:** Enable common alert schema in action group

### Duplicate Incidents

**Cause:** Multiple action groups sending same alert

**Fix:** Remove duplicate action groups from alert rule

---

## Advanced Configuration

### Multi-Subscription Setup

**Option A: Deploy Alert Bridge in each subscription**
- Isolated per subscription
- Separate billing per deployment

**Option B: Use one Alert Bridge for all subscriptions**
1. Deploy Alert Bridge in "hub" subscription
2. Get webhook URL
3. Create action groups in each subscription pointing to same URL
4. All alerts go to same SolarWinds instance

### Filtering by Resource Group

Alert Bridge processes ALL alerts sent to it. To filter:

1. Create separate action groups for different resource groups
2. Or use [severity filtering](../guides/severity-filtering)
3. Or configure SolarWinds rules to auto-close certain incidents

### Custom Fields

Alert Bridge includes:
- Alert ID (for deduplication)
- Resource information
- Metric values
- Azure portal links

For additional custom fields, see [Advanced Configuration](../guides/advanced-configuration).

---

## Next Steps

- **[Configure Priority Mapping](../guides/priority-mapping)**
- **[Set Up Severity Filtering](../guides/severity-filtering)**
- **[Test Your Integration](./quickstart.md#step-4-test-it)**

---

**Need help?** Contact [support@cynteocloud.com](mailto:support@cynteocloud.com)