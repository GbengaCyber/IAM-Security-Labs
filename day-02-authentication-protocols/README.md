# Day 02 — Lab 1: SAML SSO Configuration

## Objective

Configure SAML-based Single Sign-On on Microsoft Entra ID, simulate an SSO test login, and understand how to diagnose SAML failures — exactly as an IAM analyst would when investigating a broken SSO ticket.

## Enterprise Context

**Ticket INC-4421:** "SSO broken for Salesforce — 200 sales reps can't log in"

This lab simulates the investigation and configuration work an IAM analyst performs when:
- Onboarding a new SaaS application to Entra ID SSO
- Diagnosing a broken SAML SSO integration
- Reading a raw SAMLResponse to identify the exact failure point

---

## Prerequisites

- Microsoft Entra ID tenant with Global Administrator access
- Microsoft 365 E5 license (or Entra ID P1/P2)
- Chrome or Edge browser (required for DevTools network capture)

---

## Key Concept Before Starting

**Why Enterprise Applications — not App Registrations?**

Entra ID has two separate areas for apps:

| Area | Purpose | Protocols |
|------|---------|-----------|
| **App Registrations** | Developer-facing — register apps you are building | OIDC, OAuth only |
| **Enterprise Applications** | IT/IAM-facing — configure SSO for apps your org uses | SAML, OIDC, Password SSO |

SAML SSO configuration lives exclusively under **Enterprise Applications**. This is the most common confusion point for new IAM engineers.

---

## Lab Steps

### Step 1 — Create a Non-Gallery Enterprise Application

**Portal path:** `entra.microsoft.com → Enterprise Applications → New application`

1. Navigate to **entra.microsoft.com** and sign in with your admin account
2. In the left sidebar click **Enterprise Applications**
3. Click **+ New application**
4. Click **"Create your own application"** at the top of the gallery page
5. Configure the panel:
   - Name: `Salesforce-SAML-Lab`
   - Select: **"Integrate any other application you don't find in the gallery (Non-gallery)"**
6. Click **Create**

> **Why Non-gallery?** Gallery apps (real Salesforce, Workday etc.) have pre-configured SAML templates. Non-gallery gives you a blank SAML configuration so you can see and control every field — this is how you learn what each field does.

---

### Step 2 — Configure SAML SSO

**Portal path:** `Salesforce-SAML-Lab → Single sign-on → SAML`

1. In the left menu of your new app, click **Single sign-on**
2. Click the **SAML** tile
3. In Section 1 "Basic SAML Configuration", click the **Edit (pencil) icon**
4. Fill in both fields:

| Field | Value | What This Is |
|-------|-------|-------------|
| Identifier (Entity ID) | `https://jwt.ms` | Unique ID that Entra ID uses to identify the SP |
| Reply URL (ACS URL) | `https://jwt.ms` | Where Entra ID sends the SAMLResponse after authentication |

5. Click **Save** and close the panel

> **In production:** These values come from the SaaS vendor (Salesforce, ServiceNow etc.) and must match exactly — a single character difference breaks SSO. The Entity ID and ACS URL are found in the vendor's SSO configuration documentation.

> **Why jwt.ms?** jwt.ms is Microsoft's token inspection tool. We use it as both the Entity ID and ACS URL so we can receive and inspect the SAMLResponse in the browser. In a real integration, these would be Salesforce's actual URLs.

---

### Step 3 — Assign a User to the Application

**Portal path:** `Salesforce-SAML-Lab → Users and groups → Add user/group`

1. In the left menu click **Users and groups**
2. Click **+ Add user/group**
3. Under Users click **None Selected**
4. Search for your admin account → select it → click **Select**
5. Click **Assign**

> **Why this step matters:** By default, no users have access to a new Enterprise Application. In production, you assign a security group (e.g. "Salesforce-Users") rather than individual users — everyone in the group gets SSO access automatically. This is how you control which employees can access which apps.

---

### Step 4 — Test the SAML SSO Flow

**Portal path:** `Salesforce-SAML-Lab → Single sign-on → Test`

1. Open **Chrome** (required for this step)
2. Press **F12** → click **Network** tab → check **"Preserve log"**
3. Go back to the Single sign-on page → scroll down to Section 5
4. Click **"Test sign in"**
5. Authenticate when prompted
6. A new tab opens with jwt.ms

**To capture the SAMLResponse:**

1. Once redirected to jwt.ms, press **F12** on that tab
2. Click **Network** tab — look for the **jwt.ms** document request at the top
3. Click it → click **Payload** tab
4. Find the `SAMLResponse` field — copy the Base64 encoded value

**To decode the SAMLResponse:**

1. Go to **samltool.com**
2. Click **SAML Decoder**
3. Paste the SAMLResponse value → click Decode

---

### Step 5 — Read the Decoded SAML Assertion

After decoding, locate and understand these fields in the XML:

```xml
<!-- Who created this assertion -->
<saml:Issuer>
  https://sts.windows.net/[TENANT-ID-REDACTED]/
</saml:Issuer>

<!-- The user being authenticated -->
<saml:NameID Format="urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress">
  admin@[TENANT-REDACTED].onmicrosoft.com
</saml:NameID>

<!-- When this assertion expires — 5 minute window -->
<saml:Conditions NotOnOrAfter="[TIMESTAMP-REDACTED]">

  <!-- Who this assertion is FOR — must match SP's Entity ID -->
  <saml:AudienceRestriction>
    <saml:Audience>https://jwt.ms</saml:Audience>
  </saml:AudienceRestriction>

</saml:Conditions>

<!-- Additional user attributes sent to the SP -->
<saml:AttributeStatement>
  <saml:Attribute Name="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress">
    <saml:AttributeValue>admin@[TENANT-REDACTED].onmicrosoft.com</saml:AttributeValue>
  </saml:Attribute>
</saml:AttributeStatement>
```

**What each field means for an IAM analyst:**

| Field | What to Check | Common Issue |
|-------|--------------|--------------|
| `Issuer` | Should be your Entra ID tenant URL | Wrong tenant configured |
| `NameID` | The user identifier the SP will receive | Format mismatch breaks user matching |
| `NotOnOrAfter` | Assertion expiry (usually 5-10 min) | Expired = broken SSO, check clock skew |
| `Audience` | Must exactly match the SP's Entity ID | Most common SSO break cause |
| `AttributeStatement` | Claims being sent to the SP | Missing claim = app can't map the user |

---

## What This Means in a Real SSO Incident

When the INC-4421 ticket says "Salesforce SSO is broken":

1. **First action:** Ask the user to reproduce the error and capture the SAMLResponse
2. **Decode at samltool.com** and check:
   - Is `NotOnOrAfter` in the past? → Clock skew or certificate rotation issue
   - Does `Audience` match Salesforce's Entity ID exactly? → Configuration mismatch
   - Is the `Signature` valid? → Certificate may have been rotated without updating Salesforce
3. **If certificate rotated:** Download new Federation Metadata XML from Entra ID → upload in Salesforce SSO settings → test

> **Most common production root cause:** Entra ID automatically rotates SAML signing certificates periodically. If the SP (Salesforce) is not updated with the new certificate, signature verification fails and SSO breaks for every user simultaneously. This is why enterprises configure automatic metadata refresh on the SP side.

---

## Lab Evidence

- ✅ Created non-gallery Enterprise Application in Entra ID
- ✅ Configured SAML Basic Configuration (Entity ID + ACS URL)
- ✅ Assigned admin user to the application
- ✅ Triggered a test SAML SSO login flow
- ✅ Decoded SAMLResponse and identified key assertion fields

---

## Interview Answer — Built From This Lab

**Q: How do you diagnose a broken SAML SSO integration?**

> "When SSO breaks, the error message users see is always generic and unhelpful. The real diagnosis comes from decoding the raw SAMLResponse. I intercept the Base64-encoded SAMLResponse in browser DevTools from the POST request to the ACS URL, then decode it at samltool.com. From the decoded XML I check four things: is the Audience claim an exact match to the SP's configured Entity ID, has the assertion expired based on NotOnOrAfter, is the NameID in the format the SP expects, and is the signature valid against the current Entra ID signing certificate. In my lab I configured a SAML app end-to-end and decoded a live assertion — the most common production issue I've seen documented is certificate rotation causing signature verification failures across all users simultaneously."

---

## SC-300 Exam Mapping

- Objective: Configure and manage enterprise application SSO
- Objective: Troubleshoot application sign-in issues
- Objective: Implement SAML-based SSO for applications
