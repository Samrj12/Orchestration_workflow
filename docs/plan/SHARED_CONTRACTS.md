# Shared Contracts

> **Created by:** Planner  
> **Status:** DRAFT → LOCKED  
> **Last Updated:** [timestamp]
>
> ⚠️ **LOCKED before Phase 3 begins. No implementation agent may deviate from these definitions.**
> Any necessary change requires TeamLead approval and must be logged in SESSION.md.

---

## Types & Interfaces

> Define all shared types/interfaces in the project's primary language.
> Every field must be present. Optional fields must be marked. No `any`.

```typescript
// Example structure — replace with actual project language and types

export type UserId = string; // UUID
export type ISODateString = string; // ISO 8601

export interface User {
  id: UserId;
  email: string;
  name: string;
  role: UserRole;
  createdAt: ISODateString;
  updatedAt: ISODateString;
  // NEVER include: passwordHash, internalTokens, or any secret fields
}

export type UserRole = 'admin' | 'member' | 'viewer';

export interface PaginatedResponse<T> {
  data: T[];
  meta: {
    total: number;
    page: number;
    perPage: number;
    totalPages: number;
  };
}
```

---

## API Contracts

> Every endpoint. Every method. Full request and response schemas.
> Frontend and backend are both bound to these shapes. No deviations.

### Standard Error Response (used by ALL endpoints on failure)

```typescript
interface ApiError {
  error: {
    code: string;         // Machine-readable error code, e.g. 'NOT_FOUND', 'UNAUTHORIZED'
    message: string;      // Human-readable message (generic for 5xx, specific for 4xx)
    fields?: Record<string, string>; // Per-field validation errors (400 validation only)
  };
}
```

### Authentication

#### POST /api/auth/login
```
Request:  { email: string, password: string }
Response 200: { token: string, refreshToken: string, user: User }
Response 400: ApiError (code: VALIDATION_ERROR, fields: {...})
Response 401: ApiError (code: INVALID_CREDENTIALS)
```

#### POST /api/auth/logout
```
Request:  {} (auth token in Authorization header)
Response 200: {}
Response 401: ApiError (code: UNAUTHORIZED)
```

#### POST /api/auth/refresh
```
Request:  { refreshToken: string }
Response 200: { token: string, refreshToken: string }
Response 401: ApiError (code: INVALID_REFRESH_TOKEN)
```

---

## Database Schema

> All entities. All fields. Types, constraints, and indexes.

```sql
-- Example — replace with actual schema

CREATE TABLE users (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email       VARCHAR(255) NOT NULL UNIQUE,
  name        VARCHAR(100) NOT NULL,
  role        VARCHAR(20) NOT NULL DEFAULT 'member'
                CHECK (role IN ('admin', 'member', 'viewer')),
  password_hash VARCHAR(255) NOT NULL,
  created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_users_email ON users(email);

-- Add all tables here
```

---

## Event / Message Contracts (if applicable)

> If the system uses queues, websockets, or event buses, define all message shapes here.

```typescript
// Example
interface UserCreatedEvent {
  type: 'user.created';
  payload: {
    userId: UserId;
    email: string;
    timestamp: ISODateString;
  };
}
```

---

## Contract Changelog

> Log every change made after locking, with TeamLead approval reference.

| Date | Change | Reason | Approved By | Domains Affected |
|------|--------|--------|-------------|-----------------|
| | | | | |
