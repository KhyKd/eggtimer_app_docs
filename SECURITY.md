# Security policy

This repository contains documentation only. Security issues almost always concern the hosted Eggtimer.app service or its MCP servers:

- `https://eggtimer.app/api/mcp`
- `https://eggtimer.app/api/mcp/server`

## Reporting a vulnerability

Email **[support@eggtimer.app](mailto:support@eggtimer.app)** with "Security" in the subject. Please include the affected endpoint or tool, steps to reproduce, and the impact you observed.

- Please **do not** open a public issue for a vulnerability.
- Never include a real workspace API key, recovery key or private webhook URL in a report. If a key may be exposed, rotate it at [eggtimer.app/automate.html](https://eggtimer.app/automate.html) first.
- Test only against your own workspace. Respect the published rate limits and do not degrade the service for others.

We aim to acknowledge reports promptly and will keep you informed while we investigate.

## Leaked key in this repository?

These docs never contain real credentials. Placeholders such as `YOUR_WORKSPACE_API_KEY` and `${EGGTIMER_API_KEY}` are not keys. If you find something that looks like a real credential here, email support@eggtimer.app.

See also: [Privacy policy](https://eggtimer.app/privacy) · [Terms](https://eggtimer.app/terms)
