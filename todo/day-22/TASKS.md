# Day 22 — Candidate analytics dashboard

## Goal
`/dashboard` shows the candidate-facing widgets that drove the differentiator pitch.

## Depends on
Day 21.

## Tasks
- [ ] Add `recharts` (`npm i recharts`) for sparklines.
- [ ] Build dashboard widgets (Server Components, Prisma aggregate queries):
  - **Profile views** — total + last 7d + sparkline. Group by `date_trunc('day')`.
  - **Resume downloads** — count + table of `{company, downloaded_at}` (last 10).
  - **Application funnel** — count per `AppStatus` for this user.
  - **Profile completeness** — reuse Day 17 widget.
  - **Saved jobs** count + latest 3.
- [ ] Cache the dashboard payload in Upstash Redis with key `dashboard:{userId}` TTL 60s. Bust on relevant mutations (`recordEvent`, `applyToJob`, etc — too many sites; just rely on TTL for v1).
- [ ] "Who viewed you" — list last 20 distinct viewers with `{companyName, viewedAt}`.
- [ ] Skill demand — count of OPEN jobs whose tags overlap user's skills (top 5 skills).
- [ ] No salary benchmark widget for v1 (defer — not enough data on day 1).

## Files created/changed
- `app/(candidate)/dashboard/page.tsx` (rewrite from Day 04 stub)
- `components/dashboard/*.tsx` (one file per widget)
- `lib/dashboard.ts` (aggregate queries)

## Done when
- Dashboard renders in under 500ms with 1k events seeded.
- Sparkline shows the last 14 days with 0-bars where no events.
- Skills with no matching jobs render "0 matching jobs — try editing your skills".
