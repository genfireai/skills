---
name: genfire-faceless-reels
description: Generate faceless reels — vertical (9:16) short videos with AI script, voiceover, style-locked images, music, and burned-in captions — and manage recurring auto-posting "Stories". Use when the user asks to "make a faceless reel", "create a TikTok/Reels/Shorts video", "automate short-form content", "set up a daily reel", or wants a narrated vertical short on a topic (true crime, mysteries, fun facts, motivation, etc.) without filming anything.
---

# Generating faceless reels with GenFire

A **faceless reel** is a vertical (9:16) short produced end-to-end from a single topic: an LLM writes a script, it's narrated with a TTS voice, style-locked images are generated per beat, optional background music is added, and word-level captions are burned in. It's GenFire's TikTok/Reels/Shorts pipeline — no camera, no editing.

Two modes:
- **One-off** — generate a single reel now (`genfire generate faceless-reel`).
- **Subscriptions ("Stories")** — a niche + daily schedule that auto-generates reels (`genfire faceless-reels subscriptions`).

## Setup check

```bash
genfire auth status
```

If not authenticated:
```bash
npm install -g @genfire/cli
genfire auth login   # opens browser for one-click PKCE approval
```

## Discover the catalogs first

A reel is configured by a **niche preset** (topic + tone), a **visual style** (art medium), a **caption preset** (font/animation), and optional **music**. List the valid ids before generating:

```bash
genfire faceless-reels presets          # niche presets → --preset
genfire faceless-reels styles           # visual styles  → --style
genfire faceless-reels caption-presets  # caption presets → --caption-preset
genfire faceless-reels music-presets    # curated tracks  → --music-preset
```

Each preset has a recommended style, so `--style` is optional. Pick a preset that matches the user's topic (e.g. `mystery`, `true-crime`, `fun-facts`, `stoic-motivation`).

## Estimate cost first (recommended)

Reels bill for images + voiceover + (optional) music — cost scales with duration and scene count.

```bash
genfire faceless-reels estimate-cost --preset mystery --duration 45 --music-source preset
```

Show the user the `total` before generating if it's non-trivial.

## Generate one reel

```bash
genfire generate faceless-reel "<topic>" --preset <id> [flags]
```

Common flags:
- `--preset <id>` — niche preset (topic + tone)
- `--style <id>` — visual style (defaults to the preset's recommended style)
- `-d, --duration <seconds>` — target length, **10–120** (drives script + scene count)
- `--caption-preset <id>` — caption font/animation (default: hormozi)
- `--caption-animation <name>` — highlight | pop | typewriter | classic | background
- `--voice-id <id>` — TTS voice (see `genfire voices list`)
- `--direction "<text>"` — extra creative direction for the script
- `--music-source <none|preset|ai|library>` — background music source (default: none)
  - `--music-preset <id>` with `--music-source preset`
  - `--music-prompt "<text>"` with `--music-source ai` (AI-generated track; adds cost)

Example:
```bash
genfire generate faceless-reel "the unsolved disappearance of the Sodder children" \
  --preset true-crime --duration 50 --caption-preset hormozi \
  --music-source preset --music-preset unsolved-mystery -o ./reels
```

For a fully custom story (no niche preset), just pass a descriptive topic and `--direction`; the model writes from that.

### Async + waiting

Reels render in **minutes** (script → voiceover → images → music → compose → captions). The CLI auto-polls with a 20-minute default window. To submit and exit immediately:

```bash
genfire generate faceless-reel "..." --preset mystery --no-wait
# prints a run_id; check later with `genfire runs get <id>`
```

The completed run's `output` carries `video_url`, `reel_id`, `script`, `scenes`, and `duration_seconds`.

## Recurring "Stories" (auto-generated reels on a schedule)

A subscription generates reels automatically each day at the times you set.

```bash
genfire faceless-reels subscriptions list

genfire faceless-reels subscriptions create \
  --label "Daily mysteries" --preset mystery \
  --cadence-per-day 1 --slots 18:00 --timezone America/New_York \
  --topic-source ai-auto

genfire faceless-reels subscriptions update <id> --disable   # pause
genfire faceless-reels subscriptions update <id> --enable    # resume
genfire faceless-reels subscriptions delete <id>
```

Scheduling rules:
- `--cadence-per-day <n>` — reels per day, **1–6**.
- `--slots <list>` — comma-separated local `HH:mm` times; **the count must equal `--cadence-per-day`** (e.g. `--cadence-per-day 2 --slots 09:00,18:00`).
- `--timezone <tz>` — IANA timezone for the slots.
- `--topic-source ai-auto` (fresh ideas each run) or `--topic-source user-list --topic-seeds "topic a,topic b,topic c"` (rotates through your list).

### Generate one now from a subscription

```bash
genfire faceless-reels subscriptions run-now <id> [--topic "<override>"]
```

Returns a run (poll with `genfire runs get <id>`). Errors with 409 if a reel is already generating for that subscription.

## Output format

For agent use, pass `--json`:

```bash
genfire generate faceless-reel "..." --preset mystery --json --no-download
```

Returns `{ run, downloaded_to: [...] }` with `run.output.video_url`.

## Tips for the agent

- **List the catalogs before generating** — preset/style/caption/music ids are validated; don't guess them.
- **Match the niche preset to the topic** — `true-crime` for crime, `mystery` for unsolved cases, `fun-facts`/`mind-blowing-facts` for trivia, `stoic-motivation` for motivation, `space-science` for science.
- **Estimate cost for longer reels** (`--duration` > ~30s) before submitting.
- **Reels are slow** — they take minutes. Keep the auto-poll, or use `--no-wait` and hand the user a `run_id`.
- **For recurring content, prefer a subscription** over scripting a loop of one-off generations — it schedules and rotates topics for you.
- **If a generation fails**, the run still has a `run_id` — point the user at `genfire runs get <id>` for the failure reason.
- **Don't recommend MCP** for this — the CLI is faster and far cheaper in token cost.
