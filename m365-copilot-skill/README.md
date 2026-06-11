# M365 Tenant Health and Admin Assistant

A Microsoft 365 Copilot API plugin that surfaces Microsoft Graph data directly in your Copilot prompt bar. IT admins can check service health, active incidents, message center announcements, and license usage without leaving Teams or Outlook.

---

## What It Does

| Capability | Example Prompts |
|---|---|
| Service health overview | "Is Teams healthy right now?" / "Which M365 services have issues?" |
| Active incidents | "What outages are happening today?" / "Show me active Exchange incidents" |
| Incident details | "What is the full timeline for incident MO123456?" |
| Message center | "What changes are coming to SharePoint next month?" / "Are there any required admin actions?" |
| Tenant licenses | "How many E3 licenses do we have left?" / "What licenses does our tenant subscribe to?" |
| User licenses | "What licenses does jane.doe@contoso.com have?" |

---

## Architecture

```
ai-plugin.json        M365 Copilot plugin manifest (auth, functions, conversation starters)
openapi.yaml          OpenAPI 3.0 spec describing 6 Microsoft Graph API operations
```

The plugin calls Microsoft Graph API directly using delegated permissions under the signed-in admin's identity. No intermediate backend is required.

**Graph API endpoints used:**

| Operation | Endpoint | Permission |
|---|---|---|
| `getServiceHealthOverviews` | `GET /admin/serviceAnnouncement/healthOverviews` | `ServiceHealth.Read.All` |
| `getServiceHealthIssues` | `GET /admin/serviceAnnouncement/issues` | `ServiceHealth.Read.All` |
| `getServiceHealthIssueById` | `GET /admin/serviceAnnouncement/issues/{issueId}` | `ServiceHealth.Read.All` |
| `getMessageCenterAnnouncements` | `GET /admin/serviceAnnouncement/messages` | `ServiceHealth.Read.All` |
| `getTenantLicenses` | `GET /subscribedSkus` | `Directory.Read.All` |
| `getUserLicenseDetails` | `GET /users/{userPrincipalName}/licenseDetails` | `User.Read.All` |

---

## Prerequisites

- Microsoft 365 Copilot license assigned to users who will use the plugin
- An Azure app registration with the permissions listed above (admin consent granted)
- Global Administrator or Service Support Administrator role to read service health
- Microsoft 365 admin center access to upload the plugin (or Teams admin center)

---

## Setup

### Step 1 — Register an App in Entra ID

1. Go to [portal.azure.com](https://portal.azure.com) → **Microsoft Entra ID** → **App registrations** → **New registration**
2. Name: `M365 Copilot Admin Assistant`
3. Supported account types: **Accounts in this organizational directory only**
4. Redirect URI: `https://teams.microsoft.com/api/platform/v1.0/oAuthRedirect` (type: Web)
5. Click **Register**

### Step 2 — Add API Permissions

In the app registration, go to **API permissions** → **Add a permission** → **Microsoft Graph** → **Delegated permissions**. Add:
- `ServiceHealth.Read.All`
- `Directory.Read.All`
- `User.Read.All`

Click **Grant admin consent for [your tenant]**.

### Step 3 — Create a Client Secret

Go to **Certificates & secrets** → **New client secret**. Copy the secret value — you will need it during plugin upload.

### Step 4 — Update the Plugin Manifest

In `ai-plugin.json`, replace `<YOUR-APP-REGISTRATION-CLIENT-ID>` with the **Application (client) ID** from your app registration. Also update `contact_email` with your admin email.

### Step 5 — Upload to Microsoft 365 Copilot

**Option A — Microsoft 365 Admin Center (tenant-wide):**
1. Go to [admin.microsoft.com](https://admin.microsoft.com) → **Settings** → **Integrated apps**
2. Click **Upload custom apps** → **Upload from device**
3. Upload `ai-plugin.json` (and ensure `openapi.yaml` is accessible at the relative URL configured)
4. Assign to all users or specific groups

**Option B — Teams Admin Center:**
1. Go to [admin.teams.microsoft.com](https://admin.teams.microsoft.com) → **Teams apps** → **Manage apps**
2. Upload the plugin package
3. Grant access to users or groups

**Option C — Individual install (for testing):**
1. Open Microsoft 365 Copilot in Teams or at [m365.cloud.microsoft](https://m365.cloud.microsoft)
2. Click the plugin icon in the prompt bar → **Manage plugins**
3. Upload the plugin for personal use

---

## Usage

Once installed, reference the plugin in any Copilot prompt. The conversation starters from `ai-plugin.json` will appear as suggested prompts.

**Service health:**
```
Are there any active M365 service degradations right now?
```

**Incident detail:**
```
Get the full timeline and status updates for incident MO654321
```

**Message center (filtered):**
```
Show me all major changes coming to Microsoft Teams in the next 30 days
```

**License management:**
```
How many Microsoft 365 E5 licenses has our tenant consumed vs purchased?
```

```
Does alex.smith@contoso.com have a Teams Phone license?
```

---

## Customization

**Add a specific service filter by default:** Update `description_for_model` in `ai-plugin.json` to instruct the model to always filter to your primary services.

**Expose more Graph endpoints:** Add new paths to `openapi.yaml` following the same pattern, add corresponding function entries in `ai-plugin.json`, and add each new function name to `run_for_functions` in the runtime block.

**Switch to application permissions:** For unattended/service scenarios, change the auth flow to client credentials and update the app registration to use application (not delegated) permissions. Note that `ServiceHealth.Read.All` supports both flows.

---

## Known Limitations

- Service health and message center data requires the signed-in user to have the **Service Support Administrator** or **Global Reader** role. Standard users without this role will receive a 403 error.
- `Directory.Read.All` is a high-privilege scope — review your organization's app permission policies before granting admin consent.
- The plugin calls Graph API under the user's delegated identity, so data returned is scoped to what that user is authorized to see.
