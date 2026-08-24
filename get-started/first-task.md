---
title: "Run your first task"
description: "From a fresh Semaphore install to a playbook running on real hosts: create a project, add a key, connect a repository, define an inventory and variable group, and run a task template."
---

# Run your first task

This is the shortest path from a fresh install to automation running on real hosts. Every step below has a page of its own if you want the detail; follow them in order and you will have a working task in about ten minutes.

<!-- widget:stepper -->

## Sign in and create a project

Open `http://your-server:3000` and sign in with the admin account created during [installation](./install.md).

A **project** is the unit of separation in Semaphore — one project per team, environment or application. Everything else you create lives inside one. Create the first one from the projects menu.

[More about projects →](../concepts/projects.md)

## Add a credential to the Key Store

Before Semaphore can clone a repository or log into a host, it needs a credential. Go to **Key Store** and create one:

- **SSH** — a private key, for both Git over SSH and for logging into target hosts.
- **Login with password** — a username plus password or token. Leave the login field empty to use it as a bare personal access token.
- **None** — a placeholder for public repositories that need no authentication.

Keys are encrypted at rest with the key you set in `access_key_encryption`.

[More about the Key Store →](../concepts/key-store.md)

## Connect a repository

Go to **Repositories** and add the repository holding your playbooks, Terraform code or scripts. Semaphore accepts:

- a local path — `/path/to/the/repo`
- a local Git repository — `file://`
- a remote repository over HTTPS or SSH — `https://`, `ssh://`

Set the branch, and select the Access Key you just created. Every task template needs a repository.

[More about repositories →](../concepts/projects.md#repositories)

## Define an inventory

For Ansible runs, create an **Inventory** — the list of hosts to act on, plus the credential Ansible uses to log into them.

- **Static** — paste the inventory into the web form.
- **File** — a path to an inventory file inside your repository, e.g. `inventory/linux-hosts.yaml`.

Each inventory needs at least a user credential; add a sudo credential if your plays escalate privileges.

[More about inventory →](../concepts/inventory-and-variables.md)

## Create a variable group

Every task template requires a **Variable Group**, even an empty one. Create one containing `{}` to start, and come back later when you have real variables and secrets to pass.

[More about variable groups →](../concepts/inventory-and-variables.md#variable-groups)

## Create a task template and run it

Go to **Task Templates** → **New Template** and pick the app you are running — Ansible, Terraform/OpenTofu, Bash, PowerShell or Python. Fill in the repository, the path to the playbook or script, the inventory and the variable group, then **Create**.

Press **Run**. The task log streams live; when it finishes, the run is kept in history with who started it and what it did.

[Per-app guides →](../apps/ansible.md)

## Automate it

Once the run is reliable, stop pressing the button:

- **Schedule it** — cron syntax or a preset interval.
- **Trigger it from Git** — a [webhook integration](../integrate/webhooks.md) fires the template on push.
- **Call it from code** — the [REST API](../integrate/api.md) launches tasks with a bearer token.
- **Give it to your team** — add members with the right [role](../concepts/projects.md#teams-and-roles) so they can run it without being able to change it.

<!-- /widget -->

## A minimal first playbook

If you do not yet have a repository to point at, this is enough to prove the whole path end to end:

```yaml
---
- hosts: all
  gather_facts: true
  tasks:
    - name: Report who and where
      ansible.builtin.debug:
        msg: "{{ inventory_hostname }} reached by {{ ansible_user }}"
```

If you target `localhost` and Semaphore is running in a container, fact gathering fails by design — see [Troubleshooting](../troubleshooting.md#gathering-facts-fails-for-localhost).

## What to read next

- [Architecture](../concepts/architecture.md) — what actually ran your task, and where.
- [Templates and tasks](../concepts/templates-and-tasks.md) — task, build and deploy types, prompts and survey variables.
- [Runners](../operate/runners.md) — move execution off the Semaphore server before it matters.
