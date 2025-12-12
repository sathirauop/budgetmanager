# Implementation Plan: Personal Financial Tracker MVP

**Branch**: `001-financial-tracker-mvp` | **Date**: 2025-12-12 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/001-financial-tracker-mvp/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command. See `.specify/templates/commands/plan.md` for the execution workflow.

## Summary

A personal financial tracking application with transaction logging, category management, and dashboard visualizations in LKR currency. The application uses Next.js 16 App Router with Server Components, Route Handlers, and Server Actions for the frontend and backend logic. PostgreSQL provides data persistence with user authentication (email/password). The design is mobile-first, visually minimal, and security-focused with automatic API retry logic and 30-minute session timeouts.

## Technical Context

**Language/Version**: TypeScript 5+ (strict mode), Node.js 20+
**Primary Dependencies**: Next.js 16 (App Router), React 19, TailwindCSS v4, PostgreSQL client (NEEDS CLARIFICATION: which library - node-postgres, Prisma, Drizzle), charting library (NEEDS CLARIFICATION: Recharts, Chart.js, or other), email service (NEEDS CLARIFICATION: Resend, SendGrid, Nodemailer)
**Storage**: PostgreSQL with user-scoped data isolation
**Testing**: NEEDS CLARIFICATION (Jest, Vitest, Playwright for E2E)
**Target Platform**: Web (browser-based), mobile-responsive (320px+ width), deployed on NEEDS CLARIFICATION (Vercel, self-hosted Node.js server)
**Project Type**: Web application (Next.js full-stack with collocated frontend and backend)
**Performance Goals**: Dashboard loads in <2s with 500 transactions, transaction entry <30s on mobile, edit/delete operations <1s, category creation <500ms
**Constraints**: Mobile-first responsive design (320px+ width), HTTPS required, secure session management, 30-minute inactivity timeout, 2-3 automatic API retries with exponential backoff
**Scale/Scope**: Multi-user with authentication, ~5 pages/routes (login, register, dashboard, transaction form, password recovery), 4 key entities (User, Transaction, Category, Monthly Summary), 24+ months historical data support

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

### I. Clean & Modular Code

✅ **Component Complexity**: Target ≤200 lines per component/function
- Auth pages, transaction forms, dashboard cards will be decomposed into small components
- Server actions isolated by domain (auth, transactions, categories)
- Utility functions for currency formatting, date handling, rounding extracted

✅ **Module Boundaries**: Clear separation of concerns
- `/app` - Next.js routes and pages
- `/src/components` - Reusable UI components
- `/src/server` - Server-side business logic (auth, DB queries, validations)
- `/src/lib` - Shared utilities (formatting, validation, constants)

✅ **Minimal Dependencies**: Explicit module interfaces
- Database queries abstracted into data access layer
- API retry logic centralized
- Chart components wrapped for reusability

### II. Next.js Best Practices

✅ **Server Components Default**: All pages are Server Components by default
- Dashboard data fetching uses async/await in Server Components
- Client Components used only for: forms (interactivity), charts (browser rendering), month picker (state)

✅ **Route Handlers & Server Actions**: RESTful API design
- Route handlers for authentication endpoints (`/api/auth/*`)
- Server Actions for mutations (create/update/delete transactions and categories)
- Proper error boundaries with `error.tsx`
- Loading states with `loading.tsx`

✅ **Next.js Image & Metadata**: Performance optimization
- Next.js Image component for any logos or icons
- Metadata API in layout.tsx and page.tsx for SEO

✅ **TypeScript Strict**: Type safety enforced
- All props, API responses, database schemas typed
- Zod or similar for runtime validation on Server Actions

### III. Security-First Development

✅ **Input Validation & Sanitization**: All user input validated
- Email validation (format, uniqueness check)
- Password strength enforcement (≥8 characters)
- Amount validation (positive numbers, banker's rounding)
- Description length limits (1-200 characters)
- Category name length limits (≤50 characters)

✅ **SQL Injection Prevention**: Parameterized queries or ORM
- Use PostgreSQL client with parameterized queries OR
- Use Prisma/Drizzle ORM for type-safe queries

✅ **Authentication & Session Security**: Secure session management
- Password hashing (bcrypt or argon2)
- Session tokens (JWT or secure cookies with httpOnly, secure, sameSite flags)
- 30-minute inactivity timeout enforced
- CSRF protection for state-changing operations

✅ **Secret Management**: Environment variables only
- DATABASE_URL, JWT_SECRET, EMAIL_API_KEY in `.env.local`
- Never exposed to client-side code
- Separate `.env.example` for documentation

✅ **XSS Prevention**: Data sanitization before rendering
- React automatic escaping (default)
- Validate/sanitize rich text if added later
- Content Security Policy headers

✅ **Data Access Control**: User-scoped queries
- All queries filtered by `user_id`
- Middleware enforces authentication on protected routes
- Database constraints prevent cross-user data access

**GATE STATUS**: ✅ PASS - All constitution principles can be met with planned architecture

---

## Post-Design Re-evaluation

*Conducted after Phase 0 (Research) and Phase 1 (Design) completion*

### I. Clean & Modular Code - ✅ CONFIRMED

**Research Decisions Support Modularity**:
- Drizzle ORM provides clear data access abstraction layer in `src/server/*/queries.ts`
- Server Actions naturally decompose by domain (auth, transactions, categories)
- Recharts components are self-contained and reusable
- NextAuth.js encapsulates authentication complexity

**Data Model Enforces Boundaries**:
- 4 entities (User, Transaction, Category, Monthly Summary) with clear relationships
- Database constraints enforce referential integrity
- Type safety from Drizzle ensures compile-time checking

**No Red Flags**: All planned components fit within 200-line limit based on data model and API specification.

### II. Next.js Best Practices - ✅ CONFIRMED

**Proper Server/Client Split Maintained**:
- Server Components: Dashboard summary, transaction lists, category dropdowns (data fetching)
- Client Components: Forms (interactivity), Charts (browser canvas), MonthPicker (state)
- Server Actions for all mutations (create/update/delete)
- Route Handlers only for auth flows (register, login, password reset)

**TypeScript Strictness Enforced**:
- Drizzle provides type inference from schema: `User`, `Transaction`, `Category` types
- Zod schemas for runtime validation in Server Actions
- All API contracts documented with TypeScript signatures

**Loading & Error States**:
- `loading.tsx` and `error.tsx` files specified in project structure
- API retry logic documented in contracts (2-3 attempts with exponential backoff)

### III. Security-First Development - ✅ CONFIRMED

**Input Validation - Comprehensive**:
- Zod schemas in `src/lib/validation.ts` for all user inputs
- Email format validation (RFC 5322)
- Password strength (≥8 characters)
- Amount validation (positive, banker's rounding for decimals)
- Description length limits (1-200 chars)
- Category name limits (≤50 chars)

**SQL Injection Prevention - Enforced**:
- Drizzle ORM uses parameterized queries by default
- No raw SQL strings in application code
- Type-safe query building

**Authentication & Session Security - Robust**:
- NextAuth.js v5 handles session management
- Bcrypt password hashing (10+ salt rounds)
- JWT tokens with httpOnly, secure, sameSite cookies
- 30-minute inactivity timeout configured
- CSRF protection built-in

**Data Access Control - Mandatory**:
- All queries in data-model.md include `WHERE user_id = $currentUserId` filter
- Example queries demonstrate user-scoped filtering
- Database indexes on `user_id` columns for performance
- Foreign key CASCADE on user deletion (data cleanup)

**Secret Management - Correct**:
- All secrets in `.env.local` (DATABASE_URL, JWT_SECRET, API keys)
- `.env.example` for documentation (no actual secrets)
- Environment variables never exposed to client bundle

**XSS Prevention - Default Safe**:
- React automatic escaping (default behavior)
- No `dangerouslySetInnerHTML` usage planned
- Content Security Policy headers recommended in quickstart

**Additional Security Measures**:
- UUID primary keys prevent enumeration attacks
- Timestamps (created_at, updated_at) for audit trails
- Database CHECK constraints on amount (> 0) and type (enum)
- RESTRICT delete on category → transactions (prevent orphaned data)

### Technology Choices Aligned with Constitution

| Technology | Constitution Principle | Alignment |
|-----------|------------------------|-----------|
| **Drizzle ORM** | Clean & Modular Code | ✅ Type-safe abstraction, minimal complexity |
| **NextAuth.js v5** | Security-First | ✅ Industry-standard auth, built-in CSRF protection |
| **Recharts** | Next.js Best Practices | ✅ React-first, declarative API, tree-shakeable |
| **Resend** | Clean & Modular Code | ✅ Simple API, single responsibility (email sending) |
| **Vitest** | Next.js Best Practices | ✅ Fast, ESM-native, Jest-compatible API |
| **Zod** | Security-First | ✅ Runtime validation, type inference |

### Complexity Assessment

**No Additional Complexity Introduced**:
- All technology choices simplify rather than complicate architecture
- No custom authentication (using NextAuth.js)
- No custom ORM (using Drizzle)
- No custom retry logic libraries (simple exponential backoff utility)

**Reduced Complexity Compared to Alternatives**:
- Drizzle is 60x smaller than Prisma (~7KB vs 500KB)
- NextAuth.js eliminates need for manual session management
- Resend simpler than SMTP configuration

**FINAL GATE STATUS**: ✅✅ PASS - Post-design evaluation confirms all constitution principles are satisfied. No violations or complexity justifications needed. Ready for Phase 2 (Task Generation).

## Project Structure

### Documentation (this feature)

```text
specs/[###-feature]/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)

```text
# Next.js App Router structure with collocated backend logic

app/
├── layout.tsx                    # Root layout with auth provider, fonts
├── page.tsx                      # Dashboard (main page, Server Component)
├── login/
│   └── page.tsx                  # Login page
├── register/
│   └── page.tsx                  # Registration page
├── forgot-password/
│   └── page.tsx                  # Password recovery request
├── reset-password/
│   └── page.tsx                  # Password reset form (with token)
├── transactions/
│   ├── new/
│   │   └── page.tsx              # New transaction form
│   └── [id]/
│       └── edit/
│           └── page.tsx          # Edit transaction form
├── api/
│   └── auth/
│       └── [...nextauth]/
│           └── route.ts          # Auth route handlers (if using NextAuth)
├── loading.tsx                   # Global loading state
├── error.tsx                     # Global error boundary
└── globals.css                   # TailwindCSS v4 imports

src/
├── components/                   # Reusable UI components
│   ├── auth/
│   │   ├── LoginForm.tsx         # Client Component (form interactivity)
│   │   ├── RegisterForm.tsx      # Client Component
│   │   └── PasswordResetForm.tsx # Client Component
│   ├── transactions/
│   │   ├── TransactionForm.tsx   # Client Component (form with validation)
│   │   ├── TransactionList.tsx   # Server Component (list rendering)
│   │   └── TransactionItem.tsx   # Server Component (row rendering)
│   ├── dashboard/
│   │   ├── SummaryCards.tsx      # Server Component (income/expense/balance)
│   │   ├── ExpenseChart.tsx      # Client Component (chart rendering)
│   │   ├── IncomeChart.tsx       # Client Component (chart rendering)
│   │   ├── MonthPicker.tsx       # Client Component (state management)
│   │   └── EmptyState.tsx        # Server Component (no data message)
│   └── ui/
│       ├── Button.tsx            # Reusable button
│       ├── Input.tsx             # Reusable input
│       ├── Select.tsx            # Reusable select/dropdown
│       ├── DatePicker.tsx        # Client Component (date input)
│       └── Toast.tsx             # Client Component (notifications)
├── server/                       # Server-side business logic
│   ├── auth/
│   │   ├── actions.ts            # Server Actions: login, register, logout, resetPassword
│   │   ├── session.ts            # Session management utilities
│   │   ├── password.ts           # Password hashing/validation
│   │   └── middleware.ts         # Auth middleware for protected routes
│   ├── transactions/
│   │   ├── actions.ts            # Server Actions: create, update, delete
│   │   ├── queries.ts            # Database queries for transactions
│   │   └── validation.ts         # Transaction validation logic
│   ├── categories/
│   │   ├── actions.ts            # Server Actions: create category
│   │   ├── queries.ts            # Database queries for categories
│   │   └── defaults.ts           # Default category seeding
│   ├── dashboard/
│   │   └── queries.ts            # Aggregation queries for dashboard
│   └── db/
│       ├── client.ts             # PostgreSQL client initialization
│       ├── schema.sql            # Database schema (users, transactions, categories)
│       └── migrations/           # Database migration files
├── lib/                          # Shared utilities
│   ├── currency.ts               # LKR formatting ("Rs. X,XXX.XX")
│   ├── date.ts                   # Date utilities
│   ├── rounding.ts               # Banker's rounding implementation
│   ├── validation.ts             # Zod schemas for validation
│   ├── api-retry.ts              # Exponential backoff retry logic
│   ├── constants.ts              # App constants (timeouts, limits)
│   └── types.ts                  # Shared TypeScript types
└── middleware.ts                 # Next.js middleware (auth checks, redirects)

public/
└── [static assets if needed]

.env.local                        # Environment variables (not committed)
.env.example                      # Example environment variables
next.config.ts                    # Next.js configuration
tsconfig.json                     # TypeScript configuration
tailwind.config.ts                # TailwindCSS v4 configuration (if needed)
postcss.config.mjs                # PostCSS configuration
package.json                      # Dependencies
```

**Structure Decision**: Next.js App Router full-stack architecture with collocated frontend and backend. All server-side logic (auth, database access, business logic) is placed in `src/server/` as requested, organized by domain. Client Components are marked explicitly and used only for interactivity (forms, charts, state). Server Components handle data fetching and rendering. This structure maintains clean separation while leveraging Next.js performance optimizations.

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

No violations detected. All architecture decisions align with constitution principles.
