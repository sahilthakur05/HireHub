# Day 19 — Employer applicant list

## Goal
Employers see all applicants for one job in a sortable table with quick filters by status.

## Depends on
Day 18.

## Tasks
- [ ] Build `/employer/jobs/[id]/applicants/page.tsx` — `requireRole(EMPLOYER)` + ownership check (`Job.companyId === user.company.id`).
- [ ] Query applications `WHERE jobId = ?` joined with candidate name, email, resume snapshot.
- [ ] Table columns: avatar/name, applied-at, status badge, headline, location, skills (top 3), actions (View, Download Resume, Change Status).
- [ ] Status filter chips: All / Applied / Reviewing / Interview / Offer / Rejected.
- [ ] Sort: newest first (default), oldest first.
- [ ] "Download Resume" creates a 5-min signed URL via existing helper.
- [ ] Click a row → drawer with full candidate profile snapshot + cover letter.
- [ ] Hard-code limit: an EMPLOYER can download max 50 resumes/day (Upstash counter); over the limit show a Premium-coming-soon banner.

## Files created/changed
- `app/(employer)/employer/jobs/[id]/applicants/page.tsx`
- `app/(employer)/employer/jobs/[id]/applicants/applicant-drawer.tsx`
- `lib/limits.ts` (download counter using Upstash)

## Done when
- An employer cannot view applicants of a job they don't own.
- The 51st resume download in 24h is blocked with a clear message.
- The drawer shows the resume + cover letter without a navigation.
