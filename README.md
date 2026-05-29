# Harness: Team-Architecture Factory for Claude Code

[![version](https://img.shields.io/badge/version-1.2.1-blue)](CHANGELOG.md)
[![license](https://img.shields.io/badge/license-MIT-green)](LICENSE)

**Harness** is a Claude Code plugin that automatically generates agent teams and specialized skills from a plain-language domain description.

Trigger it with a prompt like `"build a harness for this project"` and the system scaffolds pre-configured agents and skills based on six architectural patterns.

---

## Table of Contents

- [Overview](#overview)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Six Architecture Patterns](#six-architecture-patterns)
- [Output Structure](#output-structure)
- [A/B Study Results](#ab-study-results)
- [Ecosystem Position](#ecosystem-position)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

Harness operates as an **L3 Meta-Factory** — a "Team-Architecture Factory" that sits above individual agent tools. Given a domain description, it:

1. Analyzes the domain and detects existing agent/skill files
2. Selects one of six team architecture patterns
3. Generates `.claude/agents/*.md` agent definition files
4.. Generates `.claude/skills/*/SKILL.md` skill files with progressive disclosure
5. Builds an orchestrator skill to coordinate the team
6. Validates the output with trigger tests and dry-run orchestration checks

---

## Installation

### Option A — Claude Code Marketplace

Search for **harness** in the Claude Code plugin marketplace and click Install.

### Option B — Global Skill (manual)

```bash
# Enable the experimental Agent Teams flag
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1

# Link the plugin from this repo
claude plugin link ./harness
```

Add the `export` line to your shell profile (`~/.bashrc`, `~/.zshrc`, etc.) to persist it across sessions.

---

## Quick Start

See [`docs/quickstart.md`](docs/quickstart.md) for a guided 5-minute walkthrough.

**TL;DR:**

```
"build a harness for a fintech risk-assessment team"
```

Claude will scaffold 3–5 domain-specific agents and a coordinating skill under `.claude/`.

---

## Six Architecture Patterns

| Pattern | Best For |
|---------|----------|
| **Pipeline** | Sequential workflows (analysis → design → implementation) |
| **Fan-out / Fan-in** | Parallel independent work merged afterward |
| **Expert Pool** | Dynamic role selection based on input type |
| **Producer-Reviewer** | Iterative quality assurance with feedback loops |
| **Supervisor** | Central agent dynamically distributing tasks to workers |
| **Hierarchical Delegation** | Recursive decomposition (max 2 tiers) |

Complex harnesses can combine patterns — e.g., parallel translation + individual review per language.

---

## Output Structure

```
.claude/
├── agents/
│   ├── agent-name-1.md
│   ├── agent-name-2.md
│   └── ...
└── skills/
    └── orchestrator/
        └── SKILL.md
```

Intermediate artifacts are stored in `_workspace/` with a `phase/agent/artifact` naming convention for auditability.

---

## A/B Study Results

An internal study across 15 software tasks found:

> Average quality scores improved from **49.5 → 79.3** (author-measured; third-party verification pending)

---

## Ecosystem Position

```
L3 Meta-Factory layer
├── harness      — Team-Architecture Factory  (this repo)
├── Archon       — Runtime-Configuration Factory
└── meta-harness — Codex runtime port
```

These are neighboring sub-layers of the same L3, not competing tools.

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). Pull requests are acknowledged within 72 hours; issues are triaged within 48 hours.

Security reports: `robin.hwang@kakaocorp.com` with subject prefix `[harness-security]`.

---

## License

[MIT](LICENSE)
