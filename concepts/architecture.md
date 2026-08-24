---
title: "Architecture"
description: "How Semaphore is put together: the server, the database, where your automation actually executes, and what changes when you add runners or a second node."
---

# Architecture

Semaphore is deliberately small. A single Go binary serves the web UI and the REST API, stores state in a relational database, and executes automation — either itself, or on a separate machine.

## The pieces

**Server.** One process, `semaphore server`. It serves the UI and the API, schedules tasks, streams logs over WebSocket, and holds the encryption keys for the Key Store. On Linux, Windows and macOS.

**Database.** SQLite, MySQL or PostgreSQL. It is the single source of truth: projects, templates, inventories, schedules, task history, users, roles and encrypted credentials all live there. SQLite is fine for one instance; MySQL or PostgreSQL is required if you ever want more than one.

**Execution environment.** Whatever runs your playbook: the tools themselves (Ansible, Terraform/OpenTofu, PowerShell, Python, Bash), their dependencies, and the clone of your repository. This is the part that varies most between deployments.

**Runners (optional).** Separate processes, usually on separate machines, that accept work from the server and execute it there instead.

## Where automation executes

This is the decision that shapes every Semaphore deployment.

**Without runners**, the server is also the executor. Tasks run in the context of the Semaphore process, with access to its filesystem. It is the simplest thing that works, and it is what you get out of the box.

**With runners**, the server never executes anything. It hands the task to a runner, which clones the repository, runs the tool, and streams results back. From a user's point of view nothing changes — the same button, the same log. What changes is the blast radius:

- A runner can sit inside a closed subnet, or an isolated container, with no path back to the control plane.
- Work spreads across several machines instead of contending on one.
- The environment a playbook needs (a specific Ansible version, cloud CLI tools, a Windows host for PowerShell) lives on the runner, not on your control node.

The protocol between them is one-directional in the way that matters: the runner connects out to the server, so the server never needs a route into the runner's network. Data in flight is protected by asymmetric encryption, with the keypair generated at registration time.

[Set up runners →](../operate/runners.md)

## The lifecycle of a task

1. Something starts a run — a person pressing **Run**, a [schedule](./templates-and-tasks.md#schedules) firing, a [webhook integration](../integrate/webhooks.md), or an [API call](../integrate/api.md).
2. Semaphore records the task and its parameters in the database.
3. An executor picks it up — the server itself, or a runner (matched by tag, on Pro).
4. The executor clones the repository at the configured branch, installs `requirements.yml` roles and collections if present, and injects variables and secrets **at runtime only**.
5. The tool runs. Output streams back and is persisted as the task log.
6. The task ends with a status — success, error or stopped — and the run stays in history with its author, its parameters and its full log.

## Scaling out

Two axes, and they are independent:

| You need more… | Add… |
|---|---|
| Automation throughput | [Runners](../operate/runners.md) — available on every plan |
| Control-plane availability | [HA nodes](../operate/high-availability.md) — Enterprise |

In HA mode several identical server nodes run behind a load balancer, sharing one PostgreSQL or MySQL database plus a Redis instance for distributed locks, queue state and pub/sub. All nodes are equal; there is no primary. Adding runners on top scales execution capacity separately from the web and API layer.

## Security boundaries worth knowing

- **Credentials** are encrypted at rest with `access_key_encryption` and only decrypted into a task's environment at the moment it runs.
- **Untrusted playbooks** can be isolated further with container execution, a chroot jail, or a dedicated non-root task user.
- **Runners** move execution out of the control plane entirely — the strongest boundary Semaphore offers.

[Security in detail →](../operate/security.md)

## Next steps

- [Projects](./projects.md) — how work is separated and who can see it.
- [Templates and tasks](./templates-and-tasks.md) — the run definition and the run.
- [Key Store](./key-store.md) — where credentials live.
