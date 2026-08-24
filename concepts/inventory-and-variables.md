---
title: "Inventory and variable groups"
description: "Inventories define which hosts Semaphore acts on; variable groups define what it passes to them. Includes static and file inventories, dynamic inventories, and the TF_VAR_ convention for Terraform."
---

# Inventory and variable groups

Two project resources answer two different questions: **which hosts** (inventory) and **with what values** (variable groups).

## Inventory

An inventory is the list of hosts Ansible will run plays against, plus the variables attached to them. Semaphore stores it in one of two ways:

- **Static** — typed or pasted into the web form, versioned by Semaphore.
- **File** — a path to an inventory file. Absolute for a file on the server, relative for one in your repository, e.g. `inventory/linux-hosts.yaml`.

YAML, JSON and TOML are all accepted. The [Ansible inventory guide](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html) covers the format itself.

Every inventory carries at least one credential from the [Key Store](./key-store.md):

- a **user credential** — required; this is what Ansible logs in with. It must be an SSH key or a login/password key.
- a **sudo credential** — optional; used to escalate privileges on the target host.

### Creating one

1. In **Key Store**, confirm you have a key of type `ssh` or `login_password`.
2. Go to **Inventory → New Inventory**.
3. Name it and select the user credential, plus a sudo credential if your plays escalate.
4. Choose the type — file or static — and provide the path or the content.
5. **Create**.

Editing is the pencil icon; deleting requires that nothing else references it. If you are unsure what does, open the delete dialog: Semaphore lists the resources still using it, with links.

### Dynamic inventories

For infrastructure that changes underneath you, an inventory can be produced by a script rather than maintained by hand — pulling hosts from NetBox, Consul, or a cloud provider's API. Semaphore runs the inventory script the same way Ansible does. See the upstream guides for [NetBox](https://semaphoreui.com/docs/user-guide/inventory/netbox-dynamic-inventory) and [Consul](https://semaphoreui.com/docs/user-guide/inventory/consul-dynamic-inventory).

If you are on a Pro plan with a managed-node limit, prefer stable identifiers — hostnames or DNS names rather than ephemeral IPs — so the same machine is counted once across runs.

### Kerberos

Inventories can authenticate to Windows hosts through Kerberos rather than SSH. See [Kerberos inventory](https://semaphoreui.com/docs/user-guide/inventory/kerberos) upstream for the keytab and realm configuration.

## Variable groups

A variable group (called *Environment* in older versions) is a named set of variables stored as JSON, plus a separate **Secrets** tab for values that must be encrypted.

**Every task template requires a variable group**, even an empty one. If you have nothing to pass yet, create one containing `{}`.

### Creating one

1. Go to **Variable Groups → New Variable Group**.
2. Name it and paste valid JSON.
3. Add sensitive values on the **Secrets** tab instead of the JSON body — they are encrypted at rest and injected only at run time.

As with inventories, a variable group in use cannot be deleted until its consumers are gone.

### How variables reach your automation

| App | How the values arrive |
|---|---|
| Ansible | As extra variables available to the playbook |
| Terraform / OpenTofu | As environment variables — prefix names with `TF_VAR_` |
| Bash | Environment variables, read as `$VARIABLE_NAME` |
| PowerShell | Environment variables, read as `$env:VARIABLE_NAME` |
| Python | Environment variables, read via `os.environ` |

### Terraform: the `TF_VAR_` convention

Terraform only picks up an environment variable as an input variable if it is prefixed with `TF_VAR_`. A variable group key `TF_VAR_region` becomes `var.region` in your code.

A worked example — passing a Hetzner Cloud token to OpenTofu without it ever appearing in a log:

1. **Variable Groups → New Group**.
2. Open the **Secrets** tab.
3. Add `TF_VAR_hcloud_token` with the token as the hidden value.
4. Save.

```hcl
terraform {
  required_providers {
    hcloud = {
      source  = "hetznercloud/hcloud"
      version = "~> 1.45"
    }
  }
}

variable "hcloud_token" {
  type        = string
  description = "Hetzner Cloud API token"
  sensitive   = true
}

provider "hcloud" {
  token = var.hcloud_token
}

resource "hcloud_server" "webserver" {
  name        = "webserver"
  image       = "ubuntu-24.04"
  server_type = "cpx11"
  location    = "ash"
  ssh_keys    = ["mysshkey"]

  public_net {
    ipv4_enabled = true
    ipv6_enabled = true
  }
}
```

`sensitive = true` keeps the token out of Terraform's own output; storing it in the Secrets tab keeps it out of Semaphore's.

## Next steps

- [Key Store](./key-store.md) — where the credentials an inventory needs come from.
- [Ansible](../apps/ansible.md) — how inventory, vaults and CLI options combine in a template.
- [Terraform / OpenTofu](../apps/terraform.md) — workspaces and the HTTP backend.
