# Day 29 — CI pipeline + tests + Lighthouse gate

## Goal
GitHub Actions runs typecheck, lint, vitest, playwright, axe, Lighthouse on every PR. Required to pass before merge.

## Depends on
Day 28.

## Tasks
- [ ] `npm i -D vitest @vitest/ui` and add `vitest.config.ts`. Cover at minimum: `lib/sanitize`, `lib/profile.computeCompleteness`, `lib/jobs/search`, `lib/chat` Server Actions, `lib/verification`.
- [ ] `npm i -D @playwright/test` (already from Day 28). Add e2e covering: sign-up → verify (use a `/test-only` mailbox view in dev), post a job, apply, kanban move, send chat message.
- [ ] Create `.github/workflows/ci.yml` running on `pull_request`:
  - `typecheck` — `tsc --noEmit`.
  - `lint` — `npm run lint`.
  - `unit` — `vitest run`.
  - `e2e` — `playwright test` (with Postgres + Cloudinary mock or test creds — gate behind a `secrets.E2E_*`).
  - `lighthouse` — `npx unlighthouse-ci --site http://localhost:3000` against the deployed Vercel preview, fail if mobile score < 90 on `/`, `/jobs`, `/jobs/[slug]`.
- [ ] Add branch protection on `main` requiring all checks.
- [ ] Add Dependabot (`.github/dependabot.yml`) — weekly, npm only.

## Files created/changed
- `vitest.config.ts`
- `playwright.config.ts`
- `tests/**`
- `.github/workflows/ci.yml`
- `.github/dependabot.yml`

## Done when
- Opening a PR shows 5 green checks before merge is allowed.
- A deliberate type error blocks merge.
- A deliberate axe regression in a critical page blocks merge.
