<p align="center">
  <img src="assets/icon-512.png" alt="Eggtimer.app icon" width="96" height="96">
</p>

<h1 align="center">Eggtimer.app MCP servers</h1>

<p align="center">
Turn a timed plan into a link: countdowns, clock-time timers, Pomodoro, Tabata, stopwatch and labelled multi-step recipes that open and run in any browser. No account needed.
</p>

<p align="center">
<a href="https://eggtimer.app">Website</a> ·
<a href="https://eggtimer.app/ai.html">AI &amp; MCP guide</a> ·
<a href="docs/tools.md">Tool reference</a> ·
<a href="https://eggtimer.app/support">Support</a>
</p>

This repository is **documentation only** for the two hosted [Model Context Protocol](https://modelcontextprotocol.io) servers run by [Eggtimer.app](https://eggtimer.app). There is no server code here and nothing to install or run locally: point your MCP client at a URL below.

## Contents

- [Servers at a glance](#servers-at-a-glance)
- [Quick connect](#quick-connect)
  - [Claude](#claude)
  - [Cursor](#cursor)
  - [VS Code](#vs-code)
  - [ChatGPT and the OpenAI Responses API](#chatgpt-and-the-openai-responses-api)
  - [Any stdio client via mcp-remote](#any-stdio-client-via-mcp-remote)
  - [Focused tool sets](#focused-tool-sets)
- [Tools](#tools)
- [Example prompts](#example-prompts)
- [How it behaves](#how-it-behaves)
- [Privacy and security](#privacy-and-security)
- [Listings](#listings)
- [Support](#support)
- [License](#license)

## Servers at a glance

| | Public link server | Server Timers |
|---|---|---|
| Endpoint | `https://eggtimer.app/api/mcp` | `https://eggtimer.app/api/mcp/server` |
| Transport | Streamable HTTP (JSON responses) | Streamable HTTP (JSON responses) |
| Auth | None | `Authorization: Bearer <workspace API key>` |
| Tools | 7, all read-only | 18 (the 7 public tools + 11 workspace tools) |
| What it does | Builds validated, shareable timer links | Runs real countdowns and step routines on Eggtimer.app's servers |
| Registry name | [`app.eggtimer/eggtimer`](server-json/eggtimer.server.json) | [`app.eggtimer/server-timers`](server-json/server-timers.server.json) |
| Docs | [ai.html#mcp](https://eggtimer.app/ai.html#mcp) | [server-timers.md](https://eggtimer.app/server-timers.md), [server-routines.md](https://eggtimer.app/server-routines.md) |

**Public link server.** No account, no key. The tools validate input and return eggtimer.app URLs, so an assistant never has to guess a link format. Creating a link starts nothing; the countdown starts when someone opens it.

**Server Timers.** For countdowns that must keep running after every browser tab closes. Schedules countdowns (1 second to 30 days) and finite multi-step routines, with status, listing, cancellation and optional delivery to a private Zapier Catch Hook. Create a free workspace and its API key at [eggtimer.app/automate.html](https://eggtimer.app/automate.html) (no email or payment details). Free plan with usage limits; see [Limits](#limits).

Protocol versions supported: `2025-11-25`, `2025-06-18`, `2025-03-26`. No sessions, SSE listener or server notifications; `GET` returns 405.

## Quick connect

The public server needs only its URL. For Server Timers, keep your workspace key in an environment variable or a client secret prompt, never in a file you commit or share. The examples use `EGGTIMER_API_KEY`.

> Use only the workspace **API key** (starts with `ew_`) in AI clients. Never paste the **recovery key** (starts with `er_`) into a client.

### Claude

**Claude (web and desktop), public server:** Settings → Connectors → Add custom connector → name `Eggtimer.app`, URL `https://eggtimer.app/api/mcp`.

**Claude Code:**

```bash
# Public server
claude mcp add --transport http eggtimer https://eggtimer.app/api/mcp

# Server Timers
claude mcp add --transport http eggtimer-server https://eggtimer.app/api/mcp/server \
  --header "Authorization: Bearer ${EGGTIMER_API_KEY}"
```

**Claude Desktop, Server Timers:** custom connectors do not take static Bearer headers, so bridge it through [`mcp-remote`](#any-stdio-client-via-mcp-remote) in `claude_desktop_config.json`.

### Cursor

`~/.cursor/mcp.json` (global) or `.cursor/mcp.json` (project):

```json
{
  "mcpServers": {
    "eggtimer": {
      "url": "https://eggtimer.app/api/mcp"
    },
    "eggtimer-server": {
      "url": "https://eggtimer.app/api/mcp/server",
      "headers": { "Authorization": "Bearer ${env:EGGTIMER_API_KEY}" }
    }
  }
}
```

### VS Code

`.vscode/mcp.json` (VS Code prompts for the key once and stores it securely):

```json
{
  "inputs": [
    {
      "type": "promptString",
      "id": "eggtimer-api-key",
      "description": "Eggtimer.app workspace API key (ew_...)",
      "password": true
    }
  ],
  "servers": {
    "eggtimer": {
      "type": "http",
      "url": "https://eggtimer.app/api/mcp"
    },
    "eggtimer-server": {
      "type": "http",
      "url": "https://eggtimer.app/api/mcp/server",
      "headers": { "Authorization": "Bearer ${input:eggtimer-api-key}" }
    }
  }
}
```

Or add the public server from a terminal:

```bash
code --add-mcp '{"name":"eggtimer","type":"http","url":"https://eggtimer.app/api/mcp"}'
```

### ChatGPT and the OpenAI Responses API

**ChatGPT, public server:** Plugins → **+** → Add custom MCP server → URL `https://eggtimer.app/api/mcp`, authentication **No authentication**. Server Timers needs a static Bearer key, which ChatGPT's custom MCP setup does not offer; use the API below instead.

**Responses API:** add an `mcp` tool. For Server Timers, pass the workspace key in `authorization` (OpenAI sends it as a Bearer token and does not store it).

```python
import os
from openai import OpenAI

client = OpenAI()

resp = client.responses.create(
    model="YOUR_MODEL",  # any model that supports the MCP tool
    tools=[
        {
            "type": "mcp",
            "server_label": "eggtimer",
            "server_description": "Create shareable browser timer links.",
            "server_url": "https://eggtimer.app/api/mcp",
            "require_approval": "never",  # all public tools are read-only
        },
        # Server Timers (optional):
        # {
        #     "type": "mcp",
        #     "server_label": "eggtimer_server",
        #     "server_url": "https://eggtimer.app/api/mcp/server",
        #     "authorization": os.environ["EGGTIMER_API_KEY"],
        #     "require_approval": "always",
        # },
    ],
    input="Give me a 12 minute tea timer link.",
)
print(resp.output_text)
```

### Any stdio client via mcp-remote

For clients that only launch local stdio servers (for example Claude Desktop's config file), bridge with [`mcp-remote`](https://www.npmjs.com/package/mcp-remote):

```json
{
  "mcpServers": {
    "eggtimer": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://eggtimer.app/api/mcp"]
    },
    "eggtimer-server": {
      "command": "npx",
      "args": [
        "-y", "mcp-remote", "https://eggtimer.app/api/mcp/server",
        "--header", "Authorization:${EGGTIMER_AUTH}"
      ],
      "env": { "EGGTIMER_AUTH": "Bearer YOUR_WORKSPACE_API_KEY" }
    }
  }
}
```

Replace `YOUR_WORKSPACE_API_KEY` locally. (No space after `Authorization:` avoids an argument-quoting bug in some clients; the space lives inside the variable.)

**Clients with native remote support** (Cline, Kilo Code, Windsurf, Zed and others) can use the URL directly: type `streamableHttp` / `http`, URL as above, plus the `Authorization` header for Server Timers.

### Focused tool sets

Server Timers accepts a `profile` query parameter to cut tool-definition overhead. A profile filters discovery and calls; it does not narrow the key's permissions.

| URL | Tools |
|---|---|
| `https://eggtimer.app/api/mcp/server` | all 18 |
| `https://eggtimer.app/api/mcp/server?profile=timers` | 5: timer create/read/cancel/list + workspace |
| `https://eggtimer.app/api/mcp/server?profile=routines` | 7: six routine/event tools + workspace |
| `https://eggtimer.app/api/mcp/server?profile=links` | the 7 public link tools |

## Tools

Full parameters, annotations and outputs: [docs/tools.md](docs/tools.md).

### Public tools

Available on both servers. All are read-only, idempotent and closed-world, and each publishes an `outputSchema`.

| Tool | What it does |
|---|---|
| [`get_capabilities`](docs/tools.md#get_capabilities) | Short feature and limits summary; `detail: "full"` for the complete reference. |
| [`list_presets`](docs/tools.md#list_presets) | Preset names, schedules and links (Pomodoro 25/5, 27/8, Tabata, 52/17, beep-test practice, stopwatch). |
| [`get_preset`](docs/tools.md#get_preset) | One preset with its full validated recipe, for inspection or customisation. |
| [`create_timer`](docs/tools.md#create_timer) | Validate a duration and return a countdown link. |
| [`create_clock_link`](docs/tools.md#create_clock_link) | Link that counts to the next local clock time (e.g. 9pm); each opener uses their own timezone. |
| [`create_recipe`](docs/tools.md#create_recipe) | Validate a data-only multi-step recipe and return its preview link. |
| [`get_integrations`](docs/tools.md#get_integrations) | Explain optional Zapier completion hooks and browser-lifetime limits. |

### Server Timers tools

The 7 public tools, plus:

| Tool | Kind | What it does |
|---|---|---|
| [`create_server_timer`](docs/tools.md#create_server_timer) | write | Start a countdown (duration or exact deadline, 1 s to 30 days) that runs without a browser. |
| [`get_server_timer`](docs/tools.md#get_server_timer) | read | Deadline, status, remaining time and hook-delivery outcome. |
| [`list_server_timers`](docs/tools.md#list_server_timers) | read | Active and up to 100 recent timers. |
| [`cancel_server_timer`](docs/tools.md#cancel_server_timer) | destructive | Cancel a timer or pending retry. Safe to repeat. |
| [`create_server_routine`](docs/tools.md#create_server_routine) | write | Start a finite multi-step routine with per-step and per-cycle events. |
| [`get_server_routine`](docs/tools.md#get_server_routine) | read | Routine timeline and current step. |
| [`get_routine_events`](docs/tools.md#get_routine_events) | read | One routine's event history and delivery outcomes. |
| [`list_server_routines`](docs/tools.md#list_server_routines) | read | Active and recent routines. |
| [`list_server_events`](docs/tools.md#list_server_events) | read | Workspace event feed, paginated. |
| [`cancel_server_routine`](docs/tools.md#cancel_server_routine) | destructive | Stop future steps and pending deliveries. |
| [`get_workspace`](docs/tools.md#get_workspace) | read | Free-plan limits, usage and idle expiry. |

## Example prompts

Public server:

- "Give me a 12 minute timer labelled Tea." → `https://eggtimer.app/Tea/for/720s`
- "Make a countdown to 9pm called Dinner that I can send to friends." → `https://eggtimer.app/Dinner/at/21:00`
- "I want to study with Pomodoro." → `https://eggtimer.app/pomodoro`
- "Eight rounds of Tabata, please."
- "Build a soft-boiled egg recipe: boil water 5 min, cook 6½ min, ice bath 2 min."
- "A 30-minute workout: 2 min warm-up, six 2-min work / 2-min rest pairs, 4 min cool-down."
- "Timer for my talk: intro 2 min, demo 10 min, Q&A 5 min, on a ring display."

Server Timers:

- "Start a server timer for 25 minutes called Focus and give me its status link."
- "Schedule a deadline for 17:00 tomorrow Berlin time and notify my Zapier hook."
- "Run Pomodoro 25/5 four times as a server routine."
- "What timers are running? Cancel the one called Laundry."

## How it behaves

- **Links, not alarms.** Public tools only build links. An assistant should never say "I started your timer" after generating one. Simple countdown links start when opened; recipe links preview and wait for Start.
- **Browser timers need the page open.** A locked phone, sleeping device, background tab or closed browser can delay or prevent the alarm. Use a native alarm for locked-phone sound.
- **Server timers start immediately** on `create_server_timer` / `create_server_routine`. Creates are idempotent via a required `idempotency_key`.
- **Webhooks are at-least-once.** Optional Zapier deliveries can arrive late, duplicated or out of order; deduplicate on the stable `event_id`.
- **Not for emergency or medical timing.** No hard real-time guarantee.

### Limits

- Edge rate limit: 60 requests per minute per IP (both servers).
- Server Timers free plan: 50 new timers or routines per workspace per UTC day; 10 active at once; durations 1 second to 30 days; routines at most 64 phases.
- Timer records expire 7 days after they end; idle workspaces expire after 90 days.
- No uptime or delivery-latency SLO.

Current figures always live at [eggtimer.app/server-timers.md](https://eggtimer.app/server-timers.md).

## Privacy and security

- **Public server:** no account, no key, no access to any account. Links are constructed from your input and not stored. Hosting providers process ordinary request data. Don't put sensitive information in labels; links can be shared and copied.
- **Server Timers:** each workspace key reaches only its own workspace. The service stores credential hashes, not key copies. Timer status never returns hook URLs or keys. Avoid sensitive titles.
- **Keys:** keep your `ew_` API key in a secret store or environment variable. Never commit it, put it in a timer URL or recipe, or paste the `er_` recovery key into an AI client. Lost API key: rotate it with the recovery key at [automate.html](https://eggtimer.app/automate.html).
- **Approvals:** the public tools are read-only and safe to auto-approve. Ask your client to confirm before Server Timers write or cancel calls, and before enabling a webhook.
- Network endpoints: `https://eggtimer.app/api/mcp`, `https://eggtimer.app/api/mcp/server`. Nothing runs locally.

Policies: [Privacy](https://eggtimer.app/privacy) · [Terms](https://eggtimer.app/terms). Vulnerability reports: [SECURITY.md](SECURITY.md).

## Listings

- Official MCP Registry ([registry.modelcontextprotocol.io](https://registry.modelcontextprotocol.io)): [`app.eggtimer/eggtimer`](https://registry.modelcontextprotocol.io/v0.1/servers/app.eggtimer%2Feggtimer/versions/latest) · [`app.eggtimer/server-timers`](https://registry.modelcontextprotocol.io/v0.1/servers/app.eggtimer%2Fserver-timers/versions/latest). Copies of both records: [`server-json/`](server-json/).
- Glama: [glama.ai/mcp/connectors/app.eggtimer/eggtimer](https://glama.ai/mcp/connectors/app.eggtimer/eggtimer)
- mcpservers.org: [mcpservers.org/servers/eggtimer-app](https://mcpservers.org/servers/eggtimer-app)

Machine-readable discovery on the site: [llms.txt](https://eggtimer.app/llms.txt) · [OpenAPI](https://eggtimer.app/openapi.json) · [recipe schema](https://eggtimer.app/recipe.schema.json) · [linking contract](https://eggtimer.app/linking.md).

## Support

- Email: [support@eggtimer.app](mailto:support@eggtimer.app)
- Support page: [eggtimer.app/support](https://eggtimer.app/support)
- Issues with these docs: open an issue in this repository.

Eggtimer.app is operated by Gotoh Industries LLC.

## License

The documentation, snippets and manifests in this repository are released under the [MIT License](LICENSE). That license covers this repository's contents only. It does not grant rights to the Eggtimer.app service, its source code, name or logo; use of the service is governed by the [Terms of Service](https://eggtimer.app/terms).
