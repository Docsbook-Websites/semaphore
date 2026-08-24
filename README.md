---
title: "Semaphore UI documentation"
description: "Semaphore UI is an open-source web UI and API for running Ansible, Terraform/OpenTofu, PowerShell, Bash and Python automation. Install it, run your first task, and operate it in production."
---

# Semaphore UI documentation

Semaphore UI is a self-hosted web UI and REST API for running automation you already have: **Ansible** playbooks, **Terraform/OpenTofu** code, **PowerShell**, **Bash** and **Python** scripts. It is written in Go, runs on Linux, Windows and macOS, and stores its data in SQLite, MySQL or PostgreSQL.

Instead of `ssh`-ing into a box to run a playbook, you point Semaphore at a Git repository, describe the run once as a **task template**, and give the people who need it a button, a schedule, or an API call.

<!-- widget:cards -->

## Start here

- [Install Semaphore](./get-started/install.md) — Docker, package manager, binary or Helm, in a few minutes {download}
- [Run your first task](./get-started/first-task.md) — From an empty install to a playbook running on real hosts {rocket}
- [Configure the server](./get-started/configuration.md) — `config.json`, environment variables and the options that matter first {settings}

## Understand the model

- [Architecture](./concepts/architecture.md) — Server, database and runners, and where your automation actually executes {network}
- [Projects](./concepts/projects.md) — The unit of separation for teams, environments and applications {folder-tree}
- [Templates and tasks](./concepts/templates-and-tasks.md) — Reusable run definitions, and the runs themselves {list-checks}
- [Inventory and variables](./concepts/inventory-and-variables.md) — Which hosts to touch, and what to pass them {server}
- [Key Store](./concepts/key-store.md) — SSH keys, tokens and vault passwords, encrypted at rest {key-round}

## Run your tools

- [Ansible](./apps/ansible.md) — Playbooks, vaults, tags, limits and verbosity {play}
- [Terraform / OpenTofu](./apps/terraform.md) — Workspaces, variables and the built-in HTTP backend {boxes}
- [Shell / Bash](./apps/shell.md) — Run scripts from your repository {terminal}
- [PowerShell](./apps/powershell.md) — Windows automation from a Windows runner {terminal-square}
- [Python](./apps/python.md) — Scripts, dependencies and `requirements.txt` {file-code}

## Operate it

- [Runners](./operate/runners.md) — Execute tasks on separate, isolated servers {cpu}
- [Security](./operate/security.md) — Encryption, isolation, TLS and hardening {shield-check}
- [Authentication](./operate/authentication.md) — LDAP, Active Directory and OpenID Connect SSO {user-check}
- [High availability](./operate/high-availability.md) — Active-active nodes behind a load balancer {layers}
- [Logs and metrics](./operate/observability.md) — Audit trail, syslog, SIEM and Prometheus {activity}
- [Upgrading](./operate/upgrading.md) — Move to a new version safely {arrow-up-circle}
- [CLI](./operate/cli.md) — Setup, users, projects, vaults, migrations {square-terminal}
- [License activation](./operate/license.md) — Turn on Pro and Enterprise features {badge-check}

## Connect it to everything else

- [REST API](./integrate/api.md) — Tokens, launching tasks, Swagger and Postman {webhook}
- [Webhook integrations](./integrate/webhooks.md) — Trigger templates from GitHub, GitLab and anything that can POST {git-merge}
- [Pipelines](./integrate/pipelines.md) — Build and deploy tasks with artifact versions {workflow}
- [Notifications](./integrate/notifications.md) — Email, Telegram, Slack, Teams, Rocket.Chat, DingTalk, Gotify {bell}

<!-- /widget -->

## What Semaphore is not

It is not a hosted service. Every plan — Community, Pro and Enterprise — runs in your own environment, and your credentials never leave it. It is also not a replacement for Ansible or Terraform: it runs the tools you already use, and adds history, access control, scheduling and an API on top.

## Getting help

- **Something is broken:** start with [Troubleshooting](./troubleshooting.md).
- **Community:** [Discord](https://discord.gg/5R6k7hNGcH).
- **Bugs and feature requests:** [GitHub Issues](https://github.com/semaphoreui/semaphore/issues).
- **Source:** [semaphoreui/semaphore](https://github.com/semaphoreui/semaphore) — MIT licensed.

<!-- widget:cta -->

## Ready to run automation as a team?

Community is free forever. Pro adds isolated project runners, Vault-backed secrets, SSO and a support SLA.

[Start a 30-day Pro trial](https://portal.semaphoreui.com/start_trial) · [Compare plans](./pricing.md)

<!-- /widget -->
