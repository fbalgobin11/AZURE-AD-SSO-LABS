# Lab 1 - SAML SSO with Azure AD

## Objective

Configure SAML-based Single Sign-On between Microsoft Azure AD (Entra ID) and a third-party Service Provider.

## Environment

**Identity Provider (IdP)**

* Microsoft Azure AD (Entra ID)

**Service Provider (SP)**

* sptest.iamshowcase.com

**Protocol**

* SAML 2.0

---
## Architecture Overview

<img width="880" height="828" alt="image" src="https://github.com/user-attachments/assets/086c4ace-75a9-4b49-b75e-e0da66d0bf7a" />



## Configuration Steps

### Step 1 - Create an Enterprise Application

In Azure AD:

* Enterprise Applications
* New Application
* Non-gallery Application

Created a new application called **SP-Test**.
<img width="1908" height="740" alt="image" src="https://github.com/user-attachments/assets/f2aab4bb-f706-4199-ba6b-9cf3f5686405" />



---

### Step 2 - Obtain Service Provider Metadata

Retrieved the following values from the Service Provider metadata:

* Entity ID
* Assertion Consumer Service (ACS) URL

Values used:

Entity ID:
https://sptest.iamshowcase.com/metadata

ACS URL:
https://sptest.iamshowcase.com/acs

<img width="747" height="389" alt="image" src="https://github.com/user-attachments/assets/a13120d9-2db9-4c6f-a616-b19f3ac51fc4" />


---

### Step 3 - Configure SAML in Azure AD

Configured the following settings in the Azure AD SAML configuration page:

* Identifier (Entity ID)
* Reply URL (ACS URL)

<img width="705" height="618" alt="image" src="https://github.com/user-attachments/assets/c80959f2-2fbe-410b-af2b-ce24a4154382" />

---

### Step 4 - Establish Federation Trust

Downloaded the Azure AD Federation Metadata XML file.

This metadata contains:

* Azure AD signing certificate
* SAML endpoints
* Federation information

<img width="701" height="368" alt="image" src="https://github.com/user-attachments/assets/5425e0bb-25fc-4e4a-8dcd-7a210b8ef7cb" />


---

### Step 5 - Upload Azure AD Metadata to the Service Provider

Uploaded the Federation Metadata XML to the Service Provider.

This established trust between Azure AD and the Service Provider.

<img width="793" height="486" alt="image" src="https://github.com/user-attachments/assets/e1ca64bf-4982-46ef-b00f-17fc6f9d2653" />


---

### Step 6 - Test Authentication

Used the generated login URL and authenticated using an Azure AD user account.

Successful authentication redirected the user back to the Service Provider.

<img width="811" height="511" alt="image" src="https://github.com/user-attachments/assets/913cc7ec-08a3-4fc6-8bc8-fcf9d5259856" />

---

## Authentication Flow

1. User accesses the Service Provider.
2. Service Provider redirects user to Azure AD.
3. User authenticates with Azure AD.
4. Azure AD issues a SAML Assertion.
5. Service Provider validates the assertion.
6. User is granted access.



---

## Key Learnings

* Azure AD can act as a SAML Identity Provider.
* SAML metadata exchange establishes trust between systems.
* The Entity ID uniquely identifies the Service Provider.
* The ACS URL determines where SAML assertions are delivered.
* Federation Metadata XML contains configuration information required by the Service Provider.
