---
title: "Upgrading"
description: "Upgrade Semaphore safely: package manager, binary, Docker and Helm paths, plus what to check before and after — database migrations, configuration changes and runner versions."
---

# Upgrading

Semaphore releases often. Staying current is how you get security fixes, so the upgrade path is worth having as a routine rather than an event.

## Before you upgrade

- **Read the release notes.** The [changelog on GitHub](https://github.com/semaphoreui/semaphore/releases) lists breaking changes and new configuration options.
- **Back up the database.** It holds everything: projects, templates, history, users and encrypted credentials.
- **Back up `config.json`** — and confirm you have `access_key_encryption` stored somewhere separate. Without it, a restored database is unreadable.

Semaphore applies database migrations automatically on start. On a large history that can take a moment; do not kill the process mid-migration.

## Package manager

Download the package for your distribution from the [releases page](https://github.com/semaphoreui/semaphore/releases) and install it over the existing one.

### Debian / Ubuntu

```bash
wget https://github.com/semaphoreui/semaphore/releases/download/v2.17.15/semaphore_2.17.15_linux_amd64.deb
sudo dpkg -i semaphore_2.17.15_linux_amd64.deb
sudo systemctl restart semaphore
```

For ARM64, replace `amd64` with `arm64`.

### CentOS / RHEL

```bash
wget https://github.com/semaphoreui/semaphore/releases/download/v2.17.15/semaphore_2.17.15_linux_amd64.rpm
sudo yum install semaphore_2.17.15_linux_amd64.rpm
sudo systemctl restart semaphore
```

## Binary

Download the archive, unpack it over the old binary, and restart the service.

### Linux

```bash
wget https://github.com/semaphoreui/semaphore/releases/download/v2.17.15/semaphore_2.17.15_linux_amd64.tar.gz
tar xf semaphore_2.17.15_linux_amd64.tar.gz
```

### Windows

```powershell
Invoke-WebRequest `
  -Uri "https://github.com/semaphoreui/semaphore/releases/download/v2.17.15/semaphore_2.17.15_windows_amd64.zip" `
  -OutFile semaphore.zip

Expand-Archive -Path semaphore.zip -DestinationPath ./
```

## Docker

Pull the new image and recreate the container:

```bash
docker compose pull
docker compose up -d
```

Pinning `semaphoreui/semaphore:latest` makes this easy but makes the version implicit. In production, pin an explicit tag and bump it deliberately.

## Kubernetes

Update the chart version or the image tag in your values, then:

```bash
helm upgrade semaphore semaphoreui/semaphore -f values.yaml
```

With multiple replicas, see [high availability](./high-availability.md) — nodes can be rolled one at a time for a zero-downtime upgrade.

## Runners

Runners use the same binary as the server. Upgrade them too, and keep them close to the server version — a runner several versions behind may not understand the payloads it receives. The usual order is server first, then runners.

## Licensing

You do **not** need to reinstall or switch builds to use Pro or Enterprise features. The version you are running activates with a license key. See [License activation](./license.md).

## After you upgrade

1. Check the server log for migration errors — `journalctl -u semaphore.service` or `docker logs`.
2. Sign in and confirm authentication still works, especially after an LDAP or OIDC-related release.
3. Run one small template end to end.
4. Confirm runners have reconnected.

## Next steps

- [Configure the server](../get-started/configuration.md) — new options land here
- [Security](./security.md) — why staying current matters
- [Troubleshooting](../troubleshooting.md)
