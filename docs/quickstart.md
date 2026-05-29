# Quickstart — 5-Minute Setup

> If you are not at Step 5 within 5 minutes, treat it as a documentation bug, not a user error.
> Please [open an issue](https://github.com/revfactory/harness/issues).

---

## Prerequisites

- Claude Code v2.x or later (`claude --version` to check)
- A shell that supports persistent `export` statements (bash, zsh)
- Network access to GitHub and Anthropic's API

---

## Step 1 — Enable the Experimental Flag

```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

Add this line to your shell profile (`~/.bashrc`, `~/.zshrc`, etc.) so it persists across sessions:

```bash
echo 'export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1' >> ~/.zshrc
source ~/.zshrc
```

---

## Step 2 — Install the Plugin

**Option A: Claude Code marketplace**

Open Claude Code → Settings → Plugins → search **harness** → Install.

**Option B: Manual link (development / testing)**

```bash
git clone https://github.com/revfactory/harness.git
claude plugin link ./harness
```

Verify installation:

```bash
claude plugin list
# harness  v1.2.1  ✓
```

---

## Step 3 — Generate a Harness

Open a Claude Code session in your project directory and run:

```
"build a harness for a fintech risk-assessment team"
```

Claude will scaffold 3–5 domain-specific agents. You will see output like:

```
✓ Created .claude/agents/risk-analyst.md
✓ Created .claude/agents/compliance-checker.md
✓ Created .claude/agents/report-writer.md
✓ Created .claude/skills/risk-orchestrator/SKILL.md
✓ Updated CLAUDE.md with harness pointer
```

---

## Step 4 — Verify the Output

```bash
ls .claude/agents/
ls .claude/skills/
```

You should see agent definition files in `.claude/agents/` and at least one orchestrator skill in `.claude/skills/`.

Quick sanity check — each agent file should have:
- YAML frontmatter with `name`, `description`, `model: "opus"`
- Sections: Core Role, Principles, Input/Output Protocol

---

## Step 5 — Run a Task

Give the team a realistic task — something equivalent to a Jira ticket:

```
"Analyze the transaction logs in /data/transactions.csv for anomalies
 and produce a risk summary report."
```

The orchestrator skill routes the task to the right agents and coordinates their output.

---

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| `Plugin not found` | Outdated Claude Code | Update: `claude update` |
| Agents not triggering | Missing experimental flag | Re-run Step 1; open a new terminal |
| Team falls back to single-agent | Flag not exported in current shell | `echo $CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` should print `1` |
| Skill not triggering | Language mismatch | Retry the prompt in English |
| High token usage | Expected for Agent Teams | See `docs/experimental-dependency.md` for enterprise options |

---

## Next Steps

- Read [`docs/experimental-dependency.md`](experimental-dependency.md) to understand the experimental flag dependency and migration roadmap.
- Explore [`skills/harness/references/team-examples.md`](../skills/harness/references/team-examples.md) for 5 worked examples.
- Extend your harness: `"add a QA agent to the risk-assessment team"`
