# Day 04 — RBAC proxy, role dashboards, email verification

> Next 16 note: what older Next versions called `middleware.ts` is now `proxy.ts` at the project root. Same functionality, new file name. Confirm against `node_modules/next/dist/docs/01-app/01-getting-started/16-proxy.md` before coding.

## Goal
Protected routes, per-role landing pages, and an email-verification flow that blocks writes until verified.

## Depends on
Day 02 (auth) and Day 03 (UI primitives).

## Tasks
- [ ] Create `proxy.ts` (Next 16's middleware replacement) at the project root that:
  - Redirects unauthenticated users hitting `/dashboard`, `/employer/*`, `/admin/*` to `/sign-in?next=...`.
  - 403s users hitting `/admin/*` without `role === ADMIN`.
  - 403s users hitting `/employer/*` without `role === EMPLOYER`.
- [ ] Add `requireRole(role)` and `requireUser()` to `lib/auth-helpers.ts`; use in Server Actions.
- [ ] Stub three dashboards:
  - `app/(candidate)/dashboard/page.tsx`
  - `app/(employer)/employer/jobs/page.tsx`
  - `app/(admin)/admin/page.tsx`
- [ ] `npm i resend react-email @react-email/components`.
- [ ] Create `lib/email.ts` with a `sendEmail({to, subject, react})` wrapper.
- [ ] Create `emails/VerifyEmail.tsx` (React Email template).
- [ ] On signup (Credentials) generate a `VerificationToken`, email link `/verify?token=...`.
- [ ] Build `/verify/page.tsx` Server Component that consumes the token and sets `User.emailVerified = now()`.
- [ ] Add a "Resend verification email" button on `/dashboard` while not verified.
- [ ] Block these Server Actions if `!emailVerified`: post job, apply to job, send message (those actions land later — add a `requireVerified()` helper now).

## Files created/changed
- `proxy.ts`
- `lib/auth-helpers.ts`
- `lib/email.ts`
- `emails/VerifyEmail.tsx`
- `app/(auth)/verify/page.tsx`
- `app/(auth)/actions.ts` (resend-token Server Action)
- Stub pages in `(candidate)`, `(employer)`, `(admin)`.

## Done when
- Hitting `/admin` while not admin shows a 403 page.
- New email/password signup gets a verification email via Resend (test inbox).
- Clicking the link flips `emailVerified` from `null` to a timestamp.
- `requireVerified()` throws if called by a non-verified user.
