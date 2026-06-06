# Konklave

> **Multi-agent AI CLI** — a council of specialized AI models that plan, code, review, test, and audit together so you don't have to.

[![Python 3.11+](https://img.shields.io/badge/python-3.11%2B-blue)](https://python.org)
[![OpenRouter](https://img.shields.io/badge/powered%20by-OpenRouter-purple)](https://openrouter.ai)
[![License: MIT](https://img.shields.io/badge/license-MIT-green)](LICENSE)
[![Release: June 15, 2026](https://img.shields.io/badge/release-June%2015%2C%202026-orange)](https://github.com/firatzr/konklave/releases)

---

## What is Konklave?

Most AI coding tools send your request to a single model and hope for the best. Konklave runs a **team**.

```
You describe what you want.
  → The Architect researches your codebase and writes a concrete plan.
  → The Coder implements the plan (on a different model family, so blind spots cancel out).
  → The Reviewer checks the result from an independent perspective.
  → The Tester actually runs the code and reads the real output.
  → The Auditor gives a final go/no-go.
```

Every role runs on the model you choose. You can mix providers freely — the point is that different model families catch each other's mistakes.

---

## Demo

> _Screenshots and a demo GIF will be published at release (June 15, 2026)._

<!-- TUI full-screen screenshot -->
<!-- Pipeline progress bar -->
<!-- Per-agent cost breakdown -->

---

## Features

### Six Specialized Roles

| Role | What it does |
|------|-------------|
| **Architect** | Reads the codebase, spawns parallel Researchers, writes a step-by-step plan |
| **Coder** | Implements the plan — assigned a different provider than the reviewers by design |
| **Reviewer** | Reads the implementation and checks it against the plan and your coding standards |
| **Tester** | Runs actual commands and verifies the result with real evidence, not guesses |
| **Auditor** | Final pass: does the output actually solve the original request? |
| **Researcher** | Lightweight parallel lookups — runs in swarms to gather context fast |

### Work Modes — One Command to Go Deeper

```bash
konklave run "your task" --work sehr_schnell   # instant, no research
konklave run "your task" --work schnell         # quick research, light review
konklave run "your task"                        # balanced default (mitte)
konklave run "your task" --work lang            # thorough, multi-round review
konklave run "your task" --work autonom         # autonomous loop until PERFECT (with budget cap)
```

| Mode | Pipeline depth |
|------|---------------|
| `sehr_schnell` | No research → code → done |
| `schnell` | One researcher, quick review |
| `mitte` | Research → plan → code → review → test |
| `lang` | Multi-researcher, multiple review/test rounds |
| `sehr_lang` | Exhaustive edge cases, trade-off exploration |
| `autonom` | Loop until Auditor approves or budget cap hit |
| `autonom-endlos` | Loop until you stop it |

### Swarm — Parallel Sub-Agents

Agents can spawn their own sub-agents for genuinely independent work. The Architect dispatches three Researchers at once instead of one-by-one. You control the aggressiveness:

```
:swarm off            # each agent works alone
:swarm encouraged     # agents use parallel helpers when it makes sense (default)
:swarm aggressiv      # maximally fan out to sub-agents
```

### Human-in-the-Loop Controls

Inject a message mid-run, change the direction, or just watch. Slash commands work in the TUI at any time:

```
:say "focus on the auth module"    inject a message into the active step
:work autonom                      switch to autonomous mode on the fly
:ask proaktiv                      make the Architect ask you more questions
:cost                              show current USD spend
:models                            reassign any role to a different model
:exit                              clean stop at the next step boundary
```

### YAML Pipelines

Pipelines are plain YAML — no Python required. Built-in templates cover the common cases; write your own for your workflow:

```
build_feature  →  research + plan + code + review + test + audit
quick          →  one-shot coder, no review
refactor       →  architect-led refactor with before/after review
debug_issue    →  reproduction → root cause → fix → regression test
research       →  deep research swarm, no code changes
autonomous_loop→  loop until done (budget-capped)
```

### Persistent Sessions

Every step is written to SQLite as it completes. Interrupted runs can be resumed from exactly where they stopped:

```bash
konklave resume   # pick a session from the list
```

Browse snapshots, rewind to any checkpoint, compare costs across runs.

### Cost Tracking

Every token is counted. The TUI shows live USD spend per agent, per model, and per run. Set a cost-warning threshold in your config and Konklave will alert you before a run gets expensive.

### Memory & Skills

After successful runs, agents write lessons to a persistent memory file. Recurring patterns are detected and saved as reusable skills — future runs pick them up automatically without re-discovering them.

### Web Dashboard

An optional FastAPI dashboard gives you session history, cost analytics, and remote config — useful when running Konklave on a server or in CI.

---

## Architecture

```
 ┌──────────────┐      ┌─────────────────────────────────────────────────┐
 │   You        │      │                   Konklave                       │
 │  (TUI / CLI) │─────►│                                                  │
 └──────────────┘      │  ┌───────────┐   ┌──────────────────────────┐   │
                       │  │ Conductor │──►│  Pipeline (YAML)         │   │
                       │  │           │   │  sequential / parallel / │   │
                       │  │           │   │  consensus / loop steps  │   │
                       │  └───────────┘   └────────────┬─────────────┘   │
                       │                               │                  │
                       │        ┌──────────────────────┼──────────┐      │
                       │        ▼           ▼           ▼          ▼      │
                       │  ┌──────────┐ ┌────────┐ ┌────────┐ ┌────────┐ │
                       │  │Architect │ │ Coder  │ │Reviewer│ │ Tester │ │
                       │  │ (Sonnet) │ │(DeepSeek│ │(Sonnet)│ │(Haiku) │ │
                       │  └────┬─────┘ └────────┘ └────────┘ └────────┘ │
                       │       │ spawns                                    │
                       │  ┌────▼──────────────────┐                      │
                       │  │ Researcher ×N (swarm)  │                      │
                       │  └───────────────────────┘                      │
                       │                               OpenRouter API     │
                       └─────────────────────────────────────────────────┘
```

- **Agents** are roles, each with its own persona, model, and isolated conversation history.
- **Pipelines** are explicit workflows in YAML: sequential steps, parallel groups, consensus votes, loop steps, and human-approval gates.
- **Conductor** orchestrates the pipeline, handles failures, retries on fallback models, and routes events.
- **Event Bus** decouples the TUI, Conductor, agents, and tools — everything communicates asynchronously.

### Default Model Panel

| Role | Default model | Reason |
|------|--------------|--------|
| Architect | `anthropic/claude-sonnet-4.6` | Strong planning and tool use |
| Coder | `deepseek/deepseek-chat-v3` | Fast, cheap, excellent at code |
| Reviewer | `anthropic/claude-sonnet-4.6` | Different family from coder → independent check |
| Tester | `anthropic/claude-haiku-4.5` | Fast and cheap for test gates |
| Researcher | `anthropic/claude-haiku-4.5` | High-volume parallel queries |
| Auditor | `anthropic/claude-sonnet-4.6` | Strict final quality gate |

Swap any role to any model on [OpenRouter](https://openrouter.ai/models) with `:models` in the TUI or via `konklave init`.

---

## Installation

**Requirements:** Python 3.11+, an [OpenRouter](https://openrouter.ai/keys) API key.

```bash
pip install konklave
konklave init          # interactive setup: language, API key, default models
konklave               # launch TUI
```

**Windows one-click:** double-click `start.bat` — it activates the environment and launches the TUI automatically.

---

## Quickstart

```bash
# One-shot headless run
konklave run "Add input validation to the login form"

# With an explicit work mode
konklave run "Refactor the payment module" --work lang

# Interactive TUI (full control)
konklave

# Resume a previous session
konklave resume

# Test your API key
konklave ping
```

---

## Configuration

```bash
konklave config        # show current settings
konklave init          # re-run setup wizard
```

Config is stored at `~/.konklave/config.toml`. The API key is stored securely in the OS keyring (Windows Credential Manager / macOS Keychain / Linux SecretService) — never in plain files.

---

## Requirements

- Python 3.11 or newer
- [OpenRouter](https://openrouter.ai) API key (free tier available)
- Windows 10/11, macOS 13+, or Linux

---

## License

MIT — see [LICENSE](LICENSE).

---

<p align="center">
  Built with <a href="https://openrouter.ai">OpenRouter</a> ·
  UI powered by <a href="https://github.com/Textualize/textual">Textual</a>
</p>
