# Data Model: Personal Financial Tracker MVP

**Date**: 2025-12-12
**Database**: PostgreSQL
**ORM**: Drizzle ORM

## Overview

This document defines the database schema for the financial tracker MVP. The model supports multi-user authentication with user-scoped data isolation, transaction management, category organization, and dashboard aggregations.

---

## Entity Relationship Diagram

```
┌─────────────┐
│    users    │
└──────┬──────┘
       │
       │ 1:N
       │
┌──────▼──────────────┐
│   transactions      │
└──────┬──────────────┘
       │ N:1
       │
┌──────▼──────────┐
│   categories    │
└─────────────────┘
```

**Relationships**:
- One user has many transactions (1:N)
- One transaction belongs to one category (N:1)
- One user has many categories (1:N, user-created only)
- Categories can be system defaults (shared across users) or user-specific

---

## Entities

### 1. users

Represents authenticated users of the system.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | `UUID` | PRIMARY KEY, DEFAULT gen_random_uuid() | Unique user identifier |
| `email` | `VARCHAR(255)` | UNIQUE, NOT NULL | User email address (login credential) |
| `password_hash` | `VARCHAR(255)` | NOT NULL | Bcrypt hashed password (never plaintext) |
| `created_at` | `TIMESTAMPTZ` | NOT NULL, DEFAULT NOW() | Account creation timestamp |
| `last_login_at` | `TIMESTAMPTZ` | NULL | Last successful login timestamp |

**Indexes**:
- `idx_users_email` on `email` (for fast login lookups)

**Validation Rules** (application layer):
- Email: Valid email format, unique across system
- Password: Minimum 8 characters before hashing
- Hashing: Use bcrypt with salt rounds ≥ 10

**Security Notes**:
- Never expose `password_hash` in API responses
- Use NextAuth.js JWT tokens for session management
- Implement rate limiting on login attempts (not database-level)

---

### 2. categories

Represents transaction categories (both system defaults and user-created).

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | `UUID` | PRIMARY KEY, DEFAULT gen_random_uuid() | Unique category identifier |
| `user_id` | `UUID` | NULL, FOREIGN KEY → users(id) ON DELETE CASCADE | Owner user (NULL for system defaults) |
| `name` | `VARCHAR(50)` | NOT NULL | Category display name |
| `type` | `VARCHAR(10)` | NOT NULL, CHECK (type IN ('income', 'expense')) | Category type |
| `is_default` | `BOOLEAN` | NOT NULL, DEFAULT FALSE | True for system defaults, false for user-created |
| `created_at` | `TIMESTAMPTZ` | NOT NULL, DEFAULT NOW() | Category creation timestamp |

**Indexes**:
- `idx_categories_user_id` on `user_id` (for filtering user categories)
- `idx_categories_type` on `type` (for filtering by income/expense)

**Unique Constraints**:
- `UNIQUE (user_id, name, type)` - Prevent duplicate category names per user per type
- NULL `user_id` allowed for system defaults

**Validation Rules** (application layer):
- Name: 1-50 characters, trim whitespace
- Type: Must be 'income' or 'expense'
- User-created categories: Must have non-NULL `user_id`
- System defaults: `user_id` = NULL, `is_default` = TRUE

**System Default Categories** (seeded on first run):

```sql
-- Expense defaults (user_id = NULL, is_default = TRUE)
Food, Transport, Rent, Utilities, Entertainment, Healthcare, Shopping, Mobile Data

-- Income defaults (user_id = NULL, is_default = TRUE)
Salary, Freelancing, Investment Returns, Business Income, Other Income
```

**Security Notes**:
- Users can only read system defaults (user_id IS NULL) + their own categories (user_id = current_user_id)
- Queries MUST filter: `WHERE user_id IS NULL OR user_id = $currentUserId`

---

### 3. transactions

Represents individual financial transactions (income or expense).

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | `UUID` | PRIMARY KEY, DEFAULT gen_random_uuid() | Unique transaction identifier |
| `user_id` | `UUID` | NOT NULL, FOREIGN KEY → users(id) ON DELETE CASCADE | Transaction owner |
| `category_id` | `UUID` | NOT NULL, FOREIGN KEY → categories(id) ON DELETE RESTRICT | Associated category |
| `amount` | `DECIMAL(12, 2)` | NOT NULL, CHECK (amount > 0) | Transaction amount in LKR (always positive) |
| `type` | `VARCHAR(10)` | NOT NULL, CHECK (type IN ('income', 'expense')) | Transaction type |
| `date` | `DATE` | NOT NULL | Transaction date (user-selected) |
| `description` | `VARCHAR(200)` | NOT NULL | Transaction description |
| `created_at` | `TIMESTAMPTZ` | NOT NULL, DEFAULT NOW() | Record creation timestamp |
| `updated_at` | `TIMESTAMPTZ` | NOT NULL, DEFAULT NOW() | Last update timestamp |

**Indexes**:
- `idx_transactions_user_id` on `user_id` (for filtering user transactions)
- `idx_transactions_date` on `date` (for monthly filtering)
- `idx_transactions_user_date` on `(user_id, date)` (composite index for dashboard queries)

**Validation Rules** (application layer):
- Amount: Positive number, up to 2 decimal places (banker's rounding applied)
- Type: Must be 'income' or 'expense'
- Date: Valid date between 2000-01-01 and 2099-12-31
- Description: 1-200 characters, required
- Category type must match transaction type (enforced in application logic)

**Security Notes**:
- ALL queries MUST filter by `user_id`: `WHERE user_id = $currentUserId`
- Delete: CASCADE from users (when user deleted, all transactions deleted)
- Delete: RESTRICT from categories (prevent deletion of category with transactions)

**Triggers** (optional, can be application-level):
- Update `updated_at` on every UPDATE

---

## Database Schema (SQL)

### Full Schema Definition

```sql
-- Enable UUID extension
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Users table
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  last_login_at TIMESTAMPTZ
);

CREATE INDEX idx_users_email ON users(email);

-- Categories table
CREATE TABLE categories (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  name VARCHAR(50) NOT NULL,
  type VARCHAR(10) NOT NULL CHECK (type IN ('income', 'expense')),
  is_default BOOLEAN NOT NULL DEFAULT FALSE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE (user_id, name, type)
);

CREATE INDEX idx_categories_user_id ON categories(user_id);
CREATE INDEX idx_categories_type ON categories(type);

-- Transactions table
CREATE TABLE transactions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  category_id UUID NOT NULL REFERENCES categories(id) ON DELETE RESTRICT,
  amount DECIMAL(12, 2) NOT NULL CHECK (amount > 0),
  type VARCHAR(10) NOT NULL CHECK (type IN ('income', 'expense')),
  date DATE NOT NULL,
  description VARCHAR(200) NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_transactions_user_id ON transactions(user_id);
CREATE INDEX idx_transactions_date ON transactions(date);
CREATE INDEX idx_transactions_user_date ON transactions(user_id, date);

-- Trigger to update updated_at timestamp
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_transactions_updated_at
BEFORE UPDATE ON transactions
FOR EACH ROW
EXECUTE FUNCTION update_updated_at_column();

-- Seed default categories
INSERT INTO categories (user_id, name, type, is_default) VALUES
  -- Expense defaults
  (NULL, 'Food', 'expense', TRUE),
  (NULL, 'Transport', 'expense', TRUE),
  (NULL, 'Rent', 'expense', TRUE),
  (NULL, 'Utilities', 'expense', TRUE),
  (NULL, 'Entertainment', 'expense', TRUE),
  (NULL, 'Healthcare', 'expense', TRUE),
  (NULL, 'Shopping', 'expense', TRUE),
  (NULL, 'Mobile Data', 'expense', TRUE),
  -- Income defaults
  (NULL, 'Salary', 'income', TRUE),
  (NULL, 'Freelancing', 'income', TRUE),
  (NULL, 'Investment Returns', 'income', TRUE),
  (NULL, 'Business Income', 'income', TRUE),
  (NULL, 'Other Income', 'income', TRUE);
```

---

## Drizzle ORM Schema (TypeScript)

**File**: `src/server/db/schema.ts`

```typescript
import { pgTable, uuid, varchar, timestamp, decimal, date, boolean, index, check } from 'drizzle-orm/pg-core';
import { relations } from 'drizzle-orm';

// Users table
export const users = pgTable('users', {
  id: uuid('id').primaryKey().defaultRandom(),
  email: varchar('email', { length: 255 }).notNull().unique(),
  passwordHash: varchar('password_hash', { length: 255 }).notNull(),
  createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow(),
  lastLoginAt: timestamp('last_login_at', { withTimezone: true }),
}, (table) => ({
  emailIdx: index('idx_users_email').on(table.email),
}));

// Categories table
export const categories = pgTable('categories', {
  id: uuid('id').primaryKey().defaultRandom(),
  userId: uuid('user_id').references(() => users.id, { onDelete: 'cascade' }),
  name: varchar('name', { length: 50 }).notNull(),
  type: varchar('type', { length: 10 }).notNull(), // 'income' | 'expense'
  isDefault: boolean('is_default').notNull().default(false),
  createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow(),
}, (table) => ({
  userIdIdx: index('idx_categories_user_id').on(table.userId),
  typeIdx: index('idx_categories_type').on(table.type),
  uniqueUserNameType: unique().on(table.userId, table.name, table.type),
  typeCheck: check('type', 'type IN (\'income\', \'expense\')'),
}));

// Transactions table
export const transactions = pgTable('transactions', {
  id: uuid('id').primaryKey().defaultRandom(),
  userId: uuid('user_id').notNull().references(() => users.id, { onDelete: 'cascade' }),
  categoryId: uuid('category_id').notNull().references(() => categories.id, { onDelete: 'restrict' }),
  amount: decimal('amount', { precision: 12, scale: 2 }).notNull(),
  type: varchar('type', { length: 10 }).notNull(), // 'income' | 'expense'
  date: date('date').notNull(),
  description: varchar('description', { length: 200 }).notNull(),
  createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).notNull().defaultNow(),
}, (table) => ({
  userIdIdx: index('idx_transactions_user_id').on(table.userId),
  dateIdx: index('idx_transactions_date').on(table.date),
  userDateIdx: index('idx_transactions_user_date').on(table.userId, table.date),
  amountCheck: check('amount', 'amount > 0'),
  typeCheck: check('type', 'type IN (\'income\', \'expense\')'),
}));

// Relations (for Drizzle joins)
export const usersRelations = relations(users, ({ many }) => ({
  transactions: many(transactions),
  categories: many(categories),
}));

export const categoriesRelations = relations(categories, ({ one, many }) => ({
  user: one(users, {
    fields: [categories.userId],
    references: [users.id],
  }),
  transactions: many(transactions),
}));

export const transactionsRelations = relations(transactions, ({ one }) => ({
  user: one(users, {
    fields: [transactions.userId],
    references: [users.id],
  }),
  category: one(categories, {
    fields: [transactions.categoryId],
    references: [categories.id],
  }),
}));

// TypeScript types (inferred from schema)
export type User = typeof users.$inferSelect;
export type NewUser = typeof users.$inferInsert;
export type Category = typeof categories.$inferSelect;
export type NewCategory = typeof categories.$inferInsert;
export type Transaction = typeof transactions.$inferSelect;
export type NewTransaction = typeof transactions.$inferInsert;
```

---

## Query Patterns

### Common Queries (Security-Aware)

#### Get User's Transactions for a Month

```typescript
// src/server/dashboard/queries.ts
import { db } from '../db/client';
import { transactions, categories } from '../db/schema';
import { eq, and, gte, lte, sql } from 'drizzle-orm';

export async function getTransactionsForMonth(userId: string, year: number, month: number) {
  const startDate = new Date(year, month - 1, 1);
  const endDate = new Date(year, month, 0);

  return await db
    .select({
      id: transactions.id,
      amount: transactions.amount,
      type: transactions.type,
      date: transactions.date,
      description: transactions.description,
      categoryName: categories.name,
    })
    .from(transactions)
    .innerJoin(categories, eq(transactions.categoryId, categories.id))
    .where(
      and(
        eq(transactions.userId, userId), // CRITICAL: User data isolation
        gte(transactions.date, startDate.toISOString().split('T')[0]),
        lte(transactions.date, endDate.toISOString().split('T')[0])
      )
    )
    .orderBy(transactions.date);
}
```

#### Get Monthly Summary

```typescript
export async function getMonthlySummary(userId: string, year: number, month: number) {
  const startDate = new Date(year, month - 1, 1);
  const endDate = new Date(year, month, 0);

  const result = await db
    .select({
      type: transactions.type,
      total: sql<number>`CAST(SUM(${transactions.amount}) AS DECIMAL(12,2))`,
    })
    .from(transactions)
    .where(
      and(
        eq(transactions.userId, userId), // CRITICAL: User data isolation
        gte(transactions.date, startDate.toISOString().split('T')[0]),
        lte(transactions.date, endDate.toISOString().split('T')[0])
      )
    )
    .groupBy(transactions.type);

  const income = result.find(r => r.type === 'income')?.total || 0;
  const expenses = result.find(r => r.type === 'expense')?.total || 0;

  return {
    income,
    expenses,
    balance: income - expenses,
  };
}
```

#### Get Categories for User

```typescript
export async function getCategoriesForUser(userId: string, type?: 'income' | 'expense') {
  const conditions = [
    sql`(${categories.userId} IS NULL OR ${categories.userId} = ${userId})`, // System defaults OR user's own
  ];

  if (type) {
    conditions.push(eq(categories.type, type));
  }

  return await db
    .select()
    .from(categories)
    .where(and(...conditions))
    .orderBy(categories.isDefault, categories.name);
}
```

---

## Data Migration Strategy

1. **Initial Setup**: Run `schema.sql` to create tables and seed default categories
2. **Drizzle Migrations**: Use `drizzle-kit generate` to create migration files
3. **Apply Migrations**: Run `drizzle-kit migrate` on deployment
4. **Version Control**: Store migrations in `src/server/db/migrations/`

**Migration Workflow**:
```bash
# Generate migration from schema changes
npx drizzle-kit generate

# Apply migrations to database
npx drizzle-kit migrate

# Push schema directly (dev only, not for production)
npx drizzle-kit push
```

---

## Security Checklist

- ✅ All user queries filtered by `user_id`
- ✅ Password hash never exposed in API responses
- ✅ Foreign key constraints enforce data integrity
- ✅ CASCADE delete on user → transactions/categories
- ✅ RESTRICT delete on category → transactions (prevent orphans)
- ✅ CHECK constraints on amount (> 0) and type (enum)
- ✅ Indexes on high-query columns for performance
- ✅ UUID primary keys (prevent enumeration attacks)
- ✅ Timestamps (created_at, updated_at) for audit trails

---

## Next Steps

1. Create `contracts/` directory with API specifications
2. Generate `quickstart.md` with database setup instructions
3. Implement data access layer in `src/server/*/queries.ts`
