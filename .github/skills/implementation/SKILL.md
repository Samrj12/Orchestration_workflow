---
name: senior-implementation
description: >
  Load this skill when implementing features, writing business logic, building APIs,
  creating data access layers, or writing any production code. Provides senior engineer
  patterns for error handling, security, typing, and code structure across any stack.
---

# Senior Implementation Skill

This skill provides prescriptive implementation patterns that apply across tech stacks. Every pattern here represents a hard-won engineering standard.

---

## Error Handling Patterns

### The Typed Result Pattern (use in any language)

Never throw exceptions for expected failures. Return typed results.

**TypeScript:**
```typescript
type Result<T, E = Error> =
  | { ok: true; value: T }
  | { ok: false; error: E };

// Usage
async function findUser(id: string): Promise<Result<User, UserNotFoundError | DatabaseError>> {
  try {
    const user = await db.users.findById(id);
    if (!user) return { ok: false, error: new UserNotFoundError(id) };
    return { ok: true, value: user };
  } catch (err) {
    return { ok: false, error: new DatabaseError('findUser', err) };
  }
}

// Caller
const result = await findUser(userId);
if (!result.ok) {
  if (result.error instanceof UserNotFoundError) return res.status(404).json(toApiError(result.error));
  return res.status(500).json(toApiError(result.error));
}
const user = result.value; // typed, safe
```

**Go equivalent:**
```go
func FindUser(id string) (User, error) {
    // errors.As pattern for typed error checking
}
```

**Python equivalent:**
```python
from dataclasses import dataclass
from typing import Generic, TypeVar

T = TypeVar('T')
E = TypeVar('E', bound=Exception)

@dataclass
class Ok(Generic[T]):
    value: T

@dataclass
class Err(Generic[E]):
    error: E

Result = Ok[T] | Err[E]
```

### Error Class Hierarchy

Define a consistent error hierarchy per domain:
```typescript
class AppError extends Error {
  constructor(
    message: string,
    public readonly code: string,
    public readonly statusCode: number,
    public readonly cause?: unknown
  ) { super(message); this.name = this.constructor.name; }
}

class NotFoundError extends AppError {
  constructor(resource: string, id: string) {
    super(`${resource} not found: ${id}`, 'NOT_FOUND', 404);
  }
}

class ValidationError extends AppError {
  constructor(public readonly fields: Record<string, string>) {
    super('Validation failed', 'VALIDATION_ERROR', 400);
  }
}

class UnauthorizedError extends AppError {
  constructor() { super('Unauthorized', 'UNAUTHORIZED', 401); }
}
```

---

## Input Validation Patterns

### Validate at the boundary — everything entering your system is untrusted

```typescript
// Use a validation library (zod, yup, class-validator — whatever is in the project)
// The point: validation at ENTRY, not deep in business logic

const CreateUserSchema = z.object({
  email: z.string().email().max(255).toLowerCase(),
  password: z.string().min(8).max(72), // bcrypt max is 72
  name: z.string().min(1).max(100).trim(),
});

// In your handler/controller — first thing, always
const parseResult = CreateUserSchema.safeParse(req.body);
if (!parseResult.success) {
  return res.status(400).json(toApiError(new ValidationError(parseResult.error.flatten().fieldErrors)));
}
const body = parseResult.data; // fully typed and validated from here down
```

---

## API Response Patterns

### Consistent envelope — never deviate from the contract

```typescript
// Success
{ "data": T, "meta"?: { pagination, etc. } }

// Error
{ "error": { "code": string, "message": string, "fields"?: Record<string, string> } }

// NEVER: raw data at root for some endpoints, envelope for others
// NEVER: different error shapes per endpoint
```

---

## Authentication & Authorization Patterns

### Auth check — must happen in middleware, not business logic

```typescript
// Middleware — applied to protected routes, NOT per-function
function requireAuth(req: Request, res: Response, next: NextFunction) {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).json({ error: { code: 'UNAUTHORIZED', message: 'Authentication required' } });
  
  const payload = verifyToken(token); // throws on invalid/expired
  if (!payload.ok) return res.status(401).json({ error: { code: 'UNAUTHORIZED', message: 'Invalid or expired token' } });
  
  req.user = payload.value; // attach to request
  next();
}

// Authorization — separate concern from authentication
function requireRole(role: UserRole) {
  return (req: Request, res: Response, next: NextFunction) => {
    if (!req.user) return res.status(401).json(/* ... */);
    if (req.user.role !== role) return res.status(403).json({ error: { code: 'FORBIDDEN', message: 'Insufficient permissions' } });
    next();
  };
}

// Resource ownership check — don't forget this one
function requireOwnership(getResourceUserId: (req: Request) => Promise<string>) {
  return async (req: Request, res: Response, next: NextFunction) => {
    const resourceUserId = await getResourceUserId(req);
    if (req.user.id !== resourceUserId && req.user.role !== 'admin') {
      return res.status(403).json(/* ... */);
    }
    next();
  };
}
```

---

## Data Access Patterns

### Repository pattern — isolate data access from business logic

```typescript
interface UserRepository {
  findById(id: string): Promise<Result<User, NotFoundError | DatabaseError>>;
  findByEmail(email: string): Promise<Result<User | null, DatabaseError>>;
  create(data: CreateUserData): Promise<Result<User, DatabaseError>>;
  update(id: string, data: UpdateUserData): Promise<Result<User, NotFoundError | DatabaseError>>;
}

// Business logic calls the interface, never the database directly
class UserService {
  constructor(private readonly users: UserRepository) {}
  
  async getUserProfile(requesterId: string, targetId: string): Promise<Result<UserProfile, AppError>> {
    // authorization check before data access
    if (requesterId !== targetId) {
      const requester = await this.users.findById(requesterId);
      if (!requester.ok) return requester;
      if (requester.value.role !== 'admin') return { ok: false, error: new ForbiddenError() };
    }
    
    const user = await this.users.findById(targetId);
    if (!user.ok) return user;
    return { ok: true, value: toUserProfile(user.value) };
  }
}
```

---

## State Management Patterns (Frontend)

### Discriminated union state — eliminates impossible states

```typescript
// DO THIS
type AuthState =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'authenticated'; user: User }
  | { status: 'error'; error: string };

// NOT THIS — allows impossible combinations like isLoading: true, user: {...}
type BadAuthState = {
  isLoading: boolean;
  user: User | null;
  error: string | null;
};
```

---

## Performance Patterns

### Database queries — think about N+1 before every query

- Never query inside a loop
- Use eager loading / joins for related data you know you need
- Add indexes on every field used in WHERE, ORDER BY, JOIN conditions
- Paginate every list endpoint — never return unbounded results

### Caching — explicit, not accidental

```typescript
// If you cache, make expiry explicit and visible
const CACHE_TTL_SECONDS = 60 * 5; // 5 minutes — visible, not magic

// Document what cache invalidation triggers exist
// If there is no invalidation plan, do not cache
```
