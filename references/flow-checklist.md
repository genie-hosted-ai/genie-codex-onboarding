# Genie Onboarding Flow Checklist

## Browser And OAuth

- Use `https://genieclaw.tech` as the canonical public app.
- Use embedded browser automation for ordinary clicks, form navigation, and redirects.
- Use the user for:
  - CAPTCHA or human verification
  - Google account choice when the right account is ambiguous
  - password, passkey, MFA, or device approval
  - payment details
- If Google OAuth opens a popup:
  - Inspect whether the embedded browser exposes it as a visible page, dialog, or separate target.
  - If visible to Codex, describe the account options and ask the user which to choose.
  - If not visible, ask the user to bring it forward or complete that step.
- If OAuth redirects to a system browser unexpectedly, return to the embedded browser path when possible. If not possible, ask the user whether to continue in the system browser.

## Checkout

- Start checkout only after the user confirms they want to subscribe.
- Never collect or echo payment data in chat.
- Pause at Stripe/payment forms and ask the user to complete them directly.
- After checkout success, return to the Genie dashboard and wait for provisioning.

## Provisioning

- Expect provisioning states such as queued, running, waiting for bootstrap, active, failed, or unreachable.
- Do not assume failure from a slow state. Wait and refresh before escalating.
- When active, open the worker through the dashboard link so the signed Clerk handoff is used.
- If the worker asks for reauthentication, use the API app worker-auth redirect rather than manually pasting secrets.

## Worker Setup

- Default first data email to the Genie sign-in email.
- Let the user change the first email or add more emails later.
- Prefer Composio-managed account connections for Gmail, Outlook, Calendar, Drive, Slack, WhatsApp, GitHub, and similar sources.
- Ask before connecting each OAuth-backed data source.
- Telegram bot token, Telegram request chat ID, and allowed user IDs are user-provided after initialization in the worker UI, not repo env vars.
- If the user wants to skip Telegram, treat that as acceptable unless they need Telegram chat immediately.

## Codex Worker Connection

- Prefer a product-provided "Copy Codex setup," "Connect Codex," or equivalent handoff over raw key handling.
- If only a worker API key is available:
  - Ask permission before storing it.
  - Store it in an untracked local file such as `.genie-worker.env`.
  - Set permissions to `0600` when possible.
  - Add or verify `.gitignore` excludes the file.
  - Never include the key in final answers, screenshots, commits, or logs.
- Read the worker `/docs` endpoint before API calls.
- Verify with a read-only endpoint first, such as worker server/status metadata.
- Use write or SSH permissions only after explaining the action and why it is needed.

## Recovery

- If signup gets stuck on OAuth, try sign-in from the public app directly instead of plugin/chat-driven deep links.
- If payment completes but the dashboard does not show subscription, refresh and wait briefly for webhook sync.
- If provisioning fails, capture the visible failure state and tell the user that provisioning needs operator attention.
- If a secret was pasted into Slack, GitHub, chat, screenshots, or another tool, stop and rotate it before continuing.
