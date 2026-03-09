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

7. **Domain sizing is the Planner's most important architectural decision.** A standard full-stack project has exactly 2 domains: BACKEND and FRONTEND. Adding a third domain requires a genuine runtime boundary (a separate service, worker, or CLI) — not feature complexity. Over-decomposed plans (5+ micro-domains for a simple CRUD app) are rejected at Phase 1 validation and sent back to the Planner.

8. **TeamLead never edits source code.** If the TeamLead identifies a defect in source files during any verification step, it writes a fix ticket and routes it to the Implementor. Editing source files is outside the TeamLead's role and degrades its coordination capability.

---

## Workflow Phases

```
PHASE 0: Requirements Interview  → TeamLead
PHASE 1: Planning & Research     → Planner
PHASE 2: Contracts               → Implementor (sequential)
PHASE 3: Implementation          → Implementors (parallel — BACKEND + FRONTEND)
PHASE 4: Review                  → Reviewers (parallel — BACKEND + FRONTEND simultaneously,
                                   each includes cross-domain contract findings at end of pass)
PHASE 5: Fix Cycle               → Implementors (targeted fixes, max 2 cycles)
PHASE 6: Complete                → TeamLead confirms with user
```

PHASE 3: Implementation → Implementors (parallel — invoke via runSubagent simultaneously)
Note: BACKEND and FRONTEND can run in parallel because SHARED_CONTRACTS.md is locked
before Phase 3 begins. Frontend codes against the contract, not against a running backend.

9. **npm install (and equivalent) runs once, in the Contracts pass, for all workspaces.** 
    Domain implementors never run package installs. If a domain implementor finds a 
    missing dependency, it writes a blocker to PROGRESS.md.
The TeamLead updates `SESSION.md → Current Phase` at every phase transition.
