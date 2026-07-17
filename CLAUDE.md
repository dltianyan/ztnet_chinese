---
env_file: .env.local
---

# ZTNET Chinese Edition - Development Rules

## Project Overview

ZTNET is a self-hosted ZeroTier network controller web UI with organization and multi-user management. This is a Chinese-localized fork of [sinamics/ztnet](https://github.com/sinamics/ztnet), published at [dltianyan/ztnet_chinese](https://github.com/dltianyan/ztnet_chinese).

- **License**: GPL v3.0 — all modifications must remain GPL v3.0 compliant
- **Tech Stack**: Next.js 16 (Pages Router), TypeScript 5, Prisma 6 + PostgreSQL, tRPC 10, Tailwind CSS + DaisyUI, better-auth, Zustand, Zod, Jest, Biome
- **i18n**: Next.js built-in i18n routing + `next-intl` for translations

## Upstream Sync Policy

- Upstream: `sinamics/ztnet` (main branch)
- Merge upstream regularly to avoid divergence
- Before merging, always review upstream changelog for breaking changes
- After a merge, run `npm ci && npx prisma migrate dev` to apply new migrations
- Run full test suite after every upstream merge: `npm test`

## Architecture

```
src/
  pages/           # Next.js Pages Router (auth, admin, network, organization, dashboard, central)
    api/           # REST API routes (/api/v1/*, /api/trpc/*, websocket)
  server/
    api/
      routers/     # tRPC routers (admin, auth, member, network, organization, etc.)
      services/    # Business logic services (memberService, networkService, authService, etc.)
      utils/       # Shared utilities (ipUtils, memberUtils)
      trpc.ts      # tRPC configuration
      root.ts      # Root router combining all sub-routers
    helpers/       # Error handlers
    db.ts          # Prisma client singleton
  components/      # React components organized by page/feature
    adminPage/     # Admin panel components
    auth/          # Authentication forms
    networkByIdPage/ # Network detail page components
    networkPage/   # Network list components
    organization/  # Organization management components
    organizationPage/ # Organization admin components
    layouts/       # Layout components (sidebar, footer)
    elements/      # Reusable UI elements (input, textarea, button)
    shared/        # Cross-cutting shared components (modals, table)
  hooks/           # Custom React hooks
  locales/         # Translation files (en, zh, zh-tw, de, es, fr, no, pl, ru, ua)
  types/           # TypeScript type definitions
  utils/           # Utility functions and stores (Zustand)
prisma/
  schema.prisma    # Database schema (PostgreSQL)
  migrations/      # Database migration history
  seeds/           # Database seed data
ztnodeid/          # ZeroTier mkworld binary sources (Go)
```

## Localization Rules

- **Primary locale for this fork**: `zh` (Simplified Chinese)
- Translation files live in `src/locales/<lang>/common.json`
- When adding new UI text, add keys to ALL locale files with English as fallback
- Use `useTranslations("namespace")` from `next-intl` in components
- Key format: `"componentNamespace.camelCaseKey"` (e.g., `"commonButtons.confirm"`)
- Current supported locales: `en`, `zh`, `zh-tw`, `fr`, `no`, `pl`, `es`, `ru`, `de`, `ua`

## Development Workflow

### Local Development

```bash
# 1. Install dependencies
npm ci

# 2. Set up .env from .env.example (adjust DATABASE_URL for localhost)
cp .env.example .env

# 3. Start PostgreSQL and run migrations
npx prisma migrate dev

# 4. Start dev server
npm run dev
```

### Docker Development

The production deployment uses Docker Compose at `/opt/1panel/docker/compose/ztnet/docker-compose.yml`. For local Docker dev:

```bash
# Build local Docker image
docker build -t ztnet-chinese:dev .

# Or use docker-compose with local image
docker compose up -d
```

### Before Commit Checklist

1. Run linter: `npm run lint`
2. Run formatter: `npm run format:fix`
3. Run tests: `npm test`
4. Ensure no `console.log` remains in production code
5. Verify translations are added to all locale files
6. TypeScript must compile without errors

## Code Style

- **Formatter**: Biome (config in `biome.json`)
- **TypeScript**: Strict mode, prefer `interface` for object shapes, `type` for unions
- **Immutability**: NEVER mutate objects — use spread operator for updates
- **Components**: Named `interface` for props, no `React.FC`
- **No `any`**: Use `unknown` for external input, narrow safely
- **Early returns**: Prefer over nested conditionals
- **File organization**: 200-400 lines typical, 800 max per file
- **Use Zod** for input validation at API boundaries

## Testing

- **Framework**: Jest (unit + integration tests in `src/__tests__/` and `src/server/api/__tests__/`)
- **Commands**: `npm run test:dev` (development), `npm test` (CI with coverage)
- **Coverage target**: 80%+
- **Structure**: AAA pattern (Arrange-Act-Assert), descriptive test names

## Security (CRITICAL)

- **NEVER hardcode secrets** — use environment variables only
- All API routes must validate input with Zod schemas
- ZeroTier controller communication must go through the server, never directly from the client
- Authentication is handled by `better-auth` — do not bypass
- Rate limiting is enabled on auth endpoints and REST API endpoints
- SQL queries must use Prisma's parameterized queries — never raw SQL with user input

## Key Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| next | ^16.2.4 | React framework |
| react | 18.2.0 | UI library |
| @prisma/client | 6.19.3 | ORM |
| @trpc/client/server | 10.45.3 | Type-safe API layer |
| better-auth | ^1.6.9 | Authentication |
| next-intl | ^4.9.1 | Internationalization |
| tailwindcss + daisyui | 3.4.19 / 4.12.10 | Styling |
| zod | ^4.3.6 | Schema validation |
| zustand | 4.3.6 | Client state management |
| jest | ^30.3.0 | Testing |
| @biomejs/biome | 1.9.4 | Linting and formatting |

## Database

- **Provider**: PostgreSQL 15
- **Connection**: Via `DATABASE_URL` env var
- **Migrations**: Run via `npx prisma migrate dev`
- **Shadow DB**: Required for migrations, configured via `MIGRATE_DATABASE_URL`
- Key models: `GlobalOptions`, `User`, `Network`, `Organization`, `OrganizationGroup`, `Planet`, `Webhook`, `ApiToken`
