---
name: genie-codex-onboarding
description: Use when guiding a user through Genie signup, checkout, worker provisioning, worker handoff, data-source onboarding, and agentless worker access through the embedded browser, runtime-discovered /docs, and a runtime-discovered worker admin API key.
---

# Genie Codex Onboarding

## Overview

Use this skill to drive the end-to-end Genie onboarding flow from the public signup app into a provisioned worker. Codex navigates, waits, reads stable page state, asks the user what data sources they want, and connects agentlessly; the user performs sensitive steps such as account auth, payment, and external OAuth consent.

Use the `browser-use:browser` skill with the `iab` backend for all browser automation. If the user does not provide a signup URL, open `https://genieos.net?genie_onboarding=codex`.

Never embed, assume, share, or persist a worker URL or API key in the skill. Each user gets their own worker URL and API key during onboarding; discover those values only in the active browser session and keep them out of final user-facing text. The Genie app treats a browser as Codex-embedded when the client `User-Agent` header contains `Codex`.

## Browser Workflow

1. Open the signup URL in the embedded browser.
2. Prefer the Clerk `Continue with Google` path for signup/sign-in. Verify that the Google button is unique and enabled, click it, then inspect the resulting browser state:
   - If a Google OAuth popup or new tab is visible, switch to/read that tab and summarize the visible account choices or required next action for the user. The user performs account selection, credential entry, consent, CAPTCHA, and any one-time-code steps.
   - If no visible popup/tab appears, inspect the current tab, browser tab list, and any launch URL or blocked-popup state exposed by the page. If an OAuth URL is available, open it deliberately in the embedded browser and then hand control to the user.
   - If the OAuth UI still cannot be shown, keep a listener/poll loop active instead of ending the skill. Watch the current tab URL, browser tab list, button states, visible Clerk/Google/CAPTCHA text, and hidden challenge state until login completes, a visible user action is required, or a concrete error appears.
   - Treat login as complete when the browser reaches `/auth/callback`, `/subscribe`, `/dashboard`, `/billing`, `/provisioning`, `/worker-auth`, or a worker origin. Continue immediately with the matching downstream step.
   - If the page is stuck in a loading/disabled auth state with no visible challenge, report that state briefly and keep listening. Do not ask the user to retry unless the page returns to an enabled state, exposes an OAuth URL, shows a visible CAPTCHA/action, or stays unchanged long enough to be a real stall.
   - If a real stall occurs, read the visible Clerk/Google state and give the user concrete options based on what is available, such as retry `Continue with Google`, open the OAuth URL manually if one is visible in the browser session, or use email/password signup instead. Do not invent URLs or account choices.
   - Fall back to email/password only when Google is unavailable, blocked, or the user requests it. Do not type credentials, payment details, or one-time codes unless the user explicitly provides non-sensitive test values.
3. After auth starts, keep watching the embedded browser until it lands on `/subscribe`, `/dashboard`, or an external checkout/auth page:
   - Prefer a URL listener such as `waitForURL("**/subscribe**")` or a short polling loop over `tab.url()` when the flow may pass through Clerk callback pages.
   - On every auth-listener tick, check `new URL(await tab.url()).pathname`. If the pathname is exactly `/subscribe`, immediately read the page state, verify the `Continue to checkout` button is unique and enabled, then click it. Do this for `/subscribe` with or without query parameters. Do not stop to merely report that `/subscribe` was reached before attempting this click.
   - The `/subscribe` checkout-launch click is part of the skill and does not require extra confirmation when the user asked to run onboarding.
   - After clicking `Continue to checkout`, wait for the next navigation. If it opens Stripe or another external checkout page, hand control to the user. The user performs payment and any sensitive verification steps.
   - If `/subscribe` is visible but the checkout button is missing, disabled, ambiguous, or an error is shown, report the exact visible state instead of guessing.
   - If the browser lands directly on `/dashboard`, continue with the dashboard/provisioning checks below.
4. After the user completes checkout, keep a browser listener running through the return and worker boot flow:
   - Watch for `/billing/success`, `/dashboard`, `/provisioning`, `/worker-auth`, or a non-`genieos.net` worker origin. Use `waitForURL` when one target is expected, or a short polling loop over `tab.url()` plus page state when the flow may move through multiple pages.
   - If the browser lands on `/billing/success`, `/dashboard`, or `/provisioning`, continue polling until the page either redirects automatically to `/worker-auth` or the worker URL, or exposes the ready/open-worker controls below.
   - Treat visible provisioning states such as queued, provisioning, waiting for bootstrap, or booting as expected transient states. Keep listening; do not declare failure while the page is still progressing.
   - If the browser redirects through `/worker-auth`, wait for the next navigation into the worker. If it lands on the worker origin, continue with the worker setup checks.
   - If boot stalls with a visible error or no progress for a long interval, report the current URL and visible provisioning state without inventing a worker URL.
5. While waiting on `/dashboard` or `/provisioning`, poll the page state with these selectors:
   - `[data-testid="dashboard-provisioning-state"]`
   - `[data-testid="dashboard-worker-open-link"]`
   - `[data-testid="dashboard-worker-url"]`
   - `[data-testid="dashboard-reveal-api-key-button"]`
   - `[data-testid="dashboard-api-key"]`
   - `[data-testid="dashboard-docs-url"]`
6. When `[data-testid="dashboard-worker-open-link"]` appears, verify it is unique and enabled, then click it. It should navigate same-tab through `/worker-auth` into the worker. Keep the URL listener active until the worker page loads.
7. The worker should land directly on `/setup/data` for the Codex embedded browser flow. If it lands elsewhere, navigate to `/setup/data?genie_onboarding=codex`.
8. After the worker page loads, keep listening until API connection information is discoverable from the authenticated embedded-browser session:
   - Discover the worker origin from the current browser URL after the `/worker-auth` handoff. Do not copy it into skill files, notes, summaries, or final user-facing text.
   - Poll `GET /api/onboarding/status` from the authenticated worker browser session when available. Use its `docs_url` as the docs/API base signal when present.
   - Retrieve the admin API key with `GET /api/admin/access-key` from the authenticated worker browser session, or from a visible setup/dashboard key reveal control if the API endpoint is unavailable. Do not reveal, paste, log, persist, or summarize the key.
   - Treat the API connection information as ready only when the current worker origin and an admin API key have both been obtained. Prefer also obtaining the docs URL, but do not block forever if the key and origin are present and `/docs` is reachable.
   - If the worker page is loaded but the key or docs URL are not ready yet, continue polling briefly and report only non-secret state such as current page and readiness, never the key or full worker URL.
9. Ask the user what data sources they want. Use the runtime-discovered admin API key to call `POST /api/admin/composio/sources` with the selected source IDs. Do not write the key into skill files, notes, summaries, or final responses.
10. On `/setup/data`, assist with data-source connection launches. The user must complete external OAuth consent themselves. Follow the data-source connector behavior below by default.
11. Poll `GET /api/onboarding/status` from the authenticated worker browser session if available, or `GET /api/admin/composio/sources` with the runtime-discovered admin key, and report selected and connected source IDs accurately.
12. If needed, use visible `/chat` hooks:
   - `[data-testid="chat-page"]`
   - `[data-testid="agentless-sidebar"]`
   - `[data-testid="agentless-docs-url"]`
   - `[data-testid="agentless-reveal-api-key-button"]`
   - `[data-testid="agentless-api-key"]`
   - `[data-testid="agentless-copy-api-key-button"]`
13. Use runtime-discovered worker origin and key for subsequent worker API calls. Do not write either value into skill files, notes, summaries, or final responses.

## Status Contract

`GET /api/onboarding/status` returns the worker onboarding state for the authenticated browser session:

```json
{
  "ready_for_chat": true,
  "ready_for_agentless": true,
  "current_step_id": "composio",
  "completed_step_ids": ["agent", "composio"],
  "agent_skipped": true,
  "composio": { "status": "configured" },
  "selected_source_ids": [],
  "connected_source_ids": [],
  "docs_url": "<runtime-discovered worker docs URL>"
}
```

Treat zero connected sources as valid only when data onboarding has been explicitly saved or skipped and `ready_for_agentless` is true. Report selected and connected source IDs accurately.

## Data Source Connector Behavior

The embedded browser may not surface app-created `window.open(..., "_blank")` tabs. Do not assume clicking `Connect` will leave the setup page visible or create a visible tab.

When the user asks to connect one or more data sources:

1. Identify the exact source card on `/setup/data` and verify its `Connect` or `Reconnect` button is unique and enabled.
2. Before clicking, confirm any sensitive data transmission unless already approved in the current user request. At minimum, name the source, the visible contact email that will be sent to the worker/Composio connection endpoint, and that an external OAuth consent page may open.
3. Click the source button after confirmation. If the app opens a visible OAuth tab, leave it to the user for consent.
4. If no new visible tab appears, inspect the current tab state and browser tab list. If the current tab changed to `about:blank` or another transient page, restore `/setup/data` before continuing unless an OAuth URL is visible.
5. If the app produced a launch URL, open/follow that runtime-discovered URL deliberately in the embedded browser instead of relying on the app popup. Do not paste or persist the URL in final user-facing text.
6. Hand control to the user on the external OAuth/login/consent page. Do not type credentials, authorization codes, or approve consent.
7. After the user completes consent and returns to Genie, poll or click refresh as needed, then report the connected and selected source IDs accurately.

## Worker API Reference

Read `references/worker-api.md` before making admin API calls. Never paste API keys or worker URLs into final user-facing text. In final responses, say that the key and worker URL were obtained during the current onboarding session.
