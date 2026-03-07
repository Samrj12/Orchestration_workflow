# Review Notes

> **Created by:** Reviewer  
> **Status:** IN_PROGRESS  
> **Last Updated:** [timestamp]

---

## Review Summary

| Domain | Pass # | Status | Critical | High | Medium | Low |
|--------|--------|--------|----------|------|--------|-----|
| | | PASS/FAIL | 0 | 0 | 0 | 0 |

**Overall Status:** 🔴 BLOCKED / 🟡 CONDITIONAL PASS / 🟢 PASS

---

## Domain Review: [DOMAIN_NAME]

> Files reviewed: [list exact files]  
> Plan reference: Tasks [DOMAIN]-01 through [DOMAIN]-0N  
> Review pass: #1

### Task [DOMAIN]-01 — [Task Name]
**Status:** ✅ PASS / ❌ FAIL

#### Findings

##### Finding [DOMAIN]-R01
- **Severity:** CRITICAL / HIGH / MEDIUM / LOW
- **File:** `src/[domain]/file.ts`
- **Line:** 47
- **Category:** SECURITY / LOGIC / CORRECTNESS / PERFORMANCE / STYLE
- **Issue:** Clear description of what is wrong
- **Why it matters:** Impact if left unfixed
- **Suggested Fix:** Specific, actionable instruction
  ```
  // Replace this:
  // bad code example
  
  // With this:
  // good code example
  ```

##### Finding [DOMAIN]-R02
<!-- ... -->

### Task [DOMAIN]-02 — [Task Name]
**Status:** ✅ PASS

<!-- No findings — all acceptance criteria met -->

---

## Integration Review (Phase 6)

> Run after ALL domains pass their domain review.
> Scope: cross-domain contract adherence, end-to-end flow, security boundaries only.

### API Contract Adherence
- [ ] Frontend calls match backend endpoint signatures
- [ ] Request body shapes match
- [ ] Response shapes match what frontend expects
- [ ] Error shapes handled correctly on frontend

### End-to-End Flow Completeness
<!-- Trace each user journey from the acceptance criteria end-to-end -->

| User Journey | Steps Verified | Status | Notes |
|-------------|----------------|--------|-------|
| | | | |

### Security Boundary Checks
- [ ] No client-side secrets
- [ ] Auth middleware applied to all protected routes
- [ ] CORS configured correctly
- [ ] Input validation present at all external boundaries

### Browser Testing Results
<!-- From integrated browser testing during review -->

| Test Scenario | Tool Used | Result | Screenshot/Log |
|--------------|-----------|--------|----------------|
| | | PASS/FAIL | |

---

## Fix Tickets Generated

> These are forwarded to TeamLead → SESSION.md → Fix Tickets table.
> Each ticket maps to exactly one finding.

| Ticket ID | Finding | Domain | File | Line | Severity | Assigned To |
|-----------|---------|--------|------|------|----------|-------------|
| FIX-001 | [DOMAIN]-R01 | | | | CRITICAL | Implementor |
| | | | | | | |

---

## Cycle Log

| Cycle | Date | Domains Re-reviewed | New Findings | Resolved Findings |
|-------|------|--------------------|--------------|--------------------|
| 1 | | | | |
| 2 | | | | |
