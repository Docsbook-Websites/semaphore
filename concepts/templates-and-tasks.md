---
title: "Task templates and tasks"
description: "A task template is a reusable definition of a run; a task is one execution of it. Learn the task, build and deploy types, prompts and survey variables, parallelism, schedules and log retention."
---

# Task templates and tasks

Semaphore separates the definition of a run from the run itself.

- A **task template** says *what to run and how*: which repository, which playbook or script, which inventory, which variables, which extra CLI flags.
- A **task** is one execution of a template, with its author, its parameters and its log.

The split is what makes automation shareable. Someone with deep knowledge writes the template once; everyone else presses a button.

## App types

Every template has an app type, which decides what Semaphore actually executes:

<!-- widget:cards -->

- [Ansible](../apps/ansible.md) — Playbooks, with inventory, vaults, tags and limits {play}
- [Terraform / OpenTofu](../apps/terraform.md) — Infrastructure code, workspaces and state {boxes}
- [Shell / Bash](../apps/shell.md) — Scripts from your repository {terminal}
- [PowerShell](../apps/powershell.md) — Windows automation {terminal-square}
- [Python](../apps/python.md) — Python scripts and their dependencies {file-code}

<!-- /widget -->

## Template types: task, build, deploy

Beyond the app, an Ansible template can be one of three types.

**Task** — just runs the playbook with the configured parameters. This is what most templates are.

**Build** — produces an artifact. Each run gets an auto-incremented version, exposed to the playbook as `semaphore_vars.task_details.target_version`. Semaphore does not store artifacts for you; it versions the runs, and your playbook decides what to build and where to put it.

**Deploy** — ships an artifact produced by a build template. Each deploy template is associated with a build template, and the version being deployed arrives as `semaphore_vars.task_details.incoming_version`. When starting a deploy you can choose which build version to ship; the latest is the default.

Together these give you a minimal pipeline without a pipeline engine. See [Pipelines](../integrate/pipelines.md) for the variables Semaphore passes in, including the environment-variable equivalents for Bash, PowerShell and Python templates.

## Parameters at run time

Templates are reusable because not everything has to be fixed in advance.

**Prompts** let a template ask for a value when a task starts — a branch, a tag, a limit, an inventory override. Enable the relevant prompt in template settings; without it, a value passed via API or schedule is ignored.

> If you launch a template through the API and pass `--limit`, you must enable the *Ansible prompts: Limit* option in the template. Otherwise the limit is silently dropped. A prompted field does **not** block an API-triggered run — it just becomes settable.

**Survey variables** define a form of named inputs that become variables inside the run, which is how you turn a template into something a non-expert can safely operate.

## Parallel runs

By default, tasks from the same template execute sequentially — a second run waits for the first. Enable **Allow parallel tasks** in template settings when concurrent runs are safe (read-only checks, per-host jobs) and leave it off when they are not (anything holding a lock or touching shared state).

## Schedules

A schedule runs a template on a cron expression or a preset interval, without anyone present.

```
┌─────── minute (0-59)
│ ┌────── hour (0-23)
│ │ ┌───── day of month (1-31)
│ │ │ ┌───── month (1-12)
│ │ │ │ ┌───── day of week (0-6, Sunday = 0)
│ │ │ │ │
* * * * *
```

| Expression | Meaning |
|---|---|
| `*/15 * * * *` | Every 15 minutes |
| `0 2 * * *` | 02:00 every day |
| `0 0 * * 0` | Midnight on Sundays |
| `0 9 1 * *` | 09:00 on the first of each month |

Schedules run in **UTC** unless you configure otherwise:

```json
{
  "schedule": {
    "timezone": "America/New_York"
  }
}
```

or `SEMAPHORE_SCHEDULE_TIMEZONE="America/New_York"`. Valid values come from the [IANA time zone database](https://www.iana.org/time-zones). Restart Semaphore after changing it.

Schedules can also supply parameters: enable prompts on the template, then set the values in the schedule so each run gets the branch, variables or flags it needs.

### What people schedule

- **Maintenance** — weekly package updates during off-hours.
- **Backups** — daily, weekly and monthly, with different retention.
- **Compliance** — recurring scans that collect reports off every host.
- **Cost control** — build a dev environment in the morning, tear it down at night.

A few habits that pay off: name schedules so the name says function *and* timing (`Weekly-Backup-Sunday-2AM`), avoid stacking heavy jobs at the same minute, and test with a short interval before committing to a monthly one.

## Tasks and their logs

While a task runs, its log streams live; the **RAW LOG** action opens the unprocessed output. Finished tasks stay in the template's history and in the project dashboard.

**Log retention is unlimited by default.** On a busy instance that grows the database indefinitely. Cap it:

```json
{ "max_tasks_per_template": 50 }
```

or `SEMAPHORE_MAX_TASKS_PER_TEMPLATE=50`.

For longer-term retention, export logs to files or a SIEM rather than keeping them in the database — see [Logs and metrics](../operate/observability.md).

## Next steps

- [Inventory and variables](./inventory-and-variables.md) — what a template targets and what it passes.
- [Pipelines](../integrate/pipelines.md) — build and deploy in practice.
- [REST API](../integrate/api.md) — launching templates from code.
