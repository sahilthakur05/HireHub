# Day 17 — Candidate profile page

## Goal
Candidates can edit their `CandidateProfile` (headline, bio, location, skills, links). Profile completeness % surfaced.

## Depends on
Day 16.

## Tasks
- [ ] Build `/profile/page.tsx` — single edit form (no separate view/edit modes for v1).
- [ ] Fields: headline (string, max 120), bio (textarea, max 2000), location, skills (multi-input chips), portfolio links (array of `{label, url}`).
- [ ] Server Action `updateCandidateProfile` with zod.
- [ ] On first visit, if no `CandidateProfile` row, create one server-side.
- [ ] Add a small `<ProfileCompleteness />` widget on the dashboard:
  - Score = % of these checks: name, headline, bio, location, ≥3 skills, ≥1 resume, email verified.
  - Click expands a checklist with links to fix each missing item.
- [ ] Sanitize bio (allow plain text + line breaks; no HTML).
- [ ] Skills field: free-form for now (no taxonomy — that's v1.1). Lowercase + trim on save.

## Files created/changed
- `app/(candidate)/profile/page.tsx`
- `app/(candidate)/profile/actions.ts`
- `components/profile-completeness.tsx`
- `lib/profile.ts` (`computeCompleteness(user)`)

## Done when
- Filling in the form persists; reloading shows the same values.
- Completeness % updates live as fields are filled.
- Skill `"  React.js  "` is stored as `"react.js"`.
