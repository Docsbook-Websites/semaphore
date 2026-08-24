---
title: "PowerShell"
description: "Run PowerShell scripts from Semaphore on Windows hosts or a Windows runner: create a template, read variables from $env:, and understand the execution requirements."
---

# PowerShell

Semaphore runs PowerShell scripts, which makes it a practical control plane for Windows estates — the same history, roles, scheduling and API as everything else.

## Create a PowerShell template

1. Go to **Task Templates** and click **New Template**.
2. Select **PowerShell** as the app type.
3. Configure it:

| Field | Description |
|---|---|
| **Name** | A descriptive name for the template |
| **Repository** | The repository containing your `.ps1` script |
| **Playbook / Script** | Relative path to the script, e.g. `scripts/deploy.ps1` |
| **Variable Groups** | Variable groups whose values are injected as environment variables |

4. **Create**, then **Run**.

## Passing variables

Variable group values are set in the environment before the script starts:

```powershell
Write-Host "Deploying to $env:TARGET_HOST"
```

Secrets behave identically but never appear in the task log.

## Where it runs

A PowerShell template needs one of:

- a **Windows runner** — a Semaphore runner deployed on a Windows host, or
- a Semaphore **server running on Windows**.

The runner route is the usual one: keep the control plane on Linux, put a runner in the Windows domain, and route PowerShell templates to it. On Pro you can [tag](../operate/runners.md#runner-tags-pro) the Windows runner and require that tag on the template, so these tasks can only land somewhere that can actually run them.

## Rules of the road

- Scripts run **non-interactively** — no prompts, no `Read-Host`.
- Exit code `0` is success; anything else fails the task. Remember that a PowerShell error does not always set a non-zero exit code — use `$ErrorActionPreference = 'Stop'` and explicit `exit` codes when it matters.

## Next steps

- [Runners](../operate/runners.md) — deploying a Windows runner
- [Key Store](../concepts/key-store.md) — credentials for domain access
- [Pipelines](../integrate/pipelines.md) — build and deploy variables in PowerShell
