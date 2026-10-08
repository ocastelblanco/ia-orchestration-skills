# ia-orchestration-skills

[![License: Apache 2.0](https://img.shields.io/badge/license-Apache_2.0-blue.svg)](LICENSE)
[![Agent Skills](https://img.shields.io/badge/format-Agent%20Skills-8A2BE2.svg)](#anatomy-of-a-skill)

**🇪🇸 [Léelo en español](README.es.md)**

Process tooling for AI-augmented software development. You won't find loose prompts here. These are complete workflows a coding agent runs the same way, project after project.

This repository is the public shelf of my practice as an **AI Orchestrator**. Once the agent writes 80 % of the code, the engineering work shifts. It stops being typing and becomes process design. What context does the agent get before it acts. In what order does it produce each artifact. How does its output get verified. And what did producing it actually cost. Every skill here encodes one of those decisions.

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

**Context is the product.** An agent without architectural context doesn't write bad code. It writes plausible code that doesn't fit, which is considerably worse, because it survives a quick review. So documentation stops being an end-of-project deliverable and becomes the system's input.

**What isn't measured gets assumed.** Working with AI opened a new gap between what *feels* fast and what *is* fast. METR's 2025 study clocked experienced developers 19 % slower with AI while they reported feeling 20 % faster. Review time eats the difference. And when that time isn't a measured field, for practical purposes it doesn't exist.

**No invented data.** Asking an LLM how many tokens it burned or what a task cost means asking for exactly the kind of fact it can't know and can convincingly confabulate. The fix isn't a sterner prompt. It's taking the field out of its hands and pulling it from an artifact instead.

---

## Available skills

| Skill | What it does |
|---|---|
| **[`project-docs-bootstrap`](skills/project-docs-bootstrap/)** | Builds a project's documentation system (`CLAUDE.md` → `PRD.md` → `tech-specs.md` → OWASP + git flow → `MEMORY.md` → `TODO.md`) with a **JIT engine** that keeps exactly 2 atomic tasks in the backlog at all times, derived by comparing the product goal against actual state. |
| **[`ai-effort-tracking`](skills/ai-effort-tracking/)** | Measures effort and **real cost** of assisted development: human time versus agent time, **verification tax**, tokens and USD per task. Works across CLI, web, desktop, mobile and API, with Anthropic, OpenAI, Google, DeepSeek, Qwen and Kimi. |
| **[`owasp-security-verification`](skills/owasp-security-verification/)** | Verifies what the agent just built against the OWASP requirements that **apply to its stack**. Every finding carries `file:line` evidence, an **ASVS 5.0** ID, a **Top 10:2025** category and the cheat sheet that fixes it. Markdown and SARIF report. |

The three interlock. `project-docs-bootstrap` emits stable identifiers (`OBJ-3`, `T-0042`) and `ai-effort-tracking` uses them as its join key. Together they answer something no generic LLM observability tool can: **what each product goal cost, in money and in human hours.** `owasp-security-verification` closes the loop: the security rules `project-docs-bootstrap` writes into `CLAUDE.md` get checked against the code, and the most severe failures feed the JIT engine as tasks.

### What makes `ai-effort-tracking` different

- **The verification tax, measured.** The gap between the agent finishing and the human sending the next prompt is review time, and it's already sitting on your disk. Hooks turn it from estimated into measured. Past a configurable threshold it gets reclassified as absence, so a session that spends the night waiting doesn't inflate the metric.
- **Genuinely multi-provider.** Anthropic, OpenAI and DeepSeek count cache tokens with mutually incompatible semantics. Normalising them wrongly throws no error: it produces presentable, wrong figures, off by factors above 3×. The skill ships the normalisation contract and the tests that pin it.
- **Flat rate and API, kept apart.** Under a subscription the marginal cost of a token is zero; under an API it's real. That's why three figures get recorded (shadow price, marginal cost, allocated share of the monthly fee), since each answers a different question.
- **It refuses to invent.** With no verified rate, cost stays `null` and the report explains why. A validator rejects any event carrying a "measured" field that didn't come from an extractor.

### What makes `owasp-security-verification` different

- **Only what applies.** It detects 14 stack facets (Angular, Node/Lambda, Firebase, Supabase, PHP/Moodle, Python, LLM, custom auth, Docker, IaC, n8n, GitHub Actions…) and hands the agent only the relevant checks, 33 to 82 out of a 116-check catalog across the test projects. The agent never loads the whole standard.
- **Verifiable, not opinion.** Every check cites ASVS 5.0.0 requirements that a script validates against the standard's official export. Where ASVS doesn't reach (containers, CI, LLMs), the check says so (`asvs_gap`) instead of forcing a mapping.
- **An evidence gate.** The agent can only mark `pass` or `fail` by citing a file and line that exist, an allowed URL, or a read-only API call. Without evidence the check stays "pending", and the report never presents it as passed.
- **Lightweight.** Dependency-free Node; 58 static rules settle the automatic part in seconds, and `npm audit` and `composer audit` cover dependencies. Live mode (headers, TLS, CORS, exposed paths, read-only AWS) is opt-in and restricted to an allowlist.

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

Requirements: an Agent Skills-compatible agent. `ai-effort-tracking` also needs **Node.js ≥ 18** for its scripts, and **Python ≥ 3.10** only if you use the Python API wrapper. `owasp-security-verification` needs **Node.js ≥ 20** and uses `npm`, `composer` and `pip-audit` when they're installed.

---

## Usage

A skill activates two ways.

The first is **automatic**: the agent matches the task against the frontmatter description. Asking "document this project" fires `project-docs-bootstrap`; asking "how much did this session cost" fires `ai-effort-tracking`.

The second is **explicit**, by name:

```bash
# Bootstrap a project's documentation
/project-docs-bootstrap

# Set up effort tracking (once per project)
/ai-effort-tracking init

# Record the work unit you just closed
/ai-effort-tracking capture

# Monthly report
/ai-effort-tracking report

# Verify the security of the component you just built
/owasp-security-verification verify
```

The scripts also run standalone, with no agent involved:

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

```bash
cd skills/owasp-security-verification

# Detected facets and suggested ASVS level
node scripts/detect-stack.mjs --root ../my-app

# Static verification (writes ../my-app/.owasp/results.json)
node scripts/verify.mjs --root ../my-app --scope src/app/payments

# Agent verdict: rejected if the evidence doesn't exist
node scripts/findings.mjs record --root ../my-app --check BASE-19 --status pass \
  --evidence api/middleware/auth.ts:12 --note "Every route goes through verifyToken"

# Markdown + SARIF report
node scripts/report.mjs --root ../my-app

# Tests and catalog validation against ASVS 5.0.0
node scripts/test.mjs
node scripts/validate-catalog.mjs
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

`SKILL.md` is the only thing the agent loads in full on activation, so it pays to keep it short and actionable. The detail lives in `references/`, which the agent reads only when `SKILL.md` points it there.

That layering (metadata always visible, body on demand, references on demand) is what lets you keep dozens of skills installed without flooding the context window.

---

## Compatibility

Written and tested with **Claude Code**, in the standard Agent Skills format, so they should work with any compatible tool. Some frontmatter fields, such as `allowed-tools`, are Claude Code extensions that other tools can safely ignore.

`ai-effort-tracking` measures work done with any model or tool. What changes between them is how much evidence is available. The Claude Code adapters are verified; the Codex CLI and Gemini CLI ones are documented with their source identified and flagged as pending empirical validation. That distinction stays visible on purpose.

---

## Contributing

Contributions are welcome, especially new adapters for other coding tools.

1. Create `skills/<name>/SKILL.md` with its frontmatter.
2. Keep `SKILL.md` under roughly 500 lines; detail goes in `references/`.
3. If you add an adapter, follow the [contract](skills/ai-effort-tracking/references/adapter-contract.md) and **pin a real payload as a test**, so a provider's format change fails loudly instead of silently.
4. Mark verification status honestly: *verified*, *plausible*, or *unresearched*.
5. Add your skill to the table in both READMEs.

Run `node skills/ai-effort-tracking/scripts/test.mjs` and `node skills/owasp-security-verification/scripts/test.mjs` before opening a PR.

---

## Roadmap

- Adapters for **Codex CLI** and **Gemini CLI**. Gemini already emits `gen_ai.client.token.usage` from OpenTelemetry's GenAI semantic conventions, so it can feed the same collector as Claude Code.
- OTLP collector and real-time dashboard, with the JSONL ledger as the ingestion format.
- Assisted estimation: predicting a new task's cost from the history of its type.
- `owasp-security-verification`: facets for native mobile (MASVS), Kubernetes and .NET/Java; optional adapters for semgrep, gitleaks, trivy and checkov.

---

## License

[Apache 2.0](LICENSE). Use, modify and redistribute freely, including commercially. If you redistribute, keep the license and the [NOTICE](NOTICE) file, and state which files you changed. The license also grants you a patent license from contributors.

Exception: [`skills/owasp-security-verification/references/`](skills/owasp-security-verification/references/) adapts OWASP material (ASVS, Top 10 and the Cheat Sheet Series) and is therefore distributed under **CC BY-SA 4.0**, the same license as its sources. That skill's code remains Apache 2.0.

---

## Author

**Oliver Castelblanco** · [@ocastelblanco](https://github.com/ocastelblanco)

I design and operate AI-augmented development processes: context systems that prevent hallucination, verification workflows that catch what the agent can't see, and instrumentation that turns productivity intuition into data that survives review.

Ideas, corrections, or a tool you'd like to see here? Open an [issue](https://github.com/ocastelblanco/ia-orchestration-skills/issues).
