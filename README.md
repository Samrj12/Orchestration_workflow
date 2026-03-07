# Copilot Agent Orchestration System

A production-grade multi-agent workflow for VS Code Copilot that takes you from requirements to reviewed, tested code — with zero context rot.

---

## Quick Start

1. Copy this entire folder into your project root
2. Open GitHub Copilot Chat in VS Code
3. Select the **TeamLead** agent from the agents dropdown
4. Describe what you want to build

That's it. TeamLead handles everything from here.

---

## File Structure

```
your-project/
├── AGENTS.md                           ← Global rules for ALL agents (auto-loaded)
├── SESSION.md                          ← Live workflow state (TeamLead maintains this)
│
├── .github/
│   ├── agents/
│   │   ├── team-lead.agent.md          ← Orchestrator: requirements + routing
│   │   ├── planner.agent.md            ← Research + architecture + task decomposition
│   │   ├── implementor.agent.md        ← Senior engineer: writes production code
│   │   └── reviewer.agent.md          ← Code review + browser testing
│   │
│   └── skills/
│       ├── implementation/
│       │   └── SKILL.md               ← Loaded by Implementor: error handling, security, patterns
│       └── review/
│           └── SKILL.md               ← Loaded by Reviewer: security checklist, browser playbooks
│
└── docs/
    ├── plan/
    │   ├── IMPLEMENTATION_PLAN.md      ← Planner output: full technical plan + task manifest
    │   └── SHARED_CONTRACTS.md        ← Planner output: types, API schemas, DB — locked
    └── review/
        └── REVIEW_NOTES.md            ← Reviewer output: findings + fix tickets
```

---

## Workflow Overview

```
User describes goal
      ↓
TeamLead: Requirements Interview (Phase 0)
  → Asks 8 structured questions in one message
  → Locks requirements in SESSION.md
      ↓
Planner: Research & Architecture (Phase 1)
  → Calls Context7 for current library docs
  → Writes SHARED_CONTRACTS.md (types, APIs, DB)
  → Writes IMPLEMENTATION_PLAN.md (tasks, dependencies, patterns)
      ↓
Implementor: Contracts Pass (Phase 2)
  → Writes shared types, API schemas, DB migrations
  → This MUST complete before parallel work starts
      ↓
Implementors × N: Parallel Implementation (Phase 3)
  → One agent per feature domain
  → Each domain owns a specific directory — no overlap
  → Each writes PROGRESS.md as persistent memory
  → Frontend domains verify with browser tools
      ↓
Reviewer: Domain Reviews (Phase 4)
  → One review pass per domain
  → Static code review + security checklist
  → Browser testing for UI domains
  → Writes structured findings to REVIEW_NOTES.md
      ↓
Implementors: Fix Cycle (Phase 5)
  → Targeted fixes only — specific file + line
  → Re-review of fixed domains
  → Cycles max twice before escalating to user
      ↓
Reviewer: Integration Review (Phase 6)
  → Cross-domain contract adherence
  → End-to-end flow verification
  → Full browser E2E testing
      ↓
TeamLead: Completion Summary (Phase 7)
  → All acceptance criteria verified
  → Backlog items surfaced
```

---

## Design Principles

### No Context Rot
Every agent reads from files, not from chat history. SESSION.md, IMPLEMENTATION_PLAN.md, PROGRESS.md, and REVIEW_NOTES.md are the memory of the system. Context resets are recoverable.

### Minimal Context Per Agent
Each agent reads only what it needs for its specific task. An implementor working on the auth domain never loads the entire codebase — just its task slice, the shared contracts, and its own progress log.

### Shared Contracts First
The parallelization map only works if domain boundaries are zero-overlap AND all cross-domain interfaces are defined before anyone starts implementing. The contracts pass enforces this.

### Vertical Domain Decomposition
Domains are feature slices (auth, dashboard, billing), not horizontal layers (frontend, backend). Each agent owns the full vertical slice of its domain — UI + API + data layer. This eliminates cross-domain dependencies within a feature.

### Fix Cycles Are Surgical
Reviewer findings generate precise fix tickets. Each ticket maps to a specific file, line, and problem. Implementors receive only the affected file + ticket context — not the full codebase again.

---

## Context7 Integration

Context7 is built into the Copilot agent runtime — no installation needed. The Planner and Implementor agents use it to:

- **Planner:** Research current library docs before writing architectural patterns. All patterns in the plan are verified against the pinned version.
- **Implementor:** Verify specific API method signatures before using them. One call to Context7 prevents hallucinated APIs that compile but fail at runtime.

The pattern: `mcp_context7_resolve-library-id` → `mcp_context7_get-library-docs` for the specific topic needed.

---

## Browser Testing Integration

The Implementor and Reviewer agents have access to VS Code's integrated browser tools (available from VS Code 1.110+):

| Tool | Used By | Purpose |
|------|---------|---------|
| `openBrowserPage` | Implementor, Reviewer | Launch the running dev server |
| `navigatePage` | Both | Navigate to the feature under test |
| `screenshotPage` | Both | Capture state for verification |
| `readPage` | Both | Verify content and accessibility |
| `clickElement`, `typeInPage`, `handleDialog` | Both | Interact with UI elements |
| `runPlaywrightCode` | Reviewer | Complex automation flows, E2E tests |

Implementors run smoke tests after implementing frontend tasks. Reviewers run thorough behavioral tests including error scenarios and edge cases.

---

## Customizing the Agents

### Adding a new domain implementor
The `implementor.agent.md` is domain-agnostic by design. TeamLead invokes it with domain-specific context each time. No need to create separate files for frontend vs backend vs mobile — the same agent adapts to its task assignment.

### Adding skills
Create a new directory under `.github/skills/` with a `SKILL.md` file. The skill's `description` field in the YAML frontmatter determines when it's loaded. Reference skills in your agent files by including their directory in the agent's context or instructions.

### Changing models
Update the `model:` field in any `.agent.md` file. You can specify a fallback list:
```yaml
model: ['claude-opus-4-5', 'claude-sonnet-4-5']
```

### Making it stack-specific
All agents and skills are intentionally stack-agnostic. If you want stack-specific versions:
1. Create `implementor-typescript.agent.md` with TypeScript-specific instructions
2. Add a `skills/typescript/SKILL.md` with TS patterns
3. TeamLead's instructions can select the right implementor based on SESSION.md tech stack
