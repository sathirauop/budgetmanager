# API Specification: Personal Financial Tracker MVP

**Date**: 2025-12-12
**Architecture**: Next.js 16 App Router with Server Actions and Route Handlers
**Authentication**: NextAuth.js v5 (JWT sessions)

## Overview

This document defines the API contracts for the financial tracker MVP. The application uses two mechanisms for API interactions:

1. **Server Actions** - For mutations (create, update, delete) from forms and interactive components
2. **Route Handlers** - For RESTful endpoints (primarily authentication flows)

All endpoints require authentication except for registration, login, and password recovery initiation.

---

## Authentication

### Session Management

- **Method**: JWT tokens managed by NextAuth.js v5
- **Session Duration**: 30 minutes of inactivity
- **Cookie**: `httpOnly`, `secure` (production), `sameSite: lax`
- **CSRF Protection**: Built-in via NextAuth.js

### Getting Current User

```typescript
// Server Component
import { auth } from '@/auth';

const session = await auth();
const userId = session?.user?.id;
```

```typescript
// Client Component
'use client';
import { useSession } from 'next-auth/react';

const { data: session, status } = useSession();
const userId = session?.user?.id;
```

---

## Route Handlers (Authentication)

### 1. Register User

**POST** `/api/auth/register`

Creates a new user account with email and password.

**Request Body**:
```typescript
{
  email: string;      // Valid email format, unique
  password: string;   // Minimum 8 characters
}
```

**Response** (200 OK):
```typescript
{
  success: true;
  user: {
    id: string;
    email: string;
  };
}
```

**Error Responses**:
- `400 Bad Request` - Invalid email format or weak password
  ```typescript
  {
    error: "Invalid email format" | "Password must be at least 8 characters"
  }
  ```
- `409 Conflict` - Email already registered
  ```typescript
  {
    error: "Email already registered. Please login or use a different email."
  }
  ```

**Validation Rules**:
- Email: RFC 5322 compliant, max 255 characters
- Password: Min 8 characters, trimmed
- Email uniqueness checked before hashing password

---

### 2. Login

**POST** `/api/auth/signin`

Authenticates user with email and password, returns JWT session token.

**Request Body**:
```typescript
{
  email: string;
  password: string;
}
```

**Response** (200 OK):
```typescript
{
  success: true;
  user: {
    id: string;
    email: string;
  };
  // JWT token set in httpOnly cookie automatically by NextAuth
}
```

**Error Responses**:
- `401 Unauthorized` - Invalid credentials
  ```typescript
  {
    error: "Invalid email or password" // Never reveal which field is wrong
  }
  ```

**Security Notes**:
- Updates `last_login_at` timestamp on successful login
- Rate limiting recommended (not implemented in MVP): 5 attempts per 15 minutes per IP

---

### 3. Logout

**POST** `/api/auth/signout`

Invalidates current session.

**Request**: No body required (uses session cookie)

**Response** (200 OK):
```typescript
{
  success: true;
}
```

**Note**: NextAuth.js automatically clears session cookie

---

### 4. Request Password Reset

**POST** `/api/auth/forgot-password`

Sends password reset email with token.

**Request Body**:
```typescript
{
  email: string;
}
```

**Response** (200 OK):
```typescript
{
  success: true;
  message: "Password reset email sent. Check your inbox."
}
```

**Error Responses**:
- `404 Not Found` - Email not registered (or return 200 for security - don't reveal user existence)
  ```typescript
  {
    success: true; // Return success even if email doesn't exist (security)
    message: "If that email exists, a password reset link has been sent."
  }
  ```

**Implementation**:
- Generate secure random token (UUID or crypto.randomBytes)
- Store token with expiry (15 minutes) in database or JWT
- Send email via Resend with reset link: `/reset-password?token=...`

---

### 5. Reset Password

**POST** `/api/auth/reset-password`

Resets password using token from email.

**Request Body**:
```typescript
{
  token: string;
  newPassword: string; // Minimum 8 characters
}
```

**Response** (200 OK):
```typescript
{
  success: true;
  message: "Password reset successfully. You can now login."
}
```

**Error Responses**:
- `400 Bad Request` - Invalid or expired token
  ```typescript
  {
    error: "Invalid or expired reset token. Please request a new one."
  }
  ```
- `400 Bad Request` - Weak password
  ```typescript
  {
    error: "Password must be at least 8 characters"
  }
  ```

---

## Server Actions (Transactions & Categories)

### Transaction Actions

**File**: `src/server/transactions/actions.ts`

#### 1. Create Transaction

```typescript
'use server';

export async function createTransaction(data: {
  amount: number;
  type: 'income' | 'expense';
  date: string; // ISO 8601 date (YYYY-MM-DD)
  description: string;
  categoryId: string;
}): Promise<{ success: boolean; transaction?: Transaction; error?: string }>;
```

**Validation**:
- `amount`: Positive number, rounded to 2 decimals (banker's rounding)
- `type`: Must be 'income' or 'expense'
- `date`: Valid date between 2000-01-01 and 2099-12-31
- `description`: 1-200 characters, trimmed
- `categoryId`: Must exist and belong to user or be system default
- Category type must match transaction type

**Returns**:
```typescript
{
  success: true;
  transaction: {
    id: string;
    amount: string; // "500.00"
    type: 'income' | 'expense';
    date: string;
    description: string;
    categoryId: string;
    createdAt: string; // ISO 8601 timestamp
  };
}
```

**Errors**:
```typescript
{
  success: false;
  error: "Amount must be positive" | "Description is required" | "Invalid category"
}
```

---

#### 2. Update Transaction

```typescript
'use server';

export async function updateTransaction(
  id: string,
  data: {
    amount?: number;
    type?: 'income' | 'expense';
    date?: string;
    description?: string;
    categoryId?: string;
  }
): Promise<{ success: boolean; transaction?: Transaction; error?: string }>;
```

**Security**:
- MUST verify transaction belongs to current user before updating
- Return 404 if transaction not found or doesn't belong to user

**Validation**: Same as create transaction

**Returns**: Same structure as create transaction

---

#### 3. Delete Transaction

```typescript
'use server';

export async function deleteTransaction(id: string): Promise<{ success: boolean; error?: string }>;
```

**Security**:
- MUST verify transaction belongs to current user before deleting
- Return 404 if transaction not found or doesn't belong to user

**Returns**:
```typescript
{
  success: true;
}
```

**Errors**:
```typescript
{
  success: false;
  error: "Transaction not found" | "Unauthorized"
}
```

---

### Category Actions

**File**: `src/server/categories/actions.ts`

#### 1. Create Category

```typescript
'use server';

export async function createCategory(data: {
  name: string;
  type: 'income' | 'expense';
}): Promise<{ success: boolean; category?: Category; error?: string }>;
```

**Validation**:
- `name`: 1-50 characters, trimmed
- `type`: Must be 'income' or 'expense'
- Name must be unique for user within type (case-insensitive check recommended)

**Returns**:
```typescript
{
  success: true;
  category: {
    id: string;
    name: string;
    type: 'income' | 'expense';
    isDefault: false;
  };
}
```

**Errors**:
```typescript
{
  success: false;
  error: "Category name already exists" | "Name must be 1-50 characters"
}
```

---

#### 2. Get Categories

```typescript
'use server';

export async function getCategories(type?: 'income' | 'expense'): Promise<Category[]>;
```

**Returns**: Array of categories (system defaults + user's custom categories)
```typescript
[
  {
    id: string;
    name: string;
    type: 'income' | 'expense';
    isDefault: boolean;
  }
]
```

**Security**: Automatically filters to system defaults (user_id IS NULL) + current user's categories

---

### Dashboard Queries

**File**: `src/server/dashboard/queries.ts`

#### 1. Get Monthly Summary

```typescript
'use server';

export async function getMonthlySummary(year: number, month: number): Promise<{
  income: string;    // "150000.00"
  expenses: string;  // "45000.00"
  balance: string;   // "105000.00"
}>;
```

**Validation**:
- `year`: 2000-2099
- `month`: 1-12

**Returns**: Formatted LKR amounts with 2 decimal places

---

#### 2. Get Transactions for Month

```typescript
'use server';

export async function getTransactionsForMonth(year: number, month: number): Promise<Transaction[]>;
```

**Returns**: Array of transactions for the specified month, sorted by date descending
```typescript
[
  {
    id: string;
    amount: string; // "500.00"
    type: 'income' | 'expense';
    date: string; // "2025-12-01"
    description: string;
    category: {
      id: string;
      name: string;
    };
    createdAt: string;
  }
]
```

---

#### 3. Get Category Breakdown

```typescript
'use server';

export async function getCategoryBreakdown(
  year: number,
  month: number,
  type: 'income' | 'expense'
): Promise<{ categoryName: string; total: string; percentage: number }[]>;
```

**Returns**: Aggregated totals by category for pie/donut chart rendering
```typescript
[
  {
    categoryName: "Food",
    total: "15000.00",
    percentage: 33.33 // Calculated: (15000 / 45000) * 100
  },
  {
    categoryName: "Rent",
    total: "25000.00",
    percentage: 55.56
  }
]
```

---

## Error Handling & Retry Logic

### Client-Side Retry

All API calls from Client Components should use the retry utility:

```typescript
// src/lib/api-retry.ts
export async function fetchWithRetry(
  fn: () => Promise<Response>,
  maxRetries = 3,
  initialDelay = 1000
): Promise<Response> {
  let lastError: Error;

  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error as Error;
      if (i < maxRetries - 1) {
        const delay = initialDelay * Math.pow(2, i); // Exponential backoff
        await new Promise(resolve => setTimeout(resolve, delay));
      }
    }
  }

  throw lastError!;
}
```

**Usage**:
```typescript
const result = await fetchWithRetry(async () => {
  return createTransaction(data);
});
```

### Server-Side Error Responses

All Server Actions return consistent error format:

```typescript
{
  success: false;
  error: string; // Human-readable error message
}
```

Route Handlers return standard HTTP status codes + JSON error objects.

---

## Data Formats

### Currency

**Format**: `"Rs. X,XXX.XX"`
- Thousands separator: comma (,)
- Decimal places: always 2
- Prefix: "Rs. " (Sri Lankan Rupee symbol with space)

**Example**: `"Rs. 150,000.00"`

### Dates

**Storage**: ISO 8601 date format (`YYYY-MM-DD`)
**Display**: Localized based on user preference (optional for MVP)

**Example**: `"2025-12-12"`

### Timestamps

**Format**: ISO 8601 with timezone (`YYYY-MM-DDTHH:mm:ss.sssZ`)

**Example**: `"2025-12-12T14:30:00.000Z"`

---

## Security Considerations

1. **Authentication**: All endpoints except register, login, and forgot-password require authenticated session
2. **Authorization**: All data queries filtered by `user_id` from session
3. **CSRF Protection**: Built-in via NextAuth.js for Route Handlers, Server Actions protected by Next.js
4. **Rate Limiting**: Recommended for login/register endpoints (not implemented in MVP)
5. **Input Validation**: All inputs validated with Zod schemas before database access
6. **SQL Injection**: Prevented by Drizzle ORM parameterized queries
7. **XSS Prevention**: React automatic escaping + Content Security Policy headers
8. **Password Security**: Bcrypt hashing with salt rounds ≥ 10
9. **Session Expiry**: 30-minute inactivity timeout enforced by NextAuth.js

---

## Testing Scenarios

### Authentication Flow
1. Register → Login → Access Dashboard → Logout
2. Register with duplicate email → 409 error
3. Login with wrong password → 401 error
4. Session expires after 30 minutes → Redirect to login

### Transaction Flow
1. Create transaction → Appears in dashboard
2. Edit transaction → Changes reflected immediately
3. Delete transaction → Removed from dashboard
4. Create transaction with invalid amount → Validation error

### API Retry Logic
1. Simulate network failure → 3 automatic retries → Error notification
2. Simulate slow response → Loading indicator shown
3. Successful retry → No error notification

---

## Next Steps

1. Implement Server Actions in `src/server/*/actions.ts`
2. Implement Route Handlers in `app/api/auth/*`
3. Create `quickstart.md` with API usage examples
4. Update agent context with API patterns
