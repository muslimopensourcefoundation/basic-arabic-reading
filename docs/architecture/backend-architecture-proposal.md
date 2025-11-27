# Backend Architecture Proposal

## Summary

Proposed backend stack:

- NestJS (TypeScript) with the Fastify adapter — modular, testable, and high-performance.
- Database: PostgreSQL with Prisma as the ORM.
- Auth: JWT with `@nestjs/jwt` and `passport-jwt` for stateless auth; refresh token flow for access token renewal; optional OAuth via Passport strategies.

### Token Expiry & Refresh

- Use short-lived access tokens (e.g., 15-30 minutes) for API authentication.
- Use long-lived refresh tokens (e.g., 7-30 days) to allow clients to obtain new access tokens without re-login.
- Store refresh tokens securely (e.g., httpOnly cookies or secure local storage).
- On access token expiry, client uses `/auth/refresh` to obtain a new access token.
- On refresh token expiry or invalidation, user must re-login.
- Implement token revocation and rotation for security (invalidate refresh tokens on logout or password change).
- For MVP, implement refresh token flow using `@nestjs/jwt` and Passport strategies; document token lifetimes and storage recommendations for clients.
- Deployment: Containerized (Docker) services, CI via GitHub Actions. Primary hosting target is Azure (for nonprofit credits and operational experience), but the stack remains portable for self-hosting on Render, Fly.io, AWS, or other platforms.

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

- User: id, email, name, passwordHash (argon2 or bcrypt), role, createdAt
- Lesson: id, slug, title, description, content(JSON), level, timestamps
- Exercise: id, lessonId, type, prompt(JSON), solution(JSON), metadata(JSON)
- Attempt: id, userId, exerciseId, answer(JSON), correct, score, clientId, createdAtClient, createdAtServer, syncedAt
- UserProgress: id, userId, lessonId, progress(JSON), lastUpdatedClient, lastUpdatedServer

### JSON Columns: Guidance & Examples

- JSON columns are suitable for flexible, evolving content (e.g., lesson blocks, prompts, progress details).
- If any internal property of a JSON column will be queried or filtered regularly (e.g., by value, type, or tag), consider extracting it as an explicit column and adding an index for performance and maintainability.
- For MVP, keep most content in JSON, but document any property that may need to be indexed or filtered in future features.

#### Example: Prompt JSON shape

```json
{
  "text": "اكتب الحرف الصحيح",
  "audioUrl": "https://.../audio.mp3",
  "letters": ["ب", "ت", "ث"],
  "type": "recognition",
  "difficulty": "easy",
  "tags": ["letter", "vowel"],
  "expected": "ب"
}
```

#### Example: UserProgress JSON shape

```json
{
  "percentComplete": 0.75,
  "completedExercises": ["ex1", "ex2"],
  "mastery": {
    "ب": 0.9,
    "ت": 0.7
  },
  "lastLessonViewed": "lesson-1"
}
```

#### Guidance

- Use explicit columns for properties that will be filtered, sorted, or joined against frequently (e.g., `difficulty`, `type`, `tags`).
- Use JSON for flexible, nested, or rarely-filtered data (e.g., lesson content blocks, exercise metadata).
- Revisit column normalization as features evolve and query patterns become clear.

### Password Security

- Store `passwordHash` using a modern, secure algorithm (argon2 preferred, bcrypt acceptable).
- Enforce a reasonable password policy (minimum length, complexity, and common password blacklist) at the API level.
- Use rate limiting and lockout mechanisms to prevent brute-force attacks on login endpoints.

### Attempt Sync & Conflict Resolution

- Each `Attempt` should include:
  - `clientId`: unique string per user/app combo (for deduplication and offline sync)
  - `createdAtClient`: timestamp from client device
  - `createdAtServer`: timestamp from backend (replaces `createdAt` for consistency)
  - `syncedAt`: timestamp when attempt was synced to server
- This enables reliable offline/online sync, deduplication, and conflict resolution. Backend should use these fields to merge attempts and avoid duplicates. Naming matches the `UserProgress` convention for clarity.
- UserProgress: id, userId, lessonId, progress(JSON), lastUpdatedClient, lastUpdatedServer

### Conflict Resolution

- To support offline/local progress and reliable sync, `UserProgress` should include both `lastUpdatedClient` (timestamp from client/local device) and `lastUpdatedServer` (timestamp from backend update).
- On sync, backend compares both timestamps and stores whichever is newer, or merges as needed.
- Alternative field names: `clientTimestamp`, `serverTimestamp`, or `updatedAtClient`, `updatedAtServer` (pick based on team convention; `lastUpdatedClient`/`lastUpdatedServer` is clear and explicit).

## API Surface

Base path: `/api/v1`

- Auth

  - POST `/auth/signup`
  - POST `/auth/login` — returns access token (short-lived, e.g. 15-30 min) and refresh token (long-lived, e.g. 7-30 days)
  - POST `/auth/refresh` — exchange refresh token for new access token
  - GET `/auth/me`

- Lessons

  - GET `/lessons` — list lessons with pagination and filtering
    - Query params:
      - `limit` (int, default: 20): max lessons per page
      - `offset` (int, default: 0): skip N lessons
      - `level` (string, optional): filter by lesson level (e.g., beginner, intermediate, advanced)
      - `q` (string, optional): search by title or description
      - `tags` (string[], optional): filter by tags
    - Example: `/lessons?level=beginner&q=alphabet&limit=10&offset=20`
  - GET `/lessons/:slug` — lesson + exercises

- Practice generation

  - POST `/practice/generate` — body: `{ lessonId?, letters?, includeVowels?, mode, seed?, count }` — returns structured prompts with canonical `expected` forms and `meta` (seed)

    - Example request:

    ```json
    {
      "lessonId": "lesson-1",
      "letters": ["ب", "ت", "ث"],
      "includeVowels": true,
      "mode": "random",
      "seed": "abc123",
      "count": 5
    }
    ```

    - Example response:

    ```json
    {
      "prompts": [
        {
          "id": "prompt-1",
          "display": "بَ",
          "expected": "بَ",
          "meta": {
            "letter": "ب",
            "vowel": "fatha",
            "form": "isolated",
            "seed": "abc123"
          }
        },
        {
          "id": "prompt-2",
          "display": "تِ",
          "expected": "تِ",
          "meta": {
            "letter": "ت",
            "vowel": "kasra",
            "form": "isolated",
            "seed": "abc123"
          }
        }
        // ...more prompts
      ],
      "seed": "abc123"
    }
    ```

Attempts & Progress

- POST `/attempts` — record attempt, return correctness and score (userId from JWT)
- POST `/progress/save` — save/merge user progress (userId from JWT)
- GET `/progress/me` — fetch progress for authenticated user (userId from JWT)
- For admin endpoints (e.g., `/progress/:userId`), require explicit admin privileges and document separately.
- `POST /progress/batch`: Accepts a batch of progress updates for efficient syncing. Useful for offline-first clients syncing multiple updates at once.
  - Example payload:
    ```json
    {
      "updates": [
        {
          "lessonId": "abc123",
          "progress": 0.8,
          "timestamp": "2025-11-27T12:34:56Z"
        },
        {
          "lessonId": "def456",
          "progress": 1.0,
          "timestamp": "2025-11-27T12:35:10Z"
        }
      ]
    }
    ```
  - Returns: Success/failure for each update, with conflict resolution applied as described (e.g., server wins except for locally newer timestamps).
- `POST /attempts/batch`: Accepts a batch of practice attempts for offline-to-online sync. Allows clients to submit multiple attempts in one request after reconnecting.
  - Example payload:
    ```json
    {
      "attempts": [
        {
          "lessonId": "abc123",
          "promptId": "p1",
          "result": "correct",
          "timestamp": "2025-11-27T12:36:00Z"
        },
        {
          "lessonId": "def456",
          "promptId": "p2",
          "result": "incorrect",
          "timestamp": "2025-11-27T12:36:15Z"
        }
      ]
    }
    ```
  - Returns: Success/failure for each attempt, with conflict resolution as needed.

### Security Practice: Auth Identity

- All user-specific endpoints should derive user identity from the JWT, not from URL/userId parameters, to prevent privilege escalation and ensure secure access control.
- Default to using the authenticated user's identity unless explicitly defining an admin use case.
- Document and restrict any endpoints that accept userId in the URL to admin-only scenarios.

All endpoints should validate input via DTOs and return typed responses.

## RNG Design (Backend & Shared Library)

- Maintain a canonical dataset of Arabic letters and diacritics with metadata including joining rules and codepoints.
- The canonical dataset and generation algorithm (seeded PRNG, joining logic) will be kept in a shared library/module/package, importable by both backend and frontend. This enables offline-first practice generation and consistent rules across the stack.
- The backend will expose the dataset via API for clients that cannot bundle it, but the preferred approach is to use the shared library for offline-first support.
- Implement a seeded PRNG service (e.g., sfc32 or mulberry32) as an injectable `RngService` (backend) and as a shared utility (frontend).
- Provide modes: `random`, `scaffolded` (weight by mastery), `targeted`.
- Ensure Unicode NFC normalization and canonical `expected` stored per prompt for scoring.
- Add unit tests to validate the dataset (e.g., completeness, codepoints, joining rules) and the joining logic in the shared library.

Example service mapping:

- `PracticeController` -> `PracticeService` -> `RngService` + `LetterRepository` (using shared library)

Dev & Ops Tools (Backend)

**MVP Essentials:**

- Nest CLI for scaffolding
- Prisma + Prisma Migrate + Prisma Studio
- Dockerfile + `docker-compose` for local Postgres + Redis
- GitHub Actions workflows: lint, test, build, deploy
- Azure DevOps pipelines and Azure App Service/Container Instances for primary hosting
- Basic logging and health check endpoints

**Post-MVP Improvements:**

- Monitoring: Azure AppInsights (priority for Azure), Sentry (targeted error tracking), Prometheus + Grafana (metrics)
- Security: Dependabot, secret scanning, scheduled dependency checks
- Document Azure-specific deployment steps, but keep Docker and CI portable for other platforms

## Local Progress, Sync, and Offline Support

- Users can practice and store progress locally (e.g., in browser localStorage or IndexedDB) before creating an account.
- On sign up or login, the frontend syncs local progress to the backend using `/progress/save` and `/attempts` endpoints. The backend merges local progress with any existing account data.
- If the user's JWT token expires while offline, the frontend continues to store new practice attempts and progress locally in a sync queue. Once the token is refreshed (user logs in or regains connectivity), the sync queue is sent to the backend for processing.
- Backend endpoints should:
  - Accept batched progress/attempts for efficient sync (`/progress/save` and `/attempts` can accept arrays).
  - Merge progress intelligently (e.g., update mastery, avoid duplicate attempts).
  - Return updated progress state after sync for frontend reconciliation.

### Technical Details

- **Frontend:**
  - Store progress and attempts in localStorage/IndexedDB with a sync queue for offline actions.
  - On login/signup, send all unsynced progress/attempts to backend.
  - If token expires, continue queueing; resume sync after token refresh.
  - Use a timestamp or unique ID per attempt/progress for deduplication.
- **Backend:**
  - `/progress/save` and `/attempts` endpoints accept batch payloads and merge with existing user data.
  - Use upsert logic in Prisma to avoid duplicates and merge progress.
  - Return merged/updated progress for frontend display.
  - Optionally log sync events for audit/debugging.

## Technical scope (Backend MVP)

- Implement NestJS app with modules: `Auth`, `Lessons`, `Practice`, `Progress`, `Users`
- Add Prisma schema and initial migrations
- Implement `/practice/generate` with seeded RNG and unit tests
- Add basic auth and attempt recording endpoints
- Support local progress sync and offline queueing as described above

## Risks & Tradeoffs (Backend)

- NestJS increases initial boilerplate but improves long-term maintainability.
- Modeling Arabic joining accurately is the primary domain risk.
- Using a managed Postgres instance (Azure Database for PostgreSQL in the MOSF deployment; Supabase / RDS / etc. are also viable for self-hosted deployments).

## Next Steps (Backend)

- Scaffold NestJS project with Fastify adapter and Prisma schema
- Implement `PracticeModule` with RNG and unit tests
- Add CI workflow and prepare a staging deployment
