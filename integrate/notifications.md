---
title: "Notifications"
description: "Send Semaphore task alerts to email, Telegram, Slack, Microsoft Teams, Rocket.Chat, DingTalk or Gotify — with the configuration keys and environment variables for each."
---

# Notifications

Semaphore can push task alerts into whatever channel your team actually watches. Every integration is configured the same way — an `*_alert` flag to switch it on, plus the destination — in `config.json` or as environment variables.

| Channel | Config keys |
|---|---|
| [Email](#email) | `email_alert`, `email_sender`, `email_host`, `email_port`, `email_secure`, `email_username`, `email_password`, `email_tls`, `email_tls_min_version` |
| [Telegram](#telegram) | `telegram_alert`, `telegram_token`, `telegram_chat` |
| [Slack](#slack) | `slack_alert`, `slack_url` |
| [Microsoft Teams](#microsoft-teams) | `microsoft_teams_alert`, `microsoft_teams_url` |
| [Rocket.Chat](#rocketchat) | `rocketchat_alert`, `rocketchat_url` |
| [DingTalk](#dingtalk) | `dingtalk_alert`, `dingtalk_url` |
| [Gotify](#gotify) | `gotify_alert`, `gotify_url`, `gotify_token` |

Every key has an environment-variable equivalent following the usual rule: `slack_alert` becomes `SEMAPHORE_SLACK_ALERT`, `slack_url` becomes `SEMAPHORE_SLACK_URL`. That is what you want in containers — webhook URLs are credentials, and they belong in secrets rather than in a config file in Git.

## Email

SMTP, with an AWS SES example:

```json
{
  "email_alert":           true,
  "email_sender":          "noreply@example.com",
  "email_host":            "email-smtp.us-east-1.amazonaws.com",
  "email_port":            "587",
  "email_secure":          true,
  "email_username":        "<smtp-username>",
  "email_password":        "<smtp-password>",
  "email_tls":             true,
  "email_tls_min_version": "1.2"
}
```

- `email_secure` — use **StartTLS** to upgrade the connection.
- `email_tls` — force TLS for SMTP connections.
- `email_tls_min_version` — minimum TLS version, e.g. `1.2`.

## Telegram

You need a bot token and a chat ID before Semaphore can send anything.

**Create the bot**

1. Message [@BotFather](https://t.me/botfather) with `/start`.
2. Follow the prompts to create a bot and note the authorization token — it is a secret.
3. Message your new bot with `/start` so it can send to you.

**Find the chat ID**

1. Message @RawDataBot with anything.
2. Copy the `id` value from the `chat` object.

**Test it before touching Semaphore**

```bash
curl -X POST "https://api.telegram.org/bot<your-bot-token>/sendMessage" \
  -d chat_id=<your-chat-id> \
  -d text="Test message from curl"
```

**Configure**

```json
{
  "telegram_alert": true,
  "telegram_token": "<your-bot-token>",
  "telegram_chat":  "<your-chat-id>"
}
```

Each project can override the global chat ID with its own, which is how you keep production alerts out of the channel where people discuss staging.

## Slack

Slack notifications go through an incoming webhook.

<!-- widget:stepper -->

## Create the app

Go to [api.slack.com/apps](https://api.slack.com/apps), click **Create New App** → **From Scratch**, name it (for example `Semaphore Bot`) and pick your workspace.

## Enable incoming webhooks

In the app settings, open **Features → Incoming Webhooks** and switch **Activate Incoming Webhooks** on.

## Create the webhook URL

Click **Add New Webhook to Workspace**, choose the destination channel, and click **Allow**. Slack gives you a URL of the form `https://hooks.slack.com/services/<team>/<channel>/<secret>`. Treat it as a credential — anyone holding it can post to that channel.

## Test it

```bash
curl -X POST -H 'Content-type: application/json' \
  --data '{"text":"Hello from Semaphore UI 🚀"}' \
  "$SLACK_WEBHOOK_URL"
```

The message should appear in the channel.

## Configure Semaphore

```json
{
  "slack_alert": true,
  "slack_url": "<your-slack-webhook-url>"
}
```

Or, better for containers:

```bash
SEMAPHORE_SLACK_ALERT=True
SEMAPHORE_SLACK_URL=<your-slack-webhook-url>
```

<!-- /widget -->

## Microsoft Teams

Create an incoming webhook connector on the target channel, then:

```json
{
  "microsoft_teams_alert": true,
  "microsoft_teams_url": "<your-teams-webhook-url>"
}
```

## Rocket.Chat

Create an incoming webhook integration in Rocket.Chat, then:

```json
{
  "rocketchat_alert": true,
  "rocketchat_url": "<your-rocketchat-webhook-url>"
}
```

## DingTalk

Add a custom robot to the group and use its webhook address:

```json
{
  "dingtalk_alert": true,
  "dingtalk_url": "<your-dingtalk-robot-url>"
}
```

## Gotify

Gotify needs both the server URL and an application token:

```json
{
  "gotify_alert": true,
  "gotify_url": "https://gotify.example.com",
  "gotify_token": "<your-gotify-app-token>"
}
```

## Choosing what to alert on

A notification on every task is a notification nobody reads. In practice:

- Route **failures** to the channel the on-call person watches.
- Route **scheduled job outcomes** somewhere quieter — a daily backup succeeding is a record, not an interruption.
- Use **per-project chat IDs or webhooks** so production and staging do not share a channel.

For a full audit trail rather than alerts, use the [audit webhook or syslog forwarding](../operate/observability.md#siem-integration) instead — that is a different job and a different destination.

## Next steps

- [Logs and metrics](../operate/observability.md) — the audit trail
- [Templates and tasks](../concepts/templates-and-tasks.md#schedules) — what runs unattended and therefore needs alerting
- [Configure the server](../get-started/configuration.md) — where these keys live
