# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

The "Acme" dashboard from the official [Next.js App Router Learn course](https://nextjs.org/learn). Work-in-progress through the course; some routes (e.g. `app/dashboard/invoices/page.tsx`) are still placeholder stubs and auth/Server Actions from later chapters are not yet implemented.

Uses **canary/RC versions**: Next.js 15 canary, React 19 RC. Partial Prerendering (PPR) is enabled (`experimental.ppr: 'incremental'` in `next.config.mjs`, with `export const experimental_ppr = true` opting routes in).

## Commands

Package manager is **pnpm** (note `node_modules/.pnpm`).

- `pnpm dev` — run the dev server (http://localhost:3000)
- `pnpm build` — production build
- `pnpm start` — serve the production build

There is no lint, test, or typecheck script configured. Type errors surface via `pnpm build` and the Next.js TS plugin in-editor.

## Database

Uses `@vercel/postgres`. All queries live in `app/lib/data.ts` as tagged-template `sql` calls returning `.rows`, typed via generics from `app/lib/definitions.ts` (e.g. `sql<Revenue>`...``). Requires Vercel Postgres connection env vars (`POSTGRES_*`) — typically a `.env` from `vercel env pull`. There is no live schema/migration in-repo; `app/lib/placeholder-data.ts` holds seed data and the course's seed route populates the DB.

## Architecture

App Router under `app/`:

- `app/lib/` — non-UI logic: `data.ts` (DB fetch functions), `definitions.ts` (hand-written domain types; the course note says an ORM would normally generate these), `utils.ts` (`formatCurrency`, `formatDateToLocal`, `generatePagination`).
- `app/ui/` — all React components, grouped by feature (`dashboard/`, `invoices/`, `customers/`). `fonts.ts` exports the `inter` and `lusitana` Google fonts; components apply `lusitana.className` for headings.
- `app/dashboard/` — dashboard routes. `layout.tsx` renders the persistent `SideNav`.

Key conventions:

- **Server Components by default**; `async` page/components `await` data functions from `app/lib/data.ts` directly. Amounts are stored in **cents** and converted/formatted with `formatCurrency` at the boundary.
- **Streaming via Suspense**: the overview page (`app/dashboard/(overview)/page.tsx`) wraps each data-fetching component (`CardWrapper`, `RevenueChart`, `LatestInvoices`) in `<Suspense>` with a matching skeleton from `app/ui/skeletons.tsx`. `fetchRevenue` has a deliberate 3s `setTimeout` to demonstrate streaming — keep this in mind when load feels slow.
- **Route groups**: `(overview)` is a route group (parenthesized, not in the URL) used so its `loading.tsx` applies only to the overview page, not sibling dashboard routes.
- **Path alias**: `@/*` maps to the repo root (e.g. `@/app/ui/...`), configured in `tsconfig.json`.