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

[INSERT SCREENSHOT HERE]

---

### Step 3 - Configure SAML in Azure AD

Configured the following settings in the Azure AD SAML configuration page:

* Identifier (Entity ID)
* Reply URL (ACS URL)

[INSERT SCREENSHOT HERE]

---

### Step 4 - Establish Federation Trust

Downloaded the Azure AD Federation Metadata XML file.

This metadata contains:

* Azure AD signing certificate
* SAML endpoints
* Federation information

[INSERT SCREENSHOT HERE]

---

### Step 5 - Upload Azure AD Metadata to the Service Provider

Uploaded the Federation Metadata XML to the Service Provider.

This established trust between Azure AD and the Service Provider.

[INSERT SCREENSHOT HERE]

---

### Step 6 - Test Authentication

Used the generated login URL and authenticated using an Azure AD user account.

Successful authentication redirected the user back to the Service Provider.

[INSERT SCREENSHOT HERE]

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
