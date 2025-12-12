# Tasks: Personal Financial Tracker MVP

**Input**: Design documents from `/specs/001-financial-tracker-mvp/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/api-specification.md

**Tests**: Tests are OPTIONAL for this MVP - not included per project constitution.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

This project uses Next.js App Router structure with server-side logic in `src/server/`:
- `app/` - Next.js routes and pages
- `src/components/` - React components
- `src/server/` - Server Actions, queries, authentication
- `src/lib/` - Utilities and validation

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and basic structure

- [ ] T001 Install core dependencies: drizzle-orm, postgres, next-auth@beta, recharts, resend, zod, bcryptjs per research.md
- [ ] T002 [P] Create environment variable template .env.example with DATABASE_URL, NEXTAUTH_URL, NEXTAUTH_SECRET, RESEND_API_KEY, FROM_EMAIL
- [ ] T003 [P] Configure TypeScript strict mode in tsconfig.json with path alias @/* → project root
- [ ] T004 [P] Setup TailwindCSS v4 configuration in app/globals.css with @theme inline directive
- [ ] T005 Create base project structure: src/components/, src/server/, src/lib/ directories
- [ ] T006 [P] Create .env.local with placeholder values (user must fill actual secrets)
- [ ] T007 [P] Add database management scripts to package.json: db:generate, db:migrate, db:push, db:studio, db:seed

**Checkpoint**: Project structure ready, dependencies installed

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

### Database Foundation

- [ ] T008 Define Drizzle ORM schema in src/server/db/schema.ts with users, categories, transactions tables per data-model.md
- [ ] T009 Create database client configuration in src/server/db/client.ts with PostgreSQL connection pooling
- [ ] T010 Create database seed script in src/server/db/seed.ts to insert default categories (Food, Transport, Rent, etc.)
- [ ] T011 Generate initial Drizzle migration files with drizzle-kit generate
- [ ] T012 Document database setup instructions (local PostgreSQL and cloud options) - reference quickstart.md

### Authentication Foundation

- [ ] T013 Configure NextAuth.js v5 in src/auth.ts with Credentials provider for email/password authentication
- [ ] T014 [P] Create password utilities in src/server/auth/password.ts with bcrypt hashing (10+ salt rounds)
- [ ] T015 [P] Create session utilities in src/server/auth/session.ts with JWT token generation and 30-minute timeout
- [ ] T016 Create authentication middleware in src/middleware.ts to protect routes requiring authentication
- [ ] T017 Create authentication route handler in app/api/auth/[...nextauth]/route.ts for NextAuth.js callbacks

### Shared Utilities

- [ ] T018 [P] Create currency formatting utility in src/lib/currency.ts for LKR format (Rs. X,XXX.XX)
- [ ] T019 [P] Create date utilities in src/lib/date.ts for date manipulation and validation (2000-2099 range)
- [ ] T020 [P] Create banker's rounding utility in src/lib/rounding.ts for amounts with 3+ decimals
- [ ] T021 [P] Create Zod validation schemas in src/lib/validation.ts for email, password, amount, description, category
- [ ] T022 [P] Create API retry utility in src/lib/api-retry.ts with exponential backoff (2-3 retries)
- [ ] T023 [P] Create shared TypeScript types in src/lib/types.ts for User, Transaction, Category, Session
- [ ] T024 [P] Create constants file in src/lib/constants.ts with timeouts, limits, default categories

### UI Foundation

- [ ] T025 [P] Create base Button component in src/components/ui/Button.tsx with TailwindCSS styling
- [ ] T026 [P] Create base Input component in src/components/ui/Input.tsx with validation error display
- [ ] T027 [P] Create base Select component in src/components/ui/Select.tsx for dropdown menus
- [ ] T028 [P] Create DatePicker component (Client) in src/components/ui/DatePicker.tsx for transaction dates
- [ ] T029 [P] Create Toast notification component (Client) in src/components/ui/Toast.tsx for error/success messages
- [ ] T030 Create root layout in app/layout.tsx with Geist fonts, auth provider (NextAuth SessionProvider), and global styles
- [ ] T031 Create global loading state in app/loading.tsx with spinner or skeleton
- [ ] T032 Create global error boundary in app/error.tsx with user-friendly error message and retry button

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - Record Daily Transactions (Priority: P1) 🎯 MVP

**Goal**: Enable users to create, edit, and delete transactions (income/expense) with amount, date, description, and category

**Independent Test**: Register account → Login → Add expense (Rs. 500, Food, today) → Verify saved → Edit amount to Rs. 600 → Verify updated → Delete transaction → Verify removed

### Authentication (Prerequisite for US1)

- [ ] T033 [P] [US1] Create registration route handler in app/api/auth/register/route.ts with email uniqueness check and password hashing
- [ ] T034 [P] [US1] Create password reset request handler in app/api/auth/forgot-password/route.ts with Resend email integration
- [ ] T035 [P] [US1] Create password reset handler in app/api/auth/reset-password/route.ts with token validation
- [ ] T036 [P] [US1] Create RegisterForm component (Client) in src/components/auth/RegisterForm.tsx with Zod validation
- [ ] T037 [P] [US1] Create LoginForm component (Client) in src/components/auth/LoginForm.tsx with error handling
- [ ] T038 [P] [US1] Create PasswordResetForm component (Client) in src/components/auth/PasswordResetForm.tsx
- [ ] T039 [US1] Create registration page in app/register/page.tsx with RegisterForm
- [ ] T040 [US1] Create login page in app/login/page.tsx with LoginForm
- [ ] T041 [US1] Create forgot password page in app/forgot-password/page.tsx
- [ ] T042 [US1] Create reset password page in app/reset-password/page.tsx with token from URL params

### Transaction Management (Core US1 Functionality)

- [ ] T043 [P] [US1] Create transaction Server Actions in src/server/transactions/actions.ts: createTransaction, updateTransaction, deleteTransaction
- [ ] T044 [P] [US1] Create transaction database queries in src/server/transactions/queries.ts: getUserTransactions, getTransactionById with user_id filtering
- [ ] T045 [P] [US1] Create transaction validation logic in src/server/transactions/validation.ts with Zod schema for amount, description, date, category
- [ ] T046 [P] [US1] Create category Server Actions in src/server/categories/actions.ts: createCategory (for inline category creation)
- [ ] T047 [P] [US1] Create category database queries in src/server/categories/queries.ts: getCategoriesForUser (system defaults + user's own)
- [ ] T048 [US1] Create TransactionForm component (Client) in src/components/transactions/TransactionForm.tsx with amount, date, description, category, type toggle
- [ ] T049 [US1] Create TransactionList component (Server) in src/components/transactions/TransactionList.tsx to display transactions
- [ ] T050 [US1] Create TransactionItem component (Server) in src/components/transactions/TransactionItem.tsx with Edit/Delete buttons
- [ ] T051 [US1] Create new transaction page in app/transactions/new/page.tsx with TransactionForm
- [ ] T052 [US1] Create edit transaction page in app/transactions/[id]/edit/page.tsx with pre-filled TransactionForm
- [ ] T053 [US1] Implement banker's rounding in TransactionForm for amounts with 3+ decimals (call src/lib/rounding.ts)
- [ ] T054 [US1] Implement API retry logic in TransactionForm for create/update/delete operations
- [ ] T055 [US1] Add loading indicators to TransactionForm during API operations
- [ ] T056 [US1] Add error notifications with Toast component when transaction operations fail

**Checkpoint**: At this point, User Story 1 should be fully functional - users can register, login, and manage transactions independently

---

## Phase 4: User Story 2 - Manage Custom Categories (Priority: P2)

**Goal**: Enable users to create custom categories on-the-fly during transaction entry, with categories filtered by type (income/expense)

**Independent Test**: Login → Add transaction → Type new category name "Pet Supplies" in category field → Select Expense type → Save → Add another expense → Verify "Pet Supplies" appears in dropdown

### Category Management (Extends US1)

- [ ] T057 [US2] Enhance TransactionForm to support inline category creation: detect new category name, call createCategory action, add to dropdown
- [ ] T058 [US2] Implement category type filtering in TransactionForm: show expense categories when type=expense, income categories when type=income
- [ ] T059 [US2] Add category name validation: 1-50 characters, trim whitespace, check uniqueness per user per type
- [ ] T060 [US2] Add category name length limit enforcement in category creation (≤50 characters with truncation warning)
- [ ] T061 [US2] Add visual indicator in category dropdown to distinguish system defaults from user-created categories
- [ ] T062 [US2] Ensure category dropdown in TransactionForm sorts: system defaults first, then user categories alphabetically

**Checkpoint**: At this point, User Stories 1 AND 2 should both work independently - users can create custom categories seamlessly

---

## Phase 5: User Story 3 - View Monthly Financial Dashboard (Priority: P3)

**Goal**: Display monthly financial summary (income, expenses, balance) with pie/donut charts for category breakdown and scrollable transaction history

**Independent Test**: Login → Add 5 transactions (3 expenses, 2 income) → Open dashboard → Verify summary cards show correct totals → Verify expense pie chart → Verify income donut chart → Change month picker → Verify data updates

### Dashboard Queries

- [ ] T063 [P] [US3] Create dashboard database queries in src/server/dashboard/queries.ts: getMonthlySummary, getTransactionsForMonth, getCategoryBreakdown
- [ ] T064 [US3] Implement getMonthlySummary to calculate total income, total expenses, balance for selected month with user_id filtering
- [ ] T065 [US3] Implement getTransactionsForMonth to fetch all transactions for selected month sorted by date descending
- [ ] T066 [US3] Implement getCategoryBreakdown to aggregate totals by category for pie/donut charts with percentage calculations

### Dashboard Components

- [ ] T067 [P] [US3] Create SummaryCards component (Server) in src/components/dashboard/SummaryCards.tsx with income, expenses, balance cards
- [ ] T068 [P] [US3] Create ExpenseChart component (Client) in src/components/dashboard/ExpenseChart.tsx using Recharts PieChart
- [ ] T069 [P] [US3] Create IncomeChart component (Client) in src/components/dashboard/IncomeChart.tsx using Recharts PieChart with innerRadius for donut
- [ ] T070 [P] [US3] Create MonthPicker component (Client) in src/components/dashboard/MonthPicker.tsx with dropdown for month/year selection
- [ ] T071 [P] [US3] Create EmptyState component (Server) in src/components/dashboard/EmptyState.tsx for "No data" message when no transactions
- [ ] T072 [US3] Implement balance color logic in SummaryCards: green for positive, red for negative
- [ ] T073 [US3] Implement responsive design for charts: stack vertically on mobile (320px width), side-by-side on desktop
- [ ] T074 [US3] Implement currency formatting in all dashboard components using src/lib/currency.ts for Rs. X,XXX.XX format

### Dashboard Page

- [ ] T075 [US3] Create main dashboard page in app/page.tsx (Server Component) fetching current month data
- [ ] T076 [US3] Integrate MonthPicker state management to refetch dashboard data when month changes
- [ ] T077 [US3] Add loading states for dashboard data fetching (use Suspense with loading.tsx)
- [ ] T078 [US3] Handle empty state when no transactions for selected month (display EmptyState component)
- [ ] T079 [US3] Ensure dashboard redirects to /login if user not authenticated (check session in page.tsx)
- [ ] T080 [US3] Integrate TransactionList component into dashboard to show transaction history below charts

**Checkpoint**: All user stories should now be independently functional - complete MVP with authentication, transactions, categories, and dashboard

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple user stories and final production readiness

### Security Hardening

- [ ] T081 [P] Add Content Security Policy headers in next.config.ts to prevent XSS attacks
- [ ] T082 [P] Verify all database queries include user_id filtering (audit src/server/*/queries.ts)
- [ ] T083 [P] Verify password hash never exposed in API responses (audit auth actions)
- [ ] T084 [P] Verify environment variables never exposed to client bundle (check all Server Actions)
- [ ] T085 Add session expiry redirect: detect expired session in middleware and redirect to /login with preserved form data

### Performance Optimization

- [ ] T086 [P] Add database indexes verification: user_id, date, user_id+date composite (check migration files)
- [ ] T087 [P] Optimize dashboard queries to use single aggregation query instead of multiple queries
- [ ] T088 Add React.lazy for charts to reduce initial bundle size (dynamic import ExpenseChart and IncomeChart)
- [ ] T089 Add pagination or virtualization to TransactionList if transaction count exceeds 100 per month

### UX Enhancements

- [ ] T090 [P] Add form validation feedback: show specific error messages for each field (email format, password length, amount positive)
- [ ] T091 [P] Add success notifications with Toast after transaction create/update/delete operations
- [ ] T092 Add keyboard shortcuts: Ctrl+N for new transaction, Escape to close forms
- [ ] T093 Add confirmation dialog for transaction deletion to prevent accidental deletes
- [ ] T094 Implement form data preservation on session expiry: save unsaved form data to localStorage, restore on re-login

### Mobile Optimization

- [ ] T095 [P] Test responsive design on 320px width (smallest mobile): verify all elements stack correctly
- [ ] T096 [P] Verify touch targets are at least 44x44px for mobile (buttons, form inputs, chart legend)
- [ ] T097 Add PWA manifest for "Add to Home Screen" capability on mobile browsers
- [ ] T098 Optimize font loading for mobile: preload Geist fonts to prevent layout shift

### Documentation & Deployment Prep

- [ ] T099 [P] Update README.md with project description, setup instructions, and technology stack
- [ ] T100 [P] Create deployment guide for Vercel in docs/deployment-vercel.md
- [ ] T101 [P] Create self-hosted deployment guide in docs/deployment-self-hosted.md
- [ ] T102 Verify all quickstart.md instructions are accurate: test database setup, environment variables, development server
- [ ] T103 Run full manual testing checklist from quickstart.md: all authentication flows, transaction CRUD, dashboard views
- [ ] T104 Run Lighthouse audit: ensure performance ≥90, accessibility ≥90, SEO ≥90
- [ ] T105 Generate production build with npm run build and verify no errors or warnings

**Checkpoint**: Production-ready MVP with all security, performance, and UX polish complete

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Story 1 (Phase 3)**: Depends on Foundational completion - Can start independently
- **User Story 2 (Phase 4)**: Depends on Foundational completion - Can start after or in parallel with US1 (enhances TransactionForm from US1)
- **User Story 3 (Phase 5)**: Depends on Foundational completion - Can start after US1 (needs transaction data to visualize)
- **Polish (Phase 6)**: Depends on all desired user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: ✅ No dependencies on other stories - Can start after Foundational
- **User Story 2 (P2)**: ⚠️ Soft dependency on US1 - Enhances TransactionForm created in US1, but could be built standalone
- **User Story 3 (P3)**: ⚠️ Soft dependency on US1 - Requires transaction data to visualize, but dashboard can show empty state

**Recommended Order**: Complete in priority order (US1 → US2 → US3) for incremental value delivery

### Within Each User Story

**User Story 1 (Authentication + Transactions)**:
1. Authentication components can be built in parallel (T033-T042)
2. Transaction Server Actions/queries can be built in parallel (T043-T047)
3. Transaction UI components built after Server Actions ready (T048-T050)
4. Pages built after components ready (T051-T052)
5. Polish features added last (T053-T056)

**User Story 2 (Custom Categories)**:
1. All tasks enhance existing TransactionForm
2. Tasks can be done sequentially as they build on each other
3. Category validation (T059-T060) before UI enhancements (T061-T062)

**User Story 3 (Dashboard)**:
1. Dashboard queries can be built in parallel (T063-T066)
2. Dashboard components can be built in parallel after queries (T067-T071)
3. Component polish can be parallel (T072-T074)
4. Page integration sequential (T075-T080)

### Parallel Opportunities

**Phase 1 (Setup)**: All tasks marked [P] can run in parallel - T002, T003, T004, T006, T007

**Phase 2 (Foundational)**:
- Database tasks sequential (T008 → T009 → T010 → T011)
- Auth tasks: T014, T015 in parallel, then T013 → T016 → T017
- Utilities: All T018-T024 in parallel
- UI components: All T025-T029 in parallel, then T030-T032

**Phase 3 (User Story 1)**:
- Auth components: T033-T038 all in parallel
- Auth pages: T039-T042 after components
- Transaction actions/queries: T043-T047 all in parallel
- Transaction UI: T048-T050 after actions
- Transaction pages: T051-T052 after UI

**Phase 4 (User Story 2)**:
- All tasks sequential (build on TransactionForm)

**Phase 5 (User Story 3)**:
- Queries: T063 separately, then T064-T066 after T063
- Components: T067-T071 all in parallel after queries
- Polish: T072-T074 in parallel
- Page: T075-T080 sequential

**Phase 6 (Polish)**:
- Security: T081-T085 all in parallel
- Performance: T086-T089 all in parallel
- UX: T090-T094 all in parallel
- Mobile: T095-T098 all in parallel
- Docs: T099-T102 in parallel, then T103-T105 sequential

---

## Parallel Example: User Story 1 (Authentication)

```bash
# Launch all authentication route handlers in parallel:
Task: "Create registration route handler in app/api/auth/register/route.ts" (T033)
Task: "Create password reset request handler in app/api/auth/forgot-password/route.ts" (T034)
Task: "Create password reset handler in app/api/auth/reset-password/route.ts" (T035)
Task: "Create RegisterForm component in src/components/auth/RegisterForm.tsx" (T036)
Task: "Create LoginForm component in src/components/auth/LoginForm.tsx" (T037)
Task: "Create PasswordResetForm component in src/components/auth/PasswordResetForm.tsx" (T038)
```

---

## Parallel Example: User Story 3 (Dashboard Components)

```bash
# Launch all dashboard components in parallel after queries complete:
Task: "Create SummaryCards component in src/components/dashboard/SummaryCards.tsx" (T067)
Task: "Create ExpenseChart component in src/components/dashboard/ExpenseChart.tsx" (T068)
Task: "Create IncomeChart component in src/components/dashboard/IncomeChart.tsx" (T069)
Task: "Create MonthPicker component in src/components/dashboard/MonthPicker.tsx" (T070)
Task: "Create EmptyState component in src/components/dashboard/EmptyState.tsx" (T071)
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

**Recommended for fastest time-to-value**:

1. Complete Phase 1: Setup (T001-T007)
2. Complete Phase 2: Foundational (T008-T032) - CRITICAL - blocks all stories
3. Complete Phase 3: User Story 1 (T033-T056) - Authentication + Transaction CRUD
4. **STOP and VALIDATE**: Test User Story 1 independently per Independent Test criteria
5. Deploy MVP to Vercel or staging environment
6. Gather user feedback on transaction logging experience

**Deliverable**: Functional transaction logger with authentication (~56 tasks)

### Incremental Delivery (All User Stories)

**Recommended for staged rollout**:

1. Complete Setup + Foundational (T001-T032) → Foundation ready
2. Add User Story 1 (T033-T056) → Test independently → Deploy/Demo (MVP!)
3. Add User Story 2 (T057-T062) → Test independently → Deploy/Demo (Enhanced categories)
4. Add User Story 3 (T063-T080) → Test independently → Deploy/Demo (Full dashboard)
5. Polish & optimize (T081-T105) → Production-ready

**Deliverable**: Fully-featured financial tracker with visualizations (~105 tasks)

### Parallel Team Strategy

**With multiple developers**:

1. **Together**: Complete Setup + Foundational (T001-T032)
2. **Once Foundational done**:
   - Developer A: User Story 1 - Authentication (T033-T042)
   - Developer B: User Story 1 - Transaction Management (T043-T056)
   - Developer C: Shared utilities and UI components (T018-T032 if not done)
3. **After US1 complete**:
   - Developer A: User Story 2 - Categories (T057-T062)
   - Developer B: User Story 3 - Dashboard Queries (T063-T066)
   - Developer C: User Story 3 - Dashboard Components (T067-T080)
4. **Final**:
   - All developers: Polish tasks in parallel (T081-T105)

**Deliverable**: Parallel development reduces total time by ~40%

---

## Notes

- **[P] tasks**: Different files, no dependencies - can run in parallel
- **[Story] label**: Maps task to specific user story for traceability (US1, US2, US3)
- **No tests included**: Per project constitution, tests are optional for MVP - focus on clean implementation
- **Independent user stories**: Each story (US1, US2, US3) can be tested and deployed independently
- **Security-first**: All tasks follow constitution security principles (input validation, user-scoped queries, password hashing)
- **Mobile-first**: All UI tasks prioritize mobile responsiveness (320px+ width)
- **Commit strategy**: Commit after each task or logical group (e.g., all auth components)
- **Stop at checkpoints**: Each checkpoint allows validation without breaking prior functionality
- **Avoid**: Vague tasks, same-file conflicts, cross-story dependencies that break independence

---

## Task Count Summary

- **Phase 1 (Setup)**: 7 tasks
- **Phase 2 (Foundational)**: 25 tasks (8 database + 5 auth + 7 utilities + 5 UI)
- **Phase 3 (User Story 1)**: 24 tasks (10 auth + 14 transactions)
- **Phase 4 (User Story 2)**: 6 tasks (category enhancements)
- **Phase 5 (User Story 3)**: 18 tasks (4 queries + 8 components + 6 page)
- **Phase 6 (Polish)**: 25 tasks (5 security + 4 performance + 5 UX + 4 mobile + 7 docs)

**Total**: 105 tasks

**MVP Scope (Recommended)**: Phases 1 + 2 + 3 = 56 tasks
**Full Feature**: All 105 tasks
