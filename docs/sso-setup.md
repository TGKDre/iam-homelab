# SSO Configuration Guide

This guide walks through connecting Gitea to Authentik as an OAuth2/OIDC identity provider, enabling single sign-on across the lab environment.

## Prerequisites

- Authentik running and accessible at `http://<your-ip>:9000`
- Gitea running and accessible at `http://<your-ip>:3000`
- Admin access to both platforms

## Step 1: Create a Custom Property Mapping in Authentik

Gitea requires a custom scope to receive group membership claims.

1. In the Authentik admin panel, go to **Customization → Property Mappings → Create**
2. Select **Scope Mapping** and fill in:
   - **Name**: `authentik gitea OAuth Mapping: OpenID 'gitea'`
   - **Scope name**: `gitea`
   - **Expression**:

```python
return {
    "groups": [group.name for group in request.user.ak_groups.all()],
}
```

3. Click **Finish**

## Step 2: Create the Application & Provider in Authentik

1. Go to **Applications → Applications → Create with Provider**
2. Configure the application:
   - **Name**: `Gitea`
   - **Slug**: `gitea`
3. Configure the provider:
   - **Provider type**: `OAuth2/OpenID Connect`
   - **Authorization flow**: `default-provider-authorization-explicit-consent`
   - **Redirect URI**: `http://<your-ip>:3000/user/oauth2/authentik/callback`
   - **Signing Key**: select the available self-signed certificate
4. Under **Advanced Protocol Settings → Selected Scopes**, add:
   - `authentik default OAuth Mapping: OpenID 'email'`
   - `authentik default OAuth Mapping: OpenID 'profile'`
   - `authentik default OAuth Mapping: OpenID 'openid'`
   - `authentik gitea OAuth Mapping: OpenID 'gitea'`
5. Click **Submit** and copy the **Client ID** and **Client Secret**

## Step 3: Add Authentik as Authentication Source in Gitea

1. In Gitea, go to **Site Administration → Identity & Access → Authentication Sources**
2. Click **Add Authentication Source** and fill in:

| Field | Value |
|---|---|
| Authentication Type | `OAuth2` |
| Authentication Name | `authentik` |
| OAuth2 Provider | `OpenID Connect` |
| Client ID | *(from Step 2)* |
| Client Secret | *(from Step 2)* |
| OpenID Connect Auto Discovery URL | `http://<your-ip>:9000/application/o/gitea/.well-known/openid-configuration` |
| Additional Scopes | `email profile` |

3. Click **Add Authentication Source**

## Step 4: Test the Integration

1. Open an incognito browser window
2. Navigate to your Gitea instance
3. Click **Sign In** — you should see a **"Sign in with authentik"** button
4. Click it — you will be redirected to Authentik to authenticate
5. After successful authentication, Authentik redirects back to Gitea with an authorization token
6. On first login, Gitea prompts you to link or register an account

## How It Works

The OAuth2 Authorization Code Flow used here:

User clicks "Sign in with authentik"
↓
Gitea redirects to Authentik with client_id + redirect_uri
↓
User authenticates at Authentik (password + MFA)
↓
Authentik issues authorization code → redirects to Gitea callback
↓
Gitea exchanges code for access token + ID token (JWT)
↓
Gitea validates JWT signature using Authentik's public key
↓
User is granted access — no password ever touches Gitea
