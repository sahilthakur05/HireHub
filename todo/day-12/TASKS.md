# Day 12 — Postgres full-text search

## Goal
Keyword search on `/jobs?q=...` ranks results by relevance (title > tags > description).

## Depends on
Day 08 (`search_vector` column exists), Day 11.

## Tasks
- [ ] Verify the generated `search_vector` column from Day 08 covers `title (A) + tags (B) + descriptionHtml (C)`.
- [ ] Add a Prisma raw query helper in `lib/jobs/search.ts`:
  ```ts
  prisma.$queryRaw`
    SELECT id, ts_rank(search_vector, plainto_tsquery('english', ${q})) AS rank
    FROM "Job"
    WHERE status = 'OPEN'
      AND search_vector @@ plainto_tsquery('english', ${q})
    ORDER BY rank DESC, "createdAt" DESC
    LIMIT ${limit} OFFSET ${offset}
  `
  ```
- [ ] Update `/jobs` page to use search when `q` is present, plain list otherwise.
- [ ] Add a `<SearchInput />` form component (GET form posting to `/jobs`) at top of the page.
- [ ] Highlight matched terms in the description snippet using `ts_headline` (separate query, optional).
- [ ] Show "X results for '<query>'" header.

## Files created/changed
- `lib/jobs/search.ts`
- `components/search-input.tsx`
- Updated `app/(marketing)/jobs/page.tsx`.

## Done when
- `q=react senior` returns `Senior React Developer` higher than `Junior Backend Engineer (mentions React)`.
- Empty `q` falls back to chronological list.
- Special characters in `q` don't break the query (use `plainto_tsquery` not `to_tsquery`).
