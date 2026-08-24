---
title: "Install Semaphore"
description: "Install Semaphore UI with Docker Compose, a distribution package, a precompiled binary, or the official Helm chart — and pick the database that fits your deployment."
---

# Install Semaphore

Semaphore ships as a single Go binary plus a web UI. Pick the installation method that matches how you already run software; every one of them ends at the same place — Semaphore listening on port `3000`.

| Method | Best for | Database |
|---|---|---|
| [Docker / Docker Compose](#docker-compose) | Fast setup, sandboxed execution, infrastructure as code | MySQL, PostgreSQL or SQLite |
| [Package manager](#package-manager) | Linux servers managed with `apt` / `dnf`, systemd integration | Any |
| [Binary file](#binary-file) | Manual installs, Windows, custom workflows | Any |
| [Kubernetes (Helm)](#kubernetes-helm) | Production clusters, declarative upgrades | External MySQL/PostgreSQL |

Semaphore supports **SQLite**, **MySQL** and **PostgreSQL**. SQLite is fine for a single instance; use MySQL or PostgreSQL if you plan to scale out or run [high availability](../operate/high-availability.md).

## Docker Compose

Create a `docker-compose.yml`:

```yaml
services:
  mysql:
    restart: unless-stopped
    image: mysql:8.0
    hostname: mysql
    volumes:
      - semaphore-mysql:/var/lib/mysql
    environment:
      MYSQL_RANDOM_ROOT_PASSWORD: 'yes'
      MYSQL_DATABASE: semaphore
      MYSQL_USER: semaphore
      MYSQL_PASSWORD: semaphore

  semaphore:
    restart: unless-stopped
    ports:
      - 3000:3000
    image: semaphoreui/semaphore:latest
    environment:
      SEMAPHORE_DB_USER: semaphore
      SEMAPHORE_DB_PASS: semaphore
      SEMAPHORE_DB_HOST: mysql
      SEMAPHORE_DB_PORT: 3306
      SEMAPHORE_DB_DIALECT: mysql
      SEMAPHORE_DB: semaphore
      SEMAPHORE_PLAYBOOK_PATH: /tmp/semaphore/
      SEMAPHORE_ADMIN_PASSWORD: changeme
      SEMAPHORE_ADMIN_NAME: admin
      SEMAPHORE_ADMIN_EMAIL: admin@localhost
      SEMAPHORE_ADMIN: admin
      SEMAPHORE_ACCESS_KEY_ENCRYPTION: gs72mPntFATGJs9qK0pQ0rKtfidlexiMjYCH9gWKhTU=
      TZ: UTC
    depends_on:
      - mysql

volumes:
  semaphore-mysql:
```

Three values must be your own before this reaches anything real:

- `MYSQL_PASSWORD` / `SEMAPHORE_DB_PASS` — the database password.
- `SEMAPHORE_ADMIN_PASSWORD` — the first admin user's password.
- `SEMAPHORE_ACCESS_KEY_ENCRYPTION` — the key that encrypts everything in the [Key Store](../concepts/key-store.md). Generate it with:

```bash
head -c32 /dev/urandom | base64
```

Then start it:

```bash
docker compose up -d
```

Semaphore is now at [http://localhost:3000](http://localhost:3000).

### PostgreSQL instead of MySQL

Swap the database service and change three variables:

```yaml
      SEMAPHORE_DB_HOST: postgres
      SEMAPHORE_DB_PORT: 5432
      SEMAPHORE_DB_DIALECT: postgres
```

### SQLite

From v2.16 you can skip the database container entirely:

```yaml
      SEMAPHORE_DB_DIALECT: sqlite
      SEMAPHORE_DB: "/etc/semaphore/semaphore.sqlite"
```

### Docker secrets instead of environment variables

Every sensitive setting also accepts a `_FILE` suffix, which reads the value from a file instead of the environment. This is the recommended pattern for Docker Swarm:

```yaml
secrets:
  semaphore_admin_pw:
    file: semaphore_admin_password.txt

services:
  semaphore:
    image: semaphoreui/semaphore:latest
    environment:
      SEMAPHORE_ADMIN_PASSWORD_FILE: /run/secrets/semaphore_admin_pw
      SEMAPHORE_ADMIN_NAME: admin
      SEMAPHORE_ADMIN_EMAIL: admin@localhost
      SEMAPHORE_ADMIN: admin
```

### Extra Python packages for your playbooks

Some Ansible modules and collections need Python libraries that are not in the image. Mount a `requirements.txt` into the config directory and Semaphore installs it on container start:

```yaml
    volumes:
      - ./requirements.txt:/etc/semaphore/requirements.txt
```

On startup the container runs `pip3 install --upgrade -r /etc/semaphore/requirements.txt`.

## Package manager

Download the package for your distribution from the [releases page](https://github.com/semaphoreui/semaphore/releases) — `*.deb` for Debian and Ubuntu, `*.rpm` for CentOS and RHEL.

### Debian / Ubuntu

```bash
wget https://github.com/semaphoreui/semaphore/releases/download/v2.17.15/semaphore_2.17.15_linux_amd64.deb
sudo dpkg -i semaphore_2.17.15_linux_amd64.deb
```

For ARM64, replace `amd64` with `arm64`.

### CentOS / RHEL

```bash
wget https://github.com/semaphoreui/semaphore/releases/download/v2.17.15/semaphore_2.17.15_linux_amd64.rpm
sudo yum install semaphore_2.17.15_linux_amd64.rpm
```

Then run the interactive setup and start the server:

```bash
semaphore setup
semaphore server --config=./config.json
```

> A package install gives you the Semaphore binary, not a Python/Ansible environment. You are responsible for installing Ansible, Terraform or whatever else your templates call, on the same host.

## Binary file

Download the `*.tar.gz` (or `*.zip` on Windows) for your platform from the [releases page](https://github.com/semaphoreui/semaphore/releases).

### Linux

```bash
wget https://github.com/semaphoreui/semaphore/releases/download/v2.17.15/semaphore_2.17.15_linux_amd64.tar.gz
tar xf semaphore_2.17.15_linux_amd64.tar.gz
./semaphore setup
```

### Windows

```powershell
Invoke-WebRequest `
  -Uri "https://github.com/semaphoreui/semaphore/releases/download/v2.17.15/semaphore_2.17.15_windows_amd64.zip" `
  -OutFile semaphore.zip

Expand-Archive -Path semaphore.zip -DestinationPath ./
./semaphore setup
```

Start it:

```bash
./semaphore server --config=./config.json
```

### Run as a systemd service

A package or binary install does not create a service unit for you. Create one:

```bash
sudo tee /etc/systemd/system/semaphore.service <<EOF
[Unit]
Description=Semaphore Ansible
Documentation=https://github.com/semaphoreui/semaphore
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
ExecReload=/bin/kill -HUP \$MAINPID
ExecStart=/path/to/semaphore server --config=/path/to/config.json
SyslogIdentifier=semaphore
Restart=always
RestartSec=10s

[Install]
WantedBy=multi-user.target
EOF
```

Replace `/path/to/semaphore` and `/path/to/config.json` with your real paths, then:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now semaphore
sudo systemctl status semaphore
```

## Kubernetes (Helm)

Semaphore publishes an official Helm chart. The full values reference lives on Artifact Hub: [Semaphore Helm Chart](https://artifacthub.io/packages/helm/semaphoreui/semaphore).

For production clusters, point the chart at an external managed PostgreSQL or MySQL instance rather than an in-cluster database, and plan for [runners](../operate/runners.md) if you want execution isolated from the web/API pods.

## Next steps

- [Run your first task](./first-task.md) — create a project, connect a repository, run a playbook.
- [Configure the server](./configuration.md) — the options worth setting before anyone else logs in.
- [Security](../operate/security.md) — put it behind TLS before it touches production.
