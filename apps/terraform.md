---
title: "Terraform / OpenTofu"
description: "Run Terraform and OpenTofu from Semaphore: create a template, pass variables with the TF_VAR_ prefix, use workspaces, the destroy and migrate-state flags, and the built-in HTTP backend on Pro."
---

# Terraform / OpenTofu

Semaphore runs Terraform and OpenTofu code the same way it runs playbooks: a template points at a repository, a directory of `.tf` files and a variable group, and each run is recorded with its plan output and its author.

## Create a template

1. Go to **Task Templates** and click **New Template**.
2. Select **Terraform** as the app type.
3. Configure the template and click **Create**.
4. Click **Run**.

Semaphore runs `terraform init` automatically before each run, so a fresh clone works without extra steps.

## Passing variables

Variables from the selected **Variable Groups** are injected as environment variables. Terraform only reads an environment variable as an input variable when it is prefixed with `TF_VAR_`:

| Variable group key | Terraform variable |
|---|---|
| `TF_VAR_region` | `var.region` |
| `TF_VAR_instance_type` | `var.instance_type` |

Put anything sensitive — cloud tokens, credentials — on the **Secrets** tab of the variable group rather than in the JSON body. Those values are encrypted at rest and only exist in the process environment while the task runs.

A full worked example, including marking the variable `sensitive` on the Terraform side, is in [Inventory and variables](../concepts/inventory-and-variables.md#terraform-the-tf_var_-convention).

## Workspaces

Semaphore supports Terraform/OpenTofu workspaces natively, so one template can manage several environments from the same code. Creating and switching workspaces, and using SSH keys for private modules, are covered in the [upstream workspaces guide](https://semaphoreui.com/docs/user-guide/apps/terraform/workspaces).

## State

By default, state is handled by whatever backend your Terraform code declares — local, S3, GCS, Azure Blob, and so on. Semaphore does not interfere.

### Built-in HTTP backend (Pro)

On Pro, Semaphore ships an HTTP state backend and a template can **override the backend** to use it — without editing your Terraform code. That gives you state storage, locking and history inside Semaphore for teams that would otherwise have to stand up a state bucket first. See the [upstream HTTP backend guide](https://semaphoreui.com/docs/user-guide/apps/terraform/states).

## Destroy and state migration

The run dialog exposes two toggles:

- **`-destroy`** — tear down what the configuration manages.
- **`-migrate-state`** — move state when the backend changes.

Both are per-run, not per-template. Consider restricting who can start this template: on Enterprise, a [custom role](../concepts/projects.md#extended-rbac-enterprise) can grant *run* on a plan template without granting it on the destroy-capable one.

## Next steps

- [Inventory and variables](../concepts/inventory-and-variables.md) — the `TF_VAR_` convention in full
- [Key Store](../concepts/key-store.md) — storing cloud credentials
- [Runners](../operate/runners.md) — running Terraform from inside the network it manages
