# 🔐 Self-Hosted IAM & SSO Infrastructure

A production-style Identity and Access Management (IAM) lab built on Oracle Cloud Infrastructure, demonstrating OAuth2/OIDC single sign-on, MFA enforcement, and self-hosted identity provider configuration using open-source tooling.

## 🏗️ Architecture
┌─────────────┐ OAuth2/OIDC ┌──────────────────┐
│ User │ ──── Login Flow ────▶ │ Authentik IdP │
│ Browser │ │ (Port 9000) │
└─────────────┘ └────────┬─────────┘
▲ │
│ JWT Token + Redirect │ Validates
│ ▼
┌──────┴──────┐ ┌──────────────────┐
│ Gitea │ ◀──── Authorized ──── │ TOTP/MFA │
│ (Port 3000)│ │ Enforcement │
└─────────────┘ └──────────────────┘


## ✅ Features

- **OAuth2/OIDC SSO** — Gitea authenticates entirely through Authentik; no passwords handled at the application layer
- **MFA Enforcement** — TOTP-based multi-factor authentication required on every login via custom Authentik flow
- **Auto-enrollment** — First-time users are automatically prompted to configure an authenticator app
- **Custom Scope Mapping** — Gitea receives group membership claims via a custom property mapping
- **Self-hosted on OCI** — Deployed on Oracle Cloud Free Tier with network-level security via Security Lists and iptables

## 🛠️ Stack

| Component | Technology | Purpose |
|---|---|---|
| Identity Provider | [Authentik 2026.2.2](https://goauthentik.io) | SSO, OAuth2/OIDC, MFA |
| Git Platform | [Gitea 1.26](https://gitea.com) | OAuth2 client / portfolio app |
| Database | PostgreSQL 16 | Authentik backend |
| Cache | Redis Alpine | Authentik session store |
| Container Runtime | Docker + Compose | Service orchestration |
| Stack Manager | [Dockge](https://dockge.kuma.pet) | Compose UI |
| Cloud | Oracle Cloud Infrastructure | Compute + Networking |
| OS | Ubuntu 22.04 | Host system |

## 🔑 IAM Concepts Demonstrated

**OAuth2 Authorization Code Flow** — Gitea acts as the OAuth2 relying party, redirecting authentication to Authentik as the authorization server. After the user authenticates, Authentik issues a signed JWT that Gitea validates to grant access — the application never handles credentials directly.

**Custom Authentication Flows** — Authentik's flow engine is used to chain stages: identifier → password → TOTP validation. The validation stage is configured with a `Configure` fallback that auto-enrolls users who haven't set up an authenticator yet.

**Scope-based Claims** — A custom property mapping injects group membership into the OIDC token, enabling role-based access decisions downstream.

## 🚀 Deployment

### Prerequisites
- Oracle Cloud (or any Ubuntu VPS)
- Docker + Docker Compose
- Ports 9000 and 3000 open in firewall

### 1. Clone and configure

```bash
git clone https://github.com/TGKDre/iam-homelab
cd iam-homelab
cp authentik/.env.example authentik/.env
# Fill in PG_PASS and AUTHENTIK_SECRET_KEY in .env
```

### 2. Generate secrets

```bash
# PG_PASS
openssl rand -base64 36

# AUTHENTIK_SECRET_KEY
openssl rand -base64 60
```

### 3. Deploy Authentik

```bash
cd authentik/
docker compose up -d
```

Access the setup wizard at `http://<your-ip>:9000/if/flow/initial-setup/`

### 4. Deploy Gitea

```bash
cd gitea/
docker compose up -d
```

Access Gitea at `http://<your-ip>:3000`

### 5. Configure SSO

Follow the [SSO Configuration Guide](./docs/sso-setup.md) to link Gitea to Authentik as an OAuth2 provider.

## 📁 Repository Structure
iam-homelab/
├── authentik/
│ ├── compose.yaml
│ └── .env.example
├── gitea/
│ └── compose.yaml
├── docs/
│ ├── sso-setup.md
│ ├── mfa-enforcement.md
│ └── architecture.md
└── README.md


## 🔒 Security Notes

- All secrets are managed via `.env` files excluded from version control
- PostgreSQL is not exposed outside the Docker network
- OCI Security Lists restrict inbound traffic to explicitly allowed ports only
- iptables rules provide a second enforcement layer at the OS level

## 📚 Related Concepts

This lab maps to enterprise IAM patterns used with **Okta**, **Azure AD**, and **Ping Identity**:

| This Lab | Enterprise Equivalent |
|---|---|
| Authentik | Okta / Azure AD / Ping Identity |
| OAuth2 Provider | OIDC Identity Provider |
| Flow + Stages | Okta Sign-On Policy + MFA Rules |
| Property Mappings | Attribute Statements / Claims Mapping |
| Outpost (next) | Okta Access Gateway / Zero Trust Proxy |

## 👤 Author

**Andre Uzoukwu** — IAM & Cybersecurity Engineer
- GitHub: [@TGKDre](https://github.com/TGKDre)
- LinkedIn: [linkedin.com/in/andre-uzoukwu-tgkdre](https://linkedin.com/in/andre-uzoukwu-tgkdre)

