---
name: genfire-runs
description: Inspect, debug, and re-download past GenFire generations. Use when the user asks about a previous generation, wants to know what happened to a job, asks to redownload an old output, asks why something failed, or wants to see their generation history.
---

# Inspecting GenFire runs

A "run" is a single generation request — image, video, audio, workflow execution, etc. Every run has a stable `id` (e.g., `run_a1b2c3...`). Use these commands to look up past work.

## List recent runs

```bash
genfire runs list
```

Optional filters:
- `--limit 50` — max 100
- `--status completed` (or `queued`, `processing`, `failed`)
- `--capability video_generation` (or `image_generation`, `speech_generation`, etc.)

For agent scripting, always pass `--json`:
```bash
genfire runs list --status completed --capability video_generation --limit 5 --json
```

Returns `{ object: 'list', data: [{ id, status, capability, model, created_at, ... }] }`.

## Inspect one run

```bash
genfire runs get <run_id>
```

Shows status, capability, model, timestamps, error info if failed, and the credit usage.

## Re-download outputs

```bash
genfire runs output <run_id> -o <destination>
```

Downloads all output URLs from a completed run to the destination. If `-o` is omitted, just prints the URLs (useful for piping to `xargs curl`).

## Common workflows

### "What was that video I made yesterday?"

```bash
genfire runs list --capability video_generation --limit 10
```

User picks the right `id` from the list, then:
```bash
genfire runs get <id>
```

### "Re-download the latest completed image"

```bash
runId=$(genfire runs list --status completed --capability image_generation --limit 1 --json | jq -r '.data[0].id')
genfire runs output "$runId" -o ./
```

### "Why did this fail?"

```bash
genfire runs get <id>
```

The `Error:` line will show the failure code and message. Common failures:
- `content_policy_violation` — prompt was rejected by the model's safety filter
- `insufficient_credits` — account ran out
- `provider_timeout` — upstream model took too long; safe to retry

## Tips for the agent

- **Always show the `id`** when displaying runs to the user — it's the only stable handle for follow-up actions.
- **Don't paginate beyond 100 runs**. If the user needs more history, point them at the dashboard.
- **For failed runs**, suggest retry via the original generation command, not by re-querying the run (runs are immutable).
- **If the user asks "what's running?"**, use `genfire runs list --status processing` (or `queued`).
