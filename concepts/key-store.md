---
title: "Key Store"
description: "Semaphore's Key Store holds SSH keys, passwords, tokens and Ansible vault passwords — encrypted at rest, injected only at run time, and optionally backed by HashiCorp Vault, OpenBao or Devolutions Server."
---

# Key Store

The Key Store is where every credential Semaphore uses lives: SSH keys for cloning repositories and logging into hosts, sudo credentials, API tokens, and Ansible vault passwords.

The rule that matters: secrets are **encrypted at rest** and passed into a task **only at the moment it runs**. They are never written into the container environment ahead of time, and never exposed in the UI once saved.

## Key types

### SSH

A private key, used both for remote Git repositories and for logging into target hosts. The optional *user* field on the key determines the SSH username.

For Git over SSH, the corresponding public key has to be registered with your Git host — [GitHub](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account), [GitLab](https://docs.gitlab.com/ee/user/ssh.html), [Bitbucket](https://support.atlassian.com/bitbucket-cloud/docs/set-up-an-ssh-key/).

### Login with password

A username plus a password or access token. Use it to:

- authenticate to remote hosts (less secure than an SSH key)
- provide sudo credentials
- authenticate to Git repositories over HTTPS
- unlock Ansible vaults

> Leave the **Login** field empty and this becomes a bare secret string — which is how you store a personal access token or an API key.

### None

A placeholder for repositories that need no authentication, such as a public open-source repository. A repository still requires *a* key, so this is the one to use.

## Where secrets are stored

The storage backend is chosen **per secret**, when you create or edit it.

### Database (default, all plans)

Secrets are encrypted and stored in Semaphore's own database. The encryption key comes from configuration:

```json
{ "access_key_encryption": "..." }
```

or `SEMAPHORE_ACCESS_KEY_ENCRYPTION`. Generate it once, and treat it as the most important value in your configuration:

```bash
head -c32 /dev/urandom | base64
```

If you lose this key, the credentials in your database become unreadable. Back it up separately from the database itself — a backup containing both is a backup of plaintext.

### HashiCorp Vault (Pro)

Secrets live in an external Vault instance and Semaphore reads them at run time. The database never holds the value. See the [upstream guide](https://semaphoreui.com/docs/user-guide/key-store/hashicorp-vault).

### OpenBao

[OpenBao](https://openbao.org) is an open-source, API-compatible fork of Vault, configured the same way. See the [upstream guide](https://semaphoreui.com/docs/user-guide/key-store/openbao).

### Devolutions Server (Enterprise)

Secrets stored in an external Devolutions Server instance. See the [upstream guide](https://semaphoreui.com/docs/user-guide/key-store/devolutions-server).

### AWS Secrets Manager and Azure Key Vault (Enterprise)

Available as secret storages on Enterprise plans.

## Syncing from an external secret manager

Rather than copying secrets in by hand, Semaphore can import them from HashiCorp Vault, OpenBao, AWS Secrets Manager, Azure Key Vault or Devolutions Server, and keep them in sync. **Sync paths** select which secrets to import and how they are named on the Semaphore side.

This is the pattern to use when a platform team owns the secret manager and Semaphore is one of several consumers — rotation happens in one place and propagates. See the [upstream guide](https://semaphoreui.com/docs/user-guide/key-store/secret-sync).

## Rotating the encryption key

The CLI can re-encrypt everything in the database under a new key:

```bash
semaphore vaults --help
```

See [CLI](../operate/cli.md) for the full command set, and run it with the server stopped.

## Practical guidance

- Prefer **SSH keys** over passwords for host access, and **personal access tokens** over account passwords for HTTPS Git.
- Give each project its own keys where you can. A key shared across projects is a key shared across teams.
- Attach **multiple vault passwords** to an Ansible template when different files use different vaults — Ansible tries each in turn.
- Keep `access_key_encryption` out of the same store as your database backups.

## Next steps

- [Inventory and variables](./inventory-and-variables.md) — where keys get attached.
- [Security](../operate/security.md) — the rest of the hardening picture.
- [Ansible](../apps/ansible.md) — vault passwords in practice.
