---
title: "Deployment Errors"
description: "Troubleshoot Azure deployment errors for Alert Bridge"
weight: 2
---

# Deployment Errors

Solutions for common errors during Azure deployment of Alert Bridge.

---

## Common Deployment Errors

### Error: Resource Provider Not Registered

**Error Message:**
```
The subscription is not registered to use namespace 'Microsoft.Logic'
```

**Cause:** Logic Apps resource provider not registered in subscription

**Solution:**
```bash
az provider register --namespace Microsoft.Logic
az provider register --namespace Microsoft.KeyVault

# Wait for registration (takes 2-5 minutes)
az provider show --namespace Microsoft.Logic --query "registrationState"
```

Or via Portal:
1. Go to **Subscriptions** → Your subscription
2. Click **Resource providers**
3. Search for `Microsoft.Logic`
4. Click **Register**

---

### Error: Location Not Available

**Error Message:**
```
The location 'region-name' is not available for resource type 'Microsoft.Logic/workflows'
```

**Cause:** Logic Apps not available in selected region

**Solution:**
1. Choose a different region
2. Check Logic Apps availability:

```bash
az provider show \
  --namespace Microsoft.Logic \
  --query "resourceTypes[?resourceType=='workflows'].locations"
```

**Recommended Regions:**
- East US
- West US 2
- North Europe
- West Europe
- Southeast Asia

---

### Error: Insufficient Permissions

**Error Message:**
```
The client 'user@company.com' with object id 'xxx' does not have authorization to perform action 'Microsoft.Logic/workflows/write'
```

**Cause:** User lacks permissions to create Logic Apps

**Solution:**

Need one of these roles:
- **Owner** - Full access
- **Contributor** - Can create resources
- **Logic App Contributor** - Can manage Logic Apps

Request access from administrator:
```bash
az role assignment create \
  --assignee user@company.com \
  --role "Logic App Contributor" \
  --scope "/subscriptions/{subscription-id}/resourceGroups/{rg}"
```

---

### Error: Invalid API Token

**Error Message:**
```
Deployment validation failed: Invalid SolarWinds API token format
```

**Cause:** API token format incorrect

**Solution:**

Token must include "Bearer " prefix:
```
✅ Correct: Bearer eyJ0eXAiOiJKV1QiLCJhbGc...
❌ Wrong: eyJ0eXAiOiJKV1QiLCJhbGc...
```

Get token from SolarWinds:
1. SolarWinds → Profile → API
2. Generate new token
3. Copy full token including "Bearer "
4. Paste into deployment form

---

### Error: Key Vault Access Denied

**Error Message:**
```
The user or application does not have access to key vault secrets
```

**Cause:** Logic App Managed Identity lacks Key Vault permissions

**Solution:**

```bash
# Get Logic App identity
IDENTITY=$(az logicapp identity show \
  --name your-logic-app \
  --resource-group your-rg \
  --query principalId -o tsv)

# Grant Key Vault access
az keyvault set-policy \
  --name your-key-vault \
  --object-id $IDENTITY \
  --secret-permissions get list
```

Or via Portal:
1. Go to Key Vault → **Access policies**
2. Click **+ Create**
3. Select **Get** and **List** for secrets
4. Choose Logic App Managed Identity
5. Click **Create**

---

### Error: Resource Group Not Found

**Error Message:**
```
Resource group 'rg-name' could not be found
```

**Cause:** Trying to deploy to non-existent resource group

**Solution:**

Create resource group first:
```bash
az group create \
  --name cab-alert-bridge \
  --location eastus
```

Or via Portal:
1. Azure Portal → **Resource groups**
2. Click **+ Create**
3. Enter name and region
4. Click **Review + create**

---

### Error: Name Already Exists

**Error Message:**
```
Logic App name 'alert-bridge' already exists
```

**Cause:** Logic App name must be globally unique

**Solution:**

Use a unique name:
```
❌ alert-bridge
✅ company-alert-bridge
✅ alert-bridge-prod-eastus
✅ company-cab-prod-001
```

Check availability:
```bash
az logicapp show \
  --name your-name \
  --resource-group your-rg
```

---

### Error: Quota Exceeded

**Error Message:**
```
Operation results in exceeding quota limits. Maximum allowed: 100, Current in use: 100
```

**Cause:** Subscription quota limit reached

**Solution:**

1. **Check current usage:**
```bash
az logicapp list --query "length([])"
```

2. **Delete unused Logic Apps:**
```bash
az logicapp delete \
  --name unused-app \
  --resource-group rg-name
```

3. **Request quota increase:**
   - Azure Portal → **Subscriptions**
   - Click **Usage + quotas**
   - Search "Logic Apps"
   - Click **Request increase**

---

### Error: Template Validation Failed

**Error Message:**
```
Template validation failed: Required parameter 'apiToken' was not provided
```

**Cause:** Missing required deployment parameters

**Solution:**

Ensure all required parameters provided:

Required Parameters:
- ✅ Resource Group
- ✅ Logic App Name
- ✅ SolarWinds API Token
- ✅ SolarWinds Base URL
- ✅ Requester Email

Optional Parameters:
- Incident Category (default: "Infrastructure")
- Incident Subcategory (default: "Azure Monitor")
- Assignee Group (default: none)

---

## Deployment Validation

### Pre-Deployment Checklist

Before deploying, verify:

- [ ] **Subscription Access** - Owner or Contributor role
- [ ] **Resource Providers** - Microsoft.Logic registered
- [ ] **Region Availability** - Logic Apps available in region
- [ ] **API Token** - Valid SolarWinds token with "Bearer " prefix
- [ ] **SolarWinds URL** - Correct API endpoint
- [ ] **Resource Group** - Exists or will be created
- [ ] **Unique Name** - Logic App name not in use

### Test Deployment

Use Azure CLI to validate template:

```bash
az deployment group validate \
  --resource-group your-rg \
  --template-file template.json \
  --parameters @parameters.json
```

---

## Post-Deployment Issues

### Logic App Not Running

**Symptom:** Logic App deployed but not processing alerts

**Checks:**

1. **Logic App Status:**
```bash
az logicapp show \
  --name your-app \
  --resource-group your-rg \
  --query "state"
```

2. **Enable if disabled:**
```bash
az logicapp start \
  --name your-app \
  --resource-group your-rg
```

3. **Check for errors:**
   - Portal → Logic App → **Overview**
   - Look for error messages or warnings

### Webhook URL Not Generated

**Symptom:** Can't find webhook URL for action groups

**Solution:**

1. Go to Logic App → **Logic app designer**
2. Click the trigger at the top
3. Click **"Get trigger URL"** or **"Copy URL"**
4. If no URL shown:
   - Save the workflow
   - Check trigger is HTTP Request type
   - Verify Logic App is enabled

### Configuration Not Applied

**Symptom:** Settings provided during deployment not working

**Solution:**

1. Check Logic App configuration:
```bash
az logicapp config appsettings list \
  --name your-app \
  --resource-group your-rg
```

2. Update if needed:
```bash
az logicapp config appsettings set \
  --name your-app \
  --resource-group your-rg \
  --settings "INCIDENT_CATEGORY=Infrastructure"
```

---

## Rollback and Recovery

### Failed Deployment Cleanup

If deployment fails, clean up:

```bash
# Delete failed deployment
az deployment group delete \
  --name deployment-name \
  --resource-group your-rg

# Delete partially created resources
az resource delete --ids /subscriptions/.../resourceGroups/.../providers/...
```

### Start Over

For complete fresh start:

```bash
# Delete resource group (caution!)
az group delete --name your-rg --yes

# Create new resource group
az group create --name your-rg --location eastus

# Redeploy
# Use Azure Marketplace or ARM template
```

---

## Getting Help

### Collect Diagnostic Information

Before contacting support, gather:

1. **Deployment Error:**
   - Full error message
   - Deployment timestamp
   - Subscription ID
   - Resource group name

2. **Activity Log:**
```bash
az monitor activity-log list \
  --resource-group your-rg \
  --offset 1h
```

3. **Deployment Details:**
```bash
az deployment group show \
  --name deployment-name \
  --resource-group your-rg
```

### Contact Support

- **Email:** support@cynteocloud.com
- **Include:** All diagnostic information above
- **Response Time:** < 24 hours

---

## See Also

- [Quick Start Guide](../getting-started/quickstart) - Step-by-step deployment
- [Common Issues](./common-issues) - Runtime troubleshooting
- [SolarWinds Setup](../getting-started/solarwinds-setup) - API token generation

---

**Questions?** Contact [support@cynteocloud.com](mailto:support@cynteocloud.com)

