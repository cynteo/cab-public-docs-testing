---
title: "Configuration Options"
description: "Available configuration options for Cynteo Alert Bridge"
weight: 4
---

# Configuration Options

Learn about the configuration options available during deployment of Cynteo Alert Bridge.

---

## Required Settings

These are configured during initial deployment:

| Variable | Type | Description | Example |
|----------|------|-------------|---------|
| `SOLARWINDS_API_TOKEN` | Secret | SolarWinds API token | `Bearer abc123...` |
| `SOLARWINDS_BASE_URL` | String | SolarWinds instance URL | `https://api.samanage.com` |
| `REQUESTER_EMAIL` | String | Default requester email | `azure@company.com` |

---

## Optional Settings

These options can be configured during deployment to customize behavior:

### Incident Mapping

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `INCIDENT_CATEGORY` | String | `"Infrastructure"` | Default category name |
| `INCIDENT_SUBCATEGORY` | String | `"Azure Monitor"` | Default subcategory |
| `ASSIGNEE_GROUP` | String | `null` | Auto-assign to team |
| `INCIDENT_PREFIX` | String | `"Azure Alert:"` | Prefix for incident names |

### Priority Mapping

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `SEV0_PRIORITY` | String | `"Critical"` | Priority for Sev0 |
| `SEV1_PRIORITY` | String | `"High"` | Priority for Sev1 |
| `SEV2_PRIORITY` | String | `"Medium"` | Priority for Sev2 |
| `SEV3_PRIORITY` | String | `"Low"` | Priority for Sev3 |
| `SEV4_PRIORITY` | String | `"Low"` | Priority for Sev4 |

### Filtering

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `SEVERITY_FILTER` | String | `null` | Only process these severities (comma-separated) |
| `IGNORE_RESOLVED` | Boolean | `false` | Don't update incidents on resolution |
| `MIN_SEVERITY` | String | `null` | Minimum severity to process |

### Advanced Options

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `DEDUP_WINDOW_HOURS` | Number | `24` | How long to track alerts for dedup |
| `MAX_DESCRIPTION_LENGTH` | Number | `32000` | Truncate descriptions |
| `TIMEZONE` | String | `"UTC"` | Timezone for timestamps |
| `ENABLE_DEBUG_LOGGING` | Boolean | `false` | Verbose logging |

**Note:** Configuration options are set during deployment. Contact your administrator or [support@cynteocloud.com](mailto:support@cynteocloud.com) for configuration changes.

---

## Configuration Examples

These examples show how different settings affect incident creation:

### Example 1: Custom Priority Mapping

```json
{
  "SEV0_PRIORITY": "Critical",
  "SEV1_PRIORITY": "High",
  "SEV2_PRIORITY": "Medium",
  "SEV3_PRIORITY": "Low",
  "SEV4_PRIORITY": "Low"
}
```

### Example 2: Severity Filtering

Only create incidents for critical and high severity:

```json
{
  "SEVERITY_FILTER": "Sev0,Sev1"
}
```

Or use minimum severity:

```json
{
  "MIN_SEVERITY": "Sev2"
}
```

### Example 3: Custom Categories

```json
{
  "INCIDENT_CATEGORY": "Cloud Services",
  "INCIDENT_SUBCATEGORY": "Azure Monitoring",
  "ASSIGNEE_GROUP": "Cloud Operations Team"
}
```

### Example 4: Multiple Environments

For different environments (dev/staging/prod), use different prefixes:

**Production:**
```json
{
  "INCIDENT_PREFIX": "[PROD] Azure Alert:",
  "ASSIGNEE_GROUP": "Production Support"
}
```

**Staging:**
```json
{
  "INCIDENT_PREFIX": "[STAGING] Azure Alert:",
  "ASSIGNEE_GROUP": "Dev Team"
}
```

---

## Understanding Your Configuration

The deployment wizard collects these settings and applies them automatically. All configuration is managed by your Azure administrator.

For questions about your specific configuration or to request changes, contact [support@cynteocloud.com](mailto:support@cynteocloud.com).

---

## See Also

- [Priority Mapping](../guides/priority-mapping) - How severities map to priorities
- [Severity Filtering](../guides/severity-filtering) - Alert filtering options
- [Custom Categories](../guides/custom-categories) - SolarWinds categories

---

**Questions?** Contact [support@cynteocloud.com](mailto:support@cynteocloud.com)

