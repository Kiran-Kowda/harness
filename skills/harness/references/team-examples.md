# Agent Team Examples

Five worked examples covering all major patterns and execution modes.

---

## Example 1: Research Team (Agent Teams Mode)

**Architecture:** Fan-out / Fan-in
**Execution mode:** Agent Teams

```
[leader / orchestrator]
    ├── TeamCreate(research-team)
    ├── TaskCreate(4 research tasks)
    ├── Team members self-coordinate (SendMessage)
    ├── Collect results (Read)
    └── Produce synthesis report
```

**Agent roster:**

| Agent | Responsibility |
|-------|---------------|
| `official-researcher` | Official docs and company blogs |
| `media-researcher` | Media coverage and investment trends |
| `community-researcher` | Community and social media sentiment |
| `background-researcher` | Background, competitors, academic context |

**Orchestrator workflow (5 steps):**

1. **Prep** — parse user input, create `_workspace/`
2. **Team setup** — `TeamCreate` + `TaskCreate`
3. **Research** — independent parallel research; agents share findings via `SendMessage`
4. **Integration** — leader reads 4 artifacts, produces synthesis report
5. **Cleanup** — close team, preserve `_workspace/`

---

## Example 2: Sci-Fi Novel Writing Team (Agent Teams Mode)

**Architecture:** Pipeline + Fan-out
**Execution mode:** Agent Teams

**Multi-phase structure:**

| Phase | Agents | Mode |
|-------|--------|------|
| Phase 1 | `worldbuilder`, `character-designer`, `plot-architect` | Parallel (Fan-out); coordinate via SendMessage for consistency |
| Phase 2 | `prose-stylist` | Sequential (Pipeline) — writes draft |
| Phase 3 | `science-consultant`, `continuity-manager` | Parallel review |
| Phase 4 | `prose-stylist` | Sequential — final revision |

**Sample agent definition (`worldbuilder`):**

```markdown
---
name: worldbuilder
description: "Expert in building sci-fi novel worlds"
model: "opus"
---

# Worldbuilder — Sci-Fi World Design Expert

You are an expert in designing the world in which a science fiction novel unfolds.
Grounded in scientific fact but expanding imagination, you build the physical, social,
and technological foundations on which the story rests.

## Core Role
1. Define the world's physical laws and technology level
2. Design social structure, political system, economic system
3. Establish historical context and current conflict structure
4. Describe environments and atmosphere for each location

## Principles
- Internal consistency is the top priority — no contradictions between setting elements
- Chain "What if this technology existed?" questions to trace ripple effects
- World-building serves the story — avoid over-engineering that blocks the plot

## Input / Output Protocol
- Input: user's world concept, genre requirements
- Output: `_workspace/01_worldbuilder_setting.md`
- Format: Markdown (sections: physics / society / technology / history / locations)

## Team Communication Protocol
- → `character-designer`: social structure, class system, occupations
- → `plot-architect`: major conflict structure, crisis elements
- ← `science-consultant`: scientific error feedback → revise setting
- Broadcast to all team members when world-setting changes

## Error Handling
- If the concept is ambiguous, propose 3 directions and ask for a choice
- If a scientific error is found, always suggest an alternative
```

---

## Example 3: Webtoon Production Team (Sub-agent Mode)

**Architecture:** Producer-Reviewer (Generation-Validation)
**Execution mode:** Sub-agents

**Pattern:**

```
Phase 1: Agent(webtoon-artist)    → panel generation
Phase 2: Agent(webtoon-reviewer)  → quality check
Phase 3: Agent(webtoon-artist)    → regenerate flagged panels (max 2 rounds)
```

**Sample agent definition (`webtoon-reviewer`):**

```markdown
---
name: webtoon-reviewer
description: "Expert in webtoon panel quality review"
model: "opus"
---

# Webtoon Reviewer — Quality Review Expert

You evaluate webtoon panels on visual completeness, storytelling clarity, and character consistency.

## Core Role
1. Evaluate composition and visual quality of each panel
2. Verify character appearance consistency across panels
3. Assess speech bubble text readability and placement
4. Review overall episode pacing and direction flow

## Principles
- Use clear 3-tier verdicts: **PASS / FIX / REDO**
- FIX = partial correction is sufficient; REDO = full regeneration required
- Judge by objective criteria (consistency, readability, composition) — not personal taste

## Input / Output Protocol
- Input: panel images in `_workspace/panels/`
- Output: `_workspace/review_report.md`
- Format:
  ```
  ## Panel {N}
  - Verdict: PASS | FIX | REDO
  - Reason: [specific reason]
  - Instructions: [specific revision direction for FIX/REDO]
  ```

## Error Handling
- If an image fails to load, mark that panel as REDO
- After 2 regeneration rounds with continued REDO, force PASS with a warning

## Collaboration
- Pass revision instructions to `webtoon-artist` (via output file)
- Re-review regenerated panels (max 2 rounds)
```

**Retry policy:**
- REDO verdict → send revision instructions to artist (specific guidance required)
- Max 2 regeneration rounds, then force PASS
- If more than 50% of panels are REDO, suggest the user revise their prompt

---

## Example 4: Code Review Team (Agent Teams Mode)

**Architecture:** Fan-out / Fan-in + Discussion
**Execution mode:** Agent Teams

**Key advantage:** "Different perspectives catching cross-domain issues that siloed review would miss"

```
[leader] → TeamCreate(review-team)
    ├── security-reviewer  — security vulnerability assessment
    ├── performance-reviewer — performance impact analysis
    └── test-reviewer       — test coverage verification
```

**Team communication pattern (cross-domain):**

```
security → performance: "This SQL query may be injectable — please also check performance"
performance → test: "N+1 query found — please verify there's a test for this"
test → security: "No tests for the auth module — what's the security priority?"
```

**Key:** Reviewers communicate directly without routing through the leader, catching cross-domain issues faster.

---

## Example 5: Supervisor Pattern — Code Migration Team (Agent Teams Mode)

**Architecture:** Supervisor
**Execution mode:** Agent Teams

```
[supervisor / leader] → analyze file list → assign batches
    ├→ [migrator-1]  (batch A)
    ├→ [migrator-2]  (batch B)
    └→ [migrator-3]  (batch C)
       ← TaskUpdate → assign more batches or reassign
```

**Dynamic allocation logic:**

1. Collect full list of target files
2. Estimate complexity (file size, import count, dependency depth)
3. Register file batches as tasks via `TaskCreate`
4. Workers self-assign via task claim
5. On `TaskUpdate` completion report:
   - Success → worker automatically requests next task
   - Failure → `SendMessage` to identify root cause → reassign
6. All tasks complete → run integration tests

**Difference from Fan-out:** Tasks are **dynamically allocated at runtime**, not fixed upfront.

---

## Output Patterns Summary

### Agent Definition File

- **Location:** `{project}/.claude/agents/{agent-name}.md`
- **Required sections:** Core Role, Principles, Input/Output Protocol, Error Handling, Collaboration
- **Additional section for Agent Teams:** Team Communication Protocol (what to send/receive, and when)

### Skill File

- **Project level:** `{project}/.claude/skills/{skill-name}/SKILL.md`
- **Global level:** `~/.claude/skills/{skill-name}/SKILL.md`

### Orchestrator Skill

- Coordinates the entire team
- Defines agent composition and workflow per scenario
- Explicitly states execution mode: Agent Teams (default) or Sub-agents
