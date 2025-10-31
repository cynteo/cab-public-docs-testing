---
title: "SolarWinds API Token"
description: "How to generate and manage SolarWinds Service Desk API tokens"
weight: 5
---

# SolarWinds API Token

Complete guide to generating and managing your SolarWinds Service Desk API token for Alert Bridge.

---

## What is an API Token?

An API token is a secure authentication credential that allows Alert Bridge to:
- ✅ Create incidents in SolarWinds
- ✅ Update existing incidents
- ✅ Add comments to incidents
- ✅ Resolve incidents when alerts clear

---

## Generating a Token

### Step 1: Log into SolarWinds

1. Go to your SolarWinds Service Desk URL
2. Log in with administrator credentials

### Step 2: Navigate to API Settings

**Option A: Modern Interface**
1. Click your **profile** (top-right)
2. Select **"Developer" or "API"**
3. Click **"Tokens"** or **"API Access"**

**Option B: Classic Interface**
1. Go to **Setup** → **Account**
2. Find **"API Tokens"** or **"Integrations"**
3. Click **"Generate New Token"**

### Step 3: Create Token

1. Click **"Generate New Token"** or **"Create Token"**
2. **Name:** `Cynteo Alert Bridge`
3. **Description:** `Azure Monitor integration`
4. **Permissions:** Select required permissions (see below)
5. Click **"Generate"** or **"Create"**

### Step 4: Copy Token

⚠️ **Important:** Copy the token NOW - you won't see it again!

```
Bearer eyJ0eXAiOiJKV1QiLCJhbGc...
```

Store securely in:
- Azure Key Vault (recommended)
- Password manager
- Secure note

---

## Required Permissions

The API token needs these permissions:

| Permission | Required | Purpose |
|------------|----------|---------|
| **Read Incidents** | ✅ Yes | Check for existing incidents (dedup) |
| **Create Incidents** | ✅ Yes | Create new incidents from alerts |
| **Update Incidents** | ✅ Yes | Update incidents when alerts fire again |
| **Add Comments** | ✅ Yes | Add alert updates as comments |
| **Resolve Incidents** | ⚠️ Optional | Auto-resolve when alerts clear |
| **Delete Incidents** | ❌ No | Not required (security best practice) |
| **Manage Users** | ❌ No | Not required |
| **Admin Access** | ❌ No | Not required |

### Minimal Permissions

For maximum security, only grant:
```
- incidents:read
- incidents:create
- incidents:update
- comments:create
```

---

## Token Format

### Full Token

```
Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJodHRwczovL2FwaS5zYW1hbmFnZS5jb20iLCJpYXQiOjE2MTYxNjE2MTYsImV4cCI6MTY0NzY5NzYxNiwianRpIjoiYWJjMTIzIn0.signature
```

### Token Parts

- **Prefix:** `Bearer ` (with space)
- **Type:** JWT (JSON Web Token)
- **Parts:** Header.Payload.Signature

---

## Configuring Alert Bridge

### Option 1: Direct (Not Recommended)

Store token directly in Logic App configuration:

```json
{
  "SOLARWINDS_API_TOKEN": "Bearer eyJ0eXAiOi..."
}
```

⚠️ **Not recommended** - Token visible in Logic App configuration

### Option 2: Azure Key Vault (Recommended)

Store token in Key Vault:

#### Step 1: Add to Key Vault

```bash
az keyvault secret set \
  --vault-name your-key-vault \
  --name solarwinds-api-token \
  --value "Bearer eyJ0eXAiOi..."
```

#### Step 2: Grant Logic App Access

```bash
# Enable Managed Identity on Logic App
az logicapp identity assign \
  --name your-logic-app \
  --resource-group your-rg

# Grant Key Vault access
az keyvault set-policy \
  --name your-key-vault \
  --object-id <logic-app-identity-id> \
  --secret-permissions get
```

#### Step 3: Reference in Logic App

```json
{
  "SOLARWINDS_API_TOKEN": "@Microsoft.KeyVault(SecretUri=https://your-vault.vault.azure.net/secrets/solarwinds-api-token/)"
}
```

---

## Testing the Token

### Via cURL

```bash
curl -X GET "https://api.samanage.com/incidents.json" \
  -H "X-Samanage-Authorization: Bearer YOUR_TOKEN" \
  -H "Accept: application/json"
```

**Expected:** List of incidents (200 OK)

**Error 401:** Token invalid or expired

### Via PowerShell

```powershell
$headers = @{
    "X-Samanage-Authorization" = "Bearer YOUR_TOKEN"
    "Accept" = "application/json"
}

Invoke-RestMethod -Uri "https://api.samanage.com/incidents.json" -Headers $headers
```

### Via Logic App Test

1. Deploy Logic App with token
2. Trigger a test alert
3. Check Logic App run history
4. Look for SolarWinds API errors

---

## Token Security

### Best Practices

1. **Use Key Vault** - Never store in plain text
2. **Rotate Regularly** - Update every 90 days
3. **Minimal Permissions** - Only grant what's needed
4. **Monitor Usage** - Track API calls for anomalies
5. **Separate Tokens** - Different tokens for dev/prod

### Rotation Schedule

| Environment | Rotation Frequency |
|-------------|-------------------|
| Production | Every 90 days |
| Staging | Every 180 days |
| Development | Annual |

### Compromised Token

If token is compromised:

1. **Immediately Revoke** in SolarWinds
2. **Generate New Token**
3. **Update Key Vault** with new token
4. **Restart Logic App**
5. **Review Audit Logs** for unauthorized access

---

## Token Management

### Viewing Active Tokens

1. SolarWinds → **API Settings**
2. View list of active tokens
3. See:
   - Token name
   - Creation date
   - Last used date
   - Expiration (if set)

### Revoking Tokens

1. Find token in list
2. Click **"Revoke"** or **"Delete"**
3. Confirm revocation
4. Token immediately invalidated

### Updating Token in Alert Bridge

After generating new token:

#### If using Key Vault:

```bash
az keyvault secret set \
  --vault-name your-key-vault \
  --name solarwinds-api-token \
  --value "Bearer NEW_TOKEN_HERE"
```

Logic App automatically picks up new value within 5 minutes.

#### If using direct configuration:

1. Go to Logic App → **Configuration**
2. Update `SOLARWINDS_API_TOKEN`
3. Click **Save**
4. Logic App restarts

---

## Troubleshooting

### 401 Unauthorized

**Causes:**
- Token expired
- Token revoked
- Token format incorrect (missing "Bearer ")
- Wrong SolarWinds instance URL

**Solutions:**
1. Generate new token
2. Verify token format: `Bearer ` + token
3. Check SolarWinds base URL matches your instance

### 403 Forbidden

**Causes:**
- Token lacks required permissions
- User account disabled
- IP restrictions

**Solutions:**
1. Check token permissions in SolarWinds
2. Verify user account is active
3. Check API access restrictions

### Token Not Found in Key Vault

**Causes:**
- Secret name mismatch
- Logic App lacks Key Vault permissions
- Secret deleted

**Solutions:**
1. Verify secret exists: `az keyvault secret show`
2. Check Logic App Managed Identity has access
3. Restore secret if deleted

---

## API Limits

Be aware of SolarWinds API rate limits:

| Plan | Requests/Hour | Requests/Day |
|------|---------------|--------------|
| Trial | 100 | 1,000 |
| Standard | 1,000 | 10,000 |
| Premium | 5,000 | 50,000 |
| Enterprise | Custom | Custom |

Alert Bridge automatically respects these limits with request queuing.

---

## See Also

- [SolarWinds Setup](../getting-started/solarwinds-setup) - Complete setup guide
- [Security Overview](../reference/security) - Security best practices
- [Environment Variables](../reference/environment-variables) - Configuration options

---

**Questions?** Contact [support@cynteocloud.com](mailto:support@cynteocloud.com)
