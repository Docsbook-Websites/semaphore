---
title: "Python"
description: "Run Python scripts from Semaphore: create a template, read variables from os.environ, and control which Python version and dependencies the execution environment provides."
---

# Python

Semaphore runs Python scripts directly — useful when the automation is a real program rather than a sequence of commands, and when the libraries you need already exist in Python.

## Create a Python template

1. Go to **Task Templates** and click **New Template**.
2. Select **Python** as the app type.
3. Configure it:

| Field | Description |
|---|---|
| **Name** | A descriptive name for the template |
| **Repository** | The repository containing your `.py` script |
| **Playbook / Script** | Relative path to the script, e.g. `scripts/deploy.py` |
| **Variable Groups** | Variable groups whose values are injected as environment variables |

4. **Create**, then **Run**.

## Passing variables

Variable group values arrive in the environment:

```python
import os

target = os.environ.get("TARGET_HOST")
print(f"Deploying to {target}")
```

Secrets from the **Secrets** tab work the same way and stay out of the task log.

## Python version and dependencies

Semaphore uses whichever `python3` is on `PATH` in the execution environment. How you control that depends on your install:

| Install | How to control it |
|---|---|
| Package or binary | Install the required `python3` on the host |
| Docker | Build a custom image with the version you need |
| Docker, extra packages only | Mount a `requirements.txt` at `/etc/semaphore/requirements.txt` — Semaphore runs `pip3 install --upgrade -r` on container start |

The `requirements.txt` route is shared with Ansible collections that need Python libraries; see [Install](../get-started/install.md#extra-python-packages-for-your-playbooks).

If different templates need conflicting dependency sets, give them separate [runners](../operate/runners.md) rather than fighting one shared environment.

## Rules of the road

- Scripts run **non-interactively** — nothing that waits on `input()`.
- Exit code `0` is success; anything else fails the task. An uncaught exception exits non-zero, which is the behaviour you want.
- Write progress to stdout: it becomes the streamed task log, and it is what someone reads at 3am.

## Next steps

- [Shell / Bash](./shell.md) — for the smaller jobs
- [Runners](../operate/runners.md) — isolating dependency sets
- [REST API](../integrate/api.md) — calling Semaphore from Python instead of the other way round
