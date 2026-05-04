# Day 03 — Design system, providers, observability

## Goal
shadcn/ui installed, app shell laid out, client-state providers mounted, Sentry + PostHog capturing events.

## Depends on
Day 01–02.

## Tasks
- [ ] `npx shadcn@latest init` — pick neutral palette, CSS variables, Tailwind v4.
- [ ] Add components on demand: `button`, `input`, `label`, `dropdown-menu`, `dialog`, `sheet`, `toast`, `avatar`, `card`, `tabs`, `badge`.
- [ ] Build `app/layout.tsx` with:
  - HTML lang, theme `<html className="...">`.
  - `<Providers>` client component (TanStack Query + Toaster).
  - Site header (logo, nav, sign-in/avatar dropdown).
  - Footer with placeholder links to legal pages (real pages on Day 05).
- [ ] `npm i zustand @tanstack/react-query @tanstack/react-query-devtools`.
- [ ] Create `app/providers.tsx` (client) with `QueryClientProvider` + Devtools (only in dev).
- [ ] Set up theme toggle (light/dark) using `next-themes` (`npm i next-themes`).
- [ ] `npm i @sentry/nextjs` and run `npx @sentry/wizard@latest -i nextjs`. Confirm Sentry captures a test error.
- [ ] `npm i posthog-js posthog-node`. Add a `PHProvider` in `app/providers.tsx`. Send `$pageview` on route change.
- [ ] Add a feature-flag bootstrap so we can later gate experiments via PostHog.

## Files created/changed
- `app/layout.tsx`, `app/providers.tsx`
- `components/ui/*` (shadcn)
- `components/site-header.tsx`, `components/site-footer.tsx`
- `lib/posthog.ts`, `sentry.{client,server,edge}.config.ts`

## Done when
- Theme toggle persists across reloads.
- React Query Devtools panel opens in dev.
- A thrown error in any route shows up in Sentry within ~30s.
- PostHog dashboard shows a `$pageview` after navigating two pages.
- Layout looks decent on mobile width (375px) — header collapses to a sheet menu.
