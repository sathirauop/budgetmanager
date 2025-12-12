# Technical Research: Personal Financial Tracker MVP

**Date**: 2025-12-12
**Context**: Resolving technology choices for Next.js 16 App Router financial tracker with PostgreSQL

## Research Questions

This document resolves all "NEEDS CLARIFICATION" items from the Technical Context section of plan.md.

---

## 1. PostgreSQL Client Library

### Decision: **Drizzle ORM**

### Rationale

- **Type-safe SQL**: Drizzle provides full TypeScript type inference for queries, schema, and results
- **Lightweight**: Minimal runtime overhead compared to Prisma (~7KB vs 500KB+)
- **SQL-like API**: Drizzle queries resemble SQL, making it easy to optimize and understand
- **Migration support**: Built-in schema migration tools with `drizzle-kit`
- **Edge-compatible**: Works with serverless/edge environments (Vercel Edge Functions)
- **PostgreSQL-first**: Excellent PostgreSQL support with prepared statements preventing SQL injection
- **Performance**: Faster than Prisma for simple queries, no schema generation overhead

### Alternatives Considered

| Library | Pros | Cons | Why Rejected |
|---------|------|------|--------------|
| **Prisma** | Mature ecosystem, excellent DX, auto-generated types | Large bundle size (500KB+), slower migration generation, client-server architecture adds complexity | Bundle size concerns for MVP, overkill for simple CRUD operations |
| **node-postgres (pg)** | Minimal, direct SQL control, zero abstraction | No type safety, manual query building, SQL injection risk if misused, no migration tooling | Violates TypeScript strict requirement, higher maintenance burden |
| **Kysely** | Type-safe, lightweight, SQL-like | Smaller community, fewer integrations | Good alternative but Drizzle has better DX and documentation for Next.js |

### Implementation Notes

- Use `drizzle-orm` with `postgres` driver
- Define schema in TypeScript: `src/server/db/schema.ts`
- Use `drizzle-kit` for migrations
- Connection pooling with `pg.Pool` for production
- Prepared statements by default (SQL injection protection)

---

## 2. Charting Library

### Decision: **Recharts**

### Rationale

- **React-first**: Built specifically for React with declarative component API
- **Responsive by default**: Charts automatically adjust to container size (critical for mobile)
- **TypeScript support**: Full type definitions included
- **Tree-shakeable**: Only import components you use (reduces bundle size)
- **Customizable**: Easy to style with TailwindCSS classes or inline styles
- **Popular**: 25K+ GitHub stars, active maintenance, extensive documentation
- **Simple API**: Pie/Donut charts require minimal configuration
- **Accessibility**: Better screen reader support than canvas-based libraries

### Alternatives Considered

| Library | Pros | Cons | Why Rejected |
|---------|------|------|--------------|
| **Chart.js** | Very popular, feature-rich | Canvas-based (harder to customize with Tailwind), imperative API, not React-native | Doesn't fit Next.js/React declarative pattern |
| **Victory** | React-native, highly customizable | Larger bundle size, more complex API for simple charts | Unnecessary complexity for pie/donut charts |
| **D3.js** | Ultimate flexibility | Steep learning curve, requires manual DOM manipulation, large bundle, not React-friendly | Overkill for basic charts, violates React best practices |
| **Tremor** | Built for dashboards, includes UI components | Opinionated styling, less flexible | Too prescriptive, want custom minimal design |

### Implementation Notes

- Use `<PieChart>`, `<Pie>`, and `<Cell>` components for expense visualization
- Use `<PieChart>` with `innerRadius` for donut chart (income visualization)
- Wrap in Client Components: `ExpenseChart.tsx`, `IncomeChart.tsx`
- Custom colors matching minimal design aesthetic
- Responsive container: `<ResponsiveContainer width="100%" height={300}>`

---

## 3. Email Service

### Decision: **Resend**

### Rationale

- **Developer-first**: Modern API designed for developers, not marketers
- **Next.js integration**: Official Next.js integration examples, optimized for Server Actions
- **Generous free tier**: 3,000 emails/month free (sufficient for MVP)
- **React Email support**: Works perfectly with React Email for type-safe email templates
- **Simple API**: Single function call to send email, no complex configuration
- **Reliable delivery**: High deliverability rates, proper SPF/DKIM setup
- **Fast**: Low latency, suitable for transactional emails (password resets)
- **TypeScript-first**: Full type safety out of the box

### Alternatives Considered

| Service | Pros | Cons | Why Rejected |
|---------|------|------|--------------|
| **SendGrid** | Mature, feature-rich, 100 emails/day free | Complex API, marketing-focused UI, slower setup | API complexity overkill for simple password reset emails |
| **Nodemailer** | Self-hosted, full control, SMTP flexibility | Requires SMTP configuration, deliverability issues, no built-in templates | Manual SMTP setup violates "simple MVP" constraint |
| **AWS SES** | Cheap at scale, AWS ecosystem | Complex IAM setup, requires AWS account, cold start issues | AWS infrastructure complexity not justified for MVP |
| **Mailgun** | Good reputation, reliable | 100 emails/day free tier limit, API less intuitive | More expensive at scale, less modern DX than Resend |

### Implementation Notes

- Install: `npm install resend`
- API key in `.env.local`: `RESEND_API_KEY`
- Send password reset emails from `src/server/auth/actions.ts`
- Simple template with reset link: `https://app.com/reset-password?token=...`
- From address: `noreply@<your-domain>.com` (requires domain verification)
- Fallback for development: Console logging or Resend sandbox mode

---

## 4. Testing Framework

### Decision: **Vitest + Playwright**

### Rationale

**Vitest** (Unit/Integration):
- **Vite-native**: Instant HMR, fast test execution (<100ms cold start)
- **Jest-compatible API**: Easy migration knowledge from Jest
- **ESM-first**: Modern module system, works perfectly with Next.js 16
- **TypeScript out-of-the-box**: No extra configuration needed
- **Better DX**: Faster than Jest, built-in watch mode, better error messages
- **Next.js compatible**: Works well with Server Actions and Route Handlers

**Playwright** (E2E):
- **Multi-browser**: Chromium, Firefox, WebKit support
- **Mobile emulation**: Test responsive design (320px requirement)
- **Auto-wait**: Reduces flakiness, waits for elements intelligently
- **Network interception**: Test API retry logic and failure scenarios
- **Trace viewer**: Debug failures with DOM snapshots and network logs
- **CI-friendly**: Headless mode, parallel execution

### Alternatives Considered

| Framework | Pros | Cons | Why Rejected |
|-----------|------|------|--------------|
| **Jest** | Most popular, huge ecosystem | Slower than Vitest, ESM support issues, CommonJS legacy | Vitest is faster and more modern |
| **Cypress** | Great DX, time-travel debugging | Slower than Playwright, no WebKit, flaky in CI | Playwright is faster and more reliable |
| **Testing Library** | React-focused, good practices | Not a full framework (needs Jest/Vitest) | Used WITH Vitest, not instead of |

### Implementation Notes

- Install: `npm install -D vitest @testing-library/react @testing-library/jest-dom`
- Install: `npm install -D playwright @playwright/test`
- Test structure:
  - `__tests__/unit/` - Pure function tests (currency formatting, rounding)
  - `__tests__/integration/` - Server Actions, database queries (with test DB)
  - `__tests__/e2e/` - Playwright tests for critical flows (login, add transaction, view dashboard)
- `vitest.config.ts` for unit tests
- `playwright.config.ts` for E2E tests
- Optional for MVP: Focus on critical security paths (auth, data isolation)

---

## 5. Deployment Platform

### Decision: **Vercel (recommended)** with self-hosted option documented

### Rationale

**Vercel** (Primary):
- **Zero-config Next.js**: Optimized for Next.js 16 with automatic edge deployment
- **Free tier**: Generous limits for MVP (100GB bandwidth, unlimited requests)
- **Automatic HTTPS**: SSL certificates managed automatically
- **Preview deployments**: Every PR gets a preview URL for testing
- **PostgreSQL integration**: Easy connection to Vercel Postgres or external DB
- **Edge Functions**: Fast global performance for Server Actions
- **Environment variables**: Secure secret management in dashboard
- **Monitoring**: Built-in analytics and error tracking

**Self-Hosted (Documented Alternative)**:
- Node.js server with `next start`
- Reverse proxy (Nginx/Caddy) for HTTPS
- PM2 or systemd for process management
- Manual PostgreSQL setup

### Alternatives Considered

| Platform | Pros | Cons | Why Rejected |
|----------|------|------|--------------|
| **Netlify** | Good free tier, similar to Vercel | Less optimized for Next.js, edge functions limitations | Not as good for Next.js-specific features |
| **Railway** | Easy PostgreSQL, good DX | Smaller free tier, less mature | Vercel is more proven for Next.js |
| **AWS (ECS/Lambda)** | Scalable, flexible | Complex setup, higher cost, requires AWS expertise | Over-engineered for MVP |
| **DigitalOcean App Platform** | Simple, predictable pricing | Less Next.js-specific optimization | Generic platform, missing Next.js-specific benefits |

### Implementation Notes

- Connect GitHub repo to Vercel
- Set environment variables in Vercel dashboard:
  - `DATABASE_URL`
  - `JWT_SECRET`
  - `RESEND_API_KEY`
- Configure PostgreSQL (Vercel Postgres or external like Supabase, Neon, Railway)
- Set up custom domain (optional for MVP)
- Document self-hosted deployment in `quickstart.md` for users who want full control

---

## 6. Session Management Strategy

### Decision: **NextAuth.js v5 (Auth.js)** with JWT sessions

### Rationale

- **Next.js 16 native**: Full App Router support, works seamlessly with Server Actions
- **Email/password built-in**: Credentials provider included
- **JWT sessions**: Stateless, no database storage needed for sessions (simpler architecture)
- **Automatic expiry**: Built-in 30-minute idle timeout configuration
- **Middleware integration**: Easy protected route enforcement
- **TypeScript support**: Full type definitions
- **Security best practices**: CSRF protection, secure cookies (httpOnly, secure, sameSite)
- **Extensible**: Easy to add OAuth later if needed

### Alternatives Considered

| Approach | Pros | Cons | Why Rejected |
|----------|------|------|--------------|
| **Custom JWT** | Full control, minimal dependencies | Manual implementation of security features, easy to get wrong | Reinventing the wheel, security risk |
| **Iron Session** | Simple, encrypted cookies | Less feature-rich, no built-in auth flows | Still requires manual implementation of auth logic |
| **Clerk** | Fully managed, beautiful UI | Third-party dependency, costs at scale, vendor lock-in | Overkill for MVP, want full control |
| **Supabase Auth** | Free, includes user management | Requires Supabase ecosystem, less Next.js-specific | Adding external dependency not needed |

### Implementation Notes

- Install: `npm install next-auth@beta` (v5 for App Router)
- Configure in `app/api/auth/[...nextauth]/route.ts`
- JWT strategy with 30-minute max age
- Password hashing with bcrypt (built-in)
- Session in Server Components: `await auth()` from `next-auth`
- Session in Client Components: `useSession()` hook
- Middleware: `export { auth as middleware } from './auth'` in `middleware.ts`
- Protect routes by checking session in Server Components or middleware

---

## Summary of Decisions

| Question | Decision | Key Reason |
|----------|----------|------------|
| PostgreSQL Client | **Drizzle ORM** | Type-safe, lightweight, SQL-like API, edge-compatible |
| Charting Library | **Recharts** | React-first, responsive, simple API for pie/donut charts |
| Email Service | **Resend** | Developer-friendly, Next.js integration, generous free tier |
| Testing Framework | **Vitest + Playwright** | Fast (Vitest), reliable (Playwright), modern ESM support |
| Deployment Platform | **Vercel** (+ self-hosted docs) | Zero-config Next.js, free tier, automatic HTTPS |
| Session Management | **NextAuth.js v5** | Next.js 16 native, JWT sessions, built-in security |

---

## Next Steps

All technical unknowns are now resolved. Proceed to Phase 1:
1. Generate `data-model.md` with PostgreSQL schema
2. Generate API contracts in `contracts/`
3. Generate `quickstart.md` with setup instructions
4. Update agent context with technology decisions
