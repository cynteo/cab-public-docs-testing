---
title: "Security Overview"
description: "Security architecture and compliance information for Cynteo Alert Bridge"
weight: 2
---

# Security Overview

Learn how Cynteo Alert Bridge protects your data and maintains security compliance.

---

## Security Architecture

### Zero Data Storage

- **No Persistence** - Alert data is processed in real-time and never stored
- **Stateless Processing** - Each alert is handled independently
- **No Logging of Sensitive Data** - Only operational logs without credentials

### Secure Credential Management

- **Azure Key Vault** - All API tokens stored encrypted in your Key Vault
- **Managed Identity** - No hardcoded credentials in code or configuration
- **Automatic Rotation** - Support for Key Vault secret rotation
- **Least Privilege** - Logic App uses minimum required permissions

### Network Security

- **HTTPS Only** - All communication encrypted in transit
- **Private Endpoints** - Optional VNet integration available
- **IP Restrictions** - Configure allowed source IPs in Logic App
- **Webhook Validation** - Azure Monitor signatures verified

---

## Compliance

### SOC 2 Type II

Cynteo Alert Bridge is SOC 2 Type II compliant:
- Annual audits performed
- Security controls verified
- Compliance report available upon request

### Data Residency

- **Your Subscription** - All resources deployed to YOUR Azure subscription
- **Your Region** - Deploy to any Azure region you choose
- **Your Control** - Full RBAC and access control
- **No Third-Party Storage** - Data never leaves your environment

### GDPR Compliance

- **No Personal Data Processing** - Only technical monitoring data
- **Data Controller** - You remain the data controller
- **Right to Delete** - Delete resources anytime
- **Privacy by Design** - Minimal data collection

---

## Access Control

### Azure RBAC

Control who can:
- View Logic App workflow
- Modify configuration
- Access run history
- Manage credentials

### SolarWinds Permissions

API token should have:
- ✅ Create incidents
- ✅ Update incidents
- ✅ Add comments
- ❌ Delete incidents (not required)
- ❌ Manage users (not required)

---

## Audit & Monitoring

### Azure Monitor Integration

- **Activity Logs** - All configuration changes logged
- **Diagnostic Logs** - Logic App execution logs
- **Alerts** - Set up alerts on Logic App failures
- **Metrics** - Track success/failure rates

### Run History

- **30-Day Retention** - Logic App maintains 30-day history
- **Success/Failure Status** - See which alerts succeeded
- **Error Details** - Debug failed executions
- **No Sensitive Data** - PII redacted from logs

---

## Security Best Practices

### Recommended Configuration

1. **Enable Diagnostic Logs**
   - Send to Log Analytics workspace
   - Monitor for anomalies
   - Set up alerting

2. **Use Managed Identity**
   - Don't use connection strings
   - Leverage Azure AD authentication
   - Enable when possible

3. **Rotate Credentials**
   - Update SolarWinds token annually
   - Use Key Vault for storage
   - Enable secret expiration alerts

4. **Network Isolation**
   - Use Private Endpoints if required
   - Configure NSG rules
   - Limit public exposure

5. **Monitor Access**
   - Review RBAC assignments quarterly
   - Audit Logic App modifications
   - Track who accesses run history

---

## Incident Response

### Security Incident Contacts

- **Email:** security@cynteocloud.com
- **Response Time:** < 4 hours for security issues
- **Escalation:** 24/7 on-call team available

### Vulnerability Disclosure

Report security vulnerabilities to:
- security@cynteocloud.com
- PGP key available upon request
- Responsible disclosure policy

---

## Compliance Documentation

Available upon request:
- SOC 2 Type II Report
- Penetration Test Results
- Security Architecture Diagrams
- Data Flow Diagrams

Contact [sales@cynteocloud.com](mailto:sales@cynteocloud.com) for access.

---

## See Also

- [Incident Fields](./incident-fields) - What data is sent
- [Environment Variables](./environment-variables) - Configuration options
- [SolarWinds Setup](../getting-started/solarwinds-setup) - API token permissions

---

**Questions?** Contact [support@cynteocloud.com](mailto:support@cynteocloud.com)

