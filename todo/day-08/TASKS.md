# Day 08 — Job model + Tiptap editor + sanitization

## Goal
A reusable `<JobEditor />` component built on Tiptap that produces sanitized HTML safe to render.

## Depends on
Day 06.

## Tasks
- [ ] Confirm `Job` model + enums (`JobStatus`, `RemoteType`, `EmploymentType`, `Experience`) are in schema (added on Day 01); if missing, migrate now.
- [ ] Run a SQL migration (`prisma migrate dev --create-only --name job-search-vector` then edit) to add the generated `tsvector` column from `PLAN.md` §4. Re-run migrate.
- [ ] `npm i @tiptap/react @tiptap/starter-kit @tiptap/extension-link @tiptap/extension-placeholder`.
- [ ] Build `components/editor/job-editor.tsx` (client) — toolbar buttons: bold, italic, bullets, numbers, link, H2/H3.
- [ ] `npm i sanitize-html` and `npm i -D @types/sanitize-html`.
- [ ] Build `lib/sanitize.ts` exporting `sanitizeJobHtml(html)` — allowlist exactly the tags Tiptap emits, strip all attributes except `href` (with `rel="noopener nofollow"` enforced) on `<a>`.
- [ ] Build `components/editor/render-html.tsx` — server component that takes raw HTML, sanitizes once, sets via `dangerouslySetInnerHTML` on a styled `prose` div.

## Files created/changed
- `components/editor/job-editor.tsx`
- `components/editor/render-html.tsx`
- `lib/sanitize.ts`
- New Prisma migration for `search_vector` GIN index.

## Done when
- Editor renders, toolbar buttons work, paste from Word produces clean HTML.
- `sanitizeJobHtml('<script>x</script><p>ok</p>')` → `<p>ok</p>`.
- `RenderHtml` displays formatted output on a test page.
- Postgres has the GIN index (`\d "Job"` in psql shows `search_vector` and the index).
