---
title: "Alert Not Creating Tickets"
description: "Debug why Azure alerts aren't creating SolarWinds incidents"
weight: 3
---

# Alert Not Creating Tickets

Troubleshoot why your Azure Monitor alerts aren't creating SolarWinds Service Desk incidents.

---

## Quick Diagnostic Checklist

Work through this checklist systematically:

- [ ] Alert is actually firing in Azure Monitor
- [ ] Action group is configured correctly
- [ ] Logic App is enabled and running
- [ ] Logic App run history shows the alert
- [ ] SolarWinds API token is valid
- [ ] No severity filtering blocking the alert
- [ ] SolarWinds API is accessible

---

## Step 1: Verify Alert is Firing

### Check Azure Monitor

1. Go to **Azure Portal** → **Monitor** → **Alerts**
2. Look for your alert in the list
3. Check **Alert state:**
   - ✅ **Fired** - Alert is active
   - ❌ **Resolved** - Alert cleared (won't trigger unless configured)
   - ❌ **Not yet evaluated** - Waiting for data

### Check Alert History

1. Click your alert rule
2. Go to **History** tab
3. Look for recent state changes
4. If no history → Alert hasn't fired yet

### Manually Fire Test Alert

Create a condition that will definitely fire:

**For Metric Alert:**
```bash
az monitor metrics alert create \
  --name test-alert \
  --resource-group your-rg \
  --scopes "/subscriptions/.../resourceGroups/.../providers/..." \
  --condition "avg Percentage CPU > 1" \
  --description "Test alert - will fire immediately"
```

Set threshold very low so it fires immediately.

---

## Step 2: Verify Action Group Configuration

### Check Action Group

1. Go to **Monitor** → **Action groups**
2. Find your action group
3. Verify it contains:
   - ✅ Webhook action
   - ✅ Correct Logic App webhook URL
   - ✅ Common alert schema enabled

### Test Action Group

1. Click your action group
2. Click **"Test"** button
3. Select **"Webhook"** action type
4. Select **"Common alert schema"**
5. Click **"Test"**
6. Check Logic App run history

### Verify Alert Rule Uses Action Group

1. Go to your alert rule
2. Check **Actions** tab
3. Verify action group is assigned
4. If not, click **"Add action group"**

---

## Step 3: Check Logic App Status

### Is Logic App Enabled?

```bash
az logicapp show \
  --name your-logic-app \
  --resource-group your-rg \
  --query "state"
```

**Expected:** `"Enabled"`

**If Disabled:**
```bash
az logicapp start \
  --name your-logic-app \
  --resource-group your-rg
```

### Check Logic App Run History

1. Go to Logic App → **Overview**
2. Click **"Runs history"**
3. Look for recent runs
4. Check run status:
   - ✅ **Succeeded** - Alert processed successfully
   - ⚠️ **Failed** - See error details
   - ❌ **No runs** - Alert not reaching Logic App

---

## Step 4: Diagnose Logic App Runs

### No Runs at All

**Cause:** Alerts not reaching Logic App

**Checks:**

1. **Webhook URL correct?**
   - Get URL: Logic App → Designer → Trigger → Copy URL
   - Compare with action group webhook
   - Must match exactly

2. **Logic App triggered manually?**
   ```bash
   # Get trigger URL
   az logicapp show --name your-app --resource-group your-rg
   
   # Test with curl
   curl -X POST "https://your-logic-app-url" \
     -H "Content-Type: application/json" \
     -d '{"test": "data"}'
   ```

3. **Network restrictions?**
   - Check if Logic App has IP restrictions
   - Ensure Azure Monitor IPs are allowed

### Runs Failing

**Cause:** Error in Logic App execution

**Steps:**

1. Click failed run
2. Expand each step
3. Look for red X (failed step)
4. Read error message

**Common Errors:**

#### SolarWinds Authentication Error

```
401 Unauthorized: Invalid API token
```

**Solutions:**
- Verify token in Logic App configuration
- Check "Bearer " prefix included
- Generate new token if expired
- Test token with curl:

```bash
curl -X GET "https://api.samanage.com/incidents.json" \
  -H "X-Samanage-Authorization: Bearer YOUR_TOKEN"
```

#### SolarWinds API Error

```
400 Bad Request: Invalid category name
```

**Solutions:**
- Verify category exists in SolarWinds
- Check spelling (case-sensitive)
- Verify priority values match SolarWinds
- Check required fields are populated

#### Timeout Error

```
504 Gateway Timeout: SolarWinds API did not respond
```

**Solutions:**
- Check SolarWinds API status
- Retry the failed run
- Check SolarWinds API rate limits
- Verify network connectivity

### Runs Succeeding But No Ticket

**Cause:** Incident created but you can't find it

**Checks:**

1. **Check Logic App output:**
   - Click succeeded run
   - Expand "Create or Update Incident" action
   - Check **Outputs**
   - Look for `incidentId` or `incidentNumber`

2. **Search SolarWinds:**
   ```
   Search for: "Azure Alert"
   Or: The specific alert name
   Filter by: Date created (today)
   ```

3. **Check SolarWinds response:**
   - In Logic App outputs, look for SolarWinds response
   - Verify incident was created (200 or 201 status)
   - Note the incident ID

---

## Step 5: Check Filtering Configuration

### Severity Filtering

Alert might be filtered out by severity:

**Check Configuration:**
```bash
az logicapp config appsettings list \
  --name your-app \
  --resource-group your-rg \
  --query "[?name=='SEVERITY_FILTER' || name=='MIN_SEVERITY']"
```

**Examples:**

```json
{
  "SEVERITY_FILTER": "Sev0,Sev1"
}
```
→ Only Sev0 and Sev1 create tickets. Your Sev2/3/4 alert is ignored.

**Solution:** Update or remove filter:
```bash
az logicapp config appsettings delete \
  --name your-app \
  --resource-group your-rg \
  --setting-names SEVERITY_FILTER MIN_SEVERITY
```

---

## Step 6: Network and Connectivity

### Can Logic App Reach SolarWinds?

Test connectivity:

1. **Via Logic App:**
   - Designer → Add new HTTP action
   - URL: `https://api.samanage.com/api_information.json`
   - Method: GET
   - Headers: API token
   - Run test

2. **Check Firewall:**
   - SolarWinds might have IP restrictions
   - Add Logic App outbound IPs to whitelist
   - Get IPs:
   ```bash
   az logicapp show \
     --name your-app \
     --resource-group your-rg \
     --query "outboundIpAddresses"
   ```

### SolarWinds API Status

Check if SolarWinds API is experiencing issues:

1. Visit [SolarWinds Status Page](https://status.samanage.com)
2. Or test directly:
   ```bash
   curl -I https://api.samanage.com
   ```

---

## Step 7: Common Alert Schema

### Verify Schema Format

Logic App expects Common Alert Schema. Check action group:

1. Go to action group
2. Click webhook action
3. Verify **"Enable common alert schema"** is ✅ checked

**If unchecked:**
1. Check the box
2. Click **Save**
3. Test again

### Validate Alert Payload

Check if Azure is sending correct format:

1. Logic App → Run History
2. Click latest run
3. Check **Trigger outputs**
4. Verify structure:

```json
{
  "schemaId": "azureMonitorCommonAlertSchema",
  "data": {
    "essentials": { ... },
    "alertContext": { ... }
  }
}
```

If different, enable common alert schema in action group.

---

## Step 8: Permissions and Access

### Logic App Managed Identity

If using Key Vault:

```bash
# Check if Managed Identity enabled
az logicapp identity show \
  --name your-app \
  --resource-group your-rg

# Grant Key Vault access
az keyvault set-policy \
  --name your-vault \
  --object-id <identity-id> \
  --secret-permissions get
```

### SolarWinds API Token Permissions

Token needs these permissions:
- ✅ Read incidents
- ✅ Create incidents
- ✅ Update incidents
- ✅ Add comments

**Verify in SolarWinds:**
1. Profile → API Tokens
2. Find your token
3. Check permissions
4. Regenerate if needed

---

## Debugging Tools

### Enable Diagnostic Logs

```bash
az monitor diagnostic-settings create \
  --name logic-app-diagnostics \
  --resource /subscriptions/.../resourceGroups/.../providers/Microsoft.Logic/workflows/your-app \
  --logs '[{"category": "WorkflowRuntime", "enabled": true}]' \
  --workspace /subscriptions/.../resourceGroups/.../providers/Microsoft.OperationalInsights/workspaces/your-workspace
```

### View Logs in Log Analytics

```kusto
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.LOGIC"
| where Resource == "YOUR-LOGIC-APP"
| where TimeGenerated > ago(1h)
| project TimeGenerated, status_s, error_message_s
| order by TimeGenerated desc
```

### Test with Sample Alert

Use this test payload:

```bash
curl -X POST "https://your-logic-app-url" \
  -H "Content-Type: application/json" \
  -d '{
  "schemaId": "azureMonitorCommonAlertSchema",
  "data": {
    "essentials": {
      "alertId": "/subscriptions/test/providers/Microsoft.AlertsManagement/alerts/test-123",
      "alertRule": "Test Alert",
      "severity": "Sev1",
      "signalType": "Metric",
      "monitorCondition": "Fired",
      "monitoringService": "Platform",
      "alertTargetIDs": ["/subscriptions/test/resourceGroups/test/providers/Microsoft.Compute/virtualMachines/test-vm"],
      "originAlertId": "test-origin",
      "firedDateTime": "2025-10-29T10:00:00Z",
      "description": "Test alert for troubleshooting"
    },
    "alertContext": {}
  }
}'
```

---

## Still Not Working?

### Collect Information

Before contacting support:

1. **Logic App run history** (screenshot or export)
2. **Alert rule configuration** (screenshot)
3. **Action group configuration** (screenshot)
4. **Error messages** (full text)
5. **Logic App configuration:**
   ```bash
   az logicapp config appsettings list \
     --name your-app \
     --resource-group your-rg
   ```

### Contact Support

- **Email:** support@cynteocloud.com
- **Include:** All information from above
- **Response Time:** < 24 hours

---

## See Also

- [Common Issues](./common-issues) - Other troubleshooting topics
- [Configure Action Groups](../guides/configure-alert-action-group) - Setup guide
- [API Documentation](../reference/api) - Technical reference

---

**Questions?** Contact [support@cynteocloud.com](mailto:support@cynteocloud.com)

