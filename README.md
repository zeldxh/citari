# Citari

Production-oriented, multi-tenant appointment/booking platform. A pnpm monorepo with two workspace
apps — a NestJS/Fastify API and a Next.js frontend — backed by PostgreSQL 17 via Prisma.

## Stack

- **API** (`apps/api`): NestJS on Fastify, Prisma ORM, PostgreSQL 17, Vitest.
- **Frontend** (`apps/frontend`): Next.js App Router, Tailwind, shadcn-style UI primitives,
  Vitest + Testing Library, Playwright for e2e.
- **Infra**: Docker Compose for local Postgres, multi-stage Dockerfiles for production images,
  GitHub Actions for CI (`pnpm quality`, integration tests, e2e) and image publishing.

## What it does

- **Public**: marketing landing page, multi-step public booking flow (`/book/[slug]/...`), and a
  customer-facing booking tracking page (`/track`).
- **Business owner back-office**: services/categories, locations, availability (business hours +
  slot generation), customers, bookings, reports, and business settings.
- **Superadmin console**: tenant management (`/admin/tenants`).

The API is organized as NestJS "domains" (admin/catalog/customers/public/reports) plus feature
modules (auth, availability, bookings, tenant, notifications, scheduling, security), with full
Prisma migrations for the relational schema (users, tenants, memberships, services, locations,
bookings, slot holds, audit events, rate-limit buckets, MFA/auth challenges, email delivery
outbox).

## Getting started

Recommended (Docker):

```bash
docker compose up --build
```

Brings up PostgreSQL, the API, and the frontend. No demo records are created — bootstrap the
first superadmin explicitly (see below). Frontend at `http://localhost:3000`.

Without Docker, on the host:

```bash
corepack enable
pnpm install --frozen-lockfile
pnpm infra:up            # starts PostgreSQL only
pnpm db:validate && pnpm db:migrate:deploy
pnpm db:generate
pnpm dev                 # runs api + frontend in parallel
```

Copy `.env.example` to `.env` first and replace every `REPLACE_...` value. The database starts
empty by design — the superadmin bootstrap is a deployment operation, not a seed:

```bash
pnpm admin:bootstrap
```

Pipe the initial password via stdin from your secret provider; never place it in a command
argument, environment file, or shell history.

## Quality gate

```bash
pnpm quality
```

Verifies the committed OpenAPI v1 contract is current, then runs lint, typecheck, coverage tests,
and build across the workspace. Run this before opening a pull request.

```bash
pnpm test:e2e   # Playwright end-to-end tests
```

## Docs

- [`docs/adr/0001-postgresql-prisma-api.md`](docs/adr/0001-postgresql-prisma-api.md) — architecture decision record
- [`docs/development.md`](docs/development.md) — local development setup
- [`docs/deployment.md`](docs/deployment.md) — production deployment, images, required runtime config
- [`docs/postgresql-migration-runbook.md`](docs/postgresql-migration-runbook.md) — migration runbook
- [`docs/security.md`](docs/security.md) — security model
- [`docs/qa.md`](docs/qa.md) — QA/testing approach
- [`docs/production-mvp-blueprint.md`](docs/production-mvp-blueprint.md) — delivery blueprint

## License

See [LICENSE](LICENSE).
