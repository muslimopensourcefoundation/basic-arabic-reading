# Backend Architecture Proposal

## Summary

Proposed backend stack:

- NestJS (TypeScript) with the Fastify adapter — modular, testable, and high-performance.
- Database: PostgreSQL with Prisma as the ORM.
- Auth: JWT with `@nestjs/jwt` and `passport-jwt` for stateless auth; optional OAuth via Passport strategies.
- Deployment: Containerized (Docker) services, CI via GitHub Actions, host backend on Render / Fly.io / AWS.

Why NestJS + Fastify?

- NestJS provides clear project structure (Modules, Controllers, Providers), DI, and testing utilities — useful for teamwork and scaling.
- The Fastify adapter preserves high throughput and low latency.

## Components (Backend)

- Framework: NestJS + Fastify adapter
- ORM: Prisma (Postgres) — typed schema, migrations, Prisma Client
- Validation: DTOs with `class-validator` + `class-transformer` (or Zod where preferred)
- Auth: `@nestjs/passport`, `@nestjs/jwt` and Guard patterns
- Background jobs: bullmq + Redis for scheduled tasks and heavy processing
- Config: `@nestjs/config` for environment-based configuration
- Testing: Jest + `@nestjs/testing`, SuperTest for e2e

## Data Model

Keep the same Prisma-like data model but list here for backend implementers:

- User: id, email, name, passwordHash, role, createdAt
- Lesson: id, slug, title, description, content(JSON), level, timestamps
- Exercise: id, lessonId, type, prompt(JSON), solution(JSON), metadata(JSON)
- Attempt: id, userId, exerciseId, answer(JSON), correct, score, createdAt
- UserProgress: id, userId, lessonId, progress(JSON), lastUpdated

Relationships and indexes are the same as in the overall proposal.

## API Surface

Base path: `/api/v1`

- Auth

  - POST `/auth/signup`
  - POST `/auth/login`
  - GET `/auth/me`

- Lessons

  - GET `/lessons` — list, filters
  - GET `/lessons/:slug` — lesson + exercises

- Practice generation

  - POST `/practice/generate` — body: `{ lessonId?, letters?, includeVowels?, mode, seed?, count }` — returns structured prompts with canonical `expected` forms and `meta` (seed)

- Attempts & Progress
  - POST `/attempts` — record attempt, return correctness and score
  - POST `/progress/save` — save/merge user progress
  - GET `/progress/:userId` — fetch user progress

All endpoints should validate input via DTOs and return typed responses.

## RNG Design (Backend details)

- Maintain a canonical dataset of Arabic letters and diacritics with metadata including joining rules and codepoints.
- Implement a seeded PRNG service (e.g., sfc32 or mulberry32) as an injectable `RngService` so `PracticeService` can be deterministic when seed provided.
- Provide modes: `random`, `scaffolded` (weight by mastery), `targeted`.
- Ensure Unicode NFC normalization and canonical `expected` stored per prompt for scoring.

Example service mapping:

- `PracticeController` -> `PracticeService` -> `RngService` + `LetterRepository`

## Dev & Ops Tools (Backend)

- Nest CLI for scaffolding
- Prisma + Prisma Migrate + Prisma Studio
- Dockerfile + `docker-compose` for local Postgres + Redis
- GitHub Actions workflows: lint, test, build, deploy
- Monitoring: Sentry, Prometheus + Grafana for metrics
- Security: Dependabot, secret scanning, scheduled dependency checks

## Technical scope (Backend MVP)

- Implement NestJS app with modules: `Auth`, `Lessons`, `Practice`, `Progress`, `Users`
- Add Prisma schema and initial migrations
- Implement `/practice/generate` with seeded RNG and unit tests
- Add basic auth and attempt recording endpoints

## Risks & Tradeoffs (Backend)

- NestJS increases initial boilerplate but improves long-term maintainability.
- Modeling Arabic joining accurately is the primary domain risk.
- Using managed DB (Supabase / RDS) reduces ops burden.

## Next Steps (Backend)

- Scaffold NestJS project with Fastify adapter and Prisma schema
- Implement `PracticeModule` with RNG and unit tests
- Add CI workflow and prepare a staging deployment
