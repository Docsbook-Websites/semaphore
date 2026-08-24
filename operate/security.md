---
title: "Security"
description: "How Semaphore handles authentication, secrets, encryption and untrusted playbooks — and what you should configure before it touches production infrastructure."
---

# Security

Semaphore holds credentials for your infrastructure and runs arbitrary code against it. That combination deserves attention. This page covers what Semaphore does by default, and what you have to do yourself.

## Authentication and authorization

**Login methods**

- **Username / password** — credentials in Semaphore's database, hashed with bcrypt.
- **LDAP** — enterprise directory integration with user and group filtering, over LDAPS.
- **OpenID Connect** — single sign-on with Google, Entra ID, Keycloak, Okta and others, including custom claims.

Setup for both is in [Authentication](./authentication.md).

**Two-factor authentication.** TOTP-based 2FA, enabled per user, with optional recovery codes:

```json
{
  "auth": {
    "totp": { "enabled": true, "allow_recovery": true }
  }
}
```

**Role-based access control.** Four built-in project roles — Owner, Manager, Task Runner, Guest — plus Enterprise custom roles that can grant permissions on individual templates. See [Projects and teams](../concepts/projects.md#teams-and-roles).

**Sessions.** Protected with secure HTTP cookies, signed and encrypted with `cookie_hash` and `cookie_encryption`, with expiry and explicit logout.

## Secrets and credentials

- **Encrypted key store.** Credentials and secret variables are encrypted at rest with AES, using the key in `access_key_encryption`.
- **Runtime-only exposure.** Secrets are passed into a job when it runs. They are not written into the container environment ahead of time.
- **External secret managers.** On Pro and Enterprise, secrets can live in HashiCorp Vault, OpenBao, AWS Secrets Manager, Azure Key Vault or Devolutions Server instead of the database, chosen per secret.

Generate the encryption key once and keep it safe:

```bash
head -c32 /dev/urandom | base64
```

> Store `access_key_encryption` somewhere other than your database backups. A backup that contains both the ciphertext and the key is a backup of plaintext.

[More on the Key Store →](../concepts/key-store.md)

## Running untrusted code

Semaphore executes user-defined playbooks and commands. Several layers reduce what a bad one can do:

- **Container isolation** — tasks execute in isolated Docker containers with no access to the host.
- **Least privilege** — containers run with minimal permissions, restrictable further with Docker flags.
- **Chroot execution** — tasks can be confined to a chroot jail (`process.chroot`).
- **Dedicated task user** — tasks can run as a non-root system user such as `semaphore`.
- **Runners** — the strongest boundary: move execution off the control plane entirely. See [Runners](./runners.md).

The practical rule: anyone who can create or edit a task template can run code in the execution environment. Manage that with roles, not with hope.

## Secure deployment

**Use HTTPS.** Semaphore supports TLS directly, or you can terminate at a reverse proxy. Built-in TLS:

```json
{
  "tls": {
    "enabled": true,
    "cert_file": "/path/to/cert/example.com.cert",
    "key_file": "/path/to/key/example.com.key"
  }
}
```

For NGINX, Apache or Caddy in front, see the [reverse proxy guides](https://semaphoreui.com/docs/admin-guide/reverse-proxy/nginx) — and remember the proxy must pass WebSocket upgrades on `/api/ws` for live task logs.

**Run behind a firewall.** Limit access to the UI and to the database to trusted networks.

**Database security.** Strong passwords, and access restricted to Semaphore alone.

## Updates and patch management

- Run the latest stable release; security fixes ship regularly.
- Read the changelog on GitHub before upgrading.
- On Docker, automate updates through a pipeline rather than doing them by hand at intervals you will forget.

[Upgrading →](./upgrading.md)

## Reporting a vulnerability

Email **security@semaphoreui.com**. Please do not publish details before a patch is available.

**Resolution targets**

| Severity | Target |
|---|---|
| Critical | 30 days |
| High | 60 days |
| Medium | 90 days |
| Low | Best effort, typically 180 days |

Out-of-cycle patches may be released for actively exploited issues in the latest stable release.

**Code security tooling.** The project uses CodeQL, Codacy, Snyk and Renovate for code and dependency analysis, and automated dependency updates. Researchers may be acknowledged in release notes if they wish.

## A pre-production checklist

<!-- widget:accordion -->

### Encryption and cookies

`access_key_encryption`, `cookie_hash` and `cookie_encryption` are set to generated values — not to anything copied from a tutorial, this page included.

### Transport

HTTPS is enabled, either natively or at a reverse proxy, and runners talk to the server over HTTPS.

### Identity

LDAP or OIDC is configured if you have a directory, TOTP is enabled, and the default admin password has been changed.

### Blast radius

Execution runs on a runner, or in a container, or under a dedicated non-root user — not as root on the control plane.

### Access

Each project has at least two Owners, and everyone else has the lowest role that lets them do their job.

### Audit

Activity logging is on and shipped somewhere outside the box. See [Logs and metrics](./observability.md).

<!-- /widget -->

## Next steps

- [Authentication](./authentication.md) — LDAP, Active Directory and OIDC
- [Logs and metrics](./observability.md) — the audit trail
- [High availability](./high-availability.md) — keeping it up
