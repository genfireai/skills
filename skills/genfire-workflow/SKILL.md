---
name: genfire-workflow
description: Run published GenFire workflows (multi-step graph pipelines for storyboards, hook packs, branded ads, product shoots). Use when the user asks for a higher-level outcome like "make me a storyboard", "give me hooks for this video", "produce a UGC ad" — anything that's a pipeline of multiple generation steps rather than a single image/video. Workflows are GenFire's differentiator vs single-shot CLIs.
---

# Running GenFire workflows

Workflows are multi-step graph pipelines published by GenFire (or the user's team). Each one accepts a typed JSON input and produces structured output. Use these instead of stitching together multiple `genfire generate` calls when a workflow already exists.

## Discover available workflows

```bash
genfire workflow list
```

Returns a table of `id`, `name`, `status`. Inspect a specific one to see its input schema:

```bash
genfire workflow get <id>
```

This prints the full JSON schema of inputs and outputs. **Read it carefully** before running — workflows reject mis-typed inputs.

## Run a workflow

```bash
genfire workflow run <id> --inputs '<json>' -o <output_dir>
```

`--inputs` accepts EITHER:
- An inline JSON string (use `'single quotes'` to avoid shell escaping)
- A path to a JSON file: `--inputs vars.json`

Example:
```bash
genfire workflow run storyboard \
  --inputs '{"prompt":"a sci-fi rooftop chase","scenes":4,"style":"cinematic"}' \
  -o ./storyboard-output
```

## Async execution

Workflows can take several minutes. The CLI polls automatically and downloads outputs to `-o` when complete. To submit and exit immediately:

```bash
genfire workflow run storyboard --inputs vars.json --no-wait
# prints a run_id; check later with `genfire runs get <id>`
```

## Tips for the agent

- **Always inspect the input schema first** with `genfire workflow get <id>` — workflows have strict validation.
- **Use the user's natural-language description to construct inputs** — translate "make me a 4-scene storyboard about a sci-fi chase" into the workflow's specific input shape.
- **For batch operations across many inputs**, use `genfire batch create` instead of looping `workflow run` (single billing event, parallel execution).
- **If a workflow doesn't exist for the user's request**, fall back to chaining `genfire generate` calls. Don't invent workflow IDs — only use ones that appear in `genfire workflow list`.
- **Workflows reuse the same auth as `genfire generate`** — no separate setup needed.
