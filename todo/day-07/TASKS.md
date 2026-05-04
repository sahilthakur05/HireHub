# Day 07 — Company verification (domain-email path only)

## Goal
A company can earn a "Verified ✓" badge by signing in with an email matching its website domain.

## Depends on
Day 06.

## Tasks
- [ ] Add helper `extractDomain(url)` in `lib/url.ts` — handles `https://www.acme.com/foo` → `acme.com`.
- [ ] On `createCompany` and `updateCompany`, check if the user's email domain matches the website domain; if so set `verifiedAt = now()` and `verificationMethod = DOMAIN_EMAIL`.
- [ ] If they change website later, re-evaluate; keep verification only if the domain still matches.
- [ ] Build a small `<VerifiedBadge company={c} />` component using shadcn `Badge` + a check icon.
- [ ] Render badge on the (future) company public page and next to the company name in (future) job lists. For now wire it into the employer dashboard header.
- [ ] Add a banner on `/employer/company` explaining the verification status and what to do to verify (use a corporate email).
- [ ] **Defer DNS-TXT path entirely** to v1.1 — add a `// TODO(v1.1): DNS_TXT path` comment near the helper and move on.

## Files created/changed
- `lib/url.ts`
- `lib/verification.ts` (the domain-match logic + tests later in Day 29)
- `components/verified-badge.tsx`
- Updated company actions to call `evaluateVerification(company, ownerEmail)`.

## Done when
- Owner with `jane@acme.com` + website `acme.com` → company is auto-verified.
- Owner with `jane@gmail.com` + website `acme.com` → not verified (banner explains why).
- Changing website to a non-matching domain clears verification.
