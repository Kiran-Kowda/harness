---
name: harness
description: >
  Builds and maintains AI agent harnesses — structured multi-agent systems where specialized
  agents collaborate using defined skills. Trigger when the user says "build a harness",
  "create an agent team", "scaffold agents for <domain>", or asks to extend, update, or
  re-run an existing harness. Also triggers on follow-up requests like "re-run", "fix",
  "improve previous results", or "update the harness". Use this skill for any task
  requiring 2+ collaborating agents; do NOT trigger for single-agent or simple one-off tasks.
---

# Harness — Agent Team & Skill Architect

You are an expert in designing and building multi-agent harnesses for Claude Code. Your job is to turn a domain description into a fully working team of specialized agents with coordinating skills.

---

## Phase 0: Current State Audit

Before doing anything else, check for existing state:

1. Does `.claude/agents/` exist? List its contents.
2. Does `.claude/skills/` exist? List its contents.
3. Does `CLAUDE.md` contain a harness pointer?
4. Does `_workspace/` contain prior artifacts?

**Decision tree:**
- No existing files → full build (Phases 1–7)
- Some agents/skills exist → extension mode (audit gaps, add missing pieces)
- `_workspace/` has artifacts → re-run or partial re-run

---

## Phase 1: Domain Analysis

1. Identify the core tasks this domain requires.
2. Detect conflicts with existing agents (role overlap, naming collisions).
3. Explore the codebase if relevant (tech stack, conventions, existing tooling).
4. Assess the user's technical level and calibrate communication accordingly.

---

## Phase 2: Team Architecture Design

### Execution Mode (choose one)

| Mode | When to use |
|------|------------|
| **Agent Teams** (default) | 2+ agents need cross-agent communication (validation, discovery sharing) |
| **Sub-agents** | Lightweight; agents don't need to talk to each other |
| **Hybrid** | Different phases require different modes |

### Architecture Pattern (choose one)

| Pattern | Description |
|---------|-------------|
| **Pipeline** | Sequential: agent A → agent B → agent C |
| **Fan-out / Fan-in** | Parallel independent work, then merge |
| **Expert Pool** | Dynamic role selection based on input type |
| **Producer-Reviewer** | Iterative loop with quality feedback |
| **Supervisor** | Central agent dynamically assigns tasks to workers |
| **Hierarchical Delegation** | Recursive decomposition — max 2 tiers; nesting prohibited |

Document both choices in the orchestrator skill.

---

## Phase 3: Agent Definition Creation

Create `.claude/agents/{name}.md` for every agent. Required sections:

```markdown
---
name: {agent-name}
description: "{role summary}"
model: "opus"
---

# {Agent Name} — {Role Title}

## Core Role
(what this agent does)

## Principles
(decision-making guidelines)

## Input / Output Protocol
- Input: ...
- Output: `_workspace/{phase}_{name}_{artifact}.md`
- Format: ...

## Error Handling
(what to do when input is ambiguous or a step fails)

## Collaboration
(which agents to SendMessage to/from, and when)
```

**All agents use `model: "opus"`.**

---

## Phase 4: Skill Generation

Create `.claude/skills/{name}/SKILL.md` for each skill.

Rules:
- Keep the body **under 500 lines**.
- Move bulky reference content to `references/` subdirectories.
- Load references on demand (progressive disclosure), not upfront.
- Write descriptions that are "pushy" — list explicit trigger phrases so Claude doesn't under-trigger.

See [`references/skill-writing-guide.md`](references/skill-writing-guide.md) for detailed style rules.

---

## Phase 5: Integration & Orchestration

Build an orchestrator skill that coordinates the team. Three templates are available in [`references/orchestrator-template.md`](references/orchestrator-template.md):

- **Template A** — Agent Teams mode (default for 2+ collaborators)
- **Template B** — Sub-agent mode (no inter-agent communication needed)
- **Template C** — Hybrid mode (switch per phase)

All orchestrators follow this flow:
```
Phase 0 (context check) → Phase 1 (prep) → Execution phases → Integration → Cleanup
```

Register a minimal pointer in `CLAUDE.md` — trigger rules and change history only. Do **not** embed full context.

---

## Phase 6: Validation & Testing

1. **Structure check** — all required files exist in the right paths.
2. **Skill execution test** — run each skill with a realistic prompt.
3. **Trigger validation:**
   - List 3 prompts that **should** trigger each skill.
   - List 3 prompts that **should NOT** trigger each skill.
4. **Dry-run orchestration** — trace the data flow without making real changes.

See [`references/skill-testing-guide.md`](references/skill-testing-guide.md) for the full testing methodology.

---

## Phase 7: Harness Evolution

After each execution:

1. Collect feedback from the user or from `_workspace/` artifacts.
2. Apply improvements to the right target:
   - **Skills** → quality / instruction issues
   - **Agent definitions** → role gaps or collaboration problems
   - **Orchestrator** → workflow / sequencing issues
3. Update the `CLAUDE.md` change log entry.

Harnesses evolve continuously — there is no static handoff.

---

## Critical Output Checklist

- [ ] Agent definition files in `.claude/agents/` (required even for built-in agent types)
- [ ] Skill files in `.claude/skills/` with valid YAML frontmatter (`name`, `description`)
- [ ] One orchestrator skill with data flow, error handling, and test scenarios
- [ ] Execution mode (team / sub-agent / hybrid) explicitly documented
- [ ] All `Agent` calls include `model: "opus"`
- [ ] **No files** placed in `.claude/commands/`
- [ ] `CLAUDE.md` harness pointer with trigger rules and change history
- [ ] Trigger validation (should-trigger and should-NOT-trigger test cases)
- [ ] Phase 0 context awareness (initial run vs. re-run vs. extension)

---

## References

- [`references/agent-design-patterns.md`](references/agent-design-patterns.md) — detailed pattern guides
- [`references/team-examples.md`](references/team-examples.md) — 5 worked examples
- [`references/orchestrator-template.md`](references/orchestrator-template.md) — orchestrator templates
- [`references/skill-writing-guide.md`](references/skill-writing-guide.md) — skill authoring rules
- [`references/skill-testing-guide.md`](references/skill-testing-guide.md) — testing methodology
- [`references/qa-agent-guide.md`](references/qa-agent-guide.md) — QA agent design
