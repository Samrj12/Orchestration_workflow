# Global Agent Instructions

These instructions apply to ALL agents in this workspace. Every agent must read and follow them before beginning any task.

---

## Shared Memory Architecture

This workflow uses a persistent file-based memory system. No agent should ever rely on chat history or assume what a previous agent did. All state is written to files. All agents read from files.

### Canonical Files — Every Agent Must Know These

| File | Owner | Purpose |
|------|-------|---------|
| `SESSION.md` | TeamLead | Master state: requirements, phase, active tasks, blockers |
| `docs/plan/IMPLEMENTATION_PLAN.md` | Planner | Full technical plan, task manifest, parallelization map |
| `docs/plan/SHARED_CONTRACTS.md` | Planner (Phase 0) | All shared types, API schemas, DB models — written first, locked |
| `docs/review/REVIEW_NOTES.md` | Reviewer | Structured domain-by-domain review findings |
| `src/<domain>/PROGRESS.md` | Implementor | Per-domain progress log, written after each pass |

### Context Discipline — The Golden Rule

**No agent ever loads the full codebase or full plan into context.**
Each agent reads ONLY the slice of information its current task requires:

- TeamLead → reads `SESSION.md` only
- Planner → reads `SESSION.md` + fetches Context7 docs for specific libraries
- Implementor → reads its assigned task block from `IMPLEMENTATION_PLAN.md` + `SHARED_CONTRACTS.md` + its own `PROGRESS.md` + only the specific files for its domain
- Reviewer → reads the plan's acceptance criteria for one domain + only that domain's changed files + `REVIEW_NOTES.md` to append findings

---

## Universal Behavioral Rules

1. **Never assume. If a requirement is ambiguous, stop and surface the ambiguity to the orchestrator before proceeding.** Write `BLOCKED: <reason>` to the relevant file and halt.

2. **Every file write must be atomic and complete.** Do not write partial plans, partial contracts, or partial progress logs. A half-written file is worse than no file.

3. **Session.md is the source of truth for workflow phase.** If SESSION.md says the phase is "Planning", no implementor should be writing code. Each agent checks the current phase before acting.

4. **Task IDs are immutable.** Once the planner assigns a task ID (e.g., `AUTH-01`), that ID is used everywhere — in the plan, in PROGRESS.md, in REVIEW_NOTES.md, in fix tickets. Never rename or renumber tasks mid-workflow.

5. **Scope creep is forbidden.** If you identify something that should be done but is not in your task assignment, do NOT do it. Write it to `SESSION.md` under "Backlog / Out of Scope" and surface it to the orchestrator.

6. **Security is not optional.** Every agent enforces: no secrets in code, no unvalidated inputs at boundaries, parameterized queries only, authentication checks on every protected route. These are not review items — they are implementation requirements.

---

## Workflow Phases

```
PHASE 0: Requirements Interview  → TeamLead
PHASE 1: Planning & Research     → Planner
PHASE 2: Contracts               → Implementor (contracts-only pass)
PHASE 3: Parallel Implementation → Implementors (per domain, parallel)
PHASE 4: Review                  → Reviewer (per domain, sequential)
PHASE 5: Fix Cycle               → Implementors (targeted fixes only)
PHASE 6: Integration Review      → Reviewer (cross-domain pass)
PHASE 7: Complete                → TeamLead confirms with user
```

The TeamLead updates `SESSION.md → Current Phase` at every phase transition.
