# Experimental Flag Dependency

> **Status:** Active · **Owner:** revfactory · **Last updated:** 2026-04-18 · **SLA:** See [Monitoring Commitment](#monitoring-commitment)

This document explains why `harness` requires `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`,
the three plausible futures of that flag, and what this repository will do in each case —
with time-boxed commitments so enterprise adopters can plan against it.

---

## Current State

### Why the flag is required

`harness` is a meta-skill factory built on top of Claude Code's **Agent Teams API**.
Three Claude Code primitives are invoked internally whenever a user runs
`claude "build a harness for <domain>"`:

| Primitive | Purpose | Flag-gated? |
|-----------|---------|------------|
| `TeamCreate` | Instantiates a multi-agent team with shared context | **Yes** |
| `SendMessage` | Routes messages between team members (supervisor ↔ worker) | **Yes** |
| `TaskCreate` | Spawns long-running subtasks inside a team | **Yes** |
| `Agent` tool (invoke) | Single-agent dispatch | No (GA) |

All three flag-gated primitives require:

```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

Without this variable set in the shell that launches `claude`, harness's generated teams
fall back to single-agent execution, which silently breaks the Pipeline, Fan-out/Fan-in,
Supervisor, and Hierarchical Delegation patterns.

### Anthropic references

The design rationale and roadmap live in three Anthropic Engineering posts. Adopters
evaluating harness should read at least the first:

1. [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)
2. [Harness design for long-running apps](https://www.anthropic.com/engineering/harness-design-long-running-apps)
3. [Scaling Managed Agents](https://www.anthropic.com/engineering/managed-agents)

---

## Dependency Graph

```
harness (v1.2.0)
  └── Agent Teams API (Claude Code)
        ├── TeamCreate            ← EXPERIMENTAL_AGENT_TEAMS=1
        ├── SendMessage           ← EXPERIMENTAL_AGENT_TEAMS=1
        ├── TaskCreate            ← EXPERIMENTAL_AGENT_TEAMS=1
        └── Agent (invoke)        ← GA (flag-independent)
              └── Anthropic Roadmap
                    ├── Scenario A: Flag removed (GA promotion)
                    ├── Scenario B: Managed Agents GA (parallel path)
                    └── Scenario C: Breaking signature change
```

---

## 3 Scenarios

### Scenario A — Flag removed (Agent Teams promoted to GA)

**Trigger:** Anthropic Changelog publishes "Agent Teams is now GA" **or** nightly CI
detects that `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` is no longer required.

**Probability:** High — the three blog posts above telegraph this path.

| Checkpoint | Action | Artifact |
|------------|--------|----------|
| **T+24h** | Open `feat/drop-experimental-flag`; remove `export` line from all docs; add `claude-code >= X.Y.Z` bound in `plugin.json` | Branch + draft PR |
| **T+48h** | Publish `docs/migrating-from-experimental.md`; update this file headline to "no flag required as of vX.Y"; pin issue "Action required: drop the export line" | Migration guide + pinned issue |
| **T+72h** | Ship **v1.3.0**: CHANGELOG entry + `gh release create` + HN follow-up | `v1.3.0` git tag + GH Release |

**Adopter impact:** Positive. Enterprise approval friction drops.

---

### Scenario B — Managed Agents reaches GA (parallel path)

**Trigger:** Anthropic publishes "Managed Agents is generally available" with a stable CLI or SDK surface.

**Probability:** Medium-high within 90 days. Managed Agents is a server-side execution model;
harness's client-side team orchestration does **not** automatically translate.

| Checkpoint | Action | Artifact |
|------------|--------|----------|
| **T+24h** | Open `feat/managed-agents-compat`; add `adapters/managed-agents/` scaffold mapping harness's 6 patterns to Managed Agents invocation; identify incompatible patterns (likely: Hierarchical Delegation) | Compat PR (draft) |
| **T+48h** | Publish blog: "Harness + Managed Agents: one layer up, not replaced" — re-frame harness as the **design-time** layer that outputs Managed Agents configs | Blog post |
| **T+72h** | Publish `docs/managed-agents-migration.md` with per-pattern matrix (which of the 6 patterns map 1:1, which need rewrite) | Migration guide |

**Strategic note:** harness re-positions as the upper layer **on top of** Managed Agents —
"Managed Agents runs the team; harness designs it."

**Adopter impact:** Neutral to positive. Existing users keep working; new users can opt into Managed Agents output.

---

### Scenario C — Breaking change (API signature mutation)

**Trigger:** Nightly CI fails against Claude Code's latest nightly build **or** Changelog
announces a renamed env var or changed `TeamCreate` signature.

**Probability:** Medium. Experimental APIs can be renamed without deprecation windows.

| Checkpoint | Action | Artifact |
|------------|--------|----------|
| **T+0 to T+24h** | CI alert fires; open `hotfix/compat-<date>`; patch affected call sites; verify tests green on both old + new signature | Hotfix branch |
| **T+24h** | Merge hotfix; push `v1.2.x` patch tag; update `docs/compatibility-matrix.md` | `v1.2.x` patch release |
| **T+72h** | If non-trivial: post on Discussions + X; otherwise CHANGELOG entry is sufficient | Conditional Discussions post |

**Adopter impact:** Users pinned to the prior Claude Code version are unaffected. Users on latest get a same-week patch.

---

## Monitoring Commitment

Missing this SLA is grounds for filing an issue with the `sla-breach` label.

| Event | SLA | Measurement |
|-------|-----|-------------|
| Anthropic publishes Agent Teams / Managed Agents change | This document updated within **72 hours** | Changelog post timestamp vs. this file's `Last updated` line |
| Nightly CI detects compat break | Hotfix branch open within **24 hours** | GitHub Actions run timestamp vs. branch creation timestamp |
| New Claude Code stable release (minor or major) | `docs/compatibility-matrix.md` row added within **7 days** | Compatibility matrix diff |

**Sources monitored:**
- Anthropic Engineering blog RSS
- `anthropics/claude-code` GitHub Releases (nightly tag)
- Anthropic Discord `#claude-code` channel

---

## FAQ for Enterprise Adopters

### Q1. We can't enable EXPERIMENTAL flags in production (SOC 2 / ISO 27001 / K-ISMS). How do we adopt harness?

Use harness **design-time only**: run it in a sandbox workstation to scaffold
`.claude/agents/` and `.claude/skills/` files, then commit the generated artifacts
into your production repo. Production Claude Code never needs the flag — only the
flag-gated `TeamCreate` runtime does. The generated single-agent skills are GA-path compatible.

### Q2. If Agent Teams goes GA (Scenario A), will my existing harness-generated code break?

No. Your `.claude/agents/*.md` and `.claude/skills/*` files are plain Markdown and
remain valid. You will be able to `unset CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` on the
day of GA. We will publish a migration note within 48 hours (see Scenario A).

### Q3. Do you guarantee an SLA in writing? What happens if you miss it?

The SLA table above is the public commitment, enforced by:
(a) a GitHub Action that comments on this file if its `Last updated` line is older
than 72 hours after a detected Changelog event,
(b) an `sla-breach` issue label adopters may apply, and
(c) a post-mortem obligation in `CONTRIBUTING.md` for any breach.

This is a community commitment, not a paid SLA. For a paid SLA, contact the maintainer (see README).

---

## Related Documents

- [`docs/quickstart.md`](./quickstart.md) — 5-minute install walkthrough
- `docs/compatibility-matrix.md` *(pending)* — Claude Code × harness version table
