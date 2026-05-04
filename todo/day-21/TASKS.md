# Day 21 — ProfileEvent tracking + endpoints

## Goal
We collect signals (profile views, resume downloads) without slowing down request paths.

## Depends on
Day 16, Day 19 (resume downloads + applicant views are the main fire points).

## Tasks
- [ ] Add `ProfileEvent` model + `EventType` and `ActorRole` enums (see `PLAN.md` §3.3). Migrate.
- [ ] Add a public-ish `/candidates/[id]/page.tsx` — only employers logged-in can view (recruiter view of a candidate). 404 for everyone else for v1.
- [ ] On render of `/candidates/[id]`, fire `recordEvent({ type: PROFILE_VIEW, candidateId, actorUserId })`. Dedupe via Upstash `SETNX key=view:{candidateId}:{actorId}:{yyyyMMdd}` with 24h TTL.
- [ ] In Day 19's resume download action, log `RESUME_DOWNLOAD` with `actorCompanyId`.
- [ ] Build `/api/track/route.ts` (POST) for client-side events (`CONTACT_CLICK` etc) — requires `requireUser()` or rejects.
- [ ] Rate-limit `/api/track` via `@upstash/ratelimit` to 60 req/min/IP.
- [ ] Drop bot UAs (regex `bot|crawl|spider`) from being recorded.
- [ ] Skip self-views and admin views.

## Files created/changed
- `prisma/schema.prisma` (+ migration)
- `lib/events.ts` (the `recordEvent` helper)
- `app/(employer)/candidates/[id]/page.tsx`
- `app/api/track/route.ts`
- `lib/ratelimit.ts` (Upstash setup — first usage)

## Done when
- Loading a candidate page from a different signed-in user inserts one `ProfileEvent` row, second visit same day inserts none.
- Curl-bombing `/api/track` from one IP is rate-limited at 60 req/min.
- A user-agent of "Googlebot" produces no row.
