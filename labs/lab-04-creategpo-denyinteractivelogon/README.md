# Lab 4 — Group Policy Object (GPO): Deny Interactive Logon

**Active Directory Group Policy — Restricting Interactive Logon for Service Accounts**

---

## Objective

This lab demonstrates how to create and apply a Group Policy Object (GPO) in Active Directory that denies interactive logon for a specific AD security group. This is a core identity and access control concept used in enterprise environments to restrict service accounts from being used for interactive sessions.

By the end of this lab you will understand:

- What a GPO is and how it is structured
- The difference between user-side and computer-side policies
- How User Rights Assignments work in Windows
- The difference between authentication and authorisation in the context of logon
- Why deny policies override allow policies
- How this pattern is used in enterprise PAM programs to control service account access

---

## Environment

| Component | Role |
|---|---|
| Active Directory Domain | Identity and policy source |
| Group Policy Management Console (GPMC) | GPO creation and linking tool |
| AD Security Group | `SG_Deny_Interactive_Logon` |
| Policy Applied To | Computers/Servers OU |
| Policy Type | Computer Configuration — User Rights Assignment |

---

## Key Concepts

### What is a GPO?
A Group Policy Object is a collection of settings that control the working environment of user accounts and computer accounts in Active Directory. GPOs are linked to AD containers (sites, domains, or OUs) and applied automatically to objects within those containers.

### Two Types of GPO Policy
GPOs contain two distinct policy sections:

| Policy Type | Applied To | Regardless Of |
|---|---|---|
| User Configuration | The user | Which computer they log on to |
| Computer Configuration | The computer | Who logs on to it |

In this lab, the **Deny Interactive Logon** policy is a **Computer Configuration** policy — it is applied to the server, not the user. This means any computer in the linked OU enforces the restriction regardless of which user attempts to log in.

### What is a User Rights Assignment?
User Rights Assignments are a subset of Computer Configuration policies that define what actions users or groups are permitted to perform on a computer — such as logging on locally, accessing the computer from the network, or shutting it down. The **Deny log on locally** and **Deny log on through Remote Desktop Services** rights are the specific policies used in this lab.

### Authentication vs Authorisation in Logon
This is one of the most important distinctions in this lab:

| Stage | Question | Handled By |
|---|---|---|
| Authentication | Who are you? | Domain Controller validates credentials |
| Authorisation | Are you allowed? | Server enforces GPO User Rights Assignment |

> ⚠️ **Deny logon policies do NOT stop authentication.** The user's credentials are validated successfully by AD before the deny policy is evaluated. The logon is blocked at the authorisation stage — after authentication has already succeeded.

---

## Architecture — How the Deny Policy Works

```
1. User attempts interactive logon (RDP, console, RunAs)
              │
              ▼
2. Server contacts Domain Controller
   AD validates: username, password, account status
   ✅ Authentication succeeds (credentials are valid)
              │
              ▼
3. DC returns user SID + all group SIDs
   (including SG_Deny_Interactive_Logon membership)
              │
              ▼
4. Server evaluates GPO (already pulled from SYSVOL)
   Computer-side policy: "Deny log on locally / via RDP
   if user is member of SG_Deny_Interactive_Logon"
              │
              ▼
5. Authorisation check
   "Does this user or any of their groups match the deny list?"
              │
        ┌─────┴─────┐
       YES           NO
        │             │
        ▼             ▼
   ❌ Logon        ✅ Logon
   BLOCKED         ALLOWED
   Event ID 4625
```

**What the blocked account can still do:**

Even though interactive logon is denied, the account remains fully functional for:
- ✅ Running Windows Services
- ✅ Executing Scheduled Tasks
- ✅ CyberArk password rotation and vaulting
- ✅ Application-to-application authentication
- ✅ Network logon (if not separately restricted)

This is exactly the intended behaviour for service accounts in an enterprise PAM environment.

---

## Lab Steps

### Step 1 — Open Group Policy Management Console

On your management server, open the **Group Policy Management Console (GPMC)**:

```
Start → Windows Administrative Tools → Group Policy Management
```

Expand your forest and domain until you can see **Group Policy Objects**.

<img width="764" height="540" alt="image" src="https://github.com/user-attachments/assets/47ff2bd7-a9bf-41d6-a0b7-c200eb2a4611" />

---

### Step 2 — Create a New GPO

Right-click **Group Policy Objects → New**.

Name the GPO:
```
Deny Interactive Logon - Service Accounts
```

Click **OK**.

<img width="562" height="263" alt="image" src="https://github.com/user-attachments/assets/cf04606d-7642-4ce3-bb34-d7c00ce9c15f" />


---

### Step 3 — Edit the GPO

Right-click the new GPO → **Edit**.

The Group Policy Management Editor opens. Navigate to:

```
Computer Configuration
  → Policies
    → Windows Settings
      → Security Settings
        → Local Policies
          → User Rights Assignment
```

<img width="834" height="592" alt="image" src="https://github.com/user-attachments/assets/b7ae0092-3237-414a-beb5-f6d1ec7906a6" />

---

### Step 4 — Configure Deny Log On Locally

In the right pane, double-click **Deny log on locally**.

1. Check **Define these policy settings**
2. Click **Add User or Group**
3. Add your AD security group: `SG_Deny_Interactive_Logon`
4. Click **OK**

<img width="676" height="406" alt="image" src="https://github.com/user-attachments/assets/6365f9f5-92df-4ebc-b9ab-38bcfd633142" />

<img width="729" height="490" alt="image" src="https://github.com/user-attachments/assets/d709c34b-23c2-4be2-9d25-7f1d277ab8f3" />

---

### Step 5 — Configure Deny Log On Through Remote Desktop Services

In the same User Rights Assignment pane, double-click **Deny log on through Remote Desktop Services**.

1. Check **Define these policy settings**
2. Click **Add User or Group**
3. Add: `SG_Deny_Interactive_Logon`
4. Click **OK**

📸 *Screenshot: Deny log on through Remote Desktop Services — SG_Deny_Interactive_Logon added*

---

### Step 6 — Create the AD Security Group

If not already created, open **Active Directory Users and Computers (ADUC)** and create the security group:

| Field | Value |
|---|---|
| Group Name | `SG_Deny_Interactive_Logon` |
| Group Scope | Global |
| Group Type | Security |

Add the service accounts or users you want to restrict as members of this group.

📸 *Screenshot: SG_Deny_Interactive_Logon security group created in ADUC*

📸 *Screenshot: Members added to SG_Deny_Interactive_Logon*

---

### Step 7 — Link the GPO to the Servers OU

Back in the **Group Policy Management Console**:

1. Navigate to the **Computers** or **Servers** OU where your target machines live
2. Right-click the OU → **Link an Existing GPO**
3. Select `Deny Interactive Logon - Service Accounts`
4. Click **OK**

> ⚠️ **Critical:** Link the GPO to the **Computers/Servers OU**, not the Users OU. This is a Computer Configuration policy — it must be applied to the computer objects that will enforce the restriction.

<img width="750" height="506" alt="image" src="https://github.com/user-attachments/assets/e7226859-34c0-4692-bf90-19d70d72387a" />

---

### Step 8 — Force Group Policy Update (Optional)

To apply the policy immediately without waiting for the default refresh interval, run on the target server:

```powershell
gpupdate /force
```

Or remotely from the management server:

```powershell
Invoke-GPUpdate -Computer <ServerName> -Force
```

📸 *Screenshot: gpupdate /force output on target server*

---

### Step 9 — Test the Policy

Attempt to log in interactively to a server in the linked OU using an account that is a member of `SG_Deny_Interactive_Logon`:

- Via RDP
- Via console logon

**Expected result:**

```
The sign-in method you're trying to use isn't allowed.
```

📸 *Screenshot: Logon denied — error message displayed*

**Verify in Event Viewer:**

On the target server, open **Event Viewer → Windows Logs → Security** and look for:

| Event ID | Description |
|---|---|
| 4625 | An account failed to log on |

The failure reason will reference **User Rights** — confirming the deny policy was enforced at authorisation, not authentication.

📸 *Screenshot: Event ID 4625 in Security Event Log — logon failure due to User Rights*

---

## Common Mistakes

| Mistake | Why It's Wrong |
|---|---|
| Linking the GPO to a User OU | Computer Configuration policies must be linked to Computer/Server OUs to take effect |
| Thinking group membership alone blocks logon | Group membership must be referenced in a GPO User Rights Assignment — AD groups don't restrict logon by themselves |
| Forgetting deny overrides allow | If a user is in both an allow group and a deny group, deny always wins — no exceptions |
| Assuming service accounts shouldn't authenticate at all | Service accounts need to authenticate for services, scheduled tasks, and PAM tools — only interactive logon should be blocked |

---

## Real-World PAM Relevance

This exact pattern is standard practice in enterprise Privileged Access Management programs. When onboarding service accounts into a PAM vault (such as CyberArk), one of the first hardening steps is to ensure those accounts cannot be used for interactive logon — only for their intended automated purpose.

The GPO approach provides this control at the infrastructure level, independently of the PAM tool. Even if a service account password were compromised, the attacker could not use it to interactively log on to any server in the restricted OU.

This is also directly relevant to compliance frameworks such as **APRA CPS 234** and **ISO 27001**, which require demonstrable controls over privileged account usage and access restriction.

---

## Key Learnings

- GPOs are applied at the OU level — the type of OU (Users vs Computers) determines which policy section takes effect
- Deny Interactive Logon is a Computer Configuration policy enforced at the server, not the user
- Authentication (credential validation) and authorisation (access rights evaluation) are separate stages — deny policies operate at authorisation
- Deny always overrides Allow — there are no exceptions to this rule in Windows User Rights Assignments
- Service accounts can and should authenticate for their intended purpose — only interactive logon needs to be restricted
- Event ID 4625 is the primary security log event for failed logon attempts — the failure reason distinguishes between bad credentials and policy denial

---

## Interview Takeaway

> *"Deny interactive logon is enforced at the computer level during authorisation, not authentication. The user's credentials are validated by AD first, then the server evaluates the GPO User Rights Assignment. If the user's group membership matches the deny policy linked to the computer's OU, the logon is blocked — even though authentication succeeded. Deny always overrides allow."*

---

## References

- [Microsoft — User Rights Assignment](https://learn.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/user-rights-assignment)
- [Microsoft — Deny log on locally](https://learn.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/deny-log-on-locally)
- [Microsoft — Deny log on through Remote Desktop Services](https://learn.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/deny-log-on-through-remote-desktop-services)
- [Microsoft — Group Policy Overview](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/group-policy/group-policy-overview)
