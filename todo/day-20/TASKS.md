# Day 20 — Kanban pipeline + status emails

## Goal
Employers manage applicants on a drag-and-drop kanban; status changes email the candidate.

## Depends on
Day 19.

## Tasks
- [ ] `npm i @dnd-kit/core @dnd-kit/sortable`.
- [ ] Build `<ApplicantKanban />` (client) with 5 columns matching `AppStatus`.
- [ ] Optimistic updates: drop card → immediately move in UI, then call Server Action `updateApplicationStatus`. On error, revert.
- [ ] Bulk actions (v1 minimal): checkbox-select rows, "Move all to Reviewing" button.
- [ ] Server Action sends `ApplicationStatusChanged.tsx` email when status transitions.
- [ ] Mute the email if the same status was set within the last 5 minutes (prevent spam from rapid drag/drop corrections).
- [ ] Add a tab on `/employer/jobs/[id]/applicants` to switch between **Table** (Day 19) and **Kanban**.

## Files created/changed
- `app/(employer)/employer/jobs/[id]/applicants/kanban.tsx`
- `app/(employer)/employer/jobs/[id]/applicants/actions.ts`
- `emails/ApplicationStatusChanged.tsx`
- Updated applicants page to host both views.

## Done when
- Dragging an applicant from Applied → Interview persists across reload.
- Candidate receives one status-change email per real transition.
- Optimistic UI reverts cleanly on a forced server error.
