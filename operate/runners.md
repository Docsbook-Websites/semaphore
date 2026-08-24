---
title: "Runners"
description: "Runners execute Semaphore tasks on separate servers — for isolation, for scale, and for reaching networks the control plane cannot. Set up the server, register a runner, run it, and route work with tags."
---

# Runners

A runner executes tasks on a machine that is not the Semaphore server. The model is the same one GitLab and GitHub Actions use:

1. You start a runner on a separate server, giving it the Semaphore address and a token.
2. The runner connects out to Semaphore and says it is ready for work.
3. When a task appears, Semaphore sends everything needed; the runner clones the repository and runs Ansible, Terraform, PowerShell or whatever the template specifies.
4. Results stream back.

To the person pressing **Run**, nothing looks different.

## Why bother

With no runners defined, the Semaphore server executes everything itself, in its own context, with access to its own filesystem. That works, and for a small install it is the right answer. Runners buy you three things:

- **Isolation.** A runner can live in a closed subnet or a locked-down container. A compromised playbook reaches the runner's environment, not your control plane, and not your database encryption key.
- **Reach.** The runner dials out to Semaphore, so Semaphore never needs a route *into* the network being automated. That is often the only way to automate a segmented environment at all.
- **Distribution.** Start several runners and work spreads across them instead of queuing on one box. It is also how you provide different execution environments — a Windows host for PowerShell, an image with cloud CLIs for Terraform.

## Set up the server

Add to the Semaphore server configuration:

```json
{
  "use_remote_runner": true,
  "runner_registration_token": "long string of random characters"
}
```

or with environment variables:

```bash
SEMAPHORE_USE_REMOTE_RUNNER=True
SEMAPHORE_RUNNER_REGISTRATION_TOKEN=long_string_of_random_characters
```

The registration token is a shared secret that lets a machine become a runner. Make it long and random, and treat it like a credential.

## Set up a runner

On the runner host, using the same `semaphore` binary:

```bash
semaphore runner setup --config /path/to/your/config/file.json
```

This writes a runner configuration file. Before running it, decide how the runner will register.

## Register the runner

There are two ways.

### Via the web UI or API

Add the runner in the Semaphore interface and it gives you a runner token. When `semaphore runner setup` asks whether you have a token, answer yes and paste it.

![Adding a runner in the web UI](https://github.com/user-attachments/assets/8b0f7890-5767-4139-932d-3e39c217fd57)

### Via the CLI

This path uses the `runner_registration_token` you set on the server. Answer **No** when setup asks about a runner token, then:

```bash
semaphore runner register --config /path/to/your/config/file.json
```

To avoid putting the token on the command line:

```bash
echo REGISTRATION_TOKEN | semaphore runner register \
  --stdin-registration-token \
  --config /path/to/your/config/file.json
```

## The runner configuration file

`semaphore runner setup` produces something like:

```json
{
  "tmp_path": "/tmp/semaphore",
  "web_host": "https://semaphore_server_host",

  "runner": {
    "token": "your runner's token"
  }
}
```

You can edit this by hand; there is no need to re-run setup. Alongside `token`, a `token_file` can point at a file the runner writes its token to, and the `runner` block also accepts options such as `max_parallel_tasks`, `webhook` and `one_off`. General options like `git_client` and `ssh_config_path` apply here too.

Re-registering with `semaphore runner register` overwrites the token in the configured file.

## Run it

```bash
semaphore runner start --config /path/to/your/config/file.json
```

The runner is now available for tasks. In production, run it under systemd — the [service unit from the install guide](../get-started/install.md#run-as-a-systemd-service) works with `runner start` substituted for `server`.

## Runner tags (Pro)

Project runners can carry one or more tags, and a template can require a tag. Tasks then run only on matching runners.

This is how you express "this template must run on Windows", "this one must run inside the PCI subnet", or "this one needs the image with `terraform` installed". Configure tags when adding the runner in the project UI, and set the required tag in template settings.

## Deregistration

From the web interface:

![Removing a runner](https://github.com/user-attachments/assets/431291eb-8f48-42c1-b56e-87fc8e9ba040)

Or from the CLI:

```bash
semaphore runner unregister --config /path/to/your/config/file.json
```

## Security

Traffic between server and runner is protected with asymmetric encryption: the server encrypts with a public key, the runner decrypts with its private key. The keypair is generated automatically when the runner registers.

> Use HTTPS between the server and its runners — always if they are not on the same private network. The payload encryption protects task data; it does not replace transport security for the rest of the exchange.

## Next steps

- [Architecture](../concepts/architecture.md) — where runners sit in the whole picture
- [High availability](./high-availability.md) — scaling the control plane too
- [Troubleshooting](../troubleshooting.md#runner-prints-error-404) — registration failures
