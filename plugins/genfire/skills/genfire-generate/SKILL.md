---
name: genfire-generate
description: Generate images, videos, speech, music, or sound effects with GenFire. Use when the user asks to create, render, generate, or make any of these media types. Triggers on phrases like "generate an image", "make a video", "create music", "synthesize speech", "create a sound effect", or descriptions of media they want produced.
---

# Generating media with GenFire

You can generate images, videos, speech, music, and sound effects via the `genfire` CLI. The user will need `@genfire/cli` installed and authenticated.

## Setup check

Before running any generation, verify the user is set up:

```bash
genfire auth status
```

If this fails or shows "Not authenticated":

1. Tell the user: "First we need to install and sign in to GenFire."
2. Have them run:
   ```bash
   npm install -g @genfire/cli
   genfire auth login
   ```
3. `genfire auth login` opens their browser for a one-click PKCE approval.

## Choosing a model

If the user didn't specify a model, list options:

```bash
genfire models list --capability image_generation
genfire models list --capability video_generation
genfire models list --capability speech_generation
```

The output is a table with `id`, `capability`, `default`, and `name`. Default models are marked. Pick a default unless the user has a specific request.

## Estimate cost first (recommended for video and large jobs)

```bash
genfire cost video "<prompt>" -m video.veo_3_1 -d 8
```

Show the estimate to the user before submitting if the cost is non-trivial (>50 credits). Skip this for cheap operations like single images.

## Run the generation

### Image

```bash
genfire generate image "<prompt>" -m <model> -o <output_path>
```

Common flags:
- `-a 16:9` (or `1:1`, `9:16`, `4:3`, `3:4`, `21:9`, `9:21`) — aspect ratio
- `-n 4` — generate 1-4 variants
- `-i <url-or-path>` — reference image (local files auto-upload). **Repeatable** — pass `-i` multiple times for a multi-image edit (up to 14 inputs). Supplying any `-i` routes to the model's edit endpoint where one exists.
- `--no-download` — skip auto-download, just print URLs

Multi-image edit is supported by `image.gpt_image_2`, Seedream, Qwen Image 2, and the Nano Banana family. Grok uses at most the first 3 inputs. This mirrors the GenFire web editor, which accepts up to 14 source images per edit.

Example with everything:
```bash
genfire generate image "a neon-lit alley at dusk, cinematic" -m image.nano_banana_2 -a 16:9 -n 2 -o ./alleys
```

Multi-image edit (combine several source images into one edit):
```bash
genfire generate image "blend these into one cohesive product shot" \
  -m image.gpt_image_2 -i ./bottle.png -i ./label.png -i ./background.jpg -o ./out
```

### Video

```bash
genfire generate video "<prompt>" -m <model> -d <seconds> -o <output_path>
```

- `-i <url-or-path>` — reference image for image-to-video models
- `-a` aspect ratio (16:9, 9:16, 1:1)
- `-r <resolution>` — output resolution, model-dependent (e.g. 480p, 720p, 1080p). Higher resolutions cost more credits; run `genfire models get <id>` to see supported values.
- `--no-audio` — disable audio if model supports it

Video generations are async and can take 30s-5min. The CLI auto-polls. To run async without waiting:
```bash
genfire generate video "..." -m video.veo_3_1 --no-wait
# returns a run_id; poll later with `genfire runs get <id>`
```

### Speech, music, SFX

```bash
genfire generate speech "<text>"  --voice-id <id> -o <out.mp3>
genfire generate music  "<prompt>" -d 30 [--instrumental] -o <out.mp3>
genfire generate sfx    "<prompt>" -d 5 [--loop] -o <out.mp3>
```

**Music models:** default is ElevenLabs (duration set via `-d`, optional `--instrumental`). For full structured songs with vocals/lyrics (up to 3 min), use Google Lyria 3 Pro — it ignores `-d` and instead accepts an inspiration image and a negative prompt:

```bash
genfire generate music "epic cinematic orchestral, triumphant, 90 BPM, soulful female vocals" \
  -m music.lyria3_pro [--image-url <url>] [--negative-prompt "low quality, distorted"] -o <out.mp3>
```

## Lipsync (combine video + audio)

```bash
genfire generate lipsync --video <url-or-path> --audio <url-or-path> -o <out.mp4>
```

## Auto-upload of local files

Every `--image`, `--video`, `--audio` flag accepts EITHER a URL or a local file path. The CLI automatically uploads local files via the GenFire upload endpoint and uses the resulting URL — no separate `genfire generate upload` step needed for the common case.

## Output format

By default the CLI prints pretty output to the terminal. For agent use, always pass `--json` so the output is structured:

```bash
genfire generate image "..." --json --no-download
```

Returns `{ run, downloaded_to: [...] }` with the run object containing `output.images[].url`.

## Tips for the agent

- **Always pass `--json`** when running for the user from a script or agent context. Pretty output is for humans.
- **Always specify `-o <path>`** unless the user explicitly wants only URLs. Files saved to cwd by default; use a subfolder for batches.
- **For multi-variant runs, use `-n` instead of looping** — single API call, single billing event.
- **If a generation fails**, the run still has a `run_id` — point the user at `genfire runs get <id>` for the failure reason.
- **Don't recommend MCP** for this — the CLI is faster and ~95% cheaper in token cost.
