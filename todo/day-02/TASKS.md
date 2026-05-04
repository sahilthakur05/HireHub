# Day 02 — Auth.js v5 (Google + GitHub + Credentials)

## Goal
Sign-in works for OAuth and email/password. Sessions issue JWTs and `User.role` is on the session.

## Depends on
Day 01 (Prisma + DB).

## Tasks
- [ ] `npm i next-auth@beta @auth/prisma-adapter bcryptjs` and `npm i -D @types/bcryptjs`.
- [ ] Create OAuth apps: Google Cloud Console + GitHub Developer Settings. Add callback URLs for localhost + future prod domain.
- [ ] Add `AUTH_SECRET`, `AUTH_GOOGLE_ID/SECRET`, `AUTH_GITHUB_ID/SECRET` to `.env.local`.
- [ ] Create `auth.ts` at repo root with Auth.js v5 config: Prisma adapter, JWT strategy, Google + GitHub + Credentials providers.
- [ ] Credentials provider: bcrypt-hash on signup, compare on signin; reject if `emailVerified == null` once verification ships (Day 04).
- [ ] In `auth.ts` `callbacks.jwt` and `callbacks.session`, attach `user.id`, `user.role`, `user.plan` to the session.
- [ ] Create `app/api/auth/[...nextauth]/route.ts` re-exporting the Auth.js handlers.
- [ ] Build minimal `/sign-in` and `/sign-up` pages (shadcn not yet — plain Tailwind is fine).
- [ ] Sign-up server action: validate with zod, hash password, create `User` with default `role = CANDIDATE`, send back redirect to `/`.
- [ ] Add a "Sign in with Google / GitHub" button group on `/sign-in`.

## Files created/changed
- `auth.ts`
- `app/api/auth/[...nextauth]/route.ts`
- `app/(auth)/sign-in/page.tsx`
- `app/(auth)/sign-up/page.tsx`
- `app/(auth)/actions.ts` (Server Action for credentials signup)
- `lib/auth-helpers.ts` — `auth()` re-export, `requireUser()`, `requireRole()`.

## Done when
- Sign in via Google works locally and creates a `User` + `Account` row.
- Sign in via GitHub works.
- Sign up with email/password works; password is bcrypt-hashed in DB.
- `auth()` from a Server Component returns the session with `role`.
- Logout works.
