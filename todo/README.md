# HireHub — 30-Day v1 Build Schedule

This folder breaks the v1 scope (from `PLAN.md`) into 30 working days. Each day is a single-day chunk for one focused engineer; if you have a teammate, the parallel-friendly days are noted at the bottom.

## v1 scope (from PLAN.md, trimmed)

Auth + RBAC · Email verification · Company profile + domain-email verification · Job posting (Tiptap) · Public search w/ Postgres FTS · Apply flow + Cloudinary resumes · Employer kanban · Candidate analytics dashboard · Text-only chat · Legal pages + GDPR delete · Sentry + PostHog · Mobile-responsive · CI gates.

**Deferred to v1.1+** (do NOT build in these 30 days): TOTP 2FA, DNS-TXT verification, PWA, i18n, instant alerts, notifications inbox UI, resume parser, chat attachments, status page, Premium tier, GDPR export, audit-log UI.

## How to use

1. Open the lowest-numbered `day-XX/TASKS.md` not yet done.
2. Work top-to-bottom through the checklist. Don't skip ahead — later days assume earlier ones shipped.
3. At end of day, commit with message `day-XX: <summary>`.
4. If a day overruns, push the unfinished items to the next day's TASKS.md instead of working late — the schedule has slack baked in but not unlimited.

## Week map

| Week | Days | Theme |
|---|---|---|
| 1 | 01–05 | Foundations: project, DB, auth, RBAC, legal scaffold |
| 2 | 06–10 | Employer: company onboarding, verification, job posting |
| 3 | 11–15 | Public discovery: list, search/filters, detail page, SEO |
| 4 | 16–20 | Apply flow + employer applicant pipeline |
| 5 | 21–25 | Candidate analytics dashboard + chat |
| 6 | 26–30 | Polish: rate limits, crons, GDPR, mobile, CI, launch |

## Parallel tracks (if you have a 2nd engineer)

Once Day 5 is done, the following pairs can run in parallel:
- Days 06–10 (employer) ‖ Days 11–15 (public discovery) — both depend only on Day 1–5.
- Days 21–22 (analytics) ‖ Days 23–25 (chat) — independent feature surfaces.

## Definition of Done (every day)

- [ ] All tasks for the day checked off.
- [ ] `npm run lint` clean.
- [ ] `tsc --noEmit` clean.
- [ ] Vercel preview deploy works.
- [ ] Manual smoke-test of the day's main flow on desktop + mobile viewport.
- [ ] Committed to git with a `day-XX:` prefix.

## Reference

Full feature spec, schemas, env vars, and architectural decisions live in `../PLAN.md`. This folder is the executable plan; `PLAN.md` is the source-of-truth design doc.
