# Day 16 — Cloudinary signed-upload route + resume management

## Goal
Candidates can upload, list, rename, and delete PDF resumes. Storage uses signed URLs scoped per user.

## Depends on
Day 04 (auth) — does NOT depend on Day 06–15.

## Tasks
- [ ] In Cloudinary dashboard create an upload preset for resumes: signed, restrict to PDFs, max 5 MB, folder `resumes/{userId}`.
- [ ] Build `app/api/upload/resume/route.ts` (POST) — `requireUser()`, generate Cloudinary signed-upload params, return them. Client uploads directly to Cloudinary.
- [ ] Build `/resumes/page.tsx` — Server Component listing the user's resumes with download / rename / delete buttons.
- [ ] Client component `<ResumeUploader />` calls the signed endpoint, then uploads via `fetch` to Cloudinary's API. On success calls Server Action `addResume({ url, filename })`.
- [ ] Server Action `deleteResume(id)` — verify ownership, delete from Cloudinary (`cloudinary.uploader.destroy`), then DB row.
- [ ] For downloads, generate a signed URL with 5-minute expiry from a Server Action (don't expose the raw `secure_url`).
- [ ] Enforce v1 limit: max **3 resumes** per candidate (Premium would unlock more later — for v1 just hard-code 3, since Premium is deferred).

## Files created/changed
- `app/api/upload/resume/route.ts`
- `app/(candidate)/resumes/page.tsx`
- `app/(candidate)/resumes/actions.ts`
- `components/resume-uploader.tsx`
- `lib/cloudinary.ts` (add `signDownload`, `destroy` helpers)

## Done when
- Uploading a 6 MB PDF is rejected client + server side.
- Uploading a 1 MB PDF creates a `Resume` row and the file appears under `resumes/{userId}/...` in Cloudinary.
- Download link expires after 5 min.
- A user cannot fetch another user's resume by guessing the URL.
