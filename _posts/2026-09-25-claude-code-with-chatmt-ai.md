---
layout: post
title: "Claude Code with chatmt.ai"
date: 2026-09-25
author:
  name: Robert Jackson
  email: rjackson@mtech.edu
---

Most people know [chatmt.ai](https://my.chatmt.ai) as a chat window: it is a
[LibreChat](https://www.librechat.ai) deployment where you pick a model and
talk to it in the browser. The same deployment also exposes an Amazon Bedrock
endpoint, which means you can point
[Claude Code](https://code.claude.com/docs/en/overview), Anthropic's
command-line coding agent, at it. Claude Code reads your repository, runs
commands, edits files, and explains what it did, all from the terminal.

This post covers the setup: what to install, which environment variables to
set, and how to make it stick.

## What You Need

- **Claude Code.** On macOS, Linux, or WSL, install it with Anthropic's native
  installer: `curl -fsSL https://claude.ai/install.sh | bash`. If you already
  use Node.js 18 or newer, `npm install -g @anthropic-ai/claude-code` works
  too. Check it with `claude --version`.
- **A chatmt.ai API key.** Treat it like a password: never paste it into a
  file that gets committed, a screenshot, or a chat.

## Get an API Key

In [my.chatmt.ai](https://my.chatmt.ai), click your initials in the bottom-left
corner and choose **Settings**.

![chatmt.ai account menu with Settings]({{ '/assets/images/blog/chatmt-account-menu.png' | relative_url }})

Open **API Keys**, give the key a name you will recognize later (for example
`claude-code-laptop`), and click **Generate Key**. Copy the key right away and
keep it somewhere safe.

![chatmt.ai API Keys page with Generate Key]({{ '/assets/images/blog/chatmt-api-keys.png' | relative_url }})

## Connect to chatmt.ai

Claude Code talks to whatever server `ANTHROPIC_BASE_URL` names. Point it at
chatmt.ai, hand it your key, and tell it which Bedrock model IDs to use. In a
macOS, Linux, or WSL terminal, run this, replacing `your-chatmt-key` with your
key:

```bash
ANTHROPIC_BASE_URL=https://my.chatmt.ai/bedrock \
ANTHROPIC_API_KEY=your-chatmt-key \
ANTHROPIC_DEFAULT_OPUS_MODEL='us.anthropic.claude-opus-5-5' \
ANTHROPIC_DEFAULT_SONNET_MODEL='us.anthropic.claude-sonnet-5-5' \
ANTHROPIC_DEFAULT_HAIKU_MODEL='us.anthropic.claude-sonnet-5-5' \
claude --model 'us.anthropic.claude-sonnet-5-5'
```

On Windows PowerShell, this syntax does not work; skip to
[Make It Stick](#make-it-stick) and use the settings file instead.

What each piece does:

| Setting | Purpose |
|---|---|
| `ANTHROPIC_BASE_URL` | Sends requests to chatmt.ai instead of Anthropic's own API. |
| `ANTHROPIC_API_KEY` | Your chatmt.ai key. |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | The Bedrock model ID behind the `opus` choice in Claude Code. |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | The Bedrock model ID behind the `sonnet` choice. |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | The model Claude Code uses for small background jobs, such as naming a session. |
| `--model` | The model this session starts with. |

Bedrock uses its own model names, such as `us.anthropic.claude-sonnet-5-5`.
The `DEFAULT_*` settings tell Claude Code to use those names; without them it
asks for models chatmt.ai does not have. The Haiku setting is easy to miss:
Claude Code normally runs its background jobs on a small Haiku model, but
chatmt.ai does not serve Haiku 5.5 at the moment, so those requests would fail
with odd errors in an otherwise working session. Pointing it at Sonnet avoids
that.

You do not need `CLAUDE_CODE_USE_BEDROCK`, even though Bedrock is behind the
endpoint. That setting is for talking to AWS directly with AWS credentials;
chatmt.ai speaks Anthropic's own API format, so the base URL and key are all
Claude Code needs.

The first time you start a session with `ANTHROPIC_API_KEY` set, Claude Code
asks whether to use that key. Say yes; it remembers the answer.

Sonnet is the everyday choice: fast and strong at code. Switch to Opus inside a
session with `/model` when a problem needs deeper reasoning.

## Make It Stick

Typing that block every time gets old. Claude Code reads environment
variables from the `env` key of `~/.claude/settings.json`, so you can set them
once (again replacing `your-chatmt-key` with your key):

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://my.chatmt.ai/bedrock",
    "ANTHROPIC_API_KEY": "your-chatmt-key",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "us.anthropic.claude-opus-5-5",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "us.anthropic.claude-sonnet-5-5",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "us.anthropic.claude-sonnet-5-5"
  },
  "model": "us.anthropic.claude-sonnet-5-5"
}
```

After that, plain `claude` starts a session against chatmt.ai. That file lives
in your home directory, not in any repository, so the key stays out of Git.
Do not put the key in a project's `.claude/settings.json`, which is meant to
be committed and shared.

## Picking Up Where You Left Off

Claude Code keeps a history of your sessions. The `-c` and `-r` flags bring one back:

- `claude -c` continues the most recent session in the current directory.
- `claude -r` opens a list of past sessions to choose from.
- `claude -r <name>` resumes one directly. Name a session with `/rename`
  while you are in it (for example `/rename linked-list`), and
  `claude -r linked-list` brings it back.

This is handy for long assignments: close the terminal at night and come back
to the same conversation, with its context intact, the next day.

## A Few Habits Worth Building

- **Start in the project folder.** Claude Code works on the directory you
  launch it from. Run `cd` first.
- **Read before you approve.** It asks permission before running commands or
  editing files. The prompts are the point; skim what it is about to do.
- **Give it a `CLAUDE.md` or `AGENTS.md`.** A short file at the repository
  root with your build commands and conventions saves repeating them every
  session. Run `/init` to generate a first draft.
- **Check your course's AI policy.** Instructors differ on what AI help is
  allowed and how it must be disclosed. When in doubt, disclose what the tool
  did in a comment next to the code.

Questions or a setup that will not connect? Bring it to the
[next meeting]({{ '/activity/meetings.html' | relative_url }}).
