# GenFire Skills

Skills for [Claude Code](https://docs.claude.com/en/docs/claude-code), Cursor, OpenCode, and other agent platforms that teach the agent how to use the GenFire CLI correctly.

Instead of paying the per-call schema cost of an MCP server, agents read a single Markdown skill file once and then shell out to `genfire` directly. Same capability, far lower token cost.

## Install

### Claude Code (recommended)

```bash
/plugin marketplace add genfireai/skills
/plugin install genfire-generate@genfire
/plugin install genfire-workflow@genfire
/plugin install genfire-runs@genfire
/plugin install genfire-account@genfire
```

Or install all at once after adding the marketplace:

```bash
/plugin install genfire-generate@genfire genfire-workflow@genfire genfire-runs@genfire genfire-account@genfire
```

### Manual install (any agent that supports SKILL.md)

```bash
git clone https://github.com/genfireai/skills.git ~/.claude/skills/genfire
```

## Skills included

| Skill | When the agent uses it |
|---|---|
| [`genfire-generate`](./skills/genfire-generate/SKILL.md) | User asks to generate an image, video, speech, music, or sound effect |
| [`genfire-workflow`](./skills/genfire-workflow/SKILL.md) | User asks to run a published workflow (storyboard, hook pack, etc.) or wants a multi-step pipeline |
| [`genfire-runs`](./skills/genfire-runs/SKILL.md) | User asks about past generations, wants to re-download an output, or wants to debug a failed run |
| [`genfire-account`](./skills/genfire-account/SKILL.md) | User asks about their account, credit balance, or available models |

## Prerequisites

Each skill expects the user to have `@genfire/cli` installed and authenticated:

```bash
npm install -g @genfire/cli
genfire auth login
```

The skills will instruct the agent to nudge the user through these steps if they haven't run them yet.

## Why skills, not MCP?

MCP injects every tool's full JSON schema into the agent's context on every request. With ~20 GenFire endpoints, that's 8-15k input tokens burned before the agent reasons.

Skills load the prompt-side guidance once (~500 tokens) and have the agent shell out via the standard Bash tool. Same capability, ~95% lower token cost. See [the launch post](https://genfire.ai/blog/cli) for benchmarks.

You can still use the [GenFire MCP server](https://github.com/genfireai/sdk) when you need conversational state or are running in an environment without shell access (e.g., chat-only clients).

## License

MIT — see [LICENSE](./LICENSE).
