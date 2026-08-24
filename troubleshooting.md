---
title: "Troubleshooting"
description: "The Semaphore problems people actually hit — runner registration errors, fact gathering on localhost, Postgres SSL panics, Git authentication failures and the four common LDAP errors — with the fix for each."
---

# Troubleshooting

The failures below account for most of what gets asked in Discord and GitHub Discussions. Each one starts with the message you will actually see.

<!-- widget:accordion -->

## Runner prints error 404 or 401

The runner reaches the server but the server rejects it. In practice this is a registration problem, not a network one.

**Check, in order:**

1. `use_remote_runner` is `true` in the **server** configuration. Without it the server has no runner endpoints, which is what produces a 404.
2. `runner_registration_token` is set on the server, and the runner used that exact value when registering via CLI.
3. `web_host` in the runner configuration points at the server's real, reachable URL — including the scheme, and including any path prefix if Semaphore sits behind a proxy.
4. The runner's token has not been invalidated by removing the runner in the web UI. Re-register with `semaphore runner register` if it has.

There is community discussion of the 401 case in [semaphoreui/semaphore#1873](https://github.com/semaphoreui/semaphore/discussions/1873).

[Runner setup →](./operate/runners.md)

## Gathering Facts fails for localhost

```
TASK [Gathering Facts] *********************************************************
fatal: [localhost]: FAILED! => changed=false
```

This happens on the Docker and Snap installations. Ansible tries to gather facts locally, but it is running inside a restricted container that does not permit it. Background: [implicit localhost](https://docs.ansible.com/ansible/latest/inventory/implicit_localhost.html).

**Two fixes.**

Disable fact gathering for the play:

```yaml
- hosts: localhost
  gather_facts: false
  roles:
    - ...
```

Or force an SSH connection instead of the local one:

```ini
[localhost]
127.0.0.1 ansible_connection=ssh ansible_ssh_user=your_localhost_user
```

## panic: pq: SSL is not enabled on the server

Semaphore is trying to reach PostgreSQL over TLS, and that PostgreSQL is not configured for it.

Either enable SSL on the database — the better answer for anything crossing a network — or disable it explicitly in the connection options:

```json
{
  "postgres": {
    "host": "localhost",
    "user": "postgres",
    "pass": "pwd",
    "name": "semaphore",
    "options": {
      "sslmode": "disable"
    }
  }
}
```

## fatal: bad numeric config value '0' for 'GIT_TERMINAL_PROMPT'

Misleading message, simple cause: you are cloning a repository over HTTPS that requires authentication, and no usable credential is attached.

1. Go to **Key Store** and create a key of type **Login with password**.
2. Enter your username for GitHub, Bitbucket or wherever the repository lives.
3. For the password, use a **personal access token**, not your account password — GitHub and Bitbucket no longer accept account passwords for Git. See [creating a personal access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token).
4. Go to **Repositories**, open your repository, and select the key.

[Key Store →](./concepts/key-store.md)

## unable to read LDAP response packet: unexpected EOF

You are connecting to the LDAP server insecurely while it expects TLS.

```json
{
  "ldap_needtls": true
}
```

If that alone does not fix it, confirm you are pointing at the LDAPS port — typically `636` rather than `389`.

## LDAP Result Code 49 "Invalid Credentials"

The bind DN or the bind password is wrong. Prove it outside Semaphore before changing anything inside it. `ldapwhoami` comes from the `openldap-clients` package:

```bash
ldapwhoami \
  -H ldap://ldap.example.com:389 \
  -D "CN=your_ldap_binddn_value_in_config" \
  -x \
  -W
```

It prompts for the password, and on success returns code **0** and echoes the DN. If this fails, the problem is your directory configuration, not Semaphore.

Further reading: [ldapsearch: Invalid credentials (49)](https://serverfault.com/q/771549/443463) and [semaphoreui/semaphore#906](https://github.com/semaphoreui/semaphore/issues/906).

## LDAP Result Code 32 "No Such Object"

The search DN does not exist in the directory, or is scoped too narrowly to contain the user you are signing in as.

Check `ldap_searchdn` against the tree — `ou=users,dc=example,dc=org` fails if your users actually live under `cn=users,cn=accounts,dc=example,dc=org`. Verify with `ldapsearch` using the same base DN and filter before putting it in the configuration.

## Live task logs do not stream

The task runs and the log appears only after it finishes, or the page stays blank.

Semaphore streams logs over WebSocket at `/api/ws`. A reverse proxy that does not forward the upgrade header breaks it. There is a working NGINX block in [High availability](./operate/high-availability.md#load-balancer) — note the separate `location /api/ws` with long timeouts.

## The database keeps growing

Task log retention is **unlimited by default**. Set a per-template cap:

```json
{ "max_tasks_per_template": 30 }
```

If you need history beyond that, ship it somewhere before it rotates — see [Logs and metrics](./operate/observability.md).

## A limit or a branch passed through the API is ignored

The corresponding **prompt** is not enabled on the template. Semaphore accepts the value and drops it. Enable the prompt in template settings; the run stays unattended. See [Templates and tasks](./concepts/templates-and-tasks.md#parameters-at-run-time).

<!-- /widget -->

## Getting more detail

Raise the log level and narrow the output to the subsystem you care about:

```bash
semaphore server --config ./config.json --log-level DEBUG --debug-filter 'runner,task_*'
```

Semaphore logs to stdout, so read it with `journalctl -u semaphore.service -f` or `docker logs -f <container>`.

## Still stuck

- **Discord:** [discord.gg/5R6k7hNGcH](https://discord.gg/5R6k7hNGcH) — the fastest route for a question.
- **GitHub Issues:** [semaphoreui/semaphore/issues](https://github.com/semaphoreui/semaphore/issues) — for bugs and feature requests.
- **Pro and Enterprise support:** Pro includes a 48-hour business-day SLA; Enterprise has custom SLA options. See [Plans and pricing](./pricing.md).

When you report something, include the Semaphore version (`semaphore version`), the install method, the database, and the relevant lines from the server log. It roughly halves the round trips.
