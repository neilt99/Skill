# O365 Security Copilot Skillset

A Microsoft Security Copilot custom plugin providing 16 KQL-based investigation skills and an AI orchestration agent for Microsoft 365 security operations. Covers email threats, identity risks, mailbox compromise, SharePoint/OneDrive data exposure, and admin audit.

---

## Overview

This plugin enables SOC analysts to investigate Microsoft 365 security incidents directly from the Security Copilot prompt bar. Skills query Microsoft Defender XDR advanced hunting tables in real time, and the included orchestration agent automatically chains the right skills together based on the indicator you provide.

**Supported investigation types:**
- Phishing and malware email analysis
- Business Email Compromise (BEC) detection
- User account compromise and MFA bypass
- Privileged role escalation and admin abuse
- Data exfiltration via SharePoint and OneDrive
- Conditional access policy gap analysis

---

## Prerequisites

All skills query Microsoft Defender XDR via the `Defender` KQL target. The following licenses and services must be active in your tenant:

| Requirement | Required For |
|---|---|
| Microsoft Security Copilot (SCU provisioned) | Running any skill |
| Microsoft Defender for Office 365 Plan 2 | `EmailEvents`, `EmailAttachmentInfo`, `EmailUrlInfo`, `EmailPostDeliveryEvents` |
| Microsoft Defender for Cloud Apps (M365 connector enabled) | `CloudAppEvents` — Exchange, SharePoint, OneDrive skills |
| Microsoft Defender for Identity | `IdentityDirectoryEvents` — role change skills |
| Microsoft Entra ID P2 | `EntraIdSignInEvents` with risk level data |
| Security Reader or higher (Defender portal) | Account uploading and running the plugin |

---

## Deployment

1. Sign in to [securitycopilot.microsoft.com](https://securitycopilot.microsoft.com)
2. Click the **Plugin** button in the prompt bar (bottom right)
3. Scroll to **Custom** and click **Add plugin**
4. Select **Security Copilot plugin** as the upload format
5. Upload `manifest.yaml` from this repository
6. Confirm the plugin **O365 Security Copilot Skillset** appears under Custom plugins
7. Enable it with the toggle if it is not already active

---

## Skill Reference

### Domain 1 — Email Security

| Skill Name | Display Name | Required Inputs | Optional Inputs | Description |
|---|---|---|---|---|
| `GetEmailByNetworkMessageId` | Get Email By Network Message ID | `networkMessageId` | — | Full email details with attachments, URLs, threat types, and delivery location |
| `GetEmailsFromSuspiciousSender` | Get Emails From Suspicious Sender | `senderAddress` | `lookbackDays` | All emails from a sender across all recipients — blast radius analysis |
| `GetEmailsWithMaliciousContent` | Get Emails With Malicious Content | — | `lookbackDays` | Tenant-wide malware, phishing, and high-confidence phish emails |
| `GetEmailDeliveryTimeline` | Get Email Delivery Timeline | `networkMessageId` | — | End-to-end delivery and post-delivery action history for one email |

### Domain 2 — Identity and Access

| Skill Name | Display Name | Required Inputs | Optional Inputs | Description |
|---|---|---|---|---|
| `GetRiskySignInsForUser` | Get Risky Sign-Ins For User | `userPrincipalName` | `lookbackDays` | Failed and high-risk sign-ins with IP, country, and risk level |
| `GetMFAStatusForUser` | Get MFA Status For User | `userPrincipalName` | `lookbackDays` | Successful sign-ins showing whether MFA was required per session |
| `GetPrivilegedRoleChanges` | Get Privileged Role Changes | — | `lookbackDays` | Entra ID privileged role assignment additions and removals |
| `GetConditionalAccessFailures` | Get Conditional Access Failures | — | `lookbackDays` | Sign-ins where conditional access policies were not applied |

### Domain 3 — Mailbox and Exchange

| Skill Name | Display Name | Required Inputs | Optional Inputs | Description |
|---|---|---|---|---|
| `GetMailboxPermissionChanges` | Get Mailbox Permission Changes | — | `lookbackDays` | Mailbox delegate and full-access permission grants (BEC indicator) |
| `GetMailForwardingRuleChanges` | Get Mail Forwarding Rule Changes | — | `lookbackDays` | New or modified inbox/transport rules forwarding email externally |
| `GetSuspiciousMailboxAccess` | Get Suspicious Mailbox Access | `userDisplayName` | `lookbackDays` | Anomalous mailbox access: multi-country or high-volume per hour |

### Domain 4 — SharePoint and OneDrive

| Skill Name | Display Name | Required Inputs | Optional Inputs | Description |
|---|---|---|---|---|
| `GetExternalSharingEvents` | Get External Sharing Events | — | `lookbackDays` | Files and folders shared externally or via anonymous links |
| `GetDLPPolicyViolations` | Get DLP Policy Violations | — | `lookbackDays` | DLP policy match events for sensitive data across O365 workloads |
| `GetMassFileOperations` | Get Mass File Operations | — | `lookbackDays`, `threshold` | Bulk file downloads or deletions per user per hour (exfiltration detection) |

### Domain 5 — Admin Audit

| Skill Name | Display Name | Required Inputs | Optional Inputs | Description |
|---|---|---|---|---|
| `GetO365AdminAuditEvents` | Get O365 Admin Audit Events | `adminUser` | `lookbackDays` | Admin actions (Set-, New-, Remove-, etc.) by a specific user |
| `GetEntraRoleChanges` | Get Entra Role Changes | — | `lookbackDays` | All Entra ID role assignment changes with actor and target details |

### Domain 6 — Orchestration Agent

| Skill Name | Display Name | Required Inputs | Optional Inputs | Description |
|---|---|---|---|---|
| `O365IncidentInvestigationAgent` | O365 Incident Investigation Agent | `indicator` | `lookbackDays` | Chains all 16 skills based on indicator type; produces structured incident summary |

---

## Agent Usage

Invoke the orchestration agent from the Security Copilot prompt bar by providing a UPN, email address, NetworkMessageId, or IP address:

```
Investigate user john.doe@contoso.com for the past 14 days
```

```
Investigate email with NetworkMessageId 1a2b3c4d-0000-0000-0000-1234567890ab
```

```
Investigate suspicious activity from sender phishing@evil.com over the last 7 days
```

The agent classifies the indicator, runs the appropriate skill chain, and returns a structured summary including severity, key findings, IOCs, and recommended next actions.

---

## KQL Table Dependencies

| Skill | Defender XDR Table(s) |
|---|---|
| `GetEmailByNetworkMessageId` | `EmailEvents`, `EmailAttachmentInfo`, `EmailUrlInfo` |
| `GetEmailsFromSuspiciousSender` | `EmailEvents` |
| `GetEmailsWithMaliciousContent` | `EmailEvents` |
| `GetEmailDeliveryTimeline` | `EmailEvents`, `EmailPostDeliveryEvents` |
| `GetRiskySignInsForUser` | `EntraIdSignInEvents` |
| `GetMFAStatusForUser` | `EntraIdSignInEvents` |
| `GetPrivilegedRoleChanges` | `IdentityDirectoryEvents` |
| `GetConditionalAccessFailures` | `EntraIdSignInEvents` |
| `GetMailboxPermissionChanges` | `CloudAppEvents` |
| `GetMailForwardingRuleChanges` | `CloudAppEvents` |
| `GetSuspiciousMailboxAccess` | `CloudAppEvents` |
| `GetExternalSharingEvents` | `CloudAppEvents` |
| `GetDLPPolicyViolations` | `CloudAppEvents` |
| `GetMassFileOperations` | `CloudAppEvents` |
| `GetO365AdminAuditEvents` | `CloudAppEvents` |
| `GetEntraRoleChanges` | `IdentityDirectoryEvents` |

---

## Known Limitations

- **`IdentityDirectoryEvents`** is populated by Microsoft Defender for Identity. For cloud-only Entra ID tenants without Defender for Identity deployed, role change events may instead appear in `CloudAppEvents` with `Application == "Microsoft Azure Active Directory"`. Update the `GetPrivilegedRoleChanges` and `GetEntraRoleChanges` templates accordingly.

- **`EntraIdSignInEvents`** replaced the deprecated `AADSignInEventsBeta` table (deprecated December 9, 2025). If running against a legacy workspace that has not migrated, replace `EntraIdSignInEvents` with `AADSignInEventsBeta` in the affected skills.

- **`CloudAppEvents` for Exchange** requires the Microsoft 365 app connector to be enabled in Defender for Cloud Apps settings. Without it, mailbox, SharePoint, and OneDrive skills will return no results.

- **DLP events in `CloudAppEvents`** depend on Microsoft Purview DLP policies being configured and the Microsoft 365 connector streaming events to Defender. Native Purview DLP alerts may also be visible in `AlertInfo`/`AlertEvidence` as an alternative data source.

---

## Customization

**Adjust the default lookback window:** All skills accept a `lookbackDays` input (default 7). Pass a higher value for broader investigations: `lookbackDays=30`.

**Adjust the mass file operation threshold:** `GetMassFileOperations` accepts a `threshold` input (default 50 operations/hour). Lower the value to increase sensitivity or raise it to reduce noise.

**Add new skills:** Extend any KQL `SkillGroup` by appending a new skill block. Follow the naming convention (no whitespace or special characters) and add the new skill name to the `ChildSkills` list in `O365IncidentInvestigationAgent`.

**Change the publisher:** Replace `YourOrganization` in the `AgentDefinitions.Publisher` field with your organization name before deploying.
