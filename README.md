# GenFire Skills

Skills for [Claude Code](https://docs.claude.com/en/docs/claude-code), Cursor, OpenCode, and other agent platforms that teach the agent how to use the GenFire CLI correctly.

Instead of paying the per-call schema cost of an MCP server, agents read a single Markdown skill file once and then shell out to `genfire` directly. Same capability, far lower token cost.

## Install

### Claude Code marketplace

```bash
/plugin marketplace add genfireai/skills
```

### Manual install (any agent)

```bash
git clone https://github.com/genfireai/skills.git ~/.claude/skills/genfire
```

Or via the [skills CLI](https://github.com/anthropics/skills):

```bash
npx skills add genfireai/skills
```

## Skills included

| Skill | When the agent uses it |
|---|---|
| [`genfire-generate`](./skills/genfire-generate.md) | User asks to generate an image, video, speech, music, or sound effect |
| [`genfire-workflow`](./skills/genfire-workflow.md) | User asks to run a published workflow (storyboard, hook pack, etc.) or wants a multi-step pipeline |
| [`genfire-runs`](./skills/genfire-runs.md) | User asks about past generations, wants to re-download an output, or wants to debug a failed run |
| [`genfire-account`](./skills/genfire-account.md) | User asks about their account, credit balance, or available models |

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
