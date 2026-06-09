# Lab 0 — AD FS + Hybrid Identity
**On-Premises Active Directory → AD FS → Microsoft Entra ID (Password Hash Sync)**

---

## Objective

This lab simulates a real enterprise hybrid identity deployment, building the on-premises identity foundation that underpins all subsequent SSO labs.

By the end of this lab you will understand:

- How on-premises Active Directory integrates with Microsoft Entra ID
- What Active Directory Federation Services (AD FS) is and how it works
- The difference between AD FS Federation and Password Hash Sync (PHS)
- Why enterprises choose one authentication method over the other
- How Entra Connect synchronises identities from on-premises AD to the cloud
- What Seamless SSO is and how it enables transparent cloud authentication
- How to read and troubleshoot AD FS event logs

---

## Environment

| Component | Role | Product |
|---|---|---|
| Domain Controller | On-premises identity source | Windows Server Active Directory |
| Federation Server | Claims-based authentication | Active Directory Federation Services (AD FS) |
| Sync Engine | Hybrid identity bridge | Microsoft Entra Connect |
| Cloud Identity Provider | Cloud IdP | Microsoft Entra ID (Azure AD) |
| Internal Domain | Lab domain | `tutionbooks.com` |
| AD FS FQDN | Federation endpoint | `fs.tutionbooks.com` |
| AD FS Service Account | Dedicated service identity | `CORP\svc_adfs` |

---

## Architecture Overview

### Intended Architecture (Full AD FS Federation)

```
User
 │
 ▼
Microsoft Entra ID
 │  (authentication redirected)
 ▼
AD FS (fs.tutionbooks.com)
 │  (validates against on-prem AD)
 ▼
Domain Controller (tutionbooks.com)
 │  (issues claims token)
 ▼
Token returned → Entra → SaaS App
```

### Actual Architecture (PHS — lab pivot)

```
On-Premises Active Directory (tutionbooks.com)
           │
           ▼
Microsoft Entra Connect
(Password Hash Sync + Seamless SSO)
           │
           ▼
Microsoft Entra ID
(Cloud authentication — no AD FS redirect)
           │
           ▼
SaaS Applications (SSO Labs)
```

> See [Why We Pivoted from AD FS to PHS](#why-we-pivoted-from-ad-fs-to-phs) for explanation.

---

## Key Concepts

### Active Directory Federation Services (AD FS)
AD FS is Microsoft's on-premises claims-based authentication service. It allows organisations to extend their on-premises identity to external applications and cloud services without synchronising passwords. AD FS acts as a Security Token Service (STS) — it validates the user against on-premises AD and issues a signed claims token that trusted applications accept.

**AD FS is used when:**
- The organisation requires authentication to remain entirely on-premises
- Password hashes must never leave the corporate network (strict security policy)
- Legacy applications require WS-Federation or SAML tokens issued by an on-prem STS

### Password Hash Sync (PHS)
PHS is a hybrid identity feature of Entra Connect that synchronises a one-way hash of the user's on-premises AD password to Entra ID. Authentication happens in the cloud directly — no on-premises component is required at login time. The actual password never leaves the network, only a hash of the hash.

**PHS is used when:**
- Simplicity and resilience are priorities
- The organisation does not need authentication to remain on-premises
- No publicly routable AD FS infrastructure is available (common in lab environments)

### Seamless SSO
An Entra ID feature that silently authenticates domain-joined Windows devices using Kerberos. When a user on a domain-joined machine accesses a cloud application, Entra ID automatically authenticates them without prompting for credentials — creating a true single sign-on experience from the Windows desktop to cloud apps.

### Entra Connect
The synchronisation engine that bridges on-premises Active Directory and Microsoft Entra ID. It replicates user and group objects from on-premises AD to the cloud, and depending on the chosen authentication method, either syncs password hashes (PHS) or configures federation trust (AD FS).

### Claims and Relying Party Trusts
In AD FS, a **Relying Party Trust** defines which applications trust AD FS for authentication. **Claim rules** define what identity attributes (claims) AD FS includes in the token — such as UPN, email, group membership, or custom attributes.

---

## Authentication Method Comparison

| Feature | AD FS Federation | Password Hash Sync |
|---|---|---|
| Authentication location | On-premises (AD FS) | Cloud (Entra ID) |
| Password leaves network | No | Hash only |
| Public domain required | ✅ Yes | ❌ No |
| Infrastructure required | AD FS servers + WAP | Entra Connect only |
| Resilience | Dependent on AD FS uptime | High — cloud-native |
| Complexity | High | Low |
| Common use case | Strict compliance, legacy | Most modern enterprises |

---

## Lab Steps

### Part 1 — Install and Configure AD FS

**Step 1 — Install AD FS Role**

1. Open **Server Manager → Manage → Add Roles and Features**
2. Select **Role-based or feature-based installation**
3. Select your server
4. Under Server Roles, check **Active Directory Federation Services (AD FS)**
5. Accept defaults for Features
6. Install — reboot if prompted

<img width="811" height="391" alt="image" src="https://github.com/user-attachments/assets/cf8b7e77-7244-474f-bd52-675866db914e" />



**Step 2 — Create AD FS Service Account**

1. Open **Active Directory Users and Computers (ADUC)**
2. Navigate to your Users OU (or create a dedicated Service Accounts OU)
3. Right-click → **New → User**

| Field | Value |
|---|---|
| Name | `svc_adfs` |
| Logon name | `svc_adfs` |
| Password | Set a strong password |
| Password never expires | ✅ |

<img width="633" height="231" alt="image" src="https://github.com/user-attachments/assets/47936aff-c6e7-450c-a3f7-a338dcac801e" />

---

**Step 3 — Create SSL Certificate**

For lab purposes, a self-signed certificate was used:

1. Open **Server Manager → Tools → IIS Manager**
2. Select your server → double-click **Server Certificates**
3. Right-click → **Create Self-Signed Certificate**
4. Name: `fs.tutionbooks.com` → OK

> In production, this would be a certificate issued by a public CA (DigiCert, Sectigo etc.) bound to a publicly resolvable FQDN.

<img width="755" height="279" alt="image" src="https://github.com/user-attachments/assets/2e18cead-5aa0-40bb-9c49-132820d22d38" />

---

**Step 4 — Configure AD FS Farm**

1. In Server Manager, click the **Notifications flag → Configure the federation service on this server**
2. Choose **Create the first federation server in a federation server farm**
3. Select the SSL certificate created in Step 3
4. Set Federation Service Name: `fs.tutionbooks.com`
5. Set Federation Service Display Name: `TutionBooks AD FS`
6. Service Account: enter credentials for `CORP\svc_adfs`
7. Review → Configure

<img width="744" height="455" alt="image" src="https://github.com/user-attachments/assets/c35bac60-0684-426c-9a8e-42dac4213afb" />

---

**Step 5 — Verify AD FS**

Inside the VM, navigate to:
```
https://fs.tutionbooks.com/adfs/ls/
```

> If the FQDN doesn't resolve, try `https://localhost/adfs/ls/` as a fallback inside the VM.

<img width="797" height="390" alt="image" src="https://github.com/user-attachments/assets/78305acd-4146-45f9-9f89-73c355518322" />

---

**Step 6 — Make AD FS Accessible from Host Machine**

To access `fs.tutionbooks.com` from your host machine, add a hosts file entry:

1. Open Notepad as **Administrator** on your host machine
2. Open `C:\Windows\System32\drivers\etc\hosts`
3. Add:
```
<VM-IP>    fs.tutionbooks.com
```
4. Save and test in browser: `https://fs.tutionbooks.com/adfs/ls/`

> In a production environment this would be handled by an internal DNS A record pointing to the AD FS server or Web Application Proxy (WAP).

<img width="758" height="562" alt="image" src="https://github.com/user-attachments/assets/fe60b0fa-b0d8-405f-8047-3720eabfeb47" />

---

### Part 2 — Test AD FS with a Relying Party Trust

To verify AD FS is issuing tokens correctly without a real application, a dummy Relying Party Trust was configured.

**Step 7 — Create Test Relying Party Trust**

1. Open **AD FS Management → Relying Party Trusts → Add Relying Party Trust**
2. Choose **Enter data about the relying party manually**
3. Choose **AD FS profile**
4. Name: `LabTestApp`
5. Set Identifier: `https://labtestapp.local/`
6. Accept defaults → Finish

<img width="855" height="288" alt="image" src="https://github.com/user-attachments/assets/ddb889c1-0f86-4422-9af2-7da44b6b0423" />

**Step 8 — Configure Claim Rules**

1. Right-click `LabTestApp` → **Edit Claim Rules**
2. Add Rule → **Send LDAP Attributes as Claims**
3. Map: **User-Principal-Name → Name ID**

<img width="692" height="448" alt="image" src="https://github.com/user-attachments/assets/ed27bebb-91f3-463d-a2f3-942bdaa44b92" />

**Step 9 — Test the Authentication Flow**

Navigate to:
```
https://fs.tutionbooks.com/adfs/ls/?wa=wsignin1.0&wtrealm=https://labtestapp.local/
```

Enter domain user credentials. You will receive an error after authentication (because `labtestapp.local` doesn't exist) — this is expected. The important thing is that AD FS redirected you, authenticated you, and attempted to issue a token, proving the federation service is operational.

**Verify via PowerShell:**
```powershell
Get-AdfsProperties
Get-AdfsEndpoint | Where-Object {$_.Enabled -eq $true}
```

---

### Part 3 — Prepare for Entra Connect

**Step 10 — Verify Connectivity from Entra Connect Server**

Run the following from the Server 2019 machine designated for Entra Connect:

```powershell
# Verify AD connectivity
ping <DCName>
nslookup tutionbooks.com

# Verify Entra connectivity
Test-NetConnection login.microsoftonline.com -Port 443
```

All must succeed before proceeding.

**Step 11 — Verify Time Sync**

```powershell
w32tm /query /status
```

Time skew between the server and Entra ID must be less than 5 minutes. Kerberos and token validation are sensitive to clock drift.

---

### Part 4 — Install Microsoft Entra Connect

**Step 12 — Install Entra Connect**

Download and run the [Microsoft Entra Connect installer](https://www.microsoft.com/en-us/download/details.aspx?id=47594).

Choose **Customize** — not Express — to control the authentication method selection.

📸 *Screenshot: Entra Connect installer — Customize option selected*

**Step 13 — Select Sign-In Method**

> ⚠️ We initially selected **Federation with AD FS** here. See [Why We Pivoted from AD FS to PHS](#why-we-pivoted-from-ad-fs-to-phs) for why this was changed to Password Hash Sync.

Select: **Password Hash Synchronization**

📸 *Screenshot: Entra Connect — Password Hash Sync selected as sign-in method*

**Step 14 — Connect to Entra Tenant**

Sign in with your **Global Administrator** account. This registers your on-premises environment with your Entra tenant.

📸 *Screenshot: Entra Connect — successfully connected to Entra tenant*

**Step 15 — Connect to On-Premises AD**

Enter **Domain Admin credentials** for `tutionbooks.com`. This allows Entra Connect to read user objects and configure synchronisation.

📸 *Screenshot: Entra Connect — on-premises AD connected*

**Step 16 — Configure Sync Scope**

For lab purposes, all Clous Users OU were synced. In production, it is recommended to create a dedicated OU and sync only that scope to keep results clean and controlled.

<img width="831" height="592" alt="image" src="https://github.com/user-attachments/assets/e8a5f478-db33-4c67-b566-33ed731a20d5" />

**Step 17 — Complete Installation and Force Initial Sync**

After the wizard completes, force an initial sync:

```powershell
Start-ADSyncSyncCycle -PolicyType Initial
```

<img width="827" height="628" alt="image" src="https://github.com/user-attachments/assets/6ac184c8-6a89-4244-90b7-783ad8dc892b" />

---

### Part 5 — Verify Hybrid Identity

**Step 18 — Verify Users in Entra ID**

Navigate to [Microsoft Entra Admin Center](https://entra.microsoft.com) → Users.

Confirm synced users show:
- **On-premises sync enabled: Yes**
- **Source: Windows Server AD**

<img width="839" height="577" alt="image" src="https://github.com/user-attachments/assets/363ccc1c-e3dc-4544-aa53-3934cd2934ca" 

<img width="761" height="539" alt="image" src="https://github.com/user-attachments/assets/68ee5e42-89b6-41c0-ac21-f00ca8a885ce" />


**Step 19 — Test Cloud Login with Synced On-Premises User**

Attempt to sign in to the [Azure Portal](https://portal.azure.com) using a synced on-premises user account.

Successful login confirms:
- Password Hash Sync is working
- On-premises credentials authenticate against Entra ID
- Hybrid identity is operational

<img width="669" height="521" alt="image" src="https://github.com/user-attachments/assets/2a505d7d-7584-4b77-b7c3-2c813c4952f8" />

---

## Why We Pivoted from AD FS to PHS

### What Was Attempted

The original lab objective was to configure full **AD FS federation** with Microsoft Entra ID, creating an architecture where:

1. Entra Connect would configure a federation trust between Entra ID and AD FS
2. All cloud authentication requests would redirect to the on-premises AD FS server
3. AD FS would validate credentials against the domain controller and issue a claims token
4. The token would be returned to Entra ID and access granted

Entra Connect was initially installed with **Federation with AD FS** selected as the sign-in method.

### Why It Didn't Work

Full AD FS federation with Entra ID has a hard requirement: **a publicly verified domain**.

Entra ID requires that the domain being federated (e.g. `tutionbooks.com`) is verified in the Entra tenant — meaning the organisation must own the domain publicly and prove ownership via a DNS TXT record. It also requires that the AD FS federation endpoint (`fs.tutionbooks.com`) is publicly resolvable so Entra ID can reach it for metadata exchange.

Since `tutionbooks.com` is an internal lab domain with no public DNS registration or ownership, neither requirement could be met:

| Requirement | Status |
|---|---|
| Publicly verified domain in Entra | ❌ Not possible — internal domain only |
| Publicly resolvable AD FS endpoint | ❌ Not possible — no public DNS |
| SSL certificate from public CA | ❌ Not possible — self-signed only |

### The Pivot

Rather than abandon the hybrid identity objective, the approach was changed to **Password Hash Sync with Seamless SSO** — which has no public domain requirement. PHS synchronises password hashes directly to Entra ID and performs cloud-native authentication, while still providing the same user experience for cloud application SSO.

The AD FS configuration completed in Parts 1 and 2 of this lab remains valid and demonstrates understanding of claims-based authentication, relying party trusts, and federation architecture — even though it was not connected to Entra ID.

### Real-World Relevance

This is a genuinely common scenario in enterprise environments. Organisations that originally deployed AD FS for Office 365 federation are increasingly migrating to PHS or Pass-Through Authentication (PTA) because:

- AD FS requires dedicated infrastructure and high availability configuration
- AD FS is a single point of failure if not properly redundant
- PHS and PTA achieve the same SSO experience with significantly lower operational overhead
- Microsoft's official recommendation for most organisations is now PHS over AD FS

---

## Troubleshooting Log

### Issue 1 — AD FS Login Page Not Loading (Event ID 364 / MSIS7065)

**Symptom:** Navigating to `https://fs.tutionbooks.com/adfs/ls/` returned an error. Fallback to `https://localhost/adfs/ls/` also failed.

**Diagnosis:** Checked **Event Viewer → Applications and Services Logs → AD FS → Admin**. Found:

```
Event ID: 364
Error: MSIS7065 — There are no registered protocol handlers on path /adfs/ls/
```

**Cause:** The AD FS installation was corrupted — the Windows Internal Database (WID) used by AD FS had a partial or failed write during installation, leaving the protocol handler registration incomplete.

**Resolution:** Full cleanup and reinstall:

1. Deleted AD FS Windows Internal Database files:
```
C:\Windows\WID\Data\*ADFS*
```
2. Deleted the AD FS registry key
3. Deleted the AD FS AD object from Active Directory
4. Reinstalled both the **AD FS role** and **Windows Internal Database feature** from Server Manager

After reinstall, Event ID 364 was gone and the AD FS login page loaded correctly.

**Lesson:** If AD FS fails post-install with MSIS7065, a clean reinstall including WID data and registry cleanup is the most reliable resolution. Event Viewer is the primary diagnostic tool for AD FS issues.

---

### Issue 2 — AD FS Federation with Entra ID Failed (No Public Domain)

**Symptom:** Entra Connect wizard failed to configure federation trust between AD FS and Entra ID.

**Cause:** Full AD FS federation requires a publicly verified domain and a publicly resolvable federation endpoint. The lab domain `tutionbooks.com` is internal only — no public DNS registration or Entra domain verification was possible.

**Resolution:** Pivoted to Password Hash Sync. See [Why We Pivoted from AD FS to PHS](#why-we-pivoted-from-ad-fs-to-phs).

---

## Summary

This lab successfully delivered:

- ✅ On-premises Active Directory domain (`tutionbooks.com`)
- ✅ AD FS installed, configured, and verified with a test Relying Party Trust
- ✅ Claims-based authentication tested end-to-end via browser
- ✅ Entra Connect installed with Password Hash Sync
- ✅ Users synchronised from on-premises AD to Microsoft Entra ID
- ✅ Seamless SSO enabled for domain-joined devices
- ✅ Cloud login verified with synced on-premises user account
- ⚠️ Full AD FS → Entra federation not completed — internal domain limitation

---

## Key Learnings

- AD FS is a claims-based Security Token Service — it validates credentials against on-premises AD and issues signed tokens to trusting applications
- Full AD FS federation with Entra ID requires a publicly verified domain — a hard requirement that many lab environments cannot meet
- PHS synchronises a hash of the password hash to Entra ID — the plaintext password never leaves the network
- Seamless SSO uses Kerberos to silently authenticate domain-joined devices to cloud applications
- Entra Connect is the bridge between on-premises AD and Entra ID — it handles both identity sync and authentication method configuration
- Event Viewer (AD FS Admin log) is the primary diagnostic tool for AD FS issues — MSIS error codes are specific and actionable
- Many enterprises are migrating away from AD FS to PHS or PTA due to operational complexity and single point of failure risk

---

## Interview Questions

**What is AD FS and what problem does it solve?**
AD FS is Microsoft's on-premises federation service. It allows organisations to extend on-premises Active Directory authentication to external applications and cloud services using claims-based tokens, without exposing AD directly or synchronising passwords.

**What is the difference between AD FS federation and Password Hash Sync?**
With AD FS federation, authentication requests are redirected to the on-premises AD FS server which validates against AD and issues tokens — passwords never leave the network. With PHS, a hash of the password hash is synchronised to Entra ID and authentication happens in the cloud. PHS is simpler and more resilient but requires password hashes to reach the cloud.

**Why would an organisation choose PHS over AD FS?**
PHS has lower operational overhead, no dedicated infrastructure requirement, higher resilience (no on-prem dependency at login time), and is Microsoft's recommended approach for most organisations. AD FS is chosen when strict compliance requirements mandate that authentication remains entirely on-premises.

**What is Seamless SSO?**
A feature of Entra Connect that enables domain-joined Windows devices to authenticate silently to Entra ID using Kerberos, without prompting for credentials. It provides a desktop-to-cloud SSO experience without requiring AD FS.

**What is a Relying Party Trust in AD FS?**
A configuration object in AD FS that defines a trusted application (the relying party). It specifies which application can receive tokens from AD FS, what claims to include, and how to sign and encrypt the token.

**What are claims in the context of AD FS?**
Claims are identity attributes about the user that AD FS includes in the issued token — such as UPN, email, group membership, or department. The receiving application uses claims for identity mapping, authorisation, and role assignment.

**What does Event ID 364 / MSIS7065 indicate in AD FS?**
It indicates that no protocol handlers are registered for the AD FS endpoint path — typically caused by a corrupted or incomplete AD FS installation. The resolution is to clean up the WID database, registry key, and AD object, then reinstall the AD FS role and Windows Internal Database feature.

---

## References

- [Microsoft Entra Connect — Choose Authentication Method](https://learn.microsoft.com/en-us/entra/identity/hybrid/connect/choose-ad-authn)
- [AD FS Overview — Microsoft Docs](https://learn.microsoft.com/en-us/windows-server/identity/ad-fs/ad-fs-overview)
- [Password Hash Sync — Microsoft Docs](https://learn.microsoft.com/en-us/entra/identity/hybrid/connect/whatis-phs)
- [Seamless SSO — Microsoft Docs](https://learn.microsoft.com/en-us/entra/identity/hybrid/connect/how-to-connect-sso)
- [Migrate from AD FS to PHS — Microsoft Docs](https://learn.microsoft.com/en-us/entra/identity/hybrid/connect/migrate-from-federation-to-cloud-authentication)
