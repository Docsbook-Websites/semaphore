---
title: "License activation"
description: "Activate Semaphore Pro or Enterprise with a license key — from the web UI for a single instance, or from configuration for Docker, Kubernetes and other automated deployments."
---

# License activation

Pro and Enterprise features are unlocked with a license key on the Semaphore you are already running.

## Before you start

- **No reinstall required.** You do not need a different build or a different image. The version you have activates with a key — though upgrading gets you the newest Pro and Enterprise features.
- Sign in with an **administrator** account.
- Have the key ready: it is in your purchase email, or in the [Semaphore UI Portal](https://portal.semaphoreui.com/auth/login).

## Activate from the web UI

Best for a single instance you administer by hand.

<!-- widget:stepper -->

## Sign in as an administrator

![Semaphore UI login screen](https://semaphoreui.com/docs/assets/subscription-login-screen.png)

## Open the Admin menu

It is in the user area in the lower-left corner.

![Admin menu trigger in the lower-left corner](https://semaphoreui.com/docs/assets/subscription-admin-menu-trigger.png)

## Select "Upgrade to PRO or EE"

![Admin menu with the Upgrade to PRO or EE item](https://semaphoreui.com/docs/assets/subscription-upgrade-menu-item.png)

## Paste the key and activate

Paste your license key into the dialog and click **ACTIVATE NEW KEY**.

![Semaphore Pro activation dialog](https://semaphoreui.com/docs/assets/subscription-activation-dialog.png)

## Confirm

After activation, the **Subscription & Billing** dialog shows your current license details.

![Subscription and Billing dialog after successful activation](https://semaphoreui.com/docs/assets/subscription-activation-success.png)

<!-- /widget -->

## Activate from configuration

For Docker, Kubernetes, systemd or anything else built from code, put the key in the server configuration instead. The options live under `subscription.*`.

```json
{
  "subscription": {
    "key": "YOUR_LICENSE_KEY"
  }
}
```

Or as an environment variable:

```bash
export SEMAPHORE_SUBSCRIPTION_KEY=YOUR_LICENSE_KEY
```

Better still, keep it out of the environment entirely and read it from a file — which is what Docker secrets and Kubernetes secrets give you:

```json
{
  "subscription": {
    "key_file": "/run/secrets/semaphore-license-key"
  }
}
```

```bash
export SEMAPHORE_SUBSCRIPTION_KEY_FILE=/run/secrets/semaphore-license-key
```

> When the key comes from configuration, Semaphore **disables** the activation and editing controls in the Subscription & Billing dialog — for both `key` and `key_file`, since the file is read into the runtime key at startup. This is deliberate: configuration is the source of truth, and nobody can drift from it through the UI.

## Manage or replace a key

Open the Admin menu and select **Subscription & Billing**.

![Admin menu with the Subscription and Billing item](https://semaphoreui.com/docs/assets/subscription-billing-menu-item.png)

For a UI-managed key, the action menu inside the dialog lets you reload, upload or reset it.

![Subscription and Billing dialog with key actions](https://semaphoreui.com/docs/assets/subscription-key-actions-menu.png)

For a configuration-managed key:

1. Replace the value of `subscription.key`, or the contents of the file referenced by `subscription.key_file`.
2. **Restart Semaphore** so the server reloads it.
3. Confirm the expected Pro or Enterprise options appear.

## What each plan unlocks

See [Plans and pricing](../pricing.md) for the full comparison, or [semaphoreui.com/pricing](https://semaphoreui.com/pricing).

<!-- widget:cta -->

## Evaluate before you buy

A 30-day Pro trial runs in your own self-hosted instance. No credit card.

[Start a 30-day Pro trial](https://portal.semaphoreui.com/start_trial) · [Compare plans](../pricing.md)

<!-- /widget -->
