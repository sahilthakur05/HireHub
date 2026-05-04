# Day 15 — Sitemap, robots, home page, company directory

## Goal
Crawlers can discover every active job; the home page sells the product.

## Depends on
Day 14.

## Tasks
- [ ] `app/sitemap.ts` — return all open jobs + company pages + static legal pages. Use `lastModified: job.updatedAt`.
- [ ] `app/robots.ts` — allow all, point at sitemap, disallow `/api/`, `/admin/`, `/account/`, `/employer/`.
- [ ] Build the home page `app/(marketing)/page.tsx`:
  - Hero with search bar + CTA (sign up as candidate / post a job).
  - "Latest jobs" — last 6.
  - Featured categories (top tags by job count, links to filtered `/jobs?tags=...`).
  - "How it works" section.
- [ ] Build `/companies/[slug]/page.tsx` — public company page: logo, banner placeholder, description, list of open jobs.
- [ ] Add `/companies` directory page (alphabetical, paginated).

## Files created/changed
- `app/sitemap.ts`, `app/robots.ts`
- `app/(marketing)/page.tsx` (rewrite default Next page)
- `app/(marketing)/companies/page.tsx`
- `app/(marketing)/companies/[slug]/page.tsx`

## Done when
- `/sitemap.xml` lists every open job with absolute URLs.
- Home page Lighthouse mobile score ≥ 90.
- Company pages render correctly + show only open jobs.
- robots.txt is valid (test via Google's robots.txt tester).
