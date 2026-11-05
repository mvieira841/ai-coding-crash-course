# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Cadence: a course platform (a mini Udemy) built with React Router (SSR, framework mode), TypeScript, SQLite and Drizzle. It's the exercise repo for the AI Coding Crash Course.

## Commands

- `npm run dev` — dev server at `http://localhost:5173`.
- `npm run typecheck` — runs `react-router typegen` and then `tsc`. Run it after adding a route so the `./+types/<route>` imports resolve.
- `npm run test` — Vitest, all tests. Single file: `npx vitest run app/services/courseService.test.ts`. Single test: add `-t "<test name>"`.
- Formatting is Prettier (`.prettierrc`); there is no lint script.
- `npm run reset` and `npm run cherry-pick` rewrite the repo to a lesson checkpoint from upstream. Only the human runs them.

## Architecture

- **Routes** are registered explicitly in `app/routes.ts`. A new file in `app/routes/` stays unreachable until it's added there. Each route's `loader`/`action` calls into services and never queries the database directly.
- **Services** (`app/services/*Service.ts`) are synchronous Drizzle functions (better-sqlite3) that take positional parameters, which is the project convention. `~/` is an alias for `app/`.
- **`app/db/index.ts`** opens the connection lazily behind a Proxy. Keep it side-effect free: a module-scope connection drags better-sqlite3 into the client bundle. Server-only modules in `app/lib/` use the `.server.ts` suffix.
- **Identity** is a cookie session holding `userId` (`app/lib/session.ts`). There is no real auth. The DevUI panel switches users through `/api/switch-user` and sets a fake country for PPP (purchasing power parity) pricing.
- **Access** for a course is resolved in one place, `app/lib/access.server.ts`. `getAccess` never throws, so a page can render a paywall preview. The `require*` wrappers throw 401/403. Call one in every action, because loader guards don't protect actions.
- **Actions** with several operations dispatch on a form field `intent`, validated by a Zod `discriminatedUnion("intent", ...)` through `parseFormData` (`app/lib/validation.ts`), which returns a field-error map. `parseParams` validates route params and throws 400.
- **Tests** sit beside the service they cover. Each one mocks `~/db` with a getter that returns `createTestDb()` from `app/test/setup.ts`, an in-memory SQLite database migrated from `drizzle/`, recreated in `beforeEach` and filled by `seedBaseData`.

## Outside the app

- `request-logger/` is a standalone proxy (`npm run request-logger`) that logs what a coding agent sends to the model. It has its own tests and no dependency on the app.
- `.sandcastle/` runs Claude Code in Docker over a PRD and a plan.
- `_internal/feature-ideas.md` is a backlog of feature ideas.
