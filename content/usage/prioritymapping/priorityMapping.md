---
title: "Priority Mapping"
description: "Configure how Azure alert severities map to SolarWinds priorities"
weight: 2
---

# Priority Mapping

Learn how to map Azure Monitor alert severities to SolarWinds Service Desk priority levels.

---

## Overview

Azure Monitor uses **severities** (Sev0-Sev4), while SolarWinds uses **priorities** (Critical, High, Medium, Low). Alert Bridge maps between these automatically.

---

## Default Mapping

Out of the box, Alert Bridge uses this mapping:

| Azure Severity | SolarWinds Priority | Description |
|----------------|---------------------|-------------|
| **Sev0** | Critical | System down, immediate action required |
| **Sev1** | High | Major functionality impaired |
| **Sev2** | Medium | Moderate impact, workaround available |
| **Sev3** | Low | Minor issue, minimal impact |
| **Sev4** | Low | Informational, no immediate action |

---

## Customizing Priority Mapping

Priority mapping is configured during deployment. The solution can be customized to match your organization's SolarWinds priority structure.

**To request a priority mapping change**, contact [support@cynteocloud.com](mailto:support@cynteocloud.com) with your desired mapping.

---

## SolarWinds Priority Values

Your SolarWinds instance may have different priority names. Common priority structures include:

**Standard SolarWinds Priorities:**
- Critical
- High
- Medium  
- Low

Your organization may use custom priority names. The mapping is configured to match your specific SolarWinds priority structure during deployment.

---

## Custom Priority Examples

Alert Bridge supports various priority mapping strategies:

### Example 1: Escalated Priorities
All critical and error alerts mapped to high priority for faster response.

### Example 2: Custom Priority Names
Organizations using custom priority names (e.g., "P1 - Critical", "P2 - Urgent") can configure the mapping accordingly.

### Example 3: Simplified Mapping
Some organizations prefer to use only two priority levels (Critical and Normal) for simplicity.

### Example 4: Environment-Specific
Different environments (Production vs Development) can use different priority mappings to reflect business impact.

---

## Verifying Priority Mapping

When an Azure alert fires, you can verify the priority mapping worked correctly:

### Check the SolarWinds Incident

1. Open SolarWinds Service Desk
2. Find the incident created from the alert
3. Verify the priority field matches expectations

For example, an Azure Sev1 alert should create a High priority incident (based on default mapping).

---

## Priority-Based Routing

Different priority levels can trigger different response workflows in SolarWinds:

### SolarWinds Assignment Rules

SolarWinds supports automatic assignment based on priority:
   - **Critical Priority** → Route to On-Call Team
   - **High Priority** → Route to Infrastructure Team
   - **Medium/Low Priority** → Route to Operations Team

This ensures the right team receives alerts based on severity.

---

## Advanced Priority Scenarios

Alert Bridge supports advanced priority mapping strategies including:

- **Resource Type Based** - Different resources can have different priority mappings
- **Environment Based** - Production alerts prioritized higher than development
- **Time-Based** - After-hours alerts automatically escalated
- **Tag-Based** - Resource tags influence priority assignment

Contact [support@cynteocloud.com](mailto:support@cynteocloud.com) to discuss custom mapping requirements.

---

## See Also

- [Configuration Options](../reference/environment-variables) - Available settings
- [Severity Filtering](./severity-filtering) - Filter by severity
- [Custom Categories](./custom-categories) - SolarWinds categories

---

**Questions?** Contact [support@cynteocloud.com](mailto:support@cynteocloud.com)
