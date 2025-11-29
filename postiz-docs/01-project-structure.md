# Project Structure

## Overview

Postiz is built as an **NX monorepo** containing multiple applications and shared libraries. This architecture enables code reuse, consistent tooling, and independent deployment of services.

## Top-Level Structure

```
postiz-app/
├── apps/                 # Applications
│   ├── backend/         # NestJS API server
│   ├── frontend/        # Next.js web application
│   ├── workers/         # Background job processors
│   ├── cron/            # Scheduled task runner
│   ├── commands/        # CLI commands
│   ├── extension/       # Browser extension
│   └── sdk/             # JavaScript SDK
├── libraries/           # Shared libraries
│   ├── nestjs-libraries/    # Backend shared code
│   ├── react-shared-libraries/ # Frontend shared code
│   └── helpers/             # Common utilities
├── .github/            # GitHub Actions workflows
├── Jenkins/            # Jenkins CI configuration
├── var/                # Variable data (docker scripts)
└── reports/            # Test and coverage reports
```

## Applications (`/apps`)

### 1. Backend (`apps/backend`)
**Purpose**: Main REST API server

**Tech Stack**:
- NestJS framework
- Express.js
- Swagger/OpenAPI documentation
- JWT authentication
- Rate limiting with Throttler

**Directory Structure**:
```
apps/backend/src/
├── api/
│   └── routes/                    # API Controllers
│       ├── auth.controller.ts     # Authentication endpoints
│       ├── posts.controller.ts    # Post management
│       ├── integrations.controller.ts # Social integrations
│       ├── analytics.controller.ts # Analytics data
│       ├── billing.controller.ts  # Payment/subscription
│       ├── media.controller.ts    # File uploads
│       ├── marketplace.controller.ts # Post marketplace
│       ├── settings.controller.ts # User/org settings
│       ├── copilot.controller.ts  # AI assistant
│       └── ...
├── public-api/
│   └── routes/                    # Public API v1
├── services/
│   └── auth/                      # Auth services
│       └── permissions/           # CASL permissions
├── app.module.ts                  # Root module
└── main.ts                        # Application entry point
```

**Key Features**:
- RESTful API with Swagger documentation
- Role-based access control (RBAC)
- OAuth integration for social platforms
- Webhook support
- Public API for third-party integrations

**Scripts**:
- `pnpm dev:backend` - Development mode with hot reload
- `pnpm build:backend` - Production build
- `pnpm start:prod:backend` - Start production server

---

### 2. Frontend (`apps/frontend`)
**Purpose**: Next.js web application

**Tech Stack**:
- Next.js 14+ (App Router)
- React 18
- TailwindCSS + Mantine UI
- TypeScript
- SWR for data fetching
- Zustand for state management

**Directory Structure**:
```
apps/frontend/src/
├── app/
│   ├── (app)/                     # Authenticated routes
│   │   ├── (site)/               # Main application
│   │   │   ├── launches/         # Post scheduling
│   │   │   ├── analytics/        # Analytics dashboard
│   │   │   ├── marketplace/      # Post marketplace
│   │   │   ├── settings/         # Settings pages
│   │   │   └── ...
│   │   ├── (preview)/            # Preview pages
│   │   ├── auth/                 # Auth pages
│   │   └── api/                  # API routes
│   └── (extension)/              # Extension popup
├── components/                    # React components
│   ├── launches/                 # Post scheduling UI
│   ├── analytics/                # Analytics charts
│   ├── marketplace/              # Marketplace UI
│   ├── settings/                 # Settings forms
│   ├── copilot/                  # AI assistant
│   └── ...
├── middleware.ts                 # Next.js middleware
└── instrumentation.ts            # Monitoring setup
```

**Key Features**:
- Server-side rendering (SSR)
- Protected routes with middleware
- Responsive design
- Real-time updates
- File upload with drag-and-drop
- Rich text editor (TipTap)
- AI-powered content generation

**Scripts**:
- `pnpm dev:frontend` - Development server
- `pnpm build:frontend` - Production build
- `pnpm start:prod:frontend` - Start production server

---

### 3. Workers (`apps/workers`)
**Purpose**: Background job processors using BullMQ

**Tech Stack**:
- NestJS Microservices
- BullMQ (Redis-based queue)
- Multiple worker types

**Responsibilities**:
- Post publishing to social platforms
- Media processing (images, videos)
- Webhook delivery
- Email notifications
- Analytics data collection
- Scheduled post execution

**Key Workers**:
- Post publication worker
- Media upload worker
- Analytics sync worker
- Notification worker
- Integration refresh worker

**Scripts**:
- `pnpm dev:workers` - Development mode
- `pnpm build:workers` - Production build
- `pnpm start:prod:workers` - Start workers

---

### 4. Cron (`apps/cron`)
**Purpose**: Scheduled tasks and maintenance jobs

**Tech Stack**:
- NestJS Schedule
- Cron expressions

**Scheduled Tasks**:
- Social platform token refresh
- Analytics data aggregation
- Database cleanup
- Subscription renewals
- Email digest sending
- Integration health checks

**Scripts**:
- `pnpm dev:cron` - Development mode
- `pnpm build:cron` - Production build
- `pnpm start:prod:cron` - Start cron service

---

### 5. Extension (`apps/extension`)
**Purpose**: Browser extension for quick posting

**Tech Stack**:
- React
- Vite with CRXJS
- WebExtension API
- Manifest V3

**Features**:
- Quick post creation from any webpage
- Content scraping
- Image extraction
- Integration with main app

---

### 6. Commands (`apps/commands`)
**Purpose**: CLI tools for database operations and maintenance

**Tech Stack**:
- NestJS Command
- Yargs

**Available Commands**:
- Database seeding
- User management
- Data migration
- System diagnostics

---

### 7. SDK (`apps/sdk`)
**Purpose**: JavaScript/TypeScript SDK for Postiz API

**Features**:
- Type-safe API client
- Promise-based interface
- Published to npm as `@postiz/node`

---

## Shared Libraries (`/libraries`)

### 1. NestJS Libraries (`libraries/nestjs-libraries`)
**Purpose**: Backend shared code and services

**Key Modules**:

```
nestjs-libraries/src/
├── database/
│   └── prisma/                    # Prisma ORM
│       ├── schema.prisma         # Database schema
│       └── [entity]/             # Service per entity
│           └── *.service.ts      # CRUD operations
├── integrations/
│   └── social/                   # Social platform integrations
│       ├── twitter.provider.ts
│       ├── instagram.provider.ts
│       ├── linkedin.provider.ts
│       └── ...
├── dtos/                         # Data Transfer Objects
│   ├── auth/
│   ├── posts/
│   ├── integrations/
│   └── ...
├── agent/                        # AI agent services
├── bull-mq-transport-new/       # Queue transport
├── emails/                       # Email templates
├── upload/                       # File upload handling
├── redis/                        # Redis client
├── openai/                       # OpenAI integration
├── short-linking/               # URL shortening
├── user/                        # User decorators/guards
└── ...
```

**Key Components**:
- **Database Services**: Prisma-based data access layer
- **Social Integrations**: Platform-specific providers (20+ platforms)
- **DTOs**: Request/response validation with class-validator
- **Queues**: BullMQ job management
- **Email**: Transactional email templates
- **AI**: OpenAI/LangChain integration

---

### 2. React Shared Libraries (`libraries/react-shared-libraries`)
**Purpose**: Frontend shared components and utilities

**Modules**:
```
react-shared-libraries/src/
├── form/                         # Form components
├── helpers/                      # Utility functions
├── toaster/                      # Toast notifications
├── translation/                  # i18n utilities
└── sentry/                       # Error tracking
```

---

### 3. Helpers (`libraries/helpers`)
**Purpose**: Common utilities for both frontend and backend

**Modules**:
```
helpers/src/
├── auth/                         # Auth utilities
├── configuration/                # Config management
├── decorators/                   # TypeScript decorators
├── subdomain/                    # Subdomain handling
├── swagger/                      # API docs helpers
└── utils/                        # Generic utilities
```

---

## Configuration Files

### Root Level
- `package.json` - Main package file with workspace scripts
- `pnpm-workspace.yaml` - pnpm workspace configuration
- `tsconfig.base.json` - Base TypeScript config
- `nx.json` - NX workspace configuration
- `docker-compose.dev.yaml` - Development docker setup
- `.env.example` - Environment variables template

### Build & Development
- `jest.config.ts` - Test configuration
- `eslint.config.mjs` - ESLint rules
- `.prettierrc` - Code formatting

---

## Development Workflow

### Monorepo Structure Benefits

1. **Code Sharing**: Shared libraries prevent duplication
2. **Consistent Tooling**: Same build tools across all apps
3. **Type Safety**: Shared TypeScript types
4. **Atomic Changes**: Update API and client together
5. **Efficient CI/CD**: Only build affected projects

### Running the Full Stack

```bash
# Install dependencies
pnpm install

# Start all services in development
pnpm dev

# Start individual services
pnpm dev:backend
pnpm dev:frontend
pnpm dev:workers
pnpm dev:cron

# Build for production
pnpm build
```

### Database Management

```bash
# Generate Prisma client
pnpm prisma-generate

# Push schema changes
pnpm prisma-db-push

# Reset database
pnpm prisma-reset
```

---

## Deployment Architecture

### Production Services

1. **Backend API** (Port 3000)
   - Handles HTTP requests
   - JWT authentication
   - Rate limiting

2. **Frontend** (Port 3003)
   - Next.js SSR
   - Served via CDN in production

3. **Workers** (Background)
   - Multiple instances for scalability
   - Process jobs from Redis queue

4. **Cron** (Background)
   - Single instance for scheduled tasks

5. **Database** (PostgreSQL)
   - Primary data store
   - Managed by Prisma

6. **Redis**
   - BullMQ job queue
   - Session storage
   - Caching

---

## Package Management

- **Package Manager**: pnpm (version 10.6.1)
- **Node Version**: 22.12.0+
- **Workspaces**: Managed via pnpm workspaces
- **Dependencies**: Shared dependencies in root, app-specific in subdirectories

---

## Key Design Patterns

1. **Service Layer**: Business logic separated from controllers
2. **Repository Pattern**: Data access through Prisma services
3. **Factory Pattern**: Dynamic integration loading
4. **Dependency Injection**: NestJS IoC container
5. **Queue Pattern**: Async job processing with BullMQ
6. **Strategy Pattern**: Different social platform strategies

---

## Testing Structure

```
# Unit tests
*.spec.ts              # Co-located with source files

# E2E tests
apps/*/test/           # Application-level tests

# Test reports
reports/               # Coverage and test results
```

---

## Next Steps

- See `02-backend-api.md` for API endpoint details
- See `03-frontend-architecture.md` for frontend components
- See `04-database-models.md` for data models
- See `05-api-capabilities.md` for API usage examples
