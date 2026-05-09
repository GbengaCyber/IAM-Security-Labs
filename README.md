# IAM & Cloud Security Lab — Microsoft Entra ID

> **A hands-on identity security lab built on Microsoft 365 E5 and Entra ID.**  
> Documenting real-world IAM analyst skills across authentication protocols, SSO configuration, incident response, and identity governance.

---

## About This Project

This lab was built to develop and document practical skills for **Cloud Security Engineer**, **IAM Analyst**, and **Identity Security Analyst** roles. Every exercise mirrors a real task performed by IAM professionals in enterprise environments.

**Tech stack:**
- Microsoft Entra ID (Azure AD)
- Microsoft 365 E5
- Microsoft Sentinel
- Microsoft Defender XDR
- PowerShell + Microsoft Graph API
- SailPoint IdentityNow
- CyberArk PAM

**Certification alignment:** SC-300 · AZ-500 · SC-200

---

## Lab Index

| Day | Topic | Status |
|-----|-------|--------|
| Day 01 | Active Directory, Hybrid Identity & Azure AD Connect | 🔜 |
| **Day 02** | **Authentication Protocols — SAML, OAuth 2.0, OIDC, JWT** | ✅ Complete |
| Day 03 | Entra ID Conditional Access & MFA Deep Dive | 🔜 |
| Day 04 | RBAC, ABAC & Least Privilege in Azure | 🔜 |
| Day 05 | Exchange Online, Mail Flow & Email Security | 🔜 |
| Day 06 | PowerShell & Microsoft Graph API for IAM | 🔜 |
| Day 07 | Week 1 Consolidation — User Lifecycle SOP | 🔜 |
| Day 08 | Microsoft Defender XDR | 🔜 |
| Day 09 | Microsoft Sentinel — SIEM, KQL & Identity Detections | 🔜 |
| Day 10 | Privileged Identity Management (PIM) & PAM | 🔜 |

---

## Repository Structure

```
iam-security-lab/
├── README.md
├── day-02-authentication-protocols/
│   ├── README.md                    ← Lab overview and objectives
│   ├── 01-concepts.md               ← Deep dive: SAML, OAuth, OIDC, JWT
│   ├── 02-saml-lab.md               ← SAML SSO lab walkthrough
│   ├── 03-oidc-lab.md               ← OIDC token lab walkthrough
│   ├── 04-oauth-consent-lab.md      ← OAuth consent phishing simulation
│   ├── 05-runbook-sso-break.md      ← Enterprise runbook: SSO break/fix
│   ├── 06-runbook-oauth-incident.md ← Enterprise runbook: OAuth consent incident
│   └── screenshots/                 ← Lab evidence screenshots
```

---

## Key Skills Demonstrated

- Configured SAML SSO on Microsoft Entra ID Enterprise Applications
- Decoded live SAMLResponse assertions to identify claim mismatches
- Built OIDC auth flows and decoded JWT ID tokens
- Identified OAuth consent phishing attack vectors
- Wrote enterprise incident response runbooks for SSO failures and OAuth incidents
- Mapped controls to SC-300 exam objectives and SOC 2 CC6 compliance

---

## Environment

| Component | Details |
|-----------|---------|
| Tenant type | Microsoft 365 E5 Developer Tenant |
| Identity Provider | Microsoft Entra ID |
| Lab domain | `[REDACTED].onmicrosoft.com` |
| Admin account | `admin@[REDACTED].onmicrosoft.com` |

> All tenant IDs, client IDs, object IDs, and personal identifiers have been redacted from this documentation.
