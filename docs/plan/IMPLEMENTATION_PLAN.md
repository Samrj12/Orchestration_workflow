# Implementation Plan

> **Created by:** Planner  
> **Status:** DRAFT → IN_REVIEW → LOCKED  
> **References:** SESSION.md  
> **Last Updated:** [timestamp]

---

## 1. Architecture Overview

### System Design
<!-- High-level architecture. Components, their responsibilities, how they interact. -->

### Key Architectural Decisions
<!-- Every major decision must include a WHY. Implementors will hit forks — they need rationale. -->

| Decision | Rationale | Alternatives Considered |
|----------|-----------|------------------------|
| | | |

### Directory Structure
```
project/
├── 
```

---

## 2. Shared Contracts (Written First — Locked Before Parallel Work)

> See `docs/plan/SHARED_CONTRACTS.md` for the full contract definitions.
> Summary of cross-boundary interfaces:

### Data Models
<!-- List entities and their key fields. Full definitions are in SHARED_CONTRACTS.md -->

### API Endpoints
<!-- List every endpoint, method, request shape, response shape -->

| Method | Path | Request | Response | Auth Required |
|--------|------|---------|----------|---------------|
| | | | | |

### Shared Types / Interfaces
<!-- List type names and what they represent. Full definitions in SHARED_CONTRACTS.md -->

---

## 3. Complexity Assessment

| Domain | Complexity | Reason | Implementation Strategy |
|--------|-----------|--------|------------------------|
| | LOW/MED/HIGH | | Single pass / Multi-phase |

---

## 4. Task Manifest

> Every task has: ID, domain, description, files to touch, dependencies, acceptance criteria.
> The Parallelization Map (Section 5) shows which tasks can run concurrently.

### PHASE 2: Contracts Pass (Sequential — Must complete before Phase 3)

#### CONTRACTS-01: Shared Types & Interfaces
- **Files:** `src/types/index.ts` (or equivalent for stack)
- **Description:** Define all shared types from Section 2
- **Acceptance Criteria:**
  - [ ] All types from SHARED_CONTRACTS.md are defined
  - [ ] No circular dependencies
  - [ ] Types are exported from a single barrel file

#### CONTRACTS-02: API Schema / OpenAPI Spec
- **Files:** `docs/api/openapi.yaml` (or equivalent)
- **Description:** Define all API contracts formally
- **Acceptance Criteria:**
  - [ ] All endpoints from Section 2 are defined
  - [ ] Request/response schemas are complete
  - [ ] Error response schemas are standardized

#### CONTRACTS-03: Database Schema / Migration
- **Files:** `db/migrations/` or `prisma/schema.prisma` or equivalent
- **Description:** All data models formalized in the DB layer
- **Acceptance Criteria:**
  - [ ] All entities defined with correct field types
  - [ ] Indexes defined for all frequently queried fields
  - [ ] Foreign key constraints correct

---

### PHASE 3: Domain Implementation (Parallel)

<!-- COPY THIS BLOCK FOR EACH DOMAIN -->

#### Domain: [DOMAIN_NAME]

##### [DOMAIN]-01: [Task Name]
- **Dependencies:** CONTRACTS-01, CONTRACTS-02 (required before starting)
- **Complexity:** LOW / MEDIUM / HIGH
- **Files to Create/Modify:**
  - `src/[domain]/`
- **Description:**
  <!-- Specific, unambiguous. What exact behavior must be implemented. -->
- **Patterns to Follow:** See `SHARED_CONTRACTS.md` → [relevant section]
- **Context7 Libraries Needed:** [list libraries the implementor may need to look up]
- **Acceptance Criteria:**
  - [ ] 
  - [ ] 
- **Security Checklist:**
  - [ ] All inputs validated
  - [ ] Auth/authorization enforced
  - [ ] No secrets hardcoded
  - [ ] Errors handled and typed

##### [DOMAIN]-02: [Task Name]
- **Dependencies:** [DOMAIN]-01
<!-- ... -->

---

## 5. Parallelization Map

```
PHASE 2 (Sequential):
  CONTRACTS-01 → CONTRACTS-02 → CONTRACTS-03
       ↓
PHASE 3 (Parallel — all start after Phase 2):
  ┌─────────────────┬─────────────────┬─────────────────┐
  │  Domain A       │  Domain B       │  Domain C       │
  │  [DOMAIN_A]-01  │  [DOMAIN_B]-01  │  [DOMAIN_C]-01  │
  │  [DOMAIN_A]-02  │  [DOMAIN_B]-02  │  [DOMAIN_C]-02  │
  └─────────────────┴─────────────────┴─────────────────┘
       ↓
PHASE 4 (Sequential per domain, can overlap):
  Review Domain A → Review Domain B → Review Domain C
       ↓
PHASE 6 (Integration Review — after all domains pass):
  Cross-domain integration check
```

**Domain file boundaries (no implementor crosses these):**

| Domain | Owns These Paths | Must NOT Touch |
|--------|-----------------|----------------|
| | | |

---

## 6. Context7 Research Summary

> Synthesized patterns from current library documentation. Implementors follow these — do NOT deviate without surfacing to orchestrator.

### [Library/Framework Name] — v[version]

**Pattern for [use case]:**
```[language]
// canonical pattern from current docs
```

**Do NOT use (deprecated in this version):**
- 

---

## 7. Environment & Configuration

<!-- What env vars, config files, or setup steps are needed -->

| Variable | Purpose | Required | Example |
|----------|---------|----------|---------|
| | | | |

---

## Plan Status

- [ ] Architecture approved by TeamLead
- [ ] Shared contracts complete
- [ ] All task IDs assigned
- [ ] Parallelization map validated (no overlapping file ownership)
- [ ] Context7 research complete for all libraries
- [ ] Plan locked — no further modifications without TeamLead approval
