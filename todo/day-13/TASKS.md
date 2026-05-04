# Day 13 — Filters + sort, URL-driven state

## Goal
Users can filter by remote type, employment type, experience, salary, tags. State lives in the URL — shareable links.

## Depends on
Day 12.

## Tasks
- [ ] Build `<JobFilters />` sidebar (desktop) + drawer (mobile via `<Sheet>`):
  - Remote: REMOTE / HYBRID / ONSITE (multi-select).
  - Employment type checkboxes.
  - Experience checkboxes.
  - Salary range slider (min) — single threshold is enough for v1.
  - Tag autocomplete (top 50 tags by frequency, server-rendered).
  - Sort: relevance (default if `q`) | newest | salary-desc.
- [ ] Form is GET, all state goes into search params: `?q=...&remote=REMOTE,HYBRID&type=FULL_TIME&minSalary=80000&tags=react,typescript&sort=newest&page=1`.
- [ ] Update `lib/jobs/search.ts` to accept `JobFilters` and compose the WHERE clause safely (use Prisma `where` for predicates that don't need raw SQL; combine with raw FTS via `$queryRaw` carefully — or move to Prisma's `Prisma.sql` template tags).
- [ ] On submit, reset `page=1`.
- [ ] Show active filter chips above results with "X" to remove individually + "Clear all".

## Files created/changed
- `components/job-filters.tsx`
- `components/active-filter-chips.tsx`
- Updated `lib/jobs/search.ts` and `/jobs` page.

## Done when
- Selecting "Remote + Full-time + React tag, sort: newest" produces a URL you can copy/paste and reload to identical results.
- Filters render as a drawer on mobile width.
- Clearing one filter at a time works.
