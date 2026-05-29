# Agent Team Design Patterns

Supplementary reference for SKILL.md Phase 2 (Team Architecture Design).

---

## Execution Modes

### Agent Teams (primary mode)

Team members are independent Claude instances. They communicate via `SendMessage` and coordinate through shared task lists (`TaskCreate`).

**Characteristics:**
- Cross-agent communication (validation, discovery sharing)
- Real-time feedback between agents
- Higher token cost
- Agent nesting is **prohibited**

**When to use:** Any time 2+ agents need to share findings, validate each other's output, or coordinate dynamically.

### Sub-agents (lightweight mode)

The main agent spawns sub-agents via the `Agent` tool. Sub-agents return results only to the main agent — no inter-agent communication.

**When to use:** Agents work independently; no cross-validation needed; lower token budget.

**Decision rule:** Default to Agent Teams. Use sub-agents only when cross-agent communication is not required.

---

## Six Architecture Patterns

### 1. Pipeline

Sequential workflow where each agent's output is the next agent's input.

```
[analyzer] → [designer] → [implementer] → [reviewer]
```

Best for: well-defined sequential processes (analysis → design → implementation).

---

### 2. Fan-out / Fan-in

Work is distributed in parallel, then merged by a leader.

```
[leader]
  ├── [worker-A]  (parallel)
  ├── [worker-B]  (parallel)
  └── [worker-C]  (parallel)
         ↓
     [leader merges]
```

This is the **most natural Agent Teams pattern**. Best for: independent parallel research, multi-language translation, parallel code review.

---

### 3. Expert Pool

A router agent selects the right specialist based on input type.

```
[router] → [expert-A | expert-B | expert-C]  (one chosen per request)
```

Best for: domains with clearly distinct sub-specialties (e.g., finance vs. legal vs. technical writing).

---

### 4. Producer-Reviewer (Generation-Validation)

Iterative quality loop between a creator and a critic.

```
[producer] → [reviewer] → feedback → [producer] → ...
```

Best for: tasks requiring quality gates (writing, code generation, design).

---

### 5. Supervisor

A central supervisor dynamically distributes tasks to workers at runtime.

```
[supervisor]
  ├→ [worker-1]  (assigned batch A)
  ├→ [worker-2]  (assigned batch B)
  └→ [worker-3]  (assigned batch C)
     ← TaskUpdate → supervisor reassigns or adds more work
```

Key difference from Fan-out: tasks are **dynamically allocated at runtime**, not fixed upfront.

Best for: large-scale file migrations, bulk processing with variable complexity.

---

### 6. Hierarchical Delegation

Recursive decomposition — a top-level orchestrator delegates to sub-orchestrators, which delegate to workers.

```
[orchestrator]
  ├── [sub-orchestrator-A]
  │     ├── [worker-1]
  │     └── [worker-2]
  └── [sub-orchestrator-B]
        ├── [worker-3]
        └── [worker-4]
```

**Maximum 2 tiers. Agent nesting beyond 2 levels is prohibited.**

Best for: very large, multi-domain projects where different domains need independent coordination.

---

## Combining Patterns

Complex harnesses often combine patterns. Examples:

- **Parallel translation + per-language review**: Fan-out (translation) + Producer-Reviewer (per language)
- **Research then synthesis**: Fan-out (research) + Pipeline (analysis → report)
- **Dynamic code migration**: Supervisor (file allocation) + Pipeline (per-file: analyze → migrate → test)

When combining, document the overall pattern and the per-phase mode clearly in the orchestrator skill.

---

## Agent Definition Files

Every agent requires a `.claude/agents/{name}.md` file, even if it uses a built-in type.

**Built-in types** (`general-purpose`, `Explore`, `Plan`) have default behaviors but no persistent role definition. Custom types allow you to define:
- Domain-specific principles
- Input/output protocols
- Team communication rules

**Key distinction:**
- **Skills** (`.claude/skills/`) — procedural guides: *how to do a task*
- **Agents** (`.claude/agents/`) — persona-driven roles: *who does the task and how they behave*

Agents invoke skills via:
1. Tool calls (highest reusability — skill lives independently)
2. Inlining (when the skill is only ever used by this agent)
3. Reference loading (load the reference file on demand)
