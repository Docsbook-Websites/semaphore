---
title: "Pipelines: build and deploy"
description: "Semaphore's build and deploy task types give you artifact versioning without a separate pipeline engine. Learn the semaphore_vars variables and their environment-variable equivalents for Bash, PowerShell and Python."
---

# Pipelines: build and deploy

Semaphore supports simple pipelines through two template types: **build** and **deploy**. Between them you get artifact versioning, a link from a deployment back to the build it shipped, and the ability to redeploy any previous version — without adding a pipeline engine to your stack.

The mechanism is one variable. Semaphore passes `semaphore_vars` into every Ansible playbook it runs, and your playbook decides what to do with it.

## `semaphore_vars` in Ansible

For a **build** task:

```yaml
semaphore_vars:
    task_details:
        type: build
        username: user123
        message: New version of some feature
        target_version: 1.5.33
```

For a **deploy** task:

```yaml
semaphore_vars:
    task_details:
        type: deploy
        username: user123
        message: Deploy new feature to servers
        incoming_version: 1.5.33
```

So a playbook can branch on the task type, name its artifact after `target_version`, and know who started the run and why.

## The same values in Bash, PowerShell and Python

For non-Ansible templates, Semaphore exposes `task_details` as environment variables:

| `task_details` field | Environment variable | Notes |
|---|---|---|
| `type` | `SEMAPHORE_TASK_DETAILS_TYPE` | `build` or `deploy` |
| `username` | `SEMAPHORE_TASK_DETAILS_USERNAME` | Who started the task |
| `message` | `SEMAPHORE_TASK_DETAILS_MESSAGE` | The task message |
| `target_version` | `SEMAPHORE_TASK_DETAILS_TARGET_VERSION` | Present on `build` tasks |
| `incoming_version` | `SEMAPHORE_TASK_DETAILS_INCOMING_VERSION` | Present on `deploy` tasks |

**Bash**

```bash
echo "$SEMAPHORE_TASK_DETAILS_TYPE"
echo "$SEMAPHORE_TASK_DETAILS_TARGET_VERSION"
```

**PowerShell**

```powershell
$env:SEMAPHORE_TASK_DETAILS_TYPE
$env:SEMAPHORE_TASK_DETAILS_INCOMING_VERSION
```

**Python**

```python
import os

task_type        = os.getenv("SEMAPHORE_TASK_DETAILS_TYPE")
target_version   = os.getenv("SEMAPHORE_TASK_DETAILS_TARGET_VERSION")
incoming_version = os.getenv("SEMAPHORE_TASK_DETAILS_INCOMING_VERSION")
```

## Build

A build task creates an [artifact](https://en.wikipedia.org/wiki/Artifact_\(software_development\)). Each run gets an auto-generated version; read `semaphore_vars.task_details.target_version` to know which one you are producing.

A typical build role:

1. Fetch the application source from Git.
2. Compile it.
3. Pack the binary as `app-{{ semaphore_vars.task_details.target_version }}.tar.gz`.
4. Upload it to an S3 bucket.

## Deploy

A deploy task ships an artifact to its destination servers. Each deploy template is associated with a build template, and `semaphore_vars.task_details.incoming_version` tells you which version to ship — the latest by default, or any earlier one you pick when starting the task.

A typical deploy role:

1. Download `app-{{ semaphore_vars.task_details.incoming_version }}.tar.gz` from the bucket to the target servers.
2. Unpack it into the destination directory.
3. Create or update configuration files.
4. Restart the service.

Because the version is an input rather than "whatever is newest", rolling back is just a deploy of an earlier version — no special path, and no special code.

## What Semaphore does not do

Semaphore **does not store artifacts**. It versions runs and passes the version to your automation; where the artifact lives — S3, a registry, an internal repository — is your decision. That is a deliberate boundary: it keeps Semaphore out of the storage business and lets you keep whatever artifact store you already have.

## Next steps

- [Templates and tasks](../concepts/templates-and-tasks.md#template-types-task-build-deploy)
- [Ansible](../apps/ansible.md) — build and deploy template settings
- [Webhook integrations](./webhooks.md) — starting a build on push
