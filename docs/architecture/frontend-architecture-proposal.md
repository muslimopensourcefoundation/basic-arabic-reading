# Frontend Architecture Proposal

## Summary

Proposed frontend stack:

- React + Vite with TypeScript — fast dev server, modern build pipeline, and excellent DX.
- State + server data: React Query (TanStack Query) for server state; Zustand or Redux Toolkit for local UI state if needed.
- Styling and components: Tailwind CSS for utility-first styles or Chakra UI / Radix for accessible components.
- i18n: `react-i18next` for translations and RTL support.

Why this choice?

- Vite + React gives fast iteration and small production bundles.
- TypeScript improves maintainability and helps contributors.
- React Query and component libraries reduce boilerplate for data fetching and accessibility.

## Components (Frontend)

### Offline and Storage Strategy

- **Connectivity Indicator:**
  - A small indicator in the header or footer shows online/offline status, providing unobtrusive feedback about connection state.
- **Sync State Indicator & Retry:**
  - When syncing in the background, show a sync state indicator. Provide a way to retry failed syncs so data is not silently lost.
- **Basic Conflict Handling Rule:**

  - For MVP, use a simple rule: "server wins except for locally newer timestamps". This ensures local changes are not lost if they are newer than server data, but otherwise server state is authoritative.

- **Offline UI/App Shell:**
  - The app shell (navigation, core UI, routing) loads and functions offline using service workers and local cache.
- **Local Storage & Queue Management:**
  - Progress, attempts, and user actions are stored in IndexedDB (preferred for React Query persistence and offline survival) or localStorage. A sync queue tracks unsynced changes and manages retries when connectivity is restored.
- **Optimistic Mutations & Sync:**
  - Mutations (e.g., practice attempts, progress updates) should optimistically update local state and queue changes for backend sync when online. React Query's mutation cache can be extended to support this pattern.
  - Conflict handling rules will be needed to resolve differences between local and server state (e.g., using timestamps, merging strategies).
- **Content Management & Lesson Caching:**
  - Lessons and exercises are cached locally after first load, allowing users to access and practice previously viewed content offline.
- **Local Mirroring of Canonical Dataset:**
  - The canonical dataset of letters, diacritics, and joining rules is bundled or synced locally, so practice generation and validation work offline using the same rules as backend.
- **Sync Logic:**

  - On reconnect or login, the app syncs local progress/attempts to the backend, resolving conflicts using timestamps and merging logic.

- Framework: React + Vite + TypeScript
- Routing: React Router
- Server state: React Query (caching, background refetching, optimistic updates)
- Local state: Zustand (small) or Redux Toolkit (structured) where needed
- UI: Tailwind CSS + Headless UI.
- Forms & validation: React Hook Form + Zod for schema-driven validation
- i18n: `react-i18next` with JSON translation files and RTL switching
- Accessibility tools: axe-core (eslint-plugin-jsx-a11y) during dev and CI
- Offline-first behavior: Users can practice, generate exercises, and store progress locally even without internet connectivity. Local progress is synced to the backend when online.

## Accessibility and i18n (Frontend-specific)

- Keyboard navigation: Add logical tab order, focus-visible outlines, and keyboard shortcuts for practice actions.
- Fonts: include high-quality Arabic fonts such as `Noto Sans Arabic` and `Scheherazade`, provide font-size controls, and high-contrast themes.
- RTL layout: Enable `dir="rtl"` on root when Arabic is active; use CSS logical properties and component-library RTL support.
- Screen readers: Use semantic HTML, ARIA where needed, and provide transliterations for glyphs.

## UI Surface and Pages (MVP)

- Landing / Lessons list (filter by level)
  - Offline: Previously loaded lessons are available and filterable; new lessons require connectivity to fetch.
- Lesson page with content blocks (text, audio, exercises)
  - Offline: Previously viewed lesson content and exercises are cached and accessible; audio is available if previously cached.
- Practice UI for generated prompts (display prompt, input, submit, hint)
  - Offline: Practice generation works locally using the canonical dataset and RNG; attempts and progress are queued for sync when online.
- Profile / progress dashboard (basic mastery view)
  - Offline: Progress reflects local state and queued changes; syncs with backend when connectivity is restored.

## Developer Tooling (Frontend)

- ESLint, Prettier for code quality
- Prettier for code formatting
- React Testing Library for basic unit/integration tests
- GitHub Actions for lint and build workflows

## Post-MVP Tooling (Frontend)

- Husky + lint-staged for pre-commit checks
- Storybook for component development and visual regression
- Cypress or Playwright for end-to-end tests
- Additional GitHub Actions for test and deployment workflows

## MVP Scope (Frontend)

- Implement responsive lesson list and lesson viewer
- Practice screen with RNG-backed prompts and immediate feedback
- Basic progress UI connected to backend progress endpoints
- Localization toggle (Arabic / English) with RTL support
- Offline-first: App works fully offline for practice and progress tracking; syncs local progress when connectivity is restored or user logs in.

## Next Steps (Frontend)

- Choose UI approach: Tailwind + headless components
- Scaffold Vite + React + TypeScript app with React Query and i18n
