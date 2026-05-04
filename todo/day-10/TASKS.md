# Day 10 — Employer job dashboard polish + expire-jobs cron

## Goal
The employer's `/employer/jobs` view shows useful stats and an `expire-jobs` cron auto-closes expired postings.

## Depends on
Day 09.

## Tasks
- [ ] Add per-job stats to the listing: views (counter on `Job.viewsCount` increment via tracker — placeholder until Day 22), applicant count (`COUNT(applications)` join). Use a single Prisma query with `_count`.
- [ ] Add filters on the page: All / Open / Draft / Closed.
- [ ] Add quick actions: Repost (clones row with new `expiresAt`), Duplicate, Close.
- [ ] Build `app/api/cron/expire-jobs/route.ts`:
  - Header `Authorization: Bearer ${CRON_SECRET}`.
  - `UPDATE "Job" SET status = 'CLOSED' WHERE status = 'OPEN' AND "expiresAt" < now()`.
  - Returns the count.
- [ ] Add to `vercel.json`: `0 * * * *` for `expire-jobs`.
- [ ] Send a Resend email to the employer when their job is auto-closed (build template `JobExpired.tsx`).

## Files created/changed
- `app/(employer)/employer/jobs/page.tsx` (rewrite with stats + filters)
- `app/(employer)/employer/jobs/actions.ts` (repost/duplicate)
- `app/api/cron/expire-jobs/route.ts`
- `emails/JobExpired.tsx`
- `vercel.json`

## Done when
- Manually setting a job's `expiresAt` to yesterday + hitting the cron endpoint locally closes it and emails the employer.
- Repost duplicates the row with a new slug, fresh `expiresAt`, status `OPEN`.
- Filters work without a full page reload (use server-side search params).
