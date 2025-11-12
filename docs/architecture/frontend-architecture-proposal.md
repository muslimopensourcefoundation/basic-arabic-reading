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

- Framework: React + Vite + TypeScript
- Routing: React Router or TanStack Router
- Server state: React Query (caching, background refetching, optimistic updates)
- Local state: Zustand (small) or Redux Toolkit (structured) where needed
- UI: Tailwind CSS + Headless UI or Chakra UI / Radix for accessible primitives
- Forms & validation: React Hook Form + Zod for schema-driven validation
- i18n: `react-i18next` with JSON translation files and RTL switching
- Accessibility tools: axe-core (eslint-plugin-jsx-a11y) during dev and CI

## Accessibility and i18n (Frontend-specific)

- Keyboard navigation: Add logical tab order, focus-visible outlines, and keyboard shortcuts for practice actions.
- Fonts: include high-quality Arabic fonts such as `Noto Sans Arabic` and `Scheherazade`, provide font-size controls, and high-contrast themes.
- RTL layout: Enable `dir="rtl"` on root when Arabic is active; use CSS logical properties and component-library RTL support.
- Screen readers: Use semantic HTML, ARIA where needed, and provide transliterations for glyphs.

## UI Surface and Pages (MVP)

- Landing / Lessons list (filter by level)
- Lesson page with content blocks (text, audio, exercises)
- Practice UI for generated prompts (display prompt, input, submit, hint)
- Profile / progress dashboard (basic mastery view)

## Developer Tooling (Frontend)

- ESLint, Prettier, Husky + lint-staged for pre-commit checks
- Storybook for component development and visual regression
- Cypress or Playwright for end-to-end tests
- GitHub Actions for lint, test, build, and deployment workflows

## MVP Scope (Frontend)

- Implement responsive lesson list and lesson viewer
- Practice screen with RNG-backed prompts and immediate feedback
- Basic progress UI connected to backend progress endpoints
- Localization toggle (Arabic / English) with RTL support

## Next Steps (Frontend)

- Choose UI approach: Tailwind + headless components or Chakra UI
- Scaffold Vite + React + TypeScript app with React Query and i18n
