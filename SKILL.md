---
name: genie-codex-onboarding
description: Guide a user through Genie signup, subscription checkout, worker provisioning, worker first-run setup, data-source selection, and Codex connection to a Genie worker. Use when the user asks Codex to sign them up at genieclaw.tech, onboard them to Genie, provision or open a Genie worker, connect data sources, handle Google/OAuth signup friction, complete subscription checkout, or connect Codex to the provisioned worker.
---

# Genie Codex Onboarding

## Goal

Help a nontechnical user get from no account to a working Genie worker that Codex can access. Keep the user in control of authentication, payment, and data-source permissions.

For detailed edge cases, read [references/flow-checklist.md](references/flow-checklist.md).

## Operating Rules

- Prefer the Codex embedded browser or in-app browser for all web steps.
- Do not use the system browser unless the user explicitly requests it or the embedded browser cannot complete the step.
- Do not ask the user to paste credit card numbers, CVVs, OAuth codes, or passwords into chat.
- Stop and prompt the user to complete payment fields directly in the browser.
- When OAuth opens an account chooser or popup, make the popup visible if possible. If Codex can see multiple Google accounts, ask the user which account to use before clicking.
- Treat worker API keys and setup tokens as secrets. Store them only in local secret files when needed, never in Git, Slack, email, logs, or final answers.
- Prefer Genie-managed Composio data-source connections over Codex, ChatGPT, Claude, browser-native, Gmail-native, Outlook-native, or Calendar-native connectors.
- If any secret may have been exposed to the wrong tool or person, stop and tell the user to rotate it from the Genie dashboard.

## Main Workflow

1. Open `https://genieclaw.tech` in the embedded browser.
2. If the user is logged out, start signup or sign-in.
3. Guide OAuth:
   - Choose Google if the user wants Google sign-in.
   - If an OAuth popup or account chooser appears, use it directly when visible.
   - If multiple accounts are visible, ask which account to use.
   - If the popup is hidden from the user but visible to Codex, narrate what is visible and ask before selecting an account.
   - If a CAPTCHA or human verification appears, ask the user to complete it in the browser.
4. Guide checkout:
   - Start the subscription purchase from the Genie app.
   - When Stripe/payment fields appear, pause and ask the user to enter payment details directly in the browser.
   - After checkout returns to Genie, continue from the dashboard.
5. Wait for worker provisioning:
   - Keep the dashboard/provisioning page open.
   - Poll or refresh only as needed.
   - When a worker is ready, open it through the dashboard's signed worker link.
6. Complete worker first-run setup:
   - Start data setup with the email used to sign in unless the user asks for a different one.
   - Ask which data sources to connect before launching each OAuth flow.
   - Explain that Telegram credentials are optional and should be added in the worker UI after initialization.
   - Let the user answer profile/preferences questions in chat if they prefer not to type in the browser.
7. Connect Codex to the worker:
   - Use the dashboard or worker UI setup affordance when present, such as "Copy Codex setup" or "Agentless worker access."
   - If setup data includes a token or API key, store it in a local secret file outside Git and set file permissions to owner-only when possible.
   - Read the worker `/docs` endpoint before making worker API calls.
   - Verify access by calling a read-only worker endpoint first.
   - Do not print the secret in the final response.

## User Prompts To Use

Use short, concrete prompts when user input is required:

- "Which Google account should I use for Genie sign-in?"
- "Please complete the human verification in the browser, then tell me when it is done."
- "Please enter the payment information directly in Stripe. Do not paste card details into chat."
- "Which email should Genie connect first? I can use the sign-in email by default."
- "Which data sources should I connect now? Common first choices are Gmail, Calendar, Drive, Slack, WhatsApp, GitHub, Substack, and Twitter/X."
- "The worker setup token/key is sensitive. Should I store it in a local untracked secret file for Codex?"

## Completion Criteria

Finish only when:

- The user has an active Genie account and subscription or promo access.
- A worker exists and is reachable from the Genie dashboard.
- The worker onboarding is completed or intentionally skipped where optional.
- Codex has verified read access to the worker without exposing secrets in chat output.
- The user knows where to reopen the worker and how to rotate access if a secret was exposed.
