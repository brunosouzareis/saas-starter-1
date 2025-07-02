# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Next.js SaaS Starter template that provides a foundation for building subscription-based applications with authentication, team management, and Stripe integration.

## Development Commands

```bash
# Development
pnpm dev              # Start development server with Turbopack

# Database
pnpm db:setup         # Initial database setup (creates .env file)
pnpm db:migrate       # Run database migrations
pnpm db:seed          # Seed database with test data (test@test.com / admin123)
pnpm db:generate      # Generate new migrations after schema changes
pnpm db:studio        # Open Drizzle Studio for database management

# Build & Production
pnpm build            # Build for production
pnpm start            # Start production server

# Stripe Local Development
stripe listen --forward-to localhost:3000/api/stripe/webhook
```

## Architecture Overview

### Technology Stack
- **Framework**: Next.js 15.4 (App Router, experimental PPR enabled)
- **Database**: PostgreSQL with Drizzle ORM
- **Authentication**: Custom JWT-based auth with cookies
- **Payments**: Stripe subscriptions
- **UI**: shadcn/ui components with Radix UI
- **Styling**: Tailwind CSS v4 with CSS variables
- **State Management**: SWR for data fetching

### Directory Structure
```
/app/                    # Next.js App Router
  ├── (dashboard)/      # Authenticated routes group
  ├── (login)/          # Auth pages (sign-in, sign-up)
  ├── api/              # API routes
  └── layout.tsx        # Root layout

/components/ui/         # shadcn/ui components (Radix-based)
/lib/                   # Core application logic
  ├── auth/            # Authentication middleware & sessions
  ├── db/              # Database schema, queries, migrations
  └── payments/        # Stripe integration
```

### Authentication Flow
- JWT tokens stored in HTTP-only cookies (24-hour expiration)
- Session validation via `getUser()` in server components
- Protected routes via middleware.ts for `/dashboard/*`
- Server Actions use `validatedActionWithUser` middleware
- Role-based access: owner/member roles per team

### Database Schema
- **users**: Basic user info with soft deletes
- **teams**: Multi-tenant with Stripe subscription data
- **teamMembers**: Many-to-many user-team relationships
- **activityLogs**: Audit trail for all user actions
- **invitations**: Team invitation system

### Component Patterns
```typescript
// UI components use CVA for variants
<Button variant="destructive" size="sm" />

// Server actions with validation
export const updateName = validatedActionWithUser({
  input: z.object({ name: z.string().min(1).max(100) }),
  async handler(input, ctx) {
    // ctx.user is automatically available
  }
});

// Data fetching with SWR
const { data, error, mutate } = useSWR('/api/user', fetcher);
```

### Stripe Integration
- Checkout via `/api/stripe/checkout`
- Webhook handling at `/api/stripe/webhook`
- Customer portal for subscription management
- Team subscription status tracked in database

### Development Patterns
- **Type Safety**: Strict TypeScript throughout
- **Validation**: Zod schemas for all user inputs
- **Error Handling**: Consistent try-catch patterns
- **Loading States**: Skeleton components for async UI
- **Accessibility**: Proper ARIA attributes and focus management

### Environment Variables
Required for development:
- `POSTGRES_URL`: Database connection
- `AUTH_SECRET`: JWT signing key
- `STRIPE_SECRET_KEY`: Stripe API key
- `STRIPE_WEBHOOK_SECRET`: Webhook validation
- `BASE_URL`: Application URL

### Common Tasks

1. **Add a new page**: Create file in `app/` following the folder structure
2. **Add API endpoint**: Create route.ts in `app/api/[endpoint]/`
3. **Modify database**: Update `lib/db/schema.ts`, then run `pnpm db:generate` and `pnpm db:migrate`
4. **Add UI component**: Check existing patterns in `components/ui/`
5. **Protected server action**: Use `validatedActionWithUser` middleware
6. **Team-scoped queries**: Use `withTeam` middleware for team access

### Important Notes
- No testing framework is configured - add Jest/Vitest as needed
- No linting/formatting tools - consider adding ESLint/Prettier
- Uses experimental Next.js features (PPR, clientSegmentCache)
- All user-facing forms should use Server Actions
- Maintain multi-tenancy by scoping queries to current team
- Follow existing UI component patterns with CVA variants