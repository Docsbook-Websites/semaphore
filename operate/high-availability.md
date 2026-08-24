---
title: "High availability"
description: "Run several Semaphore nodes active-active behind a load balancer, sharing a PostgreSQL or MySQL database and Redis for coordination. Architecture, configuration, NGINX example and job execution flow."
---

# High availability

> High availability is available in the **Semaphore Enterprise** edition.

Semaphore supports active-active HA: several instances run at once behind a load balancer, and every one of them can serve UI requests, API calls, scheduled jobs and task execution. If a node fails, the rest carry on.

## Architecture

**Load balancer.** Users connect through NGINX, HAProxy or a cloud load balancer, which distributes HTTP and WebSocket traffic across the nodes.

**Semaphore nodes.** Identical instances. Any node can take a request, start a job, process a schedule and push real-time updates. All nodes are equal — there is no primary and no standby, and therefore no failover procedure to get wrong.

**Shared database.** All nodes connect to one PostgreSQL or MySQL instance, which holds projects, templates, inventories, schedules, history, users and RBAC.

> SQLite and BoltDB cannot be used in HA mode — they do not support concurrent access from multiple processes. Use PostgreSQL or MySQL.

**Redis.** The coordination layer, doing three jobs:

- **Distributed locks** — only one instance executes a given job, so nothing runs twice.
- **Shared task queue state** — all nodes see the same queue and agree on who takes what.
- **Pub/sub messaging** — task updates, cluster notifications, cache invalidation and UI state propagate between nodes in real time.

## Prerequisites

- A **Semaphore Enterprise** subscription key.
- A shared **PostgreSQL** or **MySQL** database reachable from every node.
- A **Redis** instance or cluster reachable from every node.
- A **load balancer** that supports HTTP and WebSocket.
- Two or more servers.

Every node must use the same database, the same Redis and the same configuration — with one exception: `ha.node_id` must be unique per node.

## Configuration

```json
{
  "dialect": "postgres",
  "postgres": {
    "host": "db.example.com:5432",
    "name": "semaphore",
    "user": "semaphore",
    "pass": "***"
  },

  "ha": {
    "enabled": true,
    "node_id": "node-1",
    "redis": {
      "addr": "redis.example.com:6379",
      "db": 0,
      "pass": "***"
    }
  },

  "cookie_hash": "...",
  "cookie_encryption": "...",
  "access_key_encryption": "..."
}
```

Or with environment variables:

```bash
SEMAPHORE_HA_ENABLED=true
SEMAPHORE_HA_NODE_ID=node-1
SEMAPHORE_HA_REDIS_ADDR=redis.example.com:6379
SEMAPHORE_HA_REDIS_DB=0
SEMAPHORE_HA_REDIS_PASS=***
```

### Reference

| Config option | Environment variable | Description |
|---|---|---|
| `ha.enabled` | `SEMAPHORE_HA_ENABLED` | Enable HA mode |
| `ha.node_id` | `SEMAPHORE_HA_NODE_ID` | Unique identifier for this node |
| `ha.redis.addr` | `SEMAPHORE_HA_REDIS_ADDR` | Redis address, e.g. `localhost:6379` |
| `ha.redis.db` | `SEMAPHORE_HA_REDIS_DB` | Redis database number |
| `ha.redis.pass` | `SEMAPHORE_HA_REDIS_PASS` | Redis password |
| `ha.redis.user` | `SEMAPHORE_HA_REDIS_USER` | Redis username |
| `ha.redis.tls` | `SEMAPHORE_HA_REDIS_TLS` | Enable TLS for the Redis connection |
| `ha.redis.tls_skip_verify` | `SEMAPHORE_HA_REDIS_TLS_SKIP_VERIFY` | Skip Redis TLS certificate verification |

Note that `cookie_hash`, `cookie_encryption` and `access_key_encryption` must be **identical** across nodes — otherwise sessions break as requests move between them, and secrets written by one node cannot be read by another.

## Load balancer

The load balancer must support **WebSocket** connections, or live task logs will not stream.

### NGINX example

```nginx
upstream semaphore {
    server node1.example.com:3000 max_fails=3 fail_timeout=10s;
    server node2.example.com:3000 max_fails=3 fail_timeout=10s;
    server node3.example.com:3000 max_fails=3 fail_timeout=10s;
}

server {
    listen 443 ssl;
    server_name semaphore.example.com;

    ssl_certificate     /etc/ssl/certs/semaphore.crt;
    ssl_certificate_key /etc/ssl/private/semaphore.key;

    location / {
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        proxy_pass http://semaphore;

        proxy_connect_timeout 3s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;

        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
        proxy_next_upstream_tries 3;
    }

    location /api/ws {
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_pass http://semaphore;

        proxy_connect_timeout 3s;
        proxy_send_timeout 1h;
        proxy_read_timeout 1h;

        proxy_next_upstream error timeout http_502 http_503 http_504;
        proxy_next_upstream_tries 3;
    }
}
```

The separate `/api/ws` block matters: WebSocket connections need hour-long timeouts, while ordinary requests should not.

## How job execution works

1. **A task is triggered.** The request lands on any node.
2. **Metadata is stored.** The receiving node writes the task to the database and signals work through Redis.
3. **A node claims it.** One node retrieves the task, acquires a distributed lock, and marks it running.
4. **It executes.** Locally, or delegated to a [remote runner](./runners.md). Progress and logs go back to the database.
5. **Results are broadcast.** Updates propagate over Redis pub/sub so every node and every connected browser stays in sync.

## Scaling execution as well

HA scales the control plane. To scale *execution*, add [runners](./runners.md) — the two are independent, which lets you grow automation capacity without adding web/API nodes, isolate execution environments, and run many tasks in parallel across your infrastructure.

## What you get

- **Reliability** — one instance failing does not stop traffic or jobs.
- **Zero-downtime maintenance** — update or restart nodes one at a time.
- **Horizontal scalability** — add nodes behind the load balancer for capacity.
- **No primary dependency** — no failover mechanism to test and get wrong.
- **Consistent cluster state** — shared database plus Redis coordination.

## FAQ

<!-- widget:accordion -->

### What is active-active high availability?

Multiple instances run at the same time and all of them serve requests. There is no primary node; any instance can handle traffic and execute jobs.

### Why does Semaphore need Redis?

Redis is the coordination layer between instances — distributed locks, shared task queue state and pub/sub messaging. Without it, two nodes could execute the same job.

### Which database should I use?

PostgreSQL or MySQL. SQLite and BoltDB do not support concurrent access from multiple processes.

### What happens when a node fails?

The load balancer routes traffic to the remaining nodes. Running jobs continue on other instances, and new jobs are picked up by any available node.

### Can I scale horizontally?

Yes — add Semaphore nodes for web and API capacity, and add runners for task execution capacity.

<!-- /widget -->

## Next steps

- [Runners](./runners.md) — scaling execution
- [Logs and metrics](./observability.md) — watching a cluster
- [License activation](./license.md) — turning on Enterprise
