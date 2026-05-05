---
name: genfire-account
description: Check the user's GenFire account info, credit balance, available models, or model pricing. Use when the user asks about their account, credits, billing, what models are available, what something costs, or wants to estimate the cost of a generation before running it.
---

# GenFire account, credits, and models

## Account snapshot

```bash
genfire account
```

Returns display name, email, plan, and current credit balance.

For a tighter check, `genfire credits` returns just the balance:
```bash
genfire credits
```

## Available models

```bash
genfire models list
```

Filter by capability:
```bash
genfire models list -c image_generation
genfire models list -c video_generation
genfire models list -c speech_generation
genfire models list -c music_generation
genfire models list -c sound_effect
```

The `default` column marks which model is used when the user doesn't specify `-m`.

For full details on a specific model (limits, capabilities, supported aspect ratios):
```bash
genfire models get <model_id>
```

## Pricing

```bash
genfire models pricing
```

Shows credits-per-call (or credits-per-second / credits-per-image, depending on the unit) for every model.

## Estimate before generating

Always recommend checking cost when the user asks for a video or large batch:

```bash
genfire cost video "..."  -m video.veo_3_1 -d 8
genfire cost image "..."  -m image.nano_banana_2 -n 4
genfire cost music "..."  -m music.elevenlabs -d 60
```

Output includes per-call cost, multiplier (count or duration), estimated total, and the user's current balance.

## Tips for the agent

- **If credits are low** (less than 100, or less than the estimated cost), warn the user before submitting expensive jobs.
- **Don't hard-code model IDs** in scripts you write for the user — `models list --json` is the source of truth and changes over time.
- **Cost estimates are approximate** — actual cost may differ slightly based on resolution, audio, and model-specific multipliers.
