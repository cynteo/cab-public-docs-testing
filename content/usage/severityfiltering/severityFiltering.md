---
title: "Severity Filtering"
description: "Filter which alert severities create SolarWinds incidents"
weight: 3
---

# Severity Filtering

Configure Alert Bridge to only create incidents for specific severity levels.

---

## Overview

Not all alerts need a ticket. Use severity filtering to:
- Reduce ticket noise
- Focus on critical issues
- Save SolarWinds API calls
- Improve team efficiency

---

## Azure Alert Severities

Azure Monitor supports 5 severity levels:

| Severity | Description | Typical Use |
|----------|-------------|-------------|
| **Sev0** | Critical | Service outage, data loss |
| **Sev1** | Error | Major degradation, multiple users affected |
| **Sev2** | Warning | Performance issues, single user affected |
| **Sev3** | Informational | Potential issues, proactive monitoring |
| **Sev4** | Verbose | Debug info, detailed diagnostics |

---

## Filtering Options

Severity filtering is configured during deployment to control which alerts create incidents.

### Option 1: Whitelist Specific Severities
Only create incidents for specific severities (e.g., only Sev0 and Sev1).

**Result:** Only critical and error alerts create incidents. Warning and informational alerts are ignored.

### Option 2: Minimum Severity Threshold
Create incidents for alerts at or above a severity threshold (e.g., Sev2 and above).

**Result:** Critical, error, and warning alerts create incidents. Informational alerts are ignored.

### Option 3: No Filtering (Default)
All severities create incidents for comprehensive tracking.

**To configure severity filtering**, specify your requirements during deployment or contact [support@cynteocloud.com](mailto:support@cynteocloud.com).

---

## Common Filtering Scenarios

### Scenario 1: Critical Only
Only create incidents for service outages (Sev0 alerts).

**Use Case:** Production environments where only critical issues need immediate attention through tickets.

### Scenario 2: Critical and High
Create incidents for major issues (Sev0 and Sev1 alerts).

**Use Case:** Balance between noise reduction and comprehensive tracking of significant issues.

### Scenario 3: Warning and Above
Exclude informational alerts (Sev3 and Sev4 filtered out).

**Use Case:** Keep informational alerts in Azure Monitor but don't create tickets for them.

### Scenario 4: Environment-Specific Filtering
Different environments can have different filtering rules.

**Example:**
- **Production:** Track Sev0, Sev1, and Sev2 alerts
- **Development:** Only track Sev0 alerts

---

## Understanding Your Filtering

When configured, severity filtering automatically processes alerts according to your rules:

- **Matching alerts** → Create/update incidents in SolarWinds
- **Filtered alerts** → Logged but no incident created

You can verify filtering is working by checking SolarWinds for which incidents are being created.

---

## Advanced Filtering Scenarios

Alert Bridge supports sophisticated filtering strategies beyond simple severity levels:

### Resource Type Filtering
Filter based on Azure resource type (e.g., only create tickets for VM and database alerts).

### Time-Based Filtering
Apply different filtering rules based on time of day or day of week (e.g., all severities during business hours, critical only after hours).

### Tag-Based Filtering
Use Azure resource tags to control which resources create incidents.

### Multi-Criteria Filtering
Combine severity with other factors like subscription, resource group, or alert type.

---

## Filter Strategy Best Practices

### Start Strict, Relax as Needed
Organizations often start with stricter filtering (critical alerts only) and gradually expand to include additional severities based on operational needs.

### Environment-Appropriate Filtering

| Environment | Typical Filter |
|-------------|----------------|
| **Production** | Sev0, Sev1, Sev2 |
| **Staging** | Sev0, Sev1 |
| **Development** | Sev0 only |

### Monitor the Full Picture
Even when alerts are filtered from ticket creation, they remain visible in Azure Monitor for trending and analysis.

---

## Questions About Filtering?

To configure or adjust severity filtering for your deployment, contact [support@cynteocloud.com](mailto:support@cynteocloud.com).

---

## See Also

- [Priority Mapping](./priority-mapping) - Map severities to priorities
- [Configuration Options](../reference/environment-variables) - Available settings
- [Azure Monitor Setup](../getting-started/azure-monitor-setup) - Alert configuration

---

**Questions?** Contact [support@cynteocloud.com](mailto:support@cynteocloud.com)
