# Azure AD SSO Hands-On Labs (SAML + OIDC)

## Overview
This project demonstrates practical implementation of Single Sign-On (SSO) using Microsoft Azure Active Directory as an Identity Provider (IdP), integrating with both SAML and OpenID Connect (OIDC) Service Providers.

It includes:
- SAML-based SSO integration (Azure AD ↔ SaaS test application)
- OIDC authentication flow (Azure AD ↔ Grafana Cloud)
- End-to-end authentication flow understanding
- Real-world IAM configuration experience

---

## Technologies Used
- Microsoft Azure Active Directory (Entra ID)
- SAML 2.0
- OpenID Connect (OIDC)
- OAuth 2.0 Authorization Code Flow
- Grafana Cloud
- SSO Test Service Provider (iamshowcase)

---

## Lab 1: SAML SSO (Azure AD ↔ SaaS SP)

### Objective
Configure SAML-based SSO between Azure AD and a third-party Service Provider.

### Key Steps
- Created Azure AD tenant (M365 Developer Program)
- Registered Enterprise Application (Non-gallery app)
- Configured:
  - Entity ID
  - Reply URL (ACS)
- Downloaded Federation Metadata XML from Azure AD
- Uploaded metadata to Service Provider
- Tested authentication flow

### Outcome
Successful SAML authentication with Azure AD acting as Identity Provider.

---

## Lab 2: OIDC SSO (Azure AD ↔ Grafana Cloud)

### Objective
Implement OpenID Connect authentication using Azure AD as IdP.

### Key Steps
- Created Grafana Cloud instance
- Registered Azure AD App (App Registration)
- Configured:
  - Redirect URI
  - Client ID & Secret
  - Issuer URL
- Enabled OIDC login in Grafana
- Tested authentication via Microsoft login

### Outcome
Successful OAuth2 Authorization Code Flow with JWT-based identity token.

---

## Key IAM Concepts Demonstrated
- Identity Provider vs Service Provider
- SAML assertion-based authentication
- OAuth2 Authorization Code Flow
- JWT token validation
- Enterprise application configuration
- Federation metadata exchange

---

## Architecture Diagrams
See `/architecture` folder for:
- SAML flow diagram
- OIDC flow diagram

---

## What I Learned
- Real-world IAM federation setup
- Differences between SAML vs OIDC in practice
- How enterprise SSO is configured in Azure AD
