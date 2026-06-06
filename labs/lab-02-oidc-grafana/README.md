# Lab 2 - OpenID Connect (OIDC) SSO with Azure AD and Grafana Cloud SaaS app

## Objective

Implement OpenID Connect (OIDC) Single Sign-On between Microsoft Azure AD (Entra ID) and Grafana Cloud using OAuth 2.0 Authorization Code Flow.

---

## Environment

**Identity Provider (IdP)**

* Microsoft Azure AD (Entra ID)

**Service Provider (SP)**

* Grafana Cloud (https://grafana.com)

**Protocol**

* OpenID Connect (OAuth 2.0 Authorization Code Flow)

---

## Architecture Overview

[INSERT OIDC FLOW DIAGRAM HERE]

---

## Configuration Steps

### Step 1 - Create Grafana Cloud Instance

* Created a free Grafana Cloud account
* Deployed a Grafana stack
* Accessed Grafana instance URL

📸 Screenshot:
<img width="1044" height="486" alt="image" src="https://github.com/user-attachments/assets/6b02740f-dde7-442b-b492-da5565d9ea0e" />


---

### Step 2 - Enable OIDC Authentication in Grafana

In Grafana:

* Navigate to **Administration → Authentication**
* Select **OpenID Connect**
* Prepare the following values:

  * Client ID
  * Client Secret
  * Issuer URL
  * Scopes
  * Redirect URI

📸 Screenshot:
<img width="1160" height="600" alt="image" src="https://github.com/user-attachments/assets/6a4cd896-40f3-4c63-8914-ff36115b08fc" />

---

### Step 3 - Register Application in Azure AD

In Azure AD:

* Go to **App Registrations → New Registration**
* Name: `Grafana-OIDC`
* Configure Redirect URI:

  ```
  https://<your-grafana-stack>.grafana.net/login/azuread
  ```

📸 Screenshot:
<img width="1071" height="519" alt="image" src="https://github.com/user-attachments/assets/9df7451e-16df-468c-993a-73d7e7459c17" />

---

### Step 4 - Configure Authentication Settings

* Enabled **ID Tokens**
* Added Redirect URI under Authentication settings

---

### Step 5 - Create Client Secret

* Go to **Certificates & Secrets**
* Generate new client secret
* Copy value (used in Grafana config)

<img width="1170" height="341" alt="image" src="https://github.com/user-attachments/assets/7bab92a1-b85c-441e-8e56-e9f15c9b1bf4" />

---

### Step 6 - Retrieve Azure Endpoints

From App Registration → Endpoints:

* Issuer URL
* Authorization Endpoint
* Token Endpoint
  
<img width="801" height="397" alt="image" src="https://github.com/user-attachments/assets/8b7a9deb-e8c5-4f59-be93-0fe8352e0bbd" />

---

### Step 7 - Configure Grafana with Azure AD

In Grafana:

* Client ID → Azure AD App ID
* Client Secret → Azure AD secret
* Issuer URL → Azure AD v2 endpoint
* Scopes:

  ```
  openid profile email
  ```
<img width="842" height="609" alt="image" src="https://github.com/user-attachments/assets/d905ff67-5443-41b2-a2e4-373ab9367ed8" />



---

### Step 8 - Test Login

* Navigate to Grafana login page
* Select Azure AD login
* Authenticate with Azure credentials
* MFA challenge (if enabled)

📸 Screenshot:
<img width="833" height="426" alt="image" src="https://github.com/user-attachments/assets/96dbf47e-26d7-43ed-aff0-0c41cfadd613" />

<img width="839" height="354" alt="image" src="https://github.com/user-attachments/assets/26ef4e08-07ce-4e28-a64b-08a1f0364267" />

<img width="834" height="334" alt="image" src="https://github.com/user-attachments/assets/8c41aee2-ae9a-45f7-98c5-3fae42ff9431" />



---

## Authentication Flow

### OIDC Authorization Code Flow

1. User accesses Grafana
2. Grafana redirects user to Azure AD
3. User authenticates (MFA if required)
4. Azure AD returns authorization code
5. Grafana exchanges code for tokens
6. Azure returns ID Token (JWT)
7. Grafana validates token and logs user in

---

## Token Exchange Flow

* Authorization Code → exchanged for Access Token + ID Token
* ID Token contains user identity claims
* JWT is validated using Azure AD public keys

---

## Key Learnings

* OIDC is modern replacement for SAML in many SaaS apps
* OAuth 2.0 handles authorization, OIDC adds identity layer
* Authorization Code Flow is the most secure SSO flow
* Client Secret is required for server-side token exchange
* JWT tokens are signed and verifiable by Azure AD
