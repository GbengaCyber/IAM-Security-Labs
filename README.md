# Scenario 1: Onboarding a New App to SSO Using SAML

## Business Context

The company signed a contract with BambooHR for 200 employee licences. The vendor confirmed they support SAML SSO with Microsoft Entra ID. The task was to configure SSO so employees can log in with their company credentials instead of a separate BambooHR password. This is one of the most common tasks an IAM analyst handles when a new SaaS application is onboarded.

## What I Did

**Created a non-gallery Enterprise Application in Entra ID**

SAML SSO is configured under Enterprise Applications, not App Registrations. App Registrations is for developers building OIDC apps. Enterprise Applications is where the IAM team manages SSO for third party tools the company uses. I created a new non-gallery application to simulate what you would do with any vendor that is not in the Microsoft gallery.

**Configured Basic SAML Settings**

The two critical fields in any SAML integration are the Entity ID and the ACS URL. These values come from the vendor's SSO documentation. The Entity ID is the unique name that identifies the service provider to Entra ID. The ACS URL is where Entra ID sends the SAMLResponse after authenticating the user. A single character mismatch in either field breaks SSO for every user silently.

| Field | Value Used in Lab |
|-------|-------------------|
| Identifier (Entity ID) | https://jwt.ms |
| Reply URL (ACS URL) | https://jwt.ms |

In a real BambooHR integration these would be BambooHR's actual SP Entity ID and ACS endpoint from their admin documentation.

**Downloaded the Federation Metadata XML**

After configuring Entra ID, I downloaded the Federation Metadata XML file. In production this file gets sent to the vendor's IT team. They upload it on their side so BambooHR can verify the digital signature on every SAMLResponse that Entra ID sends. Without this file the vendor cannot validate who signed the assertion and SSO will not work.

**Assigned Users to the Application**

By default nobody has access to a new Enterprise Application. In production you assign a security group such as BambooHR-Users. Anyone added to that group automatically gets SSO access. Anyone removed from the group loses access immediately. This is how IAM teams manage application access at scale without touching individual accounts.

**Tested the SSO Flow and Read the Live Assertion**

I used rcFederation Tracer to capture the SAML POST request and read the decoded assertion. The tool showed the complete XML that Entra ID sent to the service provider, including every claim.

Key fields from the live assertion:

| Field | Value | Why It Matters |
|-------|-------|----------------|
| Issuer | https://sts.windows.net/[REDACTED]/ | Confirms Entra ID signed the assertion |
| NameID | [REDACTED]@[REDACTED].onmicrosoft.com | The user identifier the app receives |
| NotOnOrAfter | 2026-05-10T00:43:42Z | Assertion expires after this time |
| StatusCode | Success | Login completed without errors |
| authnmethodsreferences | password + multipleauthn | User authenticated with password and MFA |
| givenname | [REDACTED] | First name claim sent to BambooHR |
| emailaddress | [REDACTED] | Email claim sent to BambooHR |
| Signature Algorithm | rsa-sha256 | How Entra ID signed the assertion |

## What This Looks Like in Production

When a vendor says SSO is broken, this is the process. You ask the user to reproduce the error while rcFederation Tracer is running. You capture the SAMLResponse and check four things: does the Audience match the Entity ID the vendor has configured, has the NotOnOrAfter timestamp expired, is the NameID in the format the app expects, and is the signature valid against the current signing certificate. Most SSO breaks in production come down to one of these four issues.

The most common production failure is certificate rotation. Entra ID rotates the SAML signing certificate periodically. If the vendor is not updated with the new Federation Metadata XML, their system tries to validate the signature using an old certificate and every user gets an error at the same time.

## Tools Used

| Tool | Purpose |
|------|---------|
| Microsoft Entra ID | Enterprise Application creation and SAML configuration |
| rcFederation Tracer | Captured and decoded the live SAML assertion |
| jwt.ms | Used as the test ACS URL to receive the SAMLResponse |

## Screenshots

---
<img width="800"  alt="image" src="https://github.com/user-attachments/assets/c00be3af-124b-4173-9df5-31824874fd9d" />

---

<img width="800"  alt="image" src="https://github.com/user-attachments/assets/aab3056d-dccb-4ebb-a952-fb66b0ad68e1" />


---

<img width="800"  alt="image" src="https://github.com/user-attachments/assets/f488f7b6-5e42-4a04-92a9-6435b260bd52" />

---

<img width="800"  alt="image" src="https://github.com/user-attachments/assets/f3b3f837-4a43-48ea-be93-115bb45ad70f" />


---

<img width="2446" height="1114" alt="image" src="https://github.com/user-attachments/assets/4864dae3-f555-410a-b83c-f7f5e26fadea" />


---

