---
title: "Alert Schema Reference"
description: "Azure Monitor Common Alert Schema explained"
weight: 3
---

# Alert Schema Reference

Understanding the Azure Monitor Common Alert Schema that Alert Bridge processes.

---

## Overview

Azure Monitor uses the **Common Alert Schema** to provide a consistent format for all alert types. This makes it easier to process alerts from different sources.

---

## Schema Structure

### Top-Level Fields

```json
{
  "schemaId": "azureMonitorCommonAlertSchema",
  "data": {
    "essentials": { ... },
    "alertContext": { ... }
  }
}
```

### Essentials Section

Core alert information (present in all alerts):

| Field | Type | Description |
|-------|------|-------------|
| `alertId` | String | Unique alert instance ID |
| `alertRule` | String | Name of the alert rule |
| `severity` | String | `Sev0`, `Sev1`, `Sev2`, `Sev3`, `Sev4` |
| `signalType` | String | `Metric`, `Log`, `Activity Log` |
| `monitorCondition` | String | `Fired`, `Resolved` |
| `monitoringService` | String | `Platform`, `Log Analytics`, etc. |
| `alertTargetIDs` | Array | Affected Azure resource IDs |
| `originAlertId` | String | Original alert identifier |
| `firedDateTime` | String | When alert fired (ISO 8601) |
| `resolvedDateTime` | String | When alert resolved (ISO 8601) |
| `description` | String | Alert description |

---

## Alert Context

Additional details specific to alert type:

### Metric Alerts

```json
{
  "alertContext": {
    "properties": {},
    "conditionType": "SingleResourceMultipleMetricCriteria",
    "condition": {
      "windowSize": "PT5M",
      "allOf": [
        {
          "metricName": "Percentage CPU",
          "metricNamespace": "Microsoft.Compute/virtualMachines",
          "operator": "GreaterThan",
          "threshold": "90",
          "timeAggregation": "Average",
          "dimensions": [],
          "metricValue": 95.5
        }
      ]
    }
  }
}
```

### Log Alerts

```json
{
  "alertContext": {
    "SearchQuery": "Heartbeat | where TimeGenerated > ago(5m)",
    "SearchIntervalStartTimeUtc": "2025-10-29T10:00:00Z",
    "SearchIntervalEndtimeUtc": "2025-10-29T10:05:00Z",
    "ResultCount": 0,
    "LinkToSearchResults": "https://portal.azure.com/...",
    "SeverityDescription": "Informational",
    "WorkspaceId": "..."
  }
}
```

### Activity Log Alerts

```json
{
  "alertContext": {
    "authorization": {
      "action": "Microsoft.Compute/virtualMachines/write",
      "scope": "/subscriptions/..."
    },
    "channels": "Operation",
    "claims": "...",
    "caller": "user@company.com",
    "eventSource": "Administrative",
    "eventTimestamp": "2025-10-29T10:00:00Z",
    "operationName": "Microsoft.Compute/virtualMachines/write",
    "operationId": "...",
    "status": "Succeeded",
    "subStatus": "",
    "submissionTimestamp": "2025-10-29T10:00:05Z"
  }
}
```

---

## How Alert Bridge Uses Schema

### Field Mapping

Alert Bridge extracts these fields to create SolarWinds incidents:

| Azure Field | SolarWinds Field | Transformation |
|-------------|------------------|----------------|
| `alertRule` | `name` | Prefixed with "Azure Alert:" |
| `severity` | `priority` | Mapped via configuration |
| `essentials + alertContext` | `description` | Rich HTML format |
| `monitorCondition` | `state` | Fired→New, Resolved→Resolved |
| `firedDateTime` | Incident timestamp | ISO 8601 to local |

### Deduplication Key

Alert Bridge uses these fields to identify unique alerts:

```
{alertId}-{originAlertId}
```

Multiple firings of the same alert **update** the existing incident rather than creating duplicates.

---

## Example: Complete Alert

```json
{
  "schemaId": "azureMonitorCommonAlertSchema",
  "data": {
    "essentials": {
      "alertId": "/subscriptions/.../providers/Microsoft.AlertsManagement/alerts/...",
      "alertRule": "High CPU Usage",
      "severity": "Sev1",
      "signalType": "Metric",
      "monitorCondition": "Fired",
      "monitoringService": "Platform",
      "alertTargetIDs": [
        "/subscriptions/.../resourceGroups/prod/providers/Microsoft.Compute/virtualMachines/vm-web-01"
      ],
      "originAlertId": "...",
      "firedDateTime": "2025-10-29T10:00:00.0000000Z",
      "description": "CPU usage is above 90% for vm-web-01"
    },
    "alertContext": {
      "properties": {},
      "conditionType": "SingleResourceMultipleMetricCriteria",
      "condition": {
        "windowSize": "PT5M",
        "allOf": [
          {
            "metricName": "Percentage CPU",
            "metricNamespace": "Microsoft.Compute/virtualMachines",
            "operator": "GreaterThan",
            "threshold": "90",
            "timeAggregation": "Average",
            "dimensions": [],
            "metricValue": 95.5,
            "webTestName": null
          }
        ],
        "windowStartTime": "2025-10-29T09:55:00Z",
        "windowEndTime": "2025-10-29T10:00:00Z"
      }
    }
  }
}
```

---

## Testing the Schema

### Send Test Alert

Use this PowerShell script to test:

```powershell
$uri = "https://your-logic-app-url"
$body = Get-Content -Path "test-alert.json"

Invoke-RestMethod -Uri $uri -Method Post -Body $body -ContentType "application/json"
```

### Validate in Logic App

1. Go to Logic App in Azure Portal
2. View **Run History**
3. Click latest run
4. Inspect **Inputs** to see raw alert JSON
5. Check **Outputs** for what was sent to SolarWinds

---

## See Also

- [Incident Fields](./incident-fields) - Output format
- [Configure Action Groups](../guides/configure-alert-action-group) - Setup guide
- [Azure Monitor Documentation](https://docs.microsoft.com/azure/azure-monitor/alerts/alerts-common-schema)

---

**Questions?** Contact [support@cynteocloud.com](mailto:support@cynteocloud.com)

