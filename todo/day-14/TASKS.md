# Day 14 — Job detail page + similar jobs + JSON-LD

## Goal
`/jobs/[slug]` is fast, SEO-rich (Google Jobs eligible), and links to similar postings.

## Depends on
Day 13.

## Tasks
- [ ] Build `app/(marketing)/jobs/[slug]/page.tsx` Server Component:
  - Fetch job + company (with verified badge + logo).
  - Render description with `<RenderHtml />` from Day 08.
  - Right rail: salary, location/remote, employment type, experience, posted/expires.
  - "Apply" button → `/jobs/[slug]/apply` (built Day 18).
  - "Save" button (Day 18 too — placeholder for now).
- [ ] `generateMetadata({ params })` — title/description from job, OG image via Next `ImageResponse` (`app/(marketing)/jobs/[slug]/opengraph-image.tsx`).
- [ ] Add `JobPosting` JSON-LD as a `<script type="application/ld+json">` per [schema.org/JobPosting](https://schema.org/JobPosting). Include: `title`, `description` (text-stripped HTML), `hiringOrganization`, `datePosted`, `validThrough`, `employmentType`, `jobLocation`, `baseSalary`.
- [ ] Similar jobs section: query top 5 by tag overlap, exclude current job. Use a Postgres array intersection.
- [ ] Increment `Job.viewsCount` (add column on schema, default 0) — debounce per session via cookie to avoid refresh inflation.

## Files created/changed
- `app/(marketing)/jobs/[slug]/page.tsx`
- `app/(marketing)/jobs/[slug]/opengraph-image.tsx`
- `lib/jobs/similar.ts`
- Prisma migration adding `Job.viewsCount Int @default(0)`.

## Done when
- Page passes Google's [Rich Results Test](https://search.google.com/test/rich-results) for `JobPosting`.
- OG image renders correctly when sharing on Slack/Twitter.
- Similar jobs section shows 3–5 relevant results.
- View count increments once per session per job.
