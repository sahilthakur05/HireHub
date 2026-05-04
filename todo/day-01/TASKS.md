# Day 01 — Project foundation & database

## Goal
Get a working Next 16 dev environment with Prisma connected to Neon Postgres and the initial schema migrated.

## Depends on
Nothing — this is day one.

## Tasks
- [ ] Sign up for Neon, create a project + a `dev` branch DB.
- [ ] Sign up for Upstash, Cloudinary, Resend, Sentry, PostHog (just create accounts; wiring on later days).
- [ ] Add `.env.local` with `DATABASE_URL`, `DIRECT_URL` (see `PLAN.md` §7).
- [ ] Add `.env.example` mirroring `.env.local` with empty values; commit it.
- [ ] `npm i prisma @prisma/client zod` and `npx prisma init`.
- [ ] In `prisma/schema.prisma` add models from `PLAN.md` §4 in this order: `User`, `CandidateProfile`, `Resume`, `Company`, `CompanyMember`, `Job`, `Application`, `SavedJob`, `AuditLog`, plus all required enums.
- [ ] Skip these tables for now (later days): `Conversation`, `Message`, `Notification`, `SavedSearch`, `ProfileEvent`, `DataExportRequest`.
- [ ] `npx prisma migrate dev --name init`.
- [ ] Create `lib/db.ts` exporting a singleton `PrismaClient` (Next dev hot-reload safe).
- [ ] Create `lib/env.ts` with `zod` parsing of `process.env` so missing vars fail loudly at boot.
- [ ] Add `.gitignore` entries for `.env*.local`, `prisma/migrations/dev.db*` (no SQLite to commit).
- [ ] Verify `npm run dev` boots and `npx prisma studio` shows empty tables.

## Files created/changed
- `prisma/schema.prisma`
- `prisma/migrations/<ts>_init/`
- `lib/db.ts`
- `lib/env.ts`
- `.env.local`, `.env.example`

## Done when
- `npx prisma migrate status` says "Database schema is up to date".
- `lib/env.ts` throws on a missing required var (test by deleting one).
- App boots on `localhost:3000` showing the default Next page.
