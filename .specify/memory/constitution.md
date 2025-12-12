<!--
Sync Impact Report
===================
Version: 0.0.0 → 1.0.0 (MINOR bump: Initial constitution ratification)

Modified Principles:
- N/A (initial version)

Added Sections:
- Core Principles (Clean & Modular Code, Next.js Best Practices, Security-First Development)
- Technology Standards
- Development Workflow
- Governance

Removed Sections:
- N/A (initial version)

Templates Status:
- ✅ .specify/templates/plan-template.md (verified - Constitution Check section present)
- ✅ .specify/templates/spec-template.md (verified - requirements alignment compatible)
- ✅ .specify/templates/tasks-template.md (verified - task categorization compatible)
- ⚠ .claude/commands/*.md (pending review for principle references)

Follow-up TODOs:
- None
-->

# Budget Manager Constitution

## Core Principles

### I. Clean & Modular Code

Code MUST be organized into discrete, reusable modules with clear responsibilities. Each component, service, or utility MUST have a single, well-defined purpose. Function and component complexity MUST be minimized through decomposition - complex logic should be broken into smaller, testable units.

**Rationale**: Modular code reduces cognitive load, improves testability, and enables team members to work independently on different features. Clear separation of concerns prevents cascading changes and makes the codebase maintainable as it scales.

**Non-negotiable rules**:
- No component or function exceeds 200 lines
- Each module exports a clear, documented interface
- Dependencies between modules are explicit and minimal
- Shared logic is extracted into utilities, not duplicated

### II. Next.js Best Practices

All code MUST follow Next.js 16+ App Router conventions and leverage framework capabilities optimally. Server Components are the default; Client Components MUST be explicitly marked with 'use client' directive only when necessary (interactivity, browser APIs, React hooks). Data fetching MUST use async/await in Server Components. Route handlers MUST follow RESTful conventions.

**Rationale**: Next.js provides performance optimizations and developer experience improvements that only work when used correctly. Misuse of Client vs Server Components degrades performance and increases bundle size. Following conventions ensures the application scales efficiently.

**Non-negotiable rules**:
- Use Server Components by default, Client Components only when required
- Implement proper loading and error states with loading.tsx and error.tsx
- Use Next.js Image component for all images
- Implement metadata API for SEO in layout.tsx and page.tsx files
- Use TypeScript strictly with proper typing for props, API responses, and server actions

### III. Security-First Development

Security MUST be considered at every stage of development. All user input MUST be validated and sanitized. Authentication and authorization checks MUST be implemented for protected resources. Sensitive data MUST NOT be exposed to the client. Environment variables MUST be used for secrets and configuration, never hardcoded.

**Rationale**: Security vulnerabilities can compromise user data, system integrity, and organizational reputation. Proactive security practices prevent costly breaches and build user trust. Next.js server/client boundary makes it critical to understand what code runs where.

**Non-negotiable rules**:
- Validate and sanitize all user input (forms, URL parameters, API requests)
- Use parameterized queries or ORMs to prevent SQL injection
- Implement CSRF protection for state-changing operations
- Never expose API keys, tokens, or secrets in client-side code
- Use HTTPS in production, enforce secure headers
- Implement proper session management and authentication flows
- Sanitize data before rendering to prevent XSS attacks
- Apply principle of least privilege for data access

## Technology Standards

**Framework**: Next.js 16+ with App Router (NOT Pages Router)

**Language**: TypeScript 5+ with strict mode enabled

**Styling**: TailwindCSS v4 with inline theme configuration via `@theme` directive

**Code Quality**:
- ESLint MUST pass with no errors before merging
- TypeScript MUST compile with no errors
- Follow established naming conventions: camelCase for variables/functions, PascalCase for components/types

**Performance**:
- Lighthouse score MUST be ≥90 for performance, accessibility, and SEO
- Core Web Vitals MUST meet "Good" thresholds
- Images MUST be optimized using next/image
- Bundle size increases require justification

## Development Workflow

**Version Control**:
- Feature branches MUST follow naming pattern: `###-feature-name`
- Commits MUST be atomic and have clear, descriptive messages
- Pull requests MUST reference related issue numbers

**Code Review Requirements**:
- All code MUST be reviewed before merging
- Reviewers MUST verify constitution compliance
- Security-sensitive changes require additional security review

**Testing Expectations**:
- Critical user paths SHOULD have integration tests
- Security-sensitive logic (auth, data access) SHOULD have unit tests
- Tests are OPTIONAL unless explicitly required in feature specification

**Documentation**:
- Complex business logic MUST include explanatory comments
- API endpoints MUST document request/response schemas
- Reusable utilities MUST include JSDoc comments

## Governance

This constitution supersedes all other development practices and conventions. All code changes, architectural decisions, and technical implementations MUST comply with the principles defined herein.

**Amendment Process**:
1. Proposed changes MUST be documented with rationale
2. Amendments require approval from project lead
3. Version number MUST be incremented following semantic versioning
4. Template files MUST be updated to reflect principle changes

**Compliance Verification**:
- All pull requests MUST be reviewed for constitution compliance
- Violations MUST be justified in writing before approval
- Repeated violations indicate a need for constitution amendment or additional training

**Complexity Justification**:
- Deviations from principles MUST be documented in plan.md Complexity Tracking section
- Justifications MUST explain why simpler alternatives are insufficient
- Complex solutions require explicit approval before implementation

**Version**: 1.0.0 | **Ratified**: 2025-12-12 | **Last Amended**: 2025-12-12
