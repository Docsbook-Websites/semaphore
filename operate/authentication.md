---
title: "Authentication: LDAP and OpenID Connect"
description: "Connect Semaphore to your identity provider — LDAP and Active Directory, or OpenID Connect single sign-on with Okta, Entra ID, Keycloak, Google, GitHub and more, including IdP-initiated login."
---

# Authentication

Semaphore authenticates users against its own database by default. For anything shared by a team, connect it to the directory you already have.

| Method | Available on |
|---|---|
| Username / password | All plans |
| Two-factor (TOTP) | Pro, Enterprise |
| LDAP / Active Directory | Pro, Enterprise |
| SSO via OIDC | Pro, Enterprise |
| Role mapping from IdP groups | Enterprise |

## LDAP and Active Directory

Configure LDAP in `config.json`:

```json
{
  "ldap_binddn": "cn=admin,dc=example,dc=org",
  "ldap_bindpassword": "admin_password",
  "ldap_server": "localhost:389",
  "ldap_searchdn": "ou=users,dc=example,dc=org",
  "ldap_searchfilter": "(&(objectClass=inetOrgPerson)(uid=%s))",
  "ldap_mappings": {
    "dn": "",
    "mail": "uid",
    "uid": "uid",
    "cn": "cn"
  },
  "ldap_enable": true,
  "ldap_needtls": true
}
```

### Options

| Config option | Environment variable | Description |
|---|---|---|
| `ldap_binddn` | `SEMAPHORE_LDAP_BIND_DN` | The LDAP user object to bind as |
| `ldap_bindpassword` | `SEMAPHORE_LDAP_BIND_PASSWORD` | Password for that user |
| `ldap_server` | `SEMAPHORE_LDAP_SERVER` | Host including port, e.g. `localhost:389` |
| `ldap_searchdn` | `SEMAPHORE_LDAP_SEARCH_DN` | Where users are searched, e.g. `ou=users,dc=example,dc=org` |
| `ldap_searchfilter` | `SEMAPHORE_LDAP_SEARCH_FILTER` | Search expression; `%s` is replaced by the entered login. Default `(&(objectClass=inetOrgPerson)(uid=%s))` |
| `ldap_mappings.mail` | `SEMAPHORE_LDAP_MAPPING_MAIL` | Email claim expression |
| `ldap_mappings.uid` | `SEMAPHORE_LDAP_MAPPING_UID` | Login claim expression |
| `ldap_mappings.cn` | `SEMAPHORE_LDAP_MAPPING_CN` | Display name claim expression |
| `ldap_enable` | `SEMAPHORE_LDAP_ENABLE` | Turn LDAP on |
| `ldap_needtls` | `SEMAPHORE_LDAP_NEEDTLS` | Connect over TLS |

### Claim expressions

Mappings accept a fallback expression, separated by `|`:

```
email | {{ .username }}@your-domain.com
```

Semaphore tries the first field; if it is empty, the expression after the pipe is evaluated.

> A mapping of just `"|"` generates a **random** value for each login. Setting `"uid": "|"` therefore creates a new user on every sign-in, which is almost never what you want in production.

### Testing your bind DN

Before blaming Semaphore, prove the bind works. `ldapwhoami` comes from the `openldap-clients` package:

```bash
ldapwhoami \
  -H ldap://ldap.example.com:389 \
  -D "CN=your_ldap_binddn_value_in_config" \
  -x \
  -W
```

It prompts for the password and should return code **0** plus the DN you specified. If it fails here, no Semaphore configuration will save you — see [Troubleshooting](../troubleshooting.md#ldap-result-code-49-invalid-credentials).

### A local test server

Useful for validating the configuration before pointing at the real directory:

```bash
docker run -d --name openldap \
  -p 1389:1389 -p 1636:1636 \
  -e LDAP_ADMIN_USERNAME=admin \
  -e LDAP_ADMIN_PASSWORD=pwd \
  -e LDAP_USERS=user1 \
  -e LDAP_PASSWORDS=pwd \
  -e LDAP_ROOT=dc=example,dc=org \
  -e LDAP_ADMIN_DN=cn=admin,dc=example,dc=org \
  bitnami/openldap:latest
```

## OpenID Connect

OIDC providers are configured under `oidc_providers`, keyed by a provider ID that appears in the redirect URL:

```json
{
  "oidc_providers": {
    "mysso": {
      "display_name": "Sign in with MySSO",
      "color": "orange",
      "icon": "login",
      "provider_url": "https://mysso-provider.com",
      "client_id": "***",
      "client_secret": "***",
      "redirect_url": "https://your-domain.com/api/auth/oidc/mysso/redirect"
    }
  }
}
```

In containers, the whole block can come from one environment variable holding valid JSON:

```bash
SEMAPHORE_OIDC_PROVIDERS='{
  "github": {
    "client_id": "***",
    "client_secret": "***"
  }
}'
```

Each configured provider adds a button to the sign-in screen.

### Provider options

| Option | Description |
|---|---|
| `display_name` | Text on the login button |
| `icon` | [MDI icon](https://pictogrammers.com/library/mdi/) shown before the name |
| `color` | Button colour |
| `client_id`, `client_secret` | Provider credentials |
| `client_id_file`, `client_secret_file` | Read them from files instead; lower priority than the inline values |
| `redirect_url`, `provider_url`, `scopes` | Standard OIDC endpoints and scopes |
| `username_claim`, `email_claim`, `name_claim` | Claim expressions, with the same `\|` fallback syntax as LDAP |
| `order` | Position of the button on the sign-in screen |
| `allow_idp_initiated` | Enable IdP-initiated login. Default `false` |
| `return_via_state` | Pass the post-login return path via the OAuth `state` parameter. Default `true` |
| `endpoint.*` | `issuer`, `auth`, `token`, `userinfo`, `jwks`, `algorithms` — for providers without discovery |

Provider-specific walkthroughs for GitHub, Google, GitLab, Gitea, Authelia, Authentik, Keycloak, Okta, PingFederate, Azure, Zitadel and Pocket-ID are maintained upstream at [semaphoreui.com/docs/admin-guide/openid](https://semaphoreui.com/docs/admin-guide/openid).

### IdP-initiated login

By default Semaphore only supports **SP-initiated** sign-in: the user opens Semaphore and clicks the provider button. With IdP-initiated login the journey can start at the identity provider instead — the Semaphore tile in an Okta dashboard, Entra *My Apps*, or a Keycloak application launcher.

Semaphore implements this with the standard [Third-Party Initiated Login](https://openid.net/specs/openid-connect-core-1_0.html#ThirdPartyInitiatedLogin) mechanism: the IdP redirects to a dedicated initiate URI, and Semaphore then starts a normal Authorization Code flow. It never accepts an unsolicited token.

Enable it per provider:

```json
{
  "oidc_providers": {
    "mysso": {
      "provider_url": "https://mysso-provider.com",
      "client_id": "***",
      "client_secret": "***",
      "redirect_url": "https://your-domain.com/api/auth/oidc/mysso/redirect",
      "allow_idp_initiated": true
    }
  }
}
```

Then set the application's **Initiate Login URI** in your IdP to:

```
https://your-domain.com/api/auth/oidc/<provider-id>/initiate
```

The IdP must send the `iss` parameter; Semaphore rejects requests whose issuer does not match the configured provider. `login_hint` is forwarded, and `target_link_uri` sets the post-login page — but only when it points back at Semaphore.

**Provider notes**

- **Okta** — set *Login initiated by* to *Either Okta or App*, and fill in the *Initiate login URI*. Okta sends both `iss` and `target_link_uri`.
- **Keycloak, Authentik, Ping, OneLogin** — set the application's launch or home URL to the initiate URI.
- **Azure AD / Entra** — *My Apps* uses an SP-initiated start URL and does not always send `iss`. Point the start URL at `https://your-domain.com/api/auth/oidc/<provider-id>/login` instead.

**Security properties:** off by default and per-provider; `iss` validated against the configured issuer to prevent provider mix-up; `target_link_uri` accepted only for Semaphore URLs, so no open redirect; and the full Authorization Code exchange with CSRF `state` and a `nonce`, so a captured token cannot be replayed into a session.

## Next steps

- [Security](./security.md) — the rest of the hardening picture
- [Projects and teams](../concepts/projects.md#teams-and-roles) — what a user can do once they are in
- [Troubleshooting](../troubleshooting.md) — the LDAP errors people actually hit
