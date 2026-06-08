# Worker API Reference

Use the authenticated worker browser session to discover agentless access. Worker URLs and API keys are per-user runtime values; never embed them in this skill or include them in final user-facing text.

1. Fetch `GET /api/onboarding/status` until `ready_for_agentless` is `true`.
2. Fetch `GET /api/admin/access-key` from the same authenticated browser session.
3. Read `GET /docs` for the current worker API contract.
4. Call `GET /api/admin/server` with the runtime-discovered bearer key to inspect server state.
5. Call `POST /api/admin/exec` with the runtime-discovered bearer key for narrow server-side commands.

Expected access-key shape:

```json
{
  "api_key": "<runtime-discovered API key>",
  "created_at": "<ISO timestamp>",
  "server_api_url": "<runtime-discovered worker URL>",
  "endpoints": {
    "server": "/api/admin/server",
    "exec": "/api/admin/exec"
  }
}
```

Do not expose the full API key or worker URL in chat, summaries, screenshots, logs, or final answers. Keep them in process memory only long enough to call the worker API. If a result includes secrets or connected-account metadata, summarize the useful non-secret state and omit the sensitive values.
