---
title: "CLI"
description: "The semaphore binary is both the server and the administration tool: interactive setup, server and runner modes, user and project management, vault re-encryption and database migrations."
---

# CLI

The `semaphore` binary is the server *and* a full administration tool. Run it with no arguments to see everything it can do:

```bash
semaphore help
```

## Command groups

| Command group | Purpose |
|---|---|
| `semaphore users` | Add, change, remove and inspect users; manage API tokens and TOTP |
| `semaphore projects` | Export and import projects — the backup story for a single project |
| `semaphore vaults` | Re-encrypt stored secrets and inspect encryption key usage |
| `semaphore runner` | Run in runner mode; register and unregister runners |
| `semaphore migrate` | Apply or roll back database migrations |

Several groups have shorter aliases: `users`/`user`, `projects`/`project`, `vaults`/`vault`, and `server`/`service`.

The full per-command reference lives upstream at [semaphoreui.com/docs/admin-guide/cli](https://semaphoreui.com/docs/admin-guide/cli).

## Global options

Accepted by every command:

| Option | Description |
|---|---|
| `--config <path>` | Path to the configuration file |
| `--no-config` | Ignore any configuration file — environment variables only |
| `--log-level <level>` | `DEBUG`, `INFO`, `WARN`, `ERROR`, `FATAL` or `PANIC`. Falls back to `SEMAPHORE_LOG_LEVEL` |
| `--debug-filter <spec>` | Narrow `DEBUG` output to namespaces, e.g. `'runner,task_*'` or `'*,-db'`. Only applies at `DEBUG`. Falls back to `SEMAPHORE_DEBUG_FILTER` |

`--debug-filter` is the difference between a usable debug session and 200MB of noise.

## Version

```bash
semaphore version
```

## Interactive setup

First-time configuration. It generates secrets, walks an interactive questionnaire, writes `config.json`, runs the database migrations and creates the first admin user:

```bash
semaphore setup
```

On completion it prints the command to start the server.

## Server mode

Start the web UI and API. `service` is an alias:

```bash
semaphore server --config /path/to/config.json
```

## Runner mode

Run Semaphore as a task runner:

```bash
semaphore runner start --config /path/to/config.json
```

The group also has `setup`, `register` and `unregister`. See [Runners](./runners.md) for the whole flow.

## Database migration

Bring the schema up to date:

```bash
semaphore migrate --config /path/to/config.json
```

Migrations also run automatically when the server starts; the explicit command is useful when you want migration and startup to be separate steps in a deployment pipeline, or when rolling back to a specific version.

## Users and tokens

`semaphore users` covers the operations you need when nobody can get in — creating an admin, resetting a password, or clearing a TOTP enrolment for someone who lost their phone. It is also the way to script initial provisioning, since it works before anyone has signed in.

## Vaults

`semaphore vaults` re-encrypts stored secrets under a new `access_key_encryption` key. Run it with the server stopped, and keep the old key until you have confirmed the new one works.

## Next steps

- [Configure the server](../get-started/configuration.md)
- [Runners](./runners.md)
- [Key Store](../concepts/key-store.md) — what `vaults` operates on
