---
title: "Shell / Bash"
description: "Run shell scripts from your repository as Semaphore tasks: create a Bash template, pass variables as environment variables, and understand how exit codes map to task status."
---

# Shell / Bash

Not everything is a playbook. Semaphore runs shell scripts with `/bin/bash`, which is the shortest way to bring an existing operational script under history, access control and scheduling.

## Create a Bash template

1. Go to **Task Templates** and click **New Template**.
2. Select **Bash** as the app type.
3. Configure it:

| Field | Description |
|---|---|
| **Name** | A descriptive name for the template |
| **Repository** | The repository containing your script |
| **Playbook / Script** | Relative path to the script, e.g. `scripts/deploy.sh` |
| **Variable Groups** | Variable groups whose values are injected as environment variables |

4. **Create**, then **Run**.

## Passing variables

Values from the selected variable groups arrive as environment variables:

```bash
#!/bin/bash
set -euo pipefail

echo "Deploying to $TARGET_HOST"
```

Secrets from the **Secrets** tab of a variable group arrive the same way, but are never written to the task log.

If the template is a **build** or **deploy** type, Semaphore also supplies the pipeline variables — `SEMAPHORE_TASK_DETAILS_TYPE`, `SEMAPHORE_TASK_DETAILS_TARGET_VERSION`, and the rest. See [Pipelines](../integrate/pipelines.md).

## Rules of the road

- Make the script executable (`chmod +x`) or give it a valid shebang (`#!/bin/bash`).
- Scripts run **non-interactively**. Anything that waits for input hangs the task until it is stopped.
- Exit code `0` means success. Any non-zero exit marks the task failed — so `set -e` is usually what you want, not a nicety.
- To act on remote hosts, use [Ansible](./ansible.md) rather than wrapping `ssh` in a script: you get inventory, credentials and per-host reporting for free.

## Where it runs

The script executes wherever the task executes — on the Semaphore server by default, or on the [runner](../operate/runners.md) that picks the task up. Whatever the script calls (`aws`, `kubectl`, `psql`) has to be installed there.

## Next steps

- [Python](./python.md) — when the script outgrows bash
- [Pipelines](../integrate/pipelines.md) — build and deploy variables
- [Runners](../operate/runners.md) — choosing where scripts execute
