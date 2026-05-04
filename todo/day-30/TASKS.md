# Day 30 — Production deploy & launch

## Goal
HireHub v1 is live at the production domain with monitoring, backups, and a verified end-to-end smoke test.

## Depends on
Day 29.

## Tasks
- [ ] Provision production secrets in Vercel (all env vars from `PLAN.md` §7) — separate Neon prod branch, separate Upstash, separate Cloudinary unsigned-preset for prod.
- [ ] Configure custom domain in Vercel; verify SSL.
- [ ] Update OAuth callback URLs in Google + GitHub to include the prod domain.
- [ ] Update Resend domain (DKIM + SPF DNS records) so emails don't go to spam.
- [ ] Update `NEXT_PUBLIC_APP_URL`, `EMAIL_FROM` to prod values.
- [ ] Run `prisma migrate deploy` against the prod Neon branch.
- [ ] Seed the prod DB with: 1 admin user (you), 5–10 demo jobs from real companies (with permission) so the homepage isn't empty on day 1.
- [ ] Smoke test on prod (use a fresh email + real phone):
  - [ ] Sign-up + verify.
  - [ ] Post a company + a job (as employer).
  - [ ] Apply (as candidate from another browser).
  - [ ] Kanban move triggers email.
  - [ ] Chat message arrives in real time.
  - [ ] Cron endpoints reachable with the secret only.
- [ ] Confirm Sentry has prod project and is receiving events (deliberate error in `/health` route, then revert).
- [ ] PostHog: set the prod project key, confirm `$pageview` events arriving.
- [ ] Document a one-page **Restore Drill** in `docs/runbooks/restore.md`: how to PITR-restore Neon to a moment in the past, swap the connection string, redeploy.
- [ ] Soft launch: post in 2–3 communities (Indie Hackers, your network). Watch Sentry + PostHog funnel for 24h; fix anything obviously broken.

## Files created/changed
- `docs/runbooks/restore.md`
- Vercel project settings (out-of-repo).
- Possibly tiny `app/health/route.ts` returning DB ping.

## Done when
- Anyone on the public internet can sign up and complete the apply flow without your help.
- Sentry shows zero unhandled errors in the first 2 hours after launch.
- You (or anyone) can locate and run the restore runbook in under 5 minutes.
- `PLAN.md` is updated with a "v1 shipped on YYYY-MM-DD" line at the top.
