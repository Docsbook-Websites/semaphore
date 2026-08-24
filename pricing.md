---
title: "Plans and pricing"
description: "Semaphore Community is free and MIT-licensed. Pro adds isolated runners, Vault secrets, SSO and a support SLA; Enterprise adds high availability, custom roles and custom SLAs. All plans are self-hosted."
---

# Plans and pricing

All three plans are **self-hosted**. Your automation and your credentials stay in your environment; paid plans unlock features, scale and support rather than moving you to a cloud service.

| | Community | Pro | Enterprise |
|---|---|---|---|
| **Price** | $0 | $490 / year | Custom |
| **For** | Personal setups and early automation | Company-grade control | Large-scale and critical infrastructure |
| **License** | MIT, open source | Subscription | Subscription |

Figures on this page come from [semaphoreui.com/pricing](https://semaphoreui.com/pricing) — check there for the current terms before you buy.

## Which plan

**Community** if you want a free starting point, one person owns the setup, and you can self-support from the docs and the community.

**Pro** if automation needs controlled execution or is shared by a team, tasks should run on project runners close to your infrastructure, or you need identity controls, exportable logs and a support SLA.

**Enterprise** if Semaphore is business-critical or governed, and you need HA, custom roles, identity group mapping, or onboarding and procurement support.

## Feature comparison

### Automation and execution

| Feature | Community | Pro | Enterprise |
|---|---|---|---|
| Ansible, Terraform, OpenTofu, Shell, Python, PowerShell from one platform | ✓ | ✓ | ✓ |
| Inventory management (static, file-based, dynamic) | ✓ | ✓ | ✓ |
| Reusable task templates | ✓ | ✓ | ✓ |
| Scheduled tasks (cron) | ✓ | ✓ | ✓ |
| Variable groups | ✓ | ✓ | ✓ |
| Distributed execution (global runners) | ✓ | ✓ | ✓ |
| Isolated execution (project runners) | — | ✓ | ✓ |
| Terraform/OpenTofu HTTP backend | — | ✓ | ✓ |
| Tag-based runner routing | — | ✓ | ✓ |

### Access control

| Feature | Community | Pro | Enterprise |
|---|---|---|---|
| Project-level access control | ✓ | ✓ | ✓ |
| Role-based access control | ✓ | ✓ | ✓ |
| Two-factor authentication (TOTP) | — | ✓ | ✓ |
| LDAP / Active Directory login | — | ✓ | ✓ |
| SSO via OIDC (Okta, Entra ID, Keycloak, Google…) | — | ✓ | ✓ |
| Role mapping from IdP groups | — | — | ✓ |
| Custom roles | — | — | ✓ |
| Granular task-level permissions | — | — | ✓ |

### Secrets and credentials

| Feature | Community | Pro | Enterprise |
|---|---|---|---|
| Key Store with runtime injection | ✓ | ✓ | ✓ |
| HashiCorp Vault storage | — | ✓ | ✓ |
| AWS Secrets Manager | — | — | ✓ |
| Azure Key Vault | — | — | ✓ |
| Devolutions Server | — | — | ✓ |

### Audit

| Feature | Community | Pro | Enterprise |
|---|---|---|---|
| Task history | ✓ | ✓ | ✓ |
| Activity history | ✓ | ✓ | ✓ |
| Server log | ✓ | ✓ | ✓ |
| Task execution summary | — | ✓ | ✓ |
| File-based log export | — | ✓ | ✓ |

### API and integrations

| Feature | Community | Pro | Enterprise |
|---|---|---|---|
| REST API | ✓ | ✓ | ✓ |
| Webhook automation triggers | ✓ | ✓ | ✓ |
| Git repositories (GitHub, GitLab, Bitbucket) | ✓ | ✓ | ✓ |
| Notifications (email, Slack, Teams and more) | ✓ | ✓ | ✓ |
| MCP server for AI-based automation workflows | ✓ | ✓ | ✓ |

### Support and licensing

| Feature | Community | Pro | Enterprise |
|---|---|---|---|
| Support SLA | — | 48h business days | Custom |
| Feature request review | Community-driven | Standard | Prioritized |
| Onboarding, upgrade and configuration help | — | — | ✓ |
| Migration assistance | — | — | ✓ |
| High availability | — | — | ✓ |
| Multi-instance licence | — | Per-instance subscription | ✓ |
| Air-gapped (offline) licence | — | — | ✓ |

Pro also covers up to **500 managed nodes**. A managed node is a unique target host — a physical server, VM or remote host — counted once per year however many times you automate it. Hosts are identified by their inventory connection target, so prefer stable hostnames over ephemeral IPs in dynamic environments.

## Common questions

<!-- widget:accordion -->

### Can I use Semaphore for free?

Yes. Community is free forever and open source under the MIT licence. It includes the core functionality: self-host it, automate with it, and contribute to it. Paid plans exist for when automation becomes part of a team's regular workflow.

### Can I try Pro before paying?

Yes — a **30-day Pro trial** runs in your own self-hosted instance, with no credit card. Activate the trial, apply it to your instance, and evaluate the Pro features. For an Enterprise evaluation, contact the Semaphore team.

### How do we activate it?

You receive a subscription key and activate it in your existing instance — no reinstall, no different build. See [License activation](./operate/license.md).

### Is it cloud-based or self-hosted?

Self-hosted, on every plan. You run Semaphore in your own environment and keep control of your infrastructure, configuration and data.

### Can we change plan later?

Yes. Many teams start on Community, move to Pro when they need advanced features and support, and choose Enterprise when Semaphore becomes part of critical production workflows.

### What payment methods are supported?

Pro is bought online by card. If your organisation needs an invoice or bank transfer, contact the Semaphore team — Enterprise is handled through custom commercial terms and paid by bank transfer.

<!-- /widget -->

<!-- widget:cta -->

## Evaluate it in your own environment

A 30-day Pro trial, self-hosted, no credit card.

[Start a 30-day Pro trial](https://portal.semaphoreui.com/start_trial) · [See full pricing](https://semaphoreui.com/pricing)

<!-- /widget -->
