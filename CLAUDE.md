# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a personal financial tracking application (budget manager) built with Next.js 16 (App Router), React 19, TypeScript, and TailwindCSS v4. The application enables users to log transactions, manage categories, and visualize financial data through a dashboard.

**Current Feature**: `001-financial-tracker-mvp` - Multi-user financial tracker with authentication, transaction management, and dashboard visualizations in LKR currency.

## Development Commands

```bash
# Install dependencies
npm install

# Start development server (http://localhost:3000)
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run linter
npm run lint
```

## Architecture

### Next.js App Router Structure

This project uses the Next.js App Router (not Pages Router):
- `app/layout.tsx`: Root layout with Geist fonts (sans and mono) configured via `next/font`
- `app/page.tsx`: Homepage component
- `app/globals.css`: Global styles with TailwindCSS v4 imports and CSS variables

### TypeScript Configuration

- Path alias `@/*` maps to project root for imports
- Strict mode enabled
- Target: ES2017
- Module resolution: bundler

### Styling System

**TailwindCSS v4** (NOT v3):
- Uses `@import "tailwindcss"` in CSS (not traditional config file)
- Custom theme defined with `@theme inline` in `globals.css`
- CSS variables for theming: `--background`, `--foreground`, `--font-sans`, `--font-mono`
- Dark mode via `prefers-color-scheme` media query
- PostCSS plugin: `@tailwindcss/postcss`

### Font Setup

Uses Next.js font optimization with two Geist fonts:
- `Geist` (sans-serif): `--font-geist-sans`
- `Geist_Mono` (monospace): `--font-geist-mono`

Both are exposed as CSS variables in the body className and available in TailwindCSS theme.

## Project Constitution

**IMPORTANT**: All code MUST comply with the project constitution at `.specify/memory/constitution.md`.

Core principles:
1. **Clean & Modular Code**: Components/functions ≤200 lines, single responsibility, minimal dependencies
2. **Next.js Best Practices**: Server Components by default, proper loading/error states, strict TypeScript
3. **Security-First**: Validate all input, protect secrets, prevent XSS/SQL injection, secure auth flows

See constitution for complete rules and rationale.

## Speckit Integration

This project uses Speckit for specification and planning workflows located in `.specify/`:
- `.specify/templates/`: Specification, plan, and task templates
- `.specify/memory/`: Project constitution and design decisions
- `.specify/scripts/`: Automation scripts

Custom slash commands are available in `.claude/commands/` for Speckit workflows like `/speckit.specify`, `/speckit.plan`, `/speckit.tasks`, etc.

## Technology Stack (Feature 001)

**Backend**:
- **Database**: PostgreSQL with Drizzle ORM (type-safe, lightweight)
- **Authentication**: NextAuth.js v5 (App Router compatible, JWT sessions, 30-min timeout)
- **Email**: Resend (password recovery, transactional emails)
- **Server Logic**: Server Actions + Route Handlers in `src/server/`

**Frontend**:
- **Charts**: Recharts (React-native, responsive pie/donut charts)
- **Forms**: Client Components with Zod validation
- **Styling**: TailwindCSS v4 with minimal, mobile-first design

**Testing** (Optional):
- **Unit**: Vitest + Testing Library
- **E2E**: Playwright

**Deployment**: Vercel (recommended) or self-hosted Node.js

**Key Architectural Decisions** (from `specs/001-financial-tracker-mvp/research.md`):
- Server Components by default, Client Components only for interactivity (forms, charts, state)
- All server-side logic in `src/server/` organized by domain (auth, transactions, categories, dashboard)
- User-scoped data isolation: All queries filtered by `user_id` from session
- API retry logic: 2-3 attempts with exponential backoff
- Security: bcrypt password hashing, SQL injection prevention via Drizzle, XSS prevention via React escaping

## Key Files

- `next.config.ts`: Next.js configuration (currently minimal)
- `tsconfig.json`: TypeScript compiler options
- `eslint.config.mjs`: ESLint configuration using Next.js presets
- `postcss.config.mjs`: PostCSS configuration for TailwindCSS v4
- `src/server/db/schema.ts`: Drizzle ORM database schema (users, transactions, categories)
- `specs/001-financial-tracker-mvp/`: Complete feature specification, data model, API contracts, and quickstart guide
