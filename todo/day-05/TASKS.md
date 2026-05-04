# Day 05 — Legal pages, terms acceptance, account settings, GDPR delete

## Goal
All legal compliance basics shipped so we can onboard real users without panicking.

## Depends on
Day 04.

## Tasks
- [ ] Create static MDX/TSX pages:
  - `app/legal/terms/page.tsx`
  - `app/legal/privacy/page.tsx`
  - `app/legal/cookies/page.tsx`
  - `app/legal/acceptable-use/page.tsx`
- [ ] Use a starter template (Termly/Iubenda free generators) and adapt; mark `// TODO: legal review` at top of each.
- [ ] Add `TERMS_VERSION` env var; default to today's date string.
- [ ] On `/sign-up` add an "I accept the Terms and Privacy Policy" checkbox. On submit, set `User.termsAcceptedAt` and `User.termsVersion`.
- [ ] Build `app/(account)/account/page.tsx` (profile), `app/(account)/account/security/page.tsx` (change password), `app/(account)/account/delete/page.tsx`.
- [ ] GDPR delete:
  - Server Action: set `User.deletedAt = now()`, `User.deleteScheduledFor = now() + 30d`, sign user out, send confirmation email.
  - Cancel-deletion link in the email + on the page (clears the fields if before grace expires).
- [ ] Add `cron/finalize-deletions/route.ts` Route Handler protected by `CRON_SECRET`. Hard-deletes users whose `deleteScheduledFor < now()`. Add to `vercel.json` as `0 1 * * *`.
- [ ] Footer now links to all four legal pages.

## Files created/changed
- `app/legal/**`
- `app/(account)/account/**`
- `app/api/cron/finalize-deletions/route.ts`
- `vercel.json` (cron section)
- Updated signup form + Server Action.

## Done when
- Signing up without checking the Terms box fails validation.
- Deleting your account schedules deletion + emails confirmation.
- Visiting cancel link within 30 days restores the account.
- Cron locally callable via `curl -H "Authorization: Bearer $CRON_SECRET" http://localhost:3000/api/cron/finalize-deletions` and removes test rows whose grace expired.
