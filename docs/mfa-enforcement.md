# MFA Enforcement Guide

This guide covers enforcing TOTP-based multi-factor authentication on all Authentik logins using custom flow stages.

## Overview

Authentik uses a **flow + stages** model for authentication. By adding an Authenticator Validation Stage to the default login flow, every user — including those authenticating via SSO to Gitea — must pass MFA before receiving an access token.

## Step 1: Create the TOTP Setup Stage

This stage handles first-time TOTP enrollment for users who haven't configured an authenticator yet.

1. Go to **Flows & Stages → Stages → Create**
2. Select **Authenticator TOTP Stage** and configure:
   - **Name**: `totp-setup-stage`
   - **Digits**: `6`
3. Click **Finish**

## Step 2: Create the Authenticator Validation Stage

This stage enforces MFA on every login and hands off to the setup stage for unenrolled users.

1. Create another stage, select **Authenticator Validation Stage** and configure:
   - **Name**: `totp-validation-stage`
   - **Device Classes**: `TOTP Authenticators` only
   - **Not configured action**: `Configure`
   - **Configuration Stage**: `totp-setup-stage`
2. Click **Finish**

## Step 3: Bind the Stage to the Login Flow

1. Go to **Flows & Stages → Flows → `default-authentication-flow`**
2. Click **Stage Bindings → Bind Stage**
3. Configure the binding:
   - **Stage**: `totp-validation-stage`
   - **Order**: `30` (runs after password stage at order 20)
   - **Evaluate when stage is run**: ✅ Enabled
4. Click **Create**

## Step 4: Test MFA Enforcement

1. Open an incognito window and navigate to `http://<your-ip>:9000`
2. Enter your username and password
3. You should be prompted to either:
   - **Scan a QR code** (first-time enrollment) — scan with Google Authenticator or Authy
   - **Enter a 6-digit code** (subsequent logins)
4. After passing MFA, the session is established and any SSO redirects (e.g. from Gitea) complete automatically

## How It Applies to SSO

Because MFA is enforced at the **identity provider level**, every application using Authentik for SSO inherits MFA automatically — no per-application configuration needed. When a user clicks "Sign in with authentik" on Gitea, the full Authentik flow runs including the TOTP validation stage before the token is issued.

Gitea SSO redirect → Authentik Login Flow
↓
Stage 10: Identifier (username)
↓
Stage 20: Password
↓
Stage 30: TOTP Validation ← enforced here
↓
OAuth2 token issued → Gitea access granted


## Enterprise Equivalent

This pattern maps directly to **Okta Sign-On Policies** and **Azure AD Conditional Access** — both enforce MFA at the identity provider before issuing tokens to downstream applications, which is the industry-standard Zero Trust approach.
