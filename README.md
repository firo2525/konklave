<div align="center">

# 🏛️ Konklave

### A council of AI models that thinks, codes, reviews, and ships — together.

<br/>

### 🚀 Releases June 15, 2026

<br/>

[![Python](https://img.shields.io/badge/Python-3.11%2B-3776ab?style=for-the-badge&logo=python&logoColor=white)](https://python.org)
[![OpenRouter](https://img.shields.io/badge/OpenRouter-Powered-7c3aed?style=for-the-badge&logo=openai&logoColor=white)](https://openrouter.ai)
[![License](https://img.shields.io/badge/License-MIT-22c55e?style=for-the-badge)](LICENSE)
[![Release](https://img.shields.io/badge/Release-June%2015%2C%202026-f97316?style=for-the-badge)](https://github.com/firo2525/konklave/releases)

<br/>

> **Stop talking to one AI. Run a team.**
>
> Konklave orchestrates six specialized agents — Architect, Coder, Reviewer, Tester, Researcher, Auditor —
> across any mix of models from OpenRouter. They catch each other's mistakes so you don't have to.

<br/>

</div>

---

## 🎬 Screenshots

<div align="center">

**Full pipeline run — Agents · Chat · Pipeline graph · Live cost tracking**
![Pipeline run](docs/screenshot-pipeline.png)

<br/>

**Swarm mode — 6 Researchers running in parallel, all writing findings simultaneously**
![Swarm of researchers](docs/screenshot-swarm.png)

<br/>

**Architect planning a large feature — 6 Coders spawned in parallel**
![Architect with parallel coders](docs/screenshot-architect.png)

</div>

---

## ✨ How It Works

Instead of one model doing everything, Konklave runs an actual **pipeline of specialists**:

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   📝 You describe the task                                      │
│          │                                                      │
│          ▼                                                      │
│   🏛️  Architect  ──spawns──►  🔍 Researcher × N (parallel)    │
│          │                                                      │
│          ▼  concrete plan                                       │
│   💻  Coder       (different model family — blind spots cancel) │
│          │                                                      │
│          ▼  implementation                                      │
│   🔎  Reviewer  ──╮                                            │
│                   ├──► consensus vote                          │
│   🛡️  Auditor   ──╯                                            │
│          │                                                      │
│          ▼  approved                                            │
│   🧪  Tester     (runs real commands, reads real output)       │
│          │                                                      │
│          ▼                                                      │
│   ✅  Result                                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

Each role runs on **the model you choose**. Mix providers freely — a DeepSeek coder with Anthropic reviewers means genuinely independent cross-checks, not just one model second-guessing itself.

---

## 🧠 Six Specialized Roles

| Role | What it does |
|:----:|:------------|
| 🏛️ **Architect** | Reads your codebase, spawns parallel Researchers, writes a concrete step-by-step plan |
| 💻 **Coder** | Implements the plan — intentionally assigned a different model family than the reviewers |
| 🔎 **Reviewer** | Checks the implementation against the plan and your coding standards |
| 🧪 **Tester** | Runs actual commands and verifies the result with real evidence, not guesses |
| 🛡️ **Auditor** | Final pass — does the output actually solve the original request? |
| 🔍 **Researcher** | Lightweight parallel lookups, runs in swarms to gather context fast |

---

## ⚡ Work Modes

One flag to control how deep the pipeline goes:

```bash
konklave run "your task" --work sehr_schnell   # ⚡ instant — straight to code, no research
konklave run "your task" --work schnell        # 🚀 quick research + light review
konklave run "your task"                       # ⚖️  balanced default
konklave run "your task" --work lang           # 🔬 multi-round review and testing
konklave run "your task" --work autonom        # 🤖 autonomous loop until PERFECT
```

| Mode | Speed | What runs |
|------|:-----:|-----------|
| `sehr_schnell` | ⚡⚡⚡ | No research → code → done |
| `schnell` | ⚡⚡ | One researcher, quick review |
| `mitte` | ⚖️ | Research → plan → code → review → test |
| `lang` | 🔬 | Multi-researcher, multiple review/test rounds |
| `sehr_lang` | 🧬 | Exhaustive edge cases, trade-off exploration |
| `autonom` | 🤖 | Loop until Auditor approves (with budget cap) |
| `autonom-endlos` | ♾️ | Loop until you stop it |

---

## 🌊 Swarm — Parallel Sub-Agents

Agents can spawn their own helpers for genuinely independent sub-tasks. Three Researchers run at once instead of sequentially. You control the aggressiveness:

```
:swarm off            →  each agent works alone
:swarm encouraged     →  agents use parallel helpers when it makes sense  (default)
:swarm aggressiv      →  maximally fan out work to sub-agents
```

---

## 🎮 Human-in-the-Loop

Inject a message mid-run, steer the direction, or just watch. Slash commands work in the TUI **at any time**:

| Command | What it does |
|---------|-------------|
| `:say "focus on auth"` | Inject a message into the active agent |
| `:work autonom` | Switch work mode on the fly |
| `:swarm aggressiv` | Turn up parallel agent spawning |
| `:ask proaktiv` | Make the Architect ask you more questions |
| `:cost` | Show live USD spend per agent |
| `:models` | Reassign any role to a different model |
| `:exit` | Clean stop at the next step boundary |

---

## 📦 Built-in Pipelines

Pipelines are plain **YAML** — no Python required. Write your own or use the built-ins:

```
🏗️  build_feature    →  research + plan + code + review + test + audit
⚡  quick            →  one-shot coder, no review overhead
♻️  refactor         →  architect-led refactor with before/after review
🐛  debug_issue      →  reproduction → root cause → fix → regression test
🔍  research         →  deep research swarm, no code changes
🤖  autonomous_loop  →  loop until done (budget-capped)
```

---

## 🏗️ Architecture

```
 You (TUI / CLI)
       │
       ▼
 ┌─────────────┐     ┌──────────────────────────────────────┐
 │  Conductor  │────►│  Pipeline YAML                       │
 │             │     │  • sequential steps                  │
 │             │     │  • parallel groups                   │
 │             │     │  • consensus votes                   │
 │             │     │  • loop steps                        │
 │             │     │  • human-approval gates              │
 └─────────────┘     └──────────────┬───────────────────────┘
                                    │
              ┌─────────────────────┼────────────────────┐
              ▼           ▼         ▼          ▼          ▼
        Architect      Coder    Reviewer    Tester    Auditor
        (Sonnet)    (DeepSeek)  (Sonnet)   (Haiku)   (Sonnet)
            │
            └──spawns──► Researcher × N
                         (parallel swarm)
                              │
                              ▼
                       OpenRouter API
                  (any model, any provider)
```

**Key design decisions:**
- 🔒 Each agent has **isolated conversation history** — no cross-contamination
- 🔄 **Event Bus** decouples UI, Conductor, agents, and tools asynchronously
- 🛡️ **Fallback models** — if a model fails, the agent auto-retries on a backup
- 💾 **SQLite persistence** — every step saved, resume from any checkpoint

---

## 🤖 Default Model Panel

| Role | Model | Why |
|------|-------|-----|
| 🏛️ Architect | `anthropic/claude-sonnet-4.6` | Strong planning and tool use |
| 💻 Coder | `deepseek/deepseek-chat-v3` | Fast, cheap, excellent at code |
| 🔎 Reviewer | `anthropic/claude-sonnet-4.6` | Different family from coder → independent check |
| 🧪 Tester | `anthropic/claude-haiku-4.5` | Fast and cheap for test gates |
| 🔍 Researcher | `anthropic/claude-haiku-4.5` | High-volume parallel queries |
| 🛡️ Auditor | `anthropic/claude-sonnet-4.6` | Strict final quality gate |

Swap any role to any model on [OpenRouter](https://openrouter.ai/models) — use `:models` in the TUI or `konklave init`.

---

## 🚀 Installation

**Requirements:** Python 3.11+ · [OpenRouter API key](https://openrouter.ai/keys) (free tier available)

```bash
pip install konklave
konklave init     # interactive setup: language, API key, default models
konklave          # launch TUI
```

> **Windows:** double-click `start.bat` — activates the environment and launches Konklave automatically.

---

## 🏁 Quickstart

```bash
# One-shot run
konklave run "Add input validation to the login form"

# With a specific depth
konklave run "Refactor the payment module" --work lang

# Full interactive TUI
konklave

# Resume an interrupted session
konklave resume

# Test your connection
konklave ping
```

---

## ⚙️ Configuration

```bash
konklave config    # show current settings
konklave init      # re-run the setup wizard
```

Config lives at `~/.konklave/config.toml`. Your API key is stored in the **OS keyring** (Windows Credential Manager / macOS Keychain / Linux SecretService) — never written to plain files.

---

## 💻 Platform Support

| Platform | Status |
|----------|:------:|
| Windows 10 / 11 | ✅ |
| macOS 13+ | ✅ |
| Linux | ✅ |

---

## 📄 License

MIT — see [LICENSE](LICENSE).

---

<div align="center">

Built with ❤️ using [OpenRouter](https://openrouter.ai) · UI powered by [Textual](https://github.com/Textualize/textual)

<br/>

⭐ **Star this repo if you find it useful!** ⭐

</div>
