# Day 28 — Mobile polish + accessibility pass

## Goal
The whole app is usable on a 375px screen and passes basic a11y checks on the apply, signup, and chat flows.

## Depends on
All previous days (you're auditing the built app).

## Tasks
- [ ] Walk every public route at 375px width (Chrome devtools). Fix any horizontal scroll, tap targets < 44px, broken modals.
- [ ] Critical flows to test on a real phone: sign-up → verify → upload resume → apply → check messages.
- [ ] Header: confirm mobile nav uses the `<Sheet>` drawer; sign-in vs avatar dropdown both work.
- [ ] Job filters: drawer slides up from the bottom, has a sticky "Apply filters" footer.
- [ ] Chat composer: stays above the keyboard on iOS Safari (use `100dvh` not `100vh`).
- [ ] `npm i -D @axe-core/playwright @playwright/test` and add a smoke test that runs axe on `/`, `/jobs`, `/sign-up`, `/dashboard`, `/messages`. Fix any "serious" or "critical" violations.
- [ ] Manual keyboard pass: tab through signup, apply, kanban — `:focus-visible` rings should be obvious.
- [ ] Add a "Skip to main content" link as the first focusable element of `app/layout.tsx`.
- [ ] All form inputs have associated `<Label>` (no placeholder-only).
- [ ] Color contrast: run axe + spot-check Tailwind grays against backgrounds. Fix any < 4.5:1.

## Files created/changed
- A handful of `tailwindcss` class tweaks across components.
- `tests/a11y.spec.ts` (Playwright + axe).
- Possibly `app/layout.tsx` for skip link.

## Done when
- Lighthouse mobile (a11y category) ≥ 95 on home, jobs, job-detail, signup, dashboard.
- `tests/a11y.spec.ts` passes with zero "serious"/"critical" rules.
- iOS Safari chat composer doesn't get hidden by the keyboard.
