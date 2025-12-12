# Quick Start Guide: Personal Financial Tracker MVP

**Last Updated**: 2025-12-12
**Branch**: `001-financial-tracker-mvp`

This guide covers environment setup, database initialization, development workflow, and deployment.

---

## Prerequisites

- **Node.js**: 20.x or higher ([Download](https://nodejs.org/))
- **PostgreSQL**: 15+ installed locally OR access to cloud database (Vercel Postgres, Neon, Supabase, Railway)
- **npm**: 10.x+ (comes with Node.js)
- **Git**: For version control

---

## 1. Clone & Install Dependencies

```bash
# Clone the repository
git clone <repository-url>
cd budgetmanager

# Checkout feature branch
git checkout 001-financial-tracker-mvp

# Install dependencies
npm install
```

**Key Dependencies** (installed automatically):
- `next@16` - Next.js framework
- `react@19` - React library
- `drizzle-orm` - Type-safe ORM for PostgreSQL
- `postgres` - PostgreSQL client
- `next-auth@beta` - Authentication (v5 for App Router)
- `recharts` - Charting library for dashboard
- `resend` - Email service for password recovery
- `zod` - Schema validation
- `bcryptjs` - Password hashing
- `@tailwindcss/postcss` - TailwindCSS v4

---

## 2. Database Setup

### Option A: Local PostgreSQL

1. **Install PostgreSQL** (if not already installed):
   - macOS: `brew install postgresql@15`
   - Windows: Download from [postgresql.org](https://www.postgresql.org/download/)
   - Linux: `sudo apt install postgresql postgresql-contrib`

2. **Create Database**:
   ```bash
   # Start PostgreSQL service
   # macOS/Linux
   brew services start postgresql@15  # or systemctl start postgresql

   # Create database
   createdb budgetmanager

   # Or via psql
   psql postgres
   CREATE DATABASE budgetmanager;
   \q
   ```

3. **Set Database URL**:
   ```bash
   # Local connection string
   DATABASE_URL="postgresql://username:password@localhost:5432/budgetmanager"
   ```

### Option B: Cloud Database (Recommended for MVP)

**Vercel Postgres** (easiest with Vercel deployment):
```bash
# Install Vercel CLI
npm i -g vercel

# Link project
vercel link

# Create PostgreSQL database
vercel postgres create

# Get connection string
vercel env pull .env.local
```

**Neon** ([neon.tech](https://neon.tech)):
- Sign up for free tier
- Create new project
- Copy connection string from dashboard

**Supabase** ([supabase.com](https://supabase.com)):
- Create project
- Go to Settings → Database → Connection String
- Use "Connection pooling" mode for serverless

**Railway** ([railway.app](https://railway.app)):
- Create PostgreSQL service
- Copy `DATABASE_URL` from Variables tab

---

## 3. Environment Variables

Create `.env.local` file in project root:

```bash
# Database
DATABASE_URL="postgresql://username:password@host:5432/budgetmanager"

# NextAuth.js (Authentication)
NEXTAUTH_URL="http://localhost:3000"
NEXTAUTH_SECRET="your-secret-key-here"  # Generate with: openssl rand -base64 32

# Resend (Email Service)
RESEND_API_KEY="re_xxxxxxxxxxxx"  # Get from https://resend.com
FROM_EMAIL="noreply@yourdomain.com"  # Must be verified domain in Resend

# Optional: Disable telemetry
NEXT_TELEMETRY_DISABLED=1
```

**Generate Secrets**:
```bash
# Generate NEXTAUTH_SECRET
openssl rand -base64 32

# Or use Node.js
node -e "console.log(require('crypto').randomBytes(32).toString('base64'))"
```

**Get Resend API Key**:
1. Sign up at [resend.com](https://resend.com) (free tier: 3,000 emails/month)
2. Add and verify your domain (or use testing domain in development)
3. Create API key from dashboard

---

## 4. Database Schema Setup

### Initialize Database

Run the database schema setup script:

```bash
# Generate Drizzle migrations
npm run db:generate

# Apply migrations to database
npm run db:migrate

# Seed default categories (run once)
npm run db:seed
```

**Add npm scripts** to `package.json`:
```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "db:generate": "drizzle-kit generate",
    "db:migrate": "drizzle-kit migrate",
    "db:push": "drizzle-kit push",
    "db:studio": "drizzle-kit studio",
    "db:seed": "tsx src/server/db/seed.ts"
  }
}
```

### Manual Setup (Alternative)

If you prefer to run SQL directly:

```bash
# Connect to database
psql $DATABASE_URL

# Run schema from data-model.md
# Copy the SQL from data-model.md and paste into psql
```

---

## 5. Development Server

Start the development server:

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) in your browser.

**Expected Behavior**:
- First visit redirects to `/login` (no authenticated session)
- Register new account at `/register`
- After login, see dashboard at `/` (initially empty)

---

## 6. Project Structure Overview

```
budgetmanager/
├── app/                        # Next.js App Router pages
│   ├── page.tsx                # Dashboard (main page)
│   ├── login/                  # Authentication pages
│   ├── register/
│   ├── transactions/           # Transaction forms
│   └── api/auth/               # Auth route handlers
├── src/
│   ├── components/             # React components
│   │   ├── auth/               # Login, Register forms
│   │   ├── transactions/       # Transaction forms & lists
│   │   ├── dashboard/          # Summary cards, charts
│   │   └── ui/                 # Reusable UI components
│   ├── server/                 # Server-side business logic
│   │   ├── auth/               # Auth actions, session
│   │   ├── transactions/       # Transaction actions
│   │   ├── categories/         # Category actions
│   │   ├── dashboard/          # Dashboard queries
│   │   └── db/                 # Database client & schema
│   └── lib/                    # Utilities
│       ├── currency.ts         # LKR formatting
│       ├── rounding.ts         # Banker's rounding
│       └── validation.ts       # Zod schemas
├── .env.local                  # Environment variables (create this)
├── .env.example                # Example env vars (commit this)
├── specs/001-.../              # Feature specifications
└── package.json
```

---

## 7. Common Development Tasks

### View Database

Use Drizzle Studio (visual database browser):

```bash
npm run db:studio
```

Opens at [https://local.drizzle.studio](https://local.drizzle.studio)

### Reset Database

```bash
# Drop all tables and recreate
psql $DATABASE_URL -c "DROP SCHEMA public CASCADE; CREATE SCHEMA public;"

# Re-run migrations
npm run db:migrate
npm run db:seed
```

### Test Email Sending (Development)

Without real email service during development:

1. Set `RESEND_API_KEY` to test mode or skip
2. Use console logging fallback in `src/server/auth/actions.ts`:

```typescript
if (process.env.NODE_ENV === 'development') {
  console.log('Password reset link:', resetUrl);
  // In production, send via Resend
}
```

### Check TypeScript

```bash
# Type check entire project
npm run type-check

# Add to package.json
"type-check": "tsc --noEmit"
```

### Run Linter

```bash
npm run lint

# Auto-fix issues
npm run lint -- --fix
```

---

## 8. Building for Production

### Local Production Build

```bash
# Create optimized production build
npm run build

# Start production server
npm start
```

### Environment Variables for Production

Ensure these are set in production environment:

```bash
# Production values
NEXTAUTH_URL="https://yourdomain.com"
NEXTAUTH_SECRET="<strong-secret-from-openssl>"
DATABASE_URL="<production-postgres-url>"
RESEND_API_KEY="<production-api-key>"
FROM_EMAIL="noreply@yourdomain.com"
NODE_ENV="production"
```

---

## 9. Deployment

### Vercel (Recommended)

1. **Install Vercel CLI**:
   ```bash
   npm i -g vercel
   ```

2. **Deploy**:
   ```bash
   vercel
   ```

3. **Configure Environment Variables**:
   - Go to Vercel Dashboard → Project → Settings → Environment Variables
   - Add all variables from `.env.local`
   - Deploy again: `vercel --prod`

4. **Database**:
   - Option 1: Use Vercel Postgres (integrated)
   - Option 2: Use external provider (Neon, Supabase) and set `DATABASE_URL`

**Automatic Deployments**:
- Connect GitHub repository in Vercel dashboard
- Every push to `main` triggers deployment
- Preview deployments for pull requests

### Self-Hosted (Alternative)

**Requirements**:
- Linux server (Ubuntu 22.04+ recommended)
- Node.js 20+ installed
- PostgreSQL 15+ database
- Reverse proxy (Nginx or Caddy) for HTTPS

**Steps**:

1. **Build Application**:
   ```bash
   npm run build
   ```

2. **Transfer Files** to server:
   ```bash
   rsync -avz .next node_modules package*.json user@server:/var/www/budgetmanager/
   ```

3. **Set Environment Variables** on server:
   ```bash
   # /var/www/budgetmanager/.env.local
   export DATABASE_URL="..."
   export NEXTAUTH_URL="https://yourdomain.com"
   export NEXTAUTH_SECRET="..."
   export RESEND_API_KEY="..."
   ```

4. **Start with PM2**:
   ```bash
   npm install -g pm2
   cd /var/www/budgetmanager
   pm2 start npm --name "budgetmanager" -- start
   pm2 startup  # Enable on boot
   pm2 save
   ```

5. **Configure Nginx**:
   ```nginx
   server {
     listen 80;
     server_name yourdomain.com;

     location / {
       proxy_pass http://localhost:3000;
       proxy_http_version 1.1;
       proxy_set_header Upgrade $http_upgrade;
       proxy_set_header Connection 'upgrade';
       proxy_set_header Host $host;
       proxy_cache_bypass $http_upgrade;
     }
   }
   ```

6. **Enable HTTPS** with Let's Encrypt:
   ```bash
   sudo apt install certbot python3-certbot-nginx
   sudo certbot --nginx -d yourdomain.com
   ```

---

## 10. Testing

### Manual Testing Checklist

**Authentication**:
- [ ] Register new account with valid email/password
- [ ] Register with duplicate email shows error
- [ ] Login with correct credentials succeeds
- [ ] Login with wrong password fails
- [ ] Logout clears session
- [ ] Session expires after 30 minutes of inactivity
- [ ] Forgot password sends email
- [ ] Reset password with valid token works

**Transactions**:
- [ ] Create income transaction appears in dashboard
- [ ] Create expense transaction appears in dashboard
- [ ] Edit transaction updates immediately
- [ ] Delete transaction removes from list
- [ ] Decimal rounding works (3+ decimals rounds correctly)
- [ ] Negative amounts are rejected
- [ ] Future dates are allowed
- [ ] Extremely long descriptions are truncated/rejected

**Dashboard**:
- [ ] Summary cards show correct totals
- [ ] Balance calculation is accurate (income - expenses)
- [ ] Positive balance shows green, negative shows red
- [ ] Expense pie chart displays correctly
- [ ] Income donut chart displays correctly
- [ ] Month picker filters data correctly
- [ ] Empty month shows "No data" message
- [ ] Mobile view (320px width) is responsive

### Automated Testing (Optional)

**Install Testing Libraries**:
```bash
npm install -D vitest @testing-library/react @testing-library/jest-dom
npm install -D playwright @playwright/test
```

**Run Tests**:
```bash
# Unit tests
npm run test

# E2E tests
npm run test:e2e
```

---

## 11. Troubleshooting

### Database Connection Errors

**Error**: `connection refused` or `could not connect to server`

**Solutions**:
- Check PostgreSQL is running: `pg_isready`
- Verify `DATABASE_URL` format: `postgresql://user:pass@host:port/db`
- Check firewall allows connections to port 5432
- For cloud databases, ensure IP whitelist includes your IP

### NextAuth.js Errors

**Error**: `NEXTAUTH_SECRET` not set

**Solution**:
```bash
# Generate new secret
openssl rand -base64 32

# Add to .env.local
NEXTAUTH_SECRET="<generated-secret>"
```

### Email Sending Fails

**Error**: Resend API returns 401 or 403

**Solutions**:
- Verify `RESEND_API_KEY` is correct
- Check domain is verified in Resend dashboard
- Use Resend sandbox domain for testing: `onboarding@resend.dev`

### Build Errors

**Error**: Module not found or type errors

**Solutions**:
```bash
# Clear cache and reinstall
rm -rf .next node_modules package-lock.json
npm install

# Type check
npm run type-check
```

---

## 12. Next Steps

After successful setup:

1. **Customize Design**: Update TailwindCSS styles in `app/globals.css` for minimal aesthetic
2. **Add Tests**: Implement unit tests for critical paths (auth, transactions)
3. **Performance**: Monitor with Vercel Analytics or Google Lighthouse
4. **Security**: Review constitution.md security checklist
5. **Documentation**: Update README.md with project-specific details

---

## Resources

- **Next.js Docs**: [nextjs.org/docs](https://nextjs.org/docs)
- **Drizzle ORM**: [orm.drizzle.team](https://orm.drizzle.team)
- **NextAuth.js**: [next-auth.js.org](https://next-auth.js.org)
- **Recharts**: [recharts.org](https://recharts.org)
- **Resend**: [resend.com/docs](https://resend.com/docs)
- **TailwindCSS v4**: [tailwindcss.com](https://tailwindcss.com)

---

## Support

For feature specification details, see:
- `specs/001-financial-tracker-mvp/spec.md` - Feature requirements
- `specs/001-financial-tracker-mvp/data-model.md` - Database schema
- `specs/001-financial-tracker-mvp/contracts/api-specification.md` - API contracts

For project constitution and development principles:
- `.specify/memory/constitution.md`
