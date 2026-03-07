---
name: senior-review
description: >
  Load this skill when reviewing code, writing test cases, performing security audits,
  or doing browser-based feature verification. Provides structured review checklists,
  security vulnerability patterns to look for, and browser testing playbooks.
---

# Senior Review Skill

This skill provides structured review checklists and patterns for identifying real bugs, security vulnerabilities, and logical defects. It is designed to keep reviews precise and actionable.

---

## The Review Mindset

A review has two goals:
1. Verify the implementation matches the requirements and contracts
2. Find defects before users or attackers do

It does not exist to refactor code you would have written differently, or to enforce stylistic preferences not in the project standards.

**Start every review by reading the acceptance criteria.** Each criterion is a test. Every defect you find is a criterion that fails.

---

## Security Vulnerability Checklist

These are the patterns that cause the most real-world damage. Check every one.

### Injection Vulnerabilities
- [ ] **SQL Injection:** Any string concatenation into queries? Use parameterized queries.
- [ ] **Command Injection:** Any user input passed to `exec`, `spawn`, `system`?
- [ ] **Path Traversal:** Any user-controlled file paths? Must be sanitized and confined to safe directory.

### Authentication & Authorization Flaws
- [ ] **Broken Auth:** Is auth middleware actually applied (check route registration, not just definition)?
- [ ] **JWT:** Are tokens verified (signature + expiry)? Is the algorithm specified (never `alg: none`)?
- [ ] **Privilege Escalation:** Can a regular user call admin endpoints by manipulating parameters?
- [ ] **IDOR:** Does the code verify the resource belongs to the requesting user? e.g., `GET /users/:id/data` — does it check `user.id === req.user.id`?
- [ ] **Password Storage:** Are passwords hashed with bcrypt/argon2/scrypt? Never MD5/SHA1/plaintext.

### Data Exposure
- [ ] **Sensitive Fields:** Does API response include fields that shouldn't be exposed (passwords, tokens, SSNs)?
- [ ] **Error Leakage:** Do error messages expose stack traces, internal paths, or DB schemas to clients?
- [ ] **Logging PII:** Are passwords, tokens, or personal data logged?

### Input Handling
- [ ] **Validation:** Is all user input validated for type, format, length, and range?
- [ ] **Mass Assignment:** Can a user set fields they shouldn't (e.g., `role: 'admin'` in a create user request)?
- [ ] **File Upload:** Is file type validated (by magic bytes, not extension)? Is size limited?

---

## Logic Review Checklist

### Boundary Conditions
- What happens at 0? at -1? at max integer?
- What happens with empty string, empty array, empty object?
- What happens with null/undefined/None inputs?
- What happens when a required external service is unavailable?

### Async Correctness
- Is there any `await` missing? (Functions that look synchronous but are async)
- Are parallel operations using `Promise.all` when they should be sequential (or vice versa)?
- Is there a race condition where two concurrent requests can produce inconsistent state?

### State Consistency
- If operation A partially succeeds and then fails, is the system in a consistent state?
- Are there any operations that should be transactions but aren't?
- Can the same action be triggered twice (e.g., double-submit) and cause duplicate data?

---

## Contract Verification (Integration Review)

For each API endpoint in SHARED_CONTRACTS.md:

```
Contract: POST /api/auth/login
  Request: { email: string, password: string }
  Response 200: { token: string, user: { id, email, name } }
  Response 401: { error: { code: 'INVALID_CREDENTIALS', message: string } }
  Response 400: { error: { code: 'VALIDATION_ERROR', fields: Record<string, string> } }

Backend check:
  [ ] Route exists at POST /api/auth/login
  [ ] Validates email and password presence and format
  [ ] Returns 200 with { token, user } shape on success
  [ ] Returns 401 with correct error code on bad credentials
  [ ] Returns 400 with field errors on validation failure

Frontend check:
  [ ] Calls POST /api/auth/login (not /login, not /auth/login)
  [ ] Sends { email, password } in request body
  [ ] Reads .token and .user from success response
  [ ] Handles 401 with specific "invalid credentials" message
  [ ] Handles 400 with per-field validation display
  [ ] Does NOT assume 200 means success (checks response structure)
```

---

## Browser Testing Playbooks

### Login Flow
```javascript
// runPlaywrightCode
// Happy path
await page.goto('/login');
await page.fill('[name="email"]', 'test@example.com');
await page.fill('[name="password"]', 'ValidPass123!');
await page.click('[type="submit"]');
await page.waitForURL('**/dashboard', { timeout: 5000 });
// Verify authenticated state
expect(await page.isVisible('[data-testid="user-menu"]')).toBe(true);

// Error path: wrong password
await page.goto('/login');
await page.fill('[name="email"]', 'test@example.com');
await page.fill('[name="password"]', 'wrongpassword');
await page.click('[type="submit"]');
await page.waitForSelector('[role="alert"]');
const errorText = await page.textContent('[role="alert"]');
expect(errorText).toContain('Invalid'); // Should not expose which field is wrong

// Error path: validation
await page.goto('/login');
await page.click('[type="submit"]'); // submit empty
await page.waitForSelector('[data-testid="email-error"]');
```

### Form Validation
```javascript
// runPlaywrightCode — generic form validation test
// 1. Submit empty form — expect field-level errors
// 2. Enter invalid format — expect format-specific error
// 3. Enter valid data — expect no errors
// 4. Submit valid form — expect success state

// Check console for errors throughout
const errors = [];
page.on('console', msg => { if (msg.type() === 'error') errors.push(msg.text()); });
// After test: expect(errors).toHaveLength(0);
```

### Accessibility Spot-Check
```javascript
// runPlaywrightCode
// Tab through all interactive elements
await page.keyboard.press('Tab');
// Verify focus is visible (not just present)
const focusedElement = await page.evaluate(() => document.activeElement?.getAttribute('data-testid'));
// Submit form with Enter key (keyboard users must be able to do this)
await page.keyboard.press('Enter');
```

---

## Writing Fix Tickets

A good fix ticket is immediately actionable. The implementor should be able to read it and write the fix without needing to re-read your full finding.

**Template:**
```
FIX-[N] | Severity: [CRITICAL/HIGH/MEDIUM/LOW]
Domain: [domain]
File: src/[domain]/file.ts, line [N]

PROBLEM: [One sentence describing the incorrect behavior]

ROOT CAUSE: [One sentence explaining why it's wrong]

FIX: [Specific, prescriptive instruction]
  - Add [specific thing] at [specific location]
  - Change [this] to [that]
  - Remove [this] because [reason]

VERIFY: [How to confirm the fix is correct]
  - [Test to run / behavior to observe]
```

Anything missing from this template means the implementor has to guess. Guessing creates new bugs.
