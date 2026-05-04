# Day 26 — Rate limiting, abuse hardening, weekly digest cron

## Goal
Production-grade safety: rate limits on every write, abuse heuristics, weekly job-match email.

## Depends on
Day 25 (Upstash already in place).

## Tasks
- [ ] In `lib/ratelimit.ts` create distinct sliding-window limiters:
  - `auth` — 5 / 15min / IP (apply to sign-in + password reset).
  - `signup` — 3 / hour / IP.
  - `apply` — 10 / hour / user.
  - `sendMessage` — 30 / min / user, 500 / day / user.
  - `track` — 60 / min / IP (set Day 21).
- [ ] Wrap relevant Server Actions / Route Handlers with the limiter and return a 429-equivalent error.
- [ ] Block these signup heuristics: more than 3 accounts from one IP/day (manual review queue), disposable email domains (use `disposable-email-domains` package — `npm i disposable-email-domains`).
- [ ] Build `app/api/cron/weekly-digest/route.ts`:
  - For each candidate with ≥1 skill, find OPEN jobs whose tags overlap created in the last 7 days, top 10.
  - Render `WeeklyDigest.tsx` Resend email; skip if zero matches.
  - Add `Mon 09:00` (`0 9 * * 1`) to `vercel.json`.
- [ ] Add unsubscribe link → `/unsubscribe?token=...` (token = HMAC of `userId+digest` so we don't need a table).

## Files created/changed
- `lib/ratelimit.ts` (expanded)
- Server Actions wrapped throughout the codebase.
- `app/api/cron/weekly-digest/route.ts`
- `app/(marketing)/unsubscribe/page.tsx`
- `emails/WeeklyDigest.tsx`

## Done when
- Hammering `/sign-in` with bad creds is blocked after 5 attempts in 15 minutes.
- Signing up with `you+test@mailinator.com` is rejected.
- Cron run produces an email for at least one seeded user with skill matches.
- Unsubscribe link sets `User.weeklyDigestEnabled = false` and short-circuits future digests.
