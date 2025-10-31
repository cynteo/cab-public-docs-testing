---
title: "Common Issues and Solutions"
description: "Quick solutions to the most common Alert Bridge problems"
weight: 1
---

# Common Issues and Solutions

Quick solutions to the most common Alert Bridge problems.

---

## Alerts Not Creating Incidents

### Symptom
Alert fires in Azure Monitor, but no incident appears in SolarWinds.

### Diagnosis Steps

**1. Check Action Group Configuration**
- Go to **Monitor** → **Alerts** → **Action groups**
- Click your action group
- Verify webhook action exists
- ✅ Confirm "Enable common alert schema" is checked

**2. Check Logic App Run History**
- Go to Alert Bridge resource group
- Click Logic App resource
- Click "Overview" → "Runs history"
- Look for recent runs

**If no runs:** Action group not triggering  
**If runs exist:** Click run to see details

**3. Check Logic App Run Details**

**Success (green checkmark):**
- Incident should be in SolarWinds
- Check SolarWinds with search for alert name

**Failed (red X):**
- Click failed step
- Read error message
- See specific error solutions below

### Common Causes and Fixes

#### Cause 1: Common Alert Schema Not Enabled

**Error:** `Cannot read property 'essentials' of undefined`

**Fix:**
1. Edit action group
2. Edit webhook action
3. ✅ Check "Enable the common alert schema"
4. Save

#### Cause 2: Invalid SolarWinds API Token

**Error:** `401 Unauthorized` or `403 Forbidden`

**Fix:**
1. Generate new API token in SolarWinds
2. Go to Key Vault resource in Alert Bridge resource group
3. Click "Secrets"
4. Find secret (name: `solarwinds-*`)
5. Click "New version"
6. Paste new token
7. Save

#### Cause 3: SolarWinds Category Doesn't Exist

**Error:** `Category not found`

**Fix:**
- Option A: Create category in SolarWinds
- Option B: Update Logic App to use existing category (contact support)

#### Cause 4: Requester Email Invalid

**Error:** `Requester not found`

**Fix:**
- Create user in SolarWinds with that email, OR
- Update Logic App to use existing email (contact support)

---

## Duplicate Incidents Created

### Symptom
Multiple incidents created for the same alert.

### Causes

**1. Alert ID Changed**
- Azure generates new alert ID
- Alert Bridge can't deduplicate

**Fix:** Not much you can do - this is Azure Monitor behavior

**2. Multiple Action Groups**
- Same alert has multiple action groups pointing to Alert Bridge

**Fix:**
1. Check alert rule → Actions
2. Remove duplicate action groups

**3. Storage Table Issue**
- Alert tracking table corrupted

**Fix:** Contact support to clear tracking table

---

## Incidents Not Resolving

### Symptom
Alert resolves in Azure but incident stays open in SolarWinds.

### Diagnosis

**1. Check Alert Resolved Notification**
- Alerts must send "Resolved" notification
- Some alert rules don't send resolution notifications

**Fix:**
1. Edit alert rule
2. **Advanced options**
3. ✅ Check "Automatically resolve alerts"
4. Save

**2. Check Logic App Resolution Logic**
- Go to Logic App run history
- Find run when alert resolved
- Check if `monitorCondition = "Resolved"`

**If not "Resolved":** Azure not sending resolution notification

---

## Incident Missing Information

### Symptom
Incident created but description is empty or incomplete.

### Cause
Legacy alert schema used instead of common alert schema.

### Fix
1. Edit action group
2. Edit webhook action
3. ✅ Ensure "Enable common alert schema" is checked
4. Save
5. Test with new alert

---

## High Latency (Slow Incident Creation)

### Symptom
Alerts take > 5 minutes to create incidents.

### Normal Latency
- **Azure Monitor detection:** 1-5 minutes
- **Action group trigger:** < 30 seconds
- **Alert Bridge processing:** < 10 seconds
- **SolarWinds API:** < 5 seconds
- **Total:** Usually 2-6 minutes

### If Longer Than 10 Minutes

**1. Check Logic App Performance**
- Go to Logic App → Overview
- Check "Failed runs"
- Check "Throttled runs"

**2. Check SolarWinds API Performance**
- If SolarWinds is slow, incidents will be delayed
- Contact SolarWinds support

**3. Check Azure Region Issues**
- Azure Service Health may show incidents
- Check [Azure Status](https://status.azure.com)

---

## Incidents Have Wrong Priority

### Symptom
Incident priority doesn't match alert severity.

### Cause
Priority mapping configured incorrectly during deployment.

### Check Current Mapping

1. Go to Alert Bridge Function App
2. **Settings** → **Configuration**
3. Check environment variables:
   - `PRIORITY_SEV0`
   - `PRIORITY_SEV1`
   - `PRIORITY_SEV2`
   - `PRIORITY_SEV3`

### Fix
See [Priority Mapping Guide](../guides/priority-mapping)

---

## Deployment Errors

### "Resource names must be unique"

**Cause:** Resource name conflict with existing resources.

**Fix:**
1. Delete the resource group if it exists
2. Deploy again (Alert Bridge generates new unique names)

### "Insufficient permissions"

**Cause:** You don't have Owner/Contributor role on subscription.

**Fix:**
1. Ask subscription owner to grant you Contributor role
2. OR ask them to deploy for you

### "Managed application already exists"

**Cause:** Previous deployment wasn't fully cleaned up.

**Fix:**
1. Go to **Managed applications**
2. Find old deployment
3. Delete it
4. Wait 5 minutes
5. Deploy again

---

## Usage and Billing Issues

### "Exceeded alert limit"

**Symptom:** Email notification that you've reached plan limit.

### What Happens
- **Soft limit:** No service interruption
- Alerts continue to create incidents
- Overage charges may apply (if configured)

### Fix
- Upgrade plan if consistently over limit
- Or implement [severity filtering](../guides/severity-filtering)

---

## API Token Rotation

### Need to Update Token

**When to rotate:**
- Every 90 days (security best practice)
- If token compromised
- If permissions change

**How to update:**

1. **Generate new token in SolarWinds**
   - See [SolarWinds Setup](../getting-started/solarwinds-setup)

2. **Update in Key Vault**
   - Go to Key Vault in Alert Bridge resource group
   - **Secrets** → Find your secret
   - **New version** → Paste new token
   - Save

3. **Test**
   - Trigger test alert
   - Verify incident created

**No restart needed** - Logic App uses Key Vault reference and picks up new version automatically!

---

## Logic App Disabled or Stopped

### Symptom
Alerts stop creating incidents suddenly.

### Check Logic App Status

1. Go to Logic App resource
2. Check **Status** at top
3. Should be "Enabled"

### If Disabled

**Re-enable:**
1. Click **"Enable"** button
2. Wait ~30 seconds
3. Test with alert

**Why it happened:**
- Manual disable
- Too many failures (Azure auto-disables)
- Billing issue

---

## Cannot Find Webhook URL

### Need webhook URL to create action group

**How to find it:**

1. Azure Portal → Your resource group
2. Click **Logic App** resource (name: `logicapp-*`)
3. Click **"Overview"** tab
4. Look for **"Workflow URL"** or **"Callback URL"**
5. Click copy button

**If URL not visible:**
- You may not have permission (need Reader role)
- Ask deployment owner to share URL

---

## SolarWinds Tickets Have Wrong Category

### Symptom
Incidents created in wrong category.

### Cause
Category configured during deployment.

### Check Current Category

1. Go to Function App
2. **Settings** → **Configuration**
3. Check `INCIDENT_CATEGORY` value

### Fix
Contact support to update category (requires redeployment or manual config)

---

## Getting Help

### Before Contacting Support

Please gather:

1. **Logic App run history**
   - Screenshot of failed run
   - Error message

2. **Alert details**
   - Alert rule name
   - Severity
   - Time it fired

3. **Action group config**
   - Screenshot showing "Enable common alert schema" status

4. **What you've tried**
   - List of troubleshooting steps

### Contact Support

**Email:** support@cynteocloud.com

**Include:**
- Problem description
- Screenshots
- Steps to reproduce
- Azure subscription ID (if sharing access)

**Response time:**
- Standard: < 24 hours
- Enterprise: < 4 hours

---

## Additional Resources

- [Deployment Errors](./deployment-errors)
- [Alert Not Creating Tickets](./alert-not-creating-tickets)
- [Quick Start Guide](../getting-started/quickstart)
- [Configuration Guides](../guides/)

---

**Still stuck?** Email [support@cynteocloud.com](mailto:support@cynteocloud.com) with details.

