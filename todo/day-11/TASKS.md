# Day 11 — Public /jobs list page (no search yet)

## Goal
Anyone can browse open jobs at `/jobs` with pagination, server-rendered.

## Depends on
Day 09.

## Tasks
- [ ] Build `app/(marketing)/jobs/page.tsx` — Server Component that reads `?page=1&limit=20`.
- [ ] Query: `WHERE status = 'OPEN' ORDER BY "createdAt" DESC LIMIT N OFFSET M`. Fetch company `select { name, slug, logoUrl, verifiedAt }` joined.
- [ ] Build `<JobCard />` component: title, company (with verified badge), location, remote tag, salary range, posted-X-ago, top 3 tags.
- [ ] Pagination component using `<Link>` with `?page=N` (no client JS needed).
- [ ] Skeleton loading states with `loading.tsx`.
- [ ] Empty state: "No open jobs yet — check back soon."
- [ ] Header nav links to `/jobs` for everyone (logged-in or not).
- [ ] Add `metadata` export with title + description.

## Files created/changed
- `app/(marketing)/jobs/page.tsx`
- `app/(marketing)/jobs/loading.tsx`
- `components/job-card.tsx`
- `components/pagination.tsx`

## Done when
- Seeded with ~30 jobs (write a `prisma/seed.ts` if not already), pagination works at 20/page.
- Cards render correctly on mobile (375px).
- Closed/draft jobs do not appear.
