# ia-orchestration-skills

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Agent Skills](https://img.shields.io/badge/format-Agent%20Skills-8A2BE2.svg)](#anatomy-of-a-skill)

**🇪🇸 [Léelo en español](README.es.md)**

Process tooling for AI-augmented software development. Not prompts — complete workflows a coding agent runs the same way, project after project.

This repository is the public shelf of my practice as an **AI Orchestrator**. When the agent writes 80 % of the code, engineering stops being typing and becomes designing the process: what context the agent gets before it acts, in what order it produces each artifact, how its output gets verified, and what it actually cost to produce. Every skill here encodes one of those decisions.

## Table of contents

- [Philosophy](#philosophy)
- [Available skills](#available-skills)
- [Installation](#installation)
- [Usage](#usage)
- [Anatomy of a skill](#anatomy-of-a-skill)
- [Compatibility](#compatibility)
- [Contributing](#contributing)
- [Roadmap](#roadmap)
- [License](#license)
- [Author](#author)

---

## Philosophy

Three ideas hold up everything in this repository.

**Context is the product.** An agent without architectural context doesn't write bad code — it writes plausible code that doesn't fit. Documentation stops being an end-of-project deliverable and becomes the system's input.

**What isn't measured gets assumed.** AI-assisted development opened a new gap between what *feels* fast and what *is* fast — METR's 2025 study measured experienced developers 19 % slower with AI while they believed they were 20 % faster. Review time eats the difference. If that time isn't a measured field, it doesn't exist.

**No invented data.** Asking an LLM to report how many tokens it spent or what a task cost is asking for exactly the kind of fact it cannot know and can convincingly confabulate. The fix isn't a sterner prompt — it's taking the field out of its hands and extracting it from an artifact.

---

## Available skills

| Skill | What it does |
|---|---|
| **[`project-docs-bootstrap`](skills/project-docs-bootstrap/)** | Builds a project's documentation system (`CLAUDE.md` → `PRD.md` → `tech-specs.md` → OWASP + git flow → `MEMORY.md` → `TODO.md`) with a **JIT engine** that keeps exactly 2 atomic tasks in the backlog at all times, derived by comparing the product goal against actual state. |
| **[`ai-effort-tracking`](skills/ai-effort-tracking/)** | Measures effort and **real cost** of assisted development: human vs. agent time, **verification tax**, tokens and USD per task. Works across CLI, web, desktop, mobile and API, with Anthropic, OpenAI, Google, DeepSeek, Qwen and Kimi. |

The two interlock: `project-docs-bootstrap` emits stable identifiers (`OBJ-3`, `T-0042`) and `ai-effort-tracking` uses them as its join key. Together they answer a question no generic LLM observability tool can: **what each product goal cost, in money and in human hours.**

### What makes `ai-effort-tracking` different

- **The verification tax, measured.** The gap between "the agent finished" and "the human sent the next prompt" is review time, and it's already on disk. Hooks turn it from estimated into measured — and past a threshold it's reclassified as absence, so a session that spends the night waiting doesn't inflate the metric.
- **Genuinely multi-provider.** Anthropic, OpenAI and DeepSeek count cache tokens with mutually incompatible semantics. Normalising them wrongly throws no error: it produces presentable, wrong figures off by more than 3×. The skill ships the normalisation contract and the tests that pin it.
- **Flat rate and API, kept apart.** Under a subscription the marginal cost of a token is zero; under an API it's real. All three figures are recorded — shadow price, marginal cost, and allocated share of the monthly fee — because each answers a different question.
- **It refuses to invent.** Without a verified rate, cost is `null` and the report says why. A validator rejects any event with a "measured" field that didn't come from an extractor.

---

## Installation

**For all your projects:**

```bash
git clone https://github.com/ocastelblanco/ia-orchestration-skills.git
cp -r ia-orchestration-skills/skills/<skill-name> ~/.claude/skills/
```

**For a single project** (versioned alongside the repo):

```bash
cp -r ia-orchestration-skills/skills/<skill-name> .claude/skills/
```

Requirements: an Agent Skills-compatible agent. `ai-effort-tracking` also needs **Node.js ≥ 18** for its scripts, and **Python ≥ 3.10** only if you use the Python API wrapper.

---

## Usage

A skill activates two ways:

1. **Automatically** — the agent matches the task against the frontmatter description. Asking "document this project" triggers `project-docs-bootstrap`; "how much did this session cost" triggers `ai-effort-tracking`.
2. **Explicitly** — by name.

```bash
# Bootstrap a project's documentation
/project-docs-bootstrap

# Set up effort tracking (once per project)
/ai-effort-tracking init

# Record the work unit you just closed
/ai-effort-tracking capture

# Monthly report
/ai-effort-tracking report
```

The scripts also run standalone, without an agent:

```bash
cd skills/ai-effort-tracking

# Migrate an existing tracking CSV
node scripts/core/ledger.mjs migrate --csv tracking.csv --dir metrics/events --project my-project

# Extract tokens, cost and review tax from a Claude Code session
node scripts/adapters/claude-code-transcript.mjs --project-dir . --aggregate true

# Generate the report
node scripts/core/report.mjs --dir metrics/events --from 2026-08-01 --out report.md

# Tests
node scripts/test.mjs
```

---

## Anatomy of a skill

```
skills/<name>/
├── SKILL.md        # Required: frontmatter (name, description) + instructions
├── references/     # Optional: material loaded only when needed
├── scripts/        # Optional: executables the agent invokes
└── templates/      # Optional: files to copy into the target project
```

`SKILL.md` is the only thing the agent loads in full on activation, so it stays short and actionable. `references/` holds the detail, which the agent reads only when `SKILL.md` points it there.

That layering — metadata always visible → body on demand → references on demand — is what lets you keep dozens of skills installed without flooding the context window.

---

## Compatibility

Written and tested with **Claude Code**, in the standard Agent Skills format, so they should work with any compatible tool. Some frontmatter fields, such as `allowed-tools`, are Claude Code extensions that other tools can safely ignore.

`ai-effort-tracking` measures work done with any model or tool; what varies is how much evidence is available. The Claude Code adapters are verified; the Codex CLI and Gemini CLI ones are documented with their source identified and flagged as pending empirical validation. That distinction is kept visible on purpose.

---

## Contributing

Contributions are welcome, especially new adapters for other coding tools.

1. Create `skills/<name>/SKILL.md` with its frontmatter.
2. Keep `SKILL.md` under ~500 lines; detail goes in `references/`.
3. If you add an adapter, follow the [contract](skills/ai-effort-tracking/references/adapter-contract.md) and **pin a real payload as a test** — so a provider's format change fails loudly instead of silently.
4. Mark verification status honestly: *verified*, *plausible*, or *unresearched*.
5. Add your skill to the table in both READMEs.

Run `node skills/ai-effort-tracking/scripts/test.mjs` before opening a PR.

---

## Roadmap

- Adapters for **Codex CLI** and **Gemini CLI** (Gemini already emits `gen_ai.client.token.usage` from OpenTelemetry's GenAI semantic conventions, so it can feed the same collector as Claude Code).
- OTLP collector and real-time dashboard, with the JSONL ledger as the ingestion format.
- Assisted estimation: predicting a new task's cost from the history of its type.

---

## License

[MIT](LICENSE) — use, copy, modify and redistribute freely, with or without attribution.

---

## Author

**Oliver Castelblanco** — [@ocastelblanco](https://github.com/ocastelblanco)

I design and operate AI-augmented development processes: context systems that prevent hallucination, verification workflows that catch what the agent can't see, and instrumentation that turns productivity intuition into data that survives review.

Ideas, corrections, or a tool you'd like to see here? Open an [issue](https://github.com/ocastelblanco/ia-orchestration-skills/issues).
