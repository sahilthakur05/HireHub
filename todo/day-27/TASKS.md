# Day 27 — Admin panel + moderation + audit log

## Goal
Minimal admin UI to suspend users, moderate jobs, and review reports. Audit-log every admin action.

## Depends on
All earlier days (admin needs data to moderate).

## Tasks
- [ ] Build `/admin/page.tsx` dashboard with totals: users, employers, jobs OPEN, applications today, signups today.
- [ ] Build `/admin/users/page.tsx` — table with search by email, role filter, suspend / unsuspend / delete actions.
- [ ] Add `User.suspendedAt` field (migration). Suspended users are signed out next request via `proxy.ts` check.
- [ ] Build `/admin/jobs/page.tsx` — table of jobs with status filter; "Force close", "Feature toggle" (no-op for v1 since no Premium yet — add field anyway: `Job.featured Boolean @default(false)`).
- [ ] Build `/admin/reports/page.tsx` — placeholder list (no Report model yet for v1; create one if there's time, otherwise stub the page with "No reports yet" empty state).
- [ ] Wrap every admin action with an `auditLog()` helper that writes to `AuditLog` with `actorId`, action name, target, IP, UA, JSON meta.
- [ ] Build `/admin/audit/page.tsx` showing the last 100 entries.

## Files created/changed
- `app/(admin)/admin/page.tsx`
- `app/(admin)/admin/users/page.tsx`
- `app/(admin)/admin/jobs/page.tsx`
- `app/(admin)/admin/reports/page.tsx`
- `app/(admin)/admin/audit/page.tsx`
- `lib/audit.ts`

## Done when
- Suspending yourself in another tab signs you out on the next click.
- Force-closing a job hides it publicly; reverting reopens it.
- Every admin click produces a row in `/admin/audit`.
