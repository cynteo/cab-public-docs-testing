---
title: "Incident Fields Reference"
description: "Complete reference of what data Alert Bridge sends to SolarWinds Service Desk incidents"
weight: 1
---

# Incident Fields Reference

Complete reference of what data Alert Bridge sends to SolarWinds Service Desk incidents.

---

## Incident Structure

Every incident created by Alert Bridge includes:

### Basic Fields

| Field | Type | Source | Example |
|-------|------|--------|---------|
| **name** | String | Azure alert rule name | `Azure Alert: High CPU Usage` |
| **priority** | String | Severity mapping | `High`, `Medium`, `Low` |
| **description** | HTML | Enriched alert data | See [Description Format](#description-format) |
| **category** | Object | Configuration | `{"name": "Infrastructure"}` |
| **subcategory** | Object | Configuration | `{"name": "Azure Monitor"}` |
| **requester** | Object | Configuration | `{"email": "azure@company.com"}` |
| **state** | String | Alert condition | `New`, `In Progress`, `Resolved` |

### Optional Fields

| Field | Type | When Included | Example |
|-------|------|---------------|---------|
| **group_assignee** | Object | If configured | `{"name": "Cloud Ops"}` |
| **resolution_description** | String | On resolution | `Resolved automatically by Azure Monitor...` |

---

## Description Format

The incident description is rich HTML containing all alert context:

### Example Description

```html
<p><strong>[ALERT] Azure Monitor Alert</strong></p>
<hr>

<p><strong>Alert Information:</strong></p>
<ul>
  <li><strong>Name:</strong> High CPU Usage</li>
  <li><strong>Severity:</strong> Sev1</li>
  <li><strong>Status:</strong> Fired</li>
  <li><strong>Signal Type:</strong> Metric</li>
  <li><strong>Monitoring Service:</strong> Platform</li>
  <li><strong>Fired At:</strong> 2025-10-29T10:00:00Z</li>
</ul>

<p><strong>Affected Resource:</strong></p>
<ul>
  <li><strong>Name:</strong> vm-prod-web-01</li>
  <li><strong>Type:</strong> Microsoft.Compute/virtualMachines</li>
  <li><strong>Resource Group:</strong> rg-production</li>
  <li><strong>Subscription:</strong> sub-12345678-abcd-...</li>
</ul>

<p><strong>Metric Details:</strong></p>
<ul>
  <li><strong>Metric:</strong> Percentage CPU</li>
  <li><strong>Condition:</strong> Average GreaterThan 80</li>
  <li><strong>Current Value:</strong> 95.5</li>
</ul>

<p><strong>Evaluation Window:</strong> PT5M</p>
<p><em>Period: 2025-10-29T09:55:00Z to 2025-10-29T10:00:00Z</em></p>

<hr>
<p><strong>Quick Actions:</strong></p>
<ul>
  <li><a href="https://portal.azure.com/#...">[VIEW] View This Alert Instance</a></li>
  <li><a href="https://portal.azure.com/#...">[CONFIGURE] Edit Alert Rule Settings</a></li>
  <li><a href="https://portal.azure.com/#...">[RESOURCE] View Affected Resource in Portal</a></li>
</ul>
```

### Description Sections

#### 1. Alert Information
- Alert rule name
- Severity (Sev0-Sev3)
- Monitor condition (Fired/Resolved)
- Signal type (Metric/Log/Activity Log)
- When it fired

#### 2. Affected Resource
- Resource name
- Resource type
- Resource group
- Subscription ID

#### 3. Metric Details (for Metric Alerts)
- Metric name
- Threshold condition
- Current value
- Time aggregation method

#### 4. Log Search Details (for Log Alerts)
- Query results count
- Search query
- Links to log results

#### 5. Activity Log Details (for Activity Alerts)
- Operation name
- Status
- Caller

#### 6. Alert History (if repeat occurrence)
- Number of occurrences
- First occurrence time
- Pattern indicator

#### 7. Quick Action Links
- View alert in Azure Portal
- Edit alert rule
- View affected resource

---

## Alert States

### State Transitions

```
New → In Progress → Resolved
 ↑                      |
 └──────────────────────┘
    (New occurrence)
```

### State Descriptions

| State | When | Incident Action |
|-------|------|-----------------|
| **New** | First time alert fires | Create incident |
| **In Progress** | Alert fires again (update) | Update incident |
| **Resolved** | Alert condition clears | Resolve incident |

---

## Priority Mapping

Default mapping (configurable):

| Azure Severity | SolarWinds Priority | Typical Use |
|----------------|---------------------|-------------|
| Sev0 | High | Critical outages, data loss |
| Sev1 | High | Major functionality impaired |
| Sev2 | Medium | Performance degradation |
| Sev3 | Low | Informational, capacity planning |

**Customize:** See [Priority Mapping Guide](../guides/priority-mapping)

---

## Category and Subcategory

### Default Values

- **Category:** Infrastructure
- **Subcategory:** Azure Monitor

### Auto-Detection

Alert Bridge automatically detects category based on resource type:

| Resource Type | Category |
|---------------|----------|
| Virtual Machines | Infrastructure |
| App Service | Application |
| SQL Database | Database |
| Storage Account | Storage |
| Virtual Network | Network |
| Key Vault | Security |
| Kubernetes | Platform |

**Override:** Configure during deployment

---

## Comments

### When Comments Are Added

Comments update existing incidents instead of creating new ones:

**Conditions for adding comment:**
1. Alert fires again on same resource
2. Time since last update > 5 minutes (configurable)
3. OR metric value changed > 10%

**Comment includes:**
- Current time
- Monitor condition
- Current metric value
- Change from last value
- Occurrence number
- Links to Azure Portal

### Example Comment

```
[UPDATE] Alert Status Update

• Monitor Condition: Fired
• Time: 2025-10-29T10:15:00Z
• Current Value: 92.3 (+2.1 from last)
• Occurrence: #3
• Pattern: Recurring issue (3 occurrences)

Alert continues to fire. Monitoring situation.

[View Alert] | [Edit Rule]
```

### Smart Deduplication

To prevent comment spam:
- Minimum 5 minutes between comments (configurable)
- Only comment if metric changed significantly
- Resolution uses `resolution_description` instead

---

## Resolution Fields

When alert resolves:

### Incident Updates

```json
{
  "state": "Resolved",
  "resolution_description": "Resolved automatically by Azure Monitor. Alert condition returned to normal."
}
```

### Resolution Description Includes

- Resolution time
- Total duration
- Total occurrences
- Automatic resolution note

### Example

```
[RESOLVED] Alert Cleared by Azure Monitor

• Resolution Time: 2025-10-29T10:30:00Z
• Total Duration: 30 minutes
• Total Occurrences: 3
• Alert automatically cleared - condition returned to normal

This incident has been automatically resolved.

[View Alert Details]
```

---

## Requester Email

The requester is the "who created this incident" field in SolarWinds.

### Requirements

- Must be valid email
- Should exist in SolarWinds as a user OR
- SolarWinds auto-creates user (if enabled)

### Best Practices

**Option A: Service Account**
```
azure-monitor@company.com
```

**Option B: Team Inbox**
```
cloudops@company.com
```

**Option C: Shared Account**
```
itsm-automation@company.com
```

**Not recommended:**
- Individual personal emails
- Distribution lists (unless they're users in SolarWinds)

---

## Field Limits

### Character Limits

| Field | Max Length | Truncation |
|-------|-----------|------------|
| name | 255 chars | `...` added |
| description | 32,000 chars | Rare, full alerts fit |
| category | 100 chars | Error if exceeded |
| priority | 20 chars | Predefined values |

### Array Limits

| Field | Max Items |
|-------|-----------|
| alertTargetIDs | 100 resources |
| configurationItems | 100 items |

---

## Data Not Included

Alert Bridge does NOT send:

- ❌ Azure subscription credentials
- ❌ Resource access keys
- ❌ Connection strings
- ❌ Secrets from Key Vault
- ❌ VM passwords
- ❌ Storage account keys

Only **metadata** and **metrics** are sent to SolarWinds.

---

## Custom Fields (Coming Soon)

Future versions will support:
- Custom incident fields
- Additional tags
- Custom descriptions
- Field mapping rules

[View Roadmap](./roadmap)

---

## Example: Full Incident Payload

What Alert Bridge sends to SolarWinds API:

```json
{
  "incident": {
    "name": "Azure Alert: High CPU Usage",
    "priority": "High",
    "description": "<p><strong>[ALERT] Azure Monitor Alert</strong></p>...",
    "category": {
      "name": "Infrastructure"
    },
    "subcategory": {
      "name": "Azure Monitor"
    },
    "requester": {
      "email": "azure-monitor@company.com"
    },
    "group_assignee": {
      "name": "Cloud Operations"
    }
  }
}
```

---

## See Also

- [Alert Schema](./alert-schema) - Input from Azure Monitor
- [Environment Variables](./environment-variables) - Configuration options
- [Priority Mapping](../guides/priority-mapping) - Customize mapping
- [SolarWinds API](https://help.samanage.com/s/article/Incidents-API) - Official docs

---

**Questions?** Contact [support@cynteocloud.com](mailto:support@cynteocloud.com)

