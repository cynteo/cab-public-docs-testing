---
title: "API"
description: "API reference for Cynteo Alert Bridge webhooks and integrations"
weight: 5
---

# API Documentation

Technical reference for integrating with Cynteo Alert Bridge.

---

## Webhook Endpoint

Alert Bridge provides a secure webhook endpoint that accepts Azure Monitor alerts.

### Endpoint URL

After deployment, a unique webhook URL is generated for your instance. This URL is:
- Automatically configured in your Action Groups during setup
- Protected with signature-based authentication
- Unique to your deployment

**🔒 Security:** The webhook URL contains authentication tokens and should be kept confidential.

---

## Request Format

### Headers

```http
POST /workflows/{id}/triggers/manual/paths/invoke HTTP/1.1
Host: prod-XX.region.logic.azure.com
Content-Type: application/json
User-Agent: Azure-Alerts/1.0
```

### Body

Alert Bridge expects Azure Monitor Common Alert Schema:

```json
{
  "schemaId": "azureMonitorCommonAlertSchema",
  "data": {
    "essentials": {
      "alertId": "string",
      "alertRule": "string",
      "severity": "Sev0|Sev1|Sev2|Sev3|Sev4",
      "signalType": "Metric|Log|Activity Log",
      "monitorCondition": "Fired|Resolved",
      "monitoringService": "string",
      "alertTargetIDs": ["string"],
      "originAlertId": "string",
      "firedDateTime": "ISO8601 string",
      "resolvedDateTime": "ISO8601 string",
      "description": "string"
    },
    "alertContext": {
      // Alert type specific context
    }
  }
}
```

See [Alert Schema Reference](./alert-schema) for complete details.

---

## Response Format

### Success Response

```http
HTTP/1.1 202 Accepted
Content-Type: application/json

{
  "status": "accepted",
  "incidentId": "12345",
  "incidentNumber": "INC-001234",
  "action": "created|updated",
  "timestamp": "2025-10-29T10:00:00Z"
}
```

### Error Responses

#### 400 Bad Request

```json
{
  "error": {
    "code": "InvalidSchema",
    "message": "Alert schema validation failed",
    "details": "Missing required field: alertRule"
  }
}
```

#### 401 Unauthorized

```json
{
  "error": {
    "code": "InvalidSignature",
    "message": "Request signature validation failed"
  }
}
```

#### 500 Internal Server Error

```json
{
  "error": {
    "code": "SolarWindsError",
    "message": "Failed to create incident in SolarWinds",
    "details": "API returned 401 Unauthorized"
  }
}
```

---

## Rate Limits

### Azure Logic Apps Limits

| Tier | Requests/Min | Requests/Day |
|------|--------------|--------------|
| Consumption | 60 | 86,400 |
| Standard | 600 | Unlimited |

### SolarWinds API Limits

| Tier | Requests/Hour |
|------|---------------|
| Free | 100 |
| Standard | 1,000 |
| Premium | 5,000 |

**Note:** Alert Bridge automatically queues requests to stay within limits.

---

## Testing the Integration

### Using Azure Action Group Test

The easiest way to test Alert Bridge:

1. Go to **Azure Portal** → **Monitor** → **Action groups**
2. Select your action group
3. Click **"Test"**
4. Select the webhook action
5. Click **"Test"**
6. Check SolarWinds for the test incident

### Using Azure Alerts

Fire a real alert to test end-to-end functionality:

1. Create a test alert rule with easy-to-trigger conditions
2. Wait for the alert to fire
3. Verify incident appears in SolarWinds
4. Check that all fields are populated correctly

---

## Integration Security

### Webhook Authentication

- Each deployment has a unique, signed webhook URL
- Authentication is handled automatically
- URLs contain time-limited signatures

### Best Practices

- **Keep webhook URLs confidential** - treat them like passwords
- **Use Azure Action Groups** - don't share webhook URLs externally
- **Monitor for unauthorized access** - review execution logs regularly

---

## Monitoring Integration Health

### Check Processing Status

You can monitor Alert Bridge processing through Azure:

1. **Azure Portal** → Your Alert Bridge resource
2. View **Run History** to see recent alert processing
3. Check for any failures or errors
4. Review execution times

### Common Status Indicators

- ✅ **Succeeded** - Alert processed and incident created/updated
- ⚠️ **Failed** - Error occurred (check details)
- ⏸️ **Skipped** - Alert filtered (by severity or other rules)

---

## See Also

- [Alert Schema](./alert-schema) - Input format details
- [Incident Fields](./incident-fields) - Output format
- [Configure Action Groups](../guides/configure-alert-action-group) - Setup guide

---

**Questions?** Contact [support@cynteocloud.com](mailto:support@cynteocloud.com)

