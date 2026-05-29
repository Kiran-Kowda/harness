# Orchestrator Skill Templates

Three templates for team coordination. Choose based on your execution mode.

---

## Template A — Agent Teams Mode

Use when 2+ agents need to communicate directly (default for most harnesses).

```markdown
---
name: {domain}-orchestrator
description: >
  Orchestrates the {domain} agent team. Trigger on initial build requests
  ("build a harness for X") AND on follow-up requests ("re-run", "fix",
  "improve previous results", "update the harness", "add X to the team").
---

# {Domain} Orchestrator

## Phase 0: Context Check

1. Read `_workspace/` — does prior output exist?
   - Yes → partial re-run or extension mode
   - No → full initial run
2. Check `.claude/agents/` for existing team members.

## Phase 1: Preparation

1. Parse user input and clarify ambiguities if needed.
2. Create `_workspace/` directory structure.
3. Brief the team via TaskCreate (shared task list).

## Phase 2: Team Execution (Agent Teams)

```
TeamCreate(team-name)
TaskCreate(task-1 → agent-A)
TaskCreate(task-2 → agent-B)
TaskCreate(task-3 → agent-C)
```

Agents coordinate directly via SendMessage. The orchestrator monitors TaskUpdate events.

## Phase 3: Integration

1. Read all `_workspace/` artifacts produced by the team.
2. Synthesize into a final output.
3. Run validation checks (see Phase 6 of SKILL.md).

## Phase 4: Cleanup

1. Write final report to `_workspace/final_report.md`.
2. Preserve `_workspace/` for future re-runs.
3. Update `CLAUDE.md` change log.
```

---

## Template B — Sub-agent Mode

Use when agents don't need to communicate with each other.

```markdown
---
name: {domain}-orchestrator
description: >
  Orchestrates {domain} tasks using sequential sub-agent dispatch.
  Trigger on: {list trigger phrases}
---

# {Domain} Orchestrator (Sub-agent Mode)

## Phase 0: Context Check
(same as Template A)

## Phase 1: Preparation
(same as Template A)

## Phase 2: Execution (Sub-agents)

```
result-A = Agent(agent-A, task-1)
result-B = Agent(agent-B, task-2)
result-C = Agent(agent-C, task-3)
```

Results flow only from sub-agent → orchestrator. No inter-agent communication.

## Phase 3: Integration
(same as Template A)

## Phase 4: Cleanup
(same as Template A)
```

---

## Template C — Hybrid Mode

Use when different phases need different execution modes.

```markdown
---
name: {domain}-orchestrator
description: >
  Orchestrates {domain} tasks with phase-varying execution modes.
  Trigger on: {list trigger phrases}
---

# {Domain} Orchestrator (Hybrid Mode)

## Phase 0: Context Check
(same as Template A)

## Phase 1: Data Collection (Sub-agent Mode)

Low-communication phase — use sub-agents for independent collection:

```
data-A = Agent(collector-A, collect task A)
data-B = Agent(collector-B, collect task B)
```

## Phase 2: Analysis & Integration (Agent Teams Mode)

High-communication phase — switch to Agent Teams for cross-validation:

```
TeamCreate(analysis-team)
TaskCreate(analysis → analyst)
TaskCreate(validation → validator)
```

Analyst and validator communicate directly via SendMessage.

## Phase 3: Cleanup
(same as Template A)
```

---

## Writing Effective Orchestrator Descriptions

The `description` field is the **only trigger mechanism**. It must include:

1. **Initial trigger keywords** — "build a harness", "create an agent team", "scaffold agents for X"
2. **Follow-up trigger keywords** — "re-run", "fix", "improve", "update", "add to the team", "previous results"
3. **Anti-trigger boundary** — what this orchestrator does NOT handle

Without follow-up keywords, the harness becomes a one-shot tool. With them, it becomes a continuously evolving system.
