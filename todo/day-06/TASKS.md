# Day 06 — Company profile model + employer onboarding

## Goal
Employers can create a company once, edit its profile, and upload a logo.

## Depends on
Day 03 (UI primitives), Day 04 (RBAC).

## Tasks
- [ ] If user picks "I'm hiring" on signup, set `role = EMPLOYER` (update Day 02 signup action).
- [ ] After first sign-in as EMPLOYER without a `Company`, redirect to `/employer/company/new`.
- [ ] Build `/employer/company/new` form with shadcn: name, slug (auto-generated from name, editable), website, size, industry dropdown, description textarea.
- [ ] Server Action `createCompany`: zod-validate, dedupe slug, set `ownerId = user.id`, create matching `CompanyMember(role: OWNER)`.
- [ ] Build `/employer/company/page.tsx` (edit form, same fields + logo upload).
- [ ] Logo upload: client uses `next-cloudinary`'s `<CldUploadButton>` with a signed-upload preset (configure in Cloudinary dashboard, restrict to images, max 2 MB).
- [ ] Save `logoUrl` on form submit.
- [ ] Show a "Company profile" link in the header for EMPLOYERs.

## Files created/changed
- `app/(employer)/employer/company/new/page.tsx`
- `app/(employer)/employer/company/page.tsx`
- `app/(employer)/employer/company/actions.ts`
- `lib/cloudinary.ts` (signed-upload helper)
- shadcn additions: `select`, `textarea`.

## Done when
- A new EMPLOYER user is force-redirected to create a company.
- Slug is unique-validated server-side.
- Logo upload returns a Cloudinary URL and renders in the header avatar.
