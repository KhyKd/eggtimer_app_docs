# Tool reference

Every tool exposed by the two Eggtimer.app MCP servers, with inputs, annotations and behaviour. Tool names are stable and each has its own anchor (for example [`#create_timer`](#create_timer)) for deep links.

- Public link server: `https://eggtimer.app/api/mcp` (no auth), 7 tools.
- Server Timers: `https://eggtimer.app/api/mcp/server` (`Authorization: Bearer <workspace API key>`), the same 7 tools plus 11 workspace tools.

The authoritative list is always the server's own `tools/list`. Every tool publishes an `outputSchema` for its `structuredContent`.

## Contents

- [Public tools](#public-tools): [`get_capabilities`](#get_capabilities) · [`list_presets`](#list_presets) · [`get_preset`](#get_preset) · [`create_timer`](#create_timer) · [`create_clock_link`](#create_clock_link) · [`create_recipe`](#create_recipe) · [`get_integrations`](#get_integrations)
- [Server timer tools](#server-timer-tools): [`create_server_timer`](#create_server_timer) · [`get_server_timer`](#get_server_timer) · [`list_server_timers`](#list_server_timers) · [`cancel_server_timer`](#cancel_server_timer)
- [Server routine tools](#server-routine-tools): [`create_server_routine`](#create_server_routine) · [`get_server_routine`](#get_server_routine) · [`get_routine_events`](#get_routine_events) · [`list_server_routines`](#list_server_routines) · [`list_server_events`](#list_server_events) · [`cancel_server_routine`](#cancel_server_routine)
- [Workspace tool](#workspace-tool): [`get_workspace`](#get_workspace)
- [Annotations summary](#annotations-summary)

## Public tools

All seven: `readOnlyHint: true`, `destructiveHint: false`, `idempotentHint: true`, `openWorldHint: false`. They inspect data or construct links only. They cannot start, stop, observe or synchronise a running browser timer, and they store nothing.

### get_capabilities

Short feature and limits summary. Does not start a timer.

| Input | Type | Required | Notes |
|---|---|---|---|
| `detail` | `"summary"` \| `"full"` | no | Default `summary` (routing and limits). Use `full` only when the complete reference is needed. |

### list_presets

Preset names, schedules and links, without recipe bodies. No inputs.

Presets include Pomodoro 25/5 ([/pomodoro](https://eggtimer.app/pomodoro)), 27/8 ([/pomo](https://eggtimer.app/pomo)), Tabata 20 s / 10 s × 8 ([/tabata](https://eggtimer.app/tabata)), Focus 52/17 ([/52-17](https://eggtimer.app/52-17)), a 20 m beep-test practice routine ([/beep-test](https://eggtimer.app/beep-test)) and a stopwatch ([/stopwatch](https://eggtimer.app/stopwatch)). Call the tool for the current list.

### get_preset

One preset with its full validated recipe, for inspecting or customising steps. Prefer [`list_presets`](#list_presets) for quick recommendations.

| Input | Type | Required | Notes |
|---|---|---|---|
| `id` | string (enum) | yes | A preset id from `list_presets`, e.g. `pomodoro`, `tabata`, `stopwatch`. |

### create_timer

Validate a duration and return a countdown link. The countdown starts when the person opens it, not when the tool runs. No background alarm or automation is scheduled.

| Input | Type | Required | Notes |
|---|---|---|---|
| `duration` | string, 1–120 chars | yes | Relative duration, e.g. `12m`, `1h20m`. Bare numbers mean minutes. Use [`create_recipe`](#create_recipe) for sequences. |
| `label` | string, 1–80 chars | no | Omit unless requested. |
| `theme` | `plain` \| `oat` \| `midnight` \| `clay` \| `lilac` | no | Omit for defaults. |
| `display` | `digits` \| `ring` \| `bar` | no | Omit for defaults. |
| `muted` | boolean | no | Omit for defaults. |

Example: `{"duration": "12m", "label": "Tea"}` returns

```json
{
  "url": "https://eggtimer.app/Tea/for/720s",
  "duration_seconds": 720,
  "label": "Tea",
  "start_behavior": "Countdown starts when the person opens the link.",
  "alarm_note": "Keep the page open; enable and test sound. No background alarm is scheduled."
}
```

### create_clock_link

Link that counts down to the next occurrence of a local clock time, e.g. `9pm` or `17:30`. Each opener uses their own timezone. Does not schedule a server timer.

| Input | Type | Required | Notes |
|---|---|---|---|
| `time` | string, ≤20 chars | yes | `9pm`, `21:00`, `17:30`… |
| `label` | string, ≤80 chars | no | |

Example: `{"time": "9pm", "label": "Dinner"}` returns `https://eggtimer.app/Dinner/at/21:00` with `"timezone": "opener_local"`.

### create_recipe

Validate a data-only multi-step recipe and return its portable preview link (`https://eggtimer.app/#recipe=…`). The person opens it and presses Start. No code runs and no webhook is armed.

| Input | Type | Required | Notes |
|---|---|---|---|
| `recipe` | object | yes | Fields below. Full schema: [recipe.schema.json](https://eggtimer.app/recipe.schema.json). |

Main `recipe` fields:

| Field | Notes |
|---|---|
| `version` | `1` or `2` (required). Version 2 supports fractional seconds, repeated intervals and step cues. |
| `steps` | 1–32 items (required): `{ "label"?: string ≤60, "seconds": 0.1…, "repeat"?: 1–32 }`. |
| `title`, `author` | Optional strings (≤80, ≤60). |
| `cycles` | 1–12 passes through all steps. For work/rest rounds, use alternating steps plus `cycles`; per-step `repeat` repeats that one step. |
| `display` | `digits`, `ring` or `bar`. |
| `theme` | Eight hex colour tokens: `paper`, `panel`, `ink`, `muted`, `line`, `accent`, `white`, `highlight`. |
| `autoAdvance`, `stepSound` | Booleans. |
| `completion` | `{ "message"?: string ≤160, "sound"?: boolean, "webhook"?: boolean }`. `webhook` only suggests a hook; it cannot authorise one. |
| `repeatLabel`, `note` | Optional strings (≤20, ≤320). |

Example input:

```json
{
  "recipe": {
    "version": 2,
    "title": "Soft-boiled eggs",
    "steps": [
      { "label": "Boil water", "seconds": 300 },
      { "label": "Cook eggs", "seconds": 390 },
      { "label": "Ice bath", "seconds": 120 }
    ]
  }
}
```

### get_integrations

Explains optional Zapier completion hooks, setup, authorisation and browser-lifetime limits. Does not connect or send anything. No inputs. Background: [webhooks.md](https://eggtimer.app/webhooks.md).

## Server timer tools

Server Timers endpoint only. These act on real server state in the key's own workspace. Ask the user before creating or cancelling. Background: [server-timers.md](https://eggtimer.app/server-timers.md).

### create_server_timer

Start a countdown on Eggtimer.app's servers. It continues after the browser closes and starts immediately. Supply exactly one of `duration_seconds` or `deadline_at`.

`readOnlyHint: false`, `destructiveHint: false`, `idempotentHint: true`, `openWorldHint: true`.

| Input | Type | Required | Notes |
|---|---|---|---|
| `idempotency_key` | string, 10–80 chars, `[A-Za-z0-9_-]` | yes | One fresh key per intentional timer; reuse it only to retry the same request. |
| `duration_seconds` | integer, 1 … 30 days | one of | |
| `deadline_at` | ISO 8601 date-time with explicit offset | one of | 1 second to 30 days in the future, e.g. `2030-01-01T17:00:00+07:00`. |
| `title` | string, 1–80 chars | no | Avoid sensitive titles. |
| `webhook_url` | string, ≤400 chars | no | Private HTTPS Zapier Catch Hook (`hooks.zapier.com` only). Authorises automatic completion delivery. Never put it in a share link. |
| `max_attempts` | integer 1–5 | no | Default 3. |

### get_server_timer

Read a timer's deadline, status, remaining time and webhook-delivery outcome. Never returns the hook URL. Read-only.

| Input | Type | Required |
|---|---|---|
| `timer_id` | 64-char hex id | yes |

### list_server_timers

List the workspace's active timers and up to 100 recent ones, with delivery status. No webhook URLs are returned. No inputs. Read-only.

### cancel_server_timer

Cancel a timer or a pending delivery retry. An in-flight or accepted webhook cannot be recalled. Repeated cancellation is safe. `destructiveHint: true`, `idempotentHint: true`.

| Input | Type | Required |
|---|---|---|
| `timer_id` | 64-char hex id | yes |

## Server routine tools

A routine runs a finite recipe on the server and records an event for each step and cycle. Background: [server-routines.md](https://eggtimer.app/server-routines.md).

### create_server_routine

Start a finite server-side recipe. It starts immediately and runs without a browser. Automatic whole-second phases only, at most 64 expanded phases and 30 days in total.

`readOnlyHint: false`, `destructiveHint: false`, `idempotentHint: true`, `openWorldHint: true`.

| Input | Type | Required | Notes |
|---|---|---|---|
| `recipe` | object | yes | Same [recipe schema](https://eggtimer.app/recipe.schema.json) as [`create_recipe`](#create_recipe), with the server restrictions above. |
| `idempotency_key` | string, 10–80 chars | yes | As for timers. |
| `webhook_url` | string, ≤400 chars | no | Private Zapier Catch Hook for the selected events. |
| `events` | array of event names | no | Which events to deliver to the hook. Default `["phase.completed"]`. All events are recorded either way. |
| `max_attempts` | integer 1–5 | no | Default 3. |
| `max_lateness_seconds` | integer 1–86400 | no | Default 300. Events later than this are marked expired, not sent. |

Event names: `routine.started`, `phase.started`, `phase.completed`, `cycle.completed`, `routine.completed`, `routine.cancelled`. Deliveries can arrive late, duplicated or out of order; deduplicate on `event_id`.

### get_server_routine

Read a routine's timeline and current step. Read-only. Input: `routine_id` (64-char hex, required).

### get_routine_events

Read one routine's retained events and each delivery outcome. Read-only. Input: `routine_id` (required).

### list_server_routines

List the workspace's active and recent routines. No inputs. Read-only.

### list_server_events

Read the workspace's retained routine event feed, newest first. Keeps at most 1000 events and 7 days. Delivery updates do not create new events. Read-only.

| Input | Type | Required | Notes |
|---|---|---|---|
| `limit` | integer 1–200 | no | |
| `before` / `after` | integer cursor | no | Paginate with the returned cursors. |
| `event` | event name | no | Filter by event type. |

### cancel_server_routine

Cancel future steps and pending deliveries. A claimed or in-flight delivery may still complete. `destructiveHint: true`, `idempotentHint: true`. Input: `routine_id` (required).

## Workspace tool

### get_workspace

Read the workspace's free-plan limits, usage and idle expiry. No secret is returned. No inputs. Read-only.

Key rotation and workspace closure are deliberately **not** MCP tools; they need the recovery key and are done at [automate.html](https://eggtimer.app/automate.html).

## Annotations summary

| Tool | readOnly | destructive | idempotent | openWorld |
|---|---|---|---|---|
| 7 public tools | ✓ | – | ✓ | – |
| `create_server_timer`, `create_server_routine` | – | – | ✓ | ✓ |
| `get_server_timer`, `list_server_timers`, `get_server_routine`, `get_routine_events`, `list_server_routines`, `list_server_events`, `get_workspace` | ✓ | – | ✓ | – |
| `cancel_server_timer`, `cancel_server_routine` | – | ✓ | ✓ | – |

[Back to README](../README.md)
