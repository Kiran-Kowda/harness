# QA Agent Design Guide

Design guide for building QA agents in a harness. Supplementary reference for SKILL.md Phase 3.

---

## The Most Common Failure Mode: Interface Mismatch

The most frequent defect in multi-agent builds is **interface mismatch** — each component works correctly in isolation, but the contract breaks at the connection point.

Examples:
- API responds with `{ projects: [...] }` but the hook expects a bare array
- File path structure and link paths don't match
- Agent A writes output to `_workspace/A.md` but Agent B reads from `_workspace/output-a.md`

A good QA agent's primary job is to catch these mismatches before they surface in production.

---

## Validation Strategies

The core principle: **read both sides simultaneously** and cross-check.

### API Response ↔ Frontend Hook

```
Read: API handler return statement
Read: frontend hook's expected response shape and type parameters
Check: do the response structure and field names match exactly?
```

### File Paths ↔ Routing

```
Read: all href values and router.push() calls in the frontend
Read: all page files and their directory paths
Check: does every link resolve to a real page?
```

### State Transition Map ↔ Actual Code

```
Read: state machine definition (if present)
Read: actual transition trigger code
Check: are all defined transitions implemented? Are there unauthorized transitions?
```

### Endpoints ↔ Hooks

```
Read: all API endpoint definitions
Read: all frontend API call hooks
Check: does every endpoint have a corresponding frontend caller?
```

---

## QA Agent Configuration

Use `general-purpose` agent type so the QA agent can:
- Run `grep` searches across files
- Execute scripts
- Make direct corrections when permitted

**Sample agent definition:**

```markdown
---
name: qa-agent
description: "Cross-validates agent outputs and interface contracts at each phase boundary"
model: "opus"
---

# QA Agent — Interface & Consistency Validator

You validate that components produced by different agents are compatible at their
interfaces. You are the last line of defence before integration.

## Core Role
1. Read output artifacts from all agents for the current phase
2. Cross-validate against expected input contracts of downstream agents
3. Flag mismatches with specific file paths, line numbers, and fix suggestions
4. Write results to `_workspace/qa_report_{phase}.md`

## Principles
- Read both sides of every interface simultaneously — never assume compatibility
- Report mismatches with exact location and a concrete fix suggestion
- Distinguish BLOCKER (will break at runtime) from WARNING (may cause issues)

## Input / Output Protocol
- Input: all `_workspace/` artifacts from the current phase
- Output: `_workspace/qa_report_{phase}.md`
- Format:
  ```
  ## Check: {interface name}
  - Status: PASS | WARNING | BLOCKER
  - Location: {file path}:{line number}
  - Issue: {specific description}
  - Fix: {concrete suggestion}
  ```

## Error Handling
- If an expected output file is missing, report as BLOCKER immediately
- If you cannot determine intent from context, report as WARNING with a question

## Collaboration
- Receives completed artifacts from all agents in the phase
- Sends the QA report to the orchestrator
- The orchestrator decides whether to fix-in-place or re-run the failing agent
```

---

## Incremental QA

Run QA immediately after each module completes — don't wait until the full build is done.

**Benefits:**
- Catch interface mismatches while the producing agent's context is still fresh
- Smaller re-run scope when issues are found
- Progressive confidence: each phase is validated before the next builds on it

**Integration with orchestrator:**

```
Phase 1 complete → QA Agent validates Phase 1 outputs → PASS → proceed to Phase 2
                                                       → BLOCKER → re-run failing agent
```

---

## When to Use a Dedicated QA Agent vs. Built-in Checks

| Situation | Recommendation |
|-----------|---------------|
| 2–3 agents, simple interfaces | Built-in checks in orchestrator |
| 4+ agents, complex cross-domain interfaces | Dedicated QA agent |
| High-stakes output (production code, published content) | Always use dedicated QA agent |
| Rapid iteration / low stakes | Skip QA agent; add assertions instead |
