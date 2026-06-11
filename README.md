# Genie Codex Onboarding Skill

This repo contains a Codex skill for guiding a user through Genie signup and worker setup.

The skill teaches Codex how to:

- open `https://genieos.net` in the embedded browser
- guide sign-in or signup without collecting passwords or OAuth secrets in chat
- handle Google OAuth account selection and CAPTCHA handoff safely
- pause while the user enters Stripe payment information directly in the browser
- wait for Genie worker provisioning
- guide first-run worker setup and data-source selection
- prefer Genie Composio connections over Codex-native or browser-native account connectors
- connect Codex to the provisioned worker without exposing worker secrets in chat, logs, Git, Slack, or email

## Files

- `SKILL.md`: the main skill instructions loaded by Codex when the skill is invoked.
- `references/flow-checklist.md`: extra checklist for OAuth, checkout, provisioning, worker setup, and recovery cases.
- `references/worker-api.md`: worker admin API reference for runtime-discovered agentless access.
- `agents/openai.yaml`: UI metadata for Codex skill lists.

## Usage

Install or copy this skill into a Codex skills directory, then invoke it explicitly:

```text
Use $genie-codex-onboarding to sign me up for Genie and connect Codex to my worker.
```

Codex should keep the user in control of authentication, payment, and data permissions. It should never ask the user to paste payment details, passwords, OAuth codes, or worker API keys into chat.
