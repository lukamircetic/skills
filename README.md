# Codex Skills

Reusable Codex skills by [Luka Mircetic](https://github.com/lukamircetic).

## Available skills

### Voice Orchestrator

[`orchestrate-voice-work`](skills/orchestrate-voice-work/SKILL.md) keeps a ChatGPT Voice session focused on orchestration. It captures spoken briefs, dispatches substantive work to visible project-specific Codex tasks using GPT-5.6 Sol, and monitors or redirects those tasks until their definitions of done are met.

Ask Codex to install it:

> Use `$skill-installer` to install `skills/orchestrate-voice-work` from `lukamircetic/skills`.

Or run the bundled installer directly:

```bash
python3 ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo lukamircetic/skills \
  --path skills/orchestrate-voice-work \
  --method git
```

Start a new Codex task after installation so the skill catalog refreshes. Then invoke it with `$orchestrate-voice-work`, or say **“Start conductor mode.”** During capture, say **“Dispatch that”** when the brief is ready.

## Repository layout

Each skill lives in its own directory under [`skills/`](skills/). A skill's behavior is defined by `SKILL.md`; optional UI metadata lives in `agents/openai.yaml`.
