# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a budget management application built with Next.js 16 (App Router), React 19, TypeScript, and TailwindCSS v4.

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

## Key Files

- `next.config.ts`: Next.js configuration (currently minimal)
- `tsconfig.json`: TypeScript compiler options
- `eslint.config.mjs`: ESLint configuration using Next.js presets
- `postcss.config.mjs`: PostCSS configuration for TailwindCSS v4
