# Day 18 — Apply flow + saved jobs + email notifications

## Goal
Candidates apply to a job by selecting an existing resume + writing a cover letter. Employer gets notified.

## Depends on
Day 14 (job detail), Day 17 (profile).

## Tasks
- [ ] Build `/jobs/[slug]/apply/page.tsx` — `requireUser()`, `requireRole(CANDIDATE)`, `requireVerified()`.
- [ ] Form: resume select (from user's resumes; if zero, prompt upload first), cover letter textarea (max 4000 chars, optional).
- [ ] Server Action `applyToJob({ jobId, resumeId, coverLetter })`:
  - Enforce unique constraint `@@unique([jobId, userId])`.
  - Snapshot `resumeUrl` to the application row (so future resume edits don't change historic applications).
  - Send Resend email `NewApplication.tsx` to the company owner.
- [ ] Build `/applications/page.tsx` — candidate's applications list with status, timestamps.
- [ ] Implement saved jobs: heart icon on `<JobCard />` and detail page — Server Action toggles `SavedJob`. Build `/saved/page.tsx` listing them.
- [ ] Hard-code v1 cap of 25 saved jobs per candidate (deferred Premium would lift this).
- [ ] On the job detail page, if user has already applied, replace "Apply" with "Applied on <date> · View application".

## Files created/changed
- `app/(marketing)/jobs/[slug]/apply/page.tsx`
- `app/(marketing)/jobs/[slug]/apply/actions.ts`
- `app/(candidate)/applications/page.tsx`
- `app/(candidate)/saved/page.tsx`
- `emails/NewApplication.tsx`
- Updated `JobCard` and detail page with save / applied states.

## Done when
- Applying twice to the same job fails with a friendly "You've already applied" toast.
- Employer receives an email within ~30s with applicant name + link to applicants view.
- Saved jobs persist across sessions; 26th save is rejected.
