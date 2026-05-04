# Day 23 — Chat schema + open-thread Server Actions

## Goal
DB-backed text-only chat between candidate ↔ company members. No realtime yet — that's Day 25.

## Depends on
Day 18 (so we can scope cold-message rule to "only if you applied" for v1).

## Tasks
- [ ] Add `Conversation`, `ConversationParticipant`, `Message`, `Side` enum (see `PLAN.md` §3.4). Skip `Attachment` for v1 (text-only chat).
- [ ] Confirm `CompanyMember` from Day 06 still exists; if not, add it now.
- [ ] Server Action `openOrFindConversation({ candidateId, companyId, jobId? })`:
  - Enforce: caller is either the candidate, OR a `CompanyMember` of `companyId`.
  - For v1, candidate can only open if they have applied to a job at that company. (Cold outreach deferred to Premium.)
  - Upsert by unique `(candidateId, companyId, jobId)`.
  - Add participants: candidate + all OWNER/ADMIN/RECRUITER/HR members of the company.
- [ ] Server Action `sendMessage({ conversationId, body })`:
  - Sanitize via `sanitize-html` (text + line breaks only, no HTML).
  - Append to `Message`, update `Conversation.lastMessageAt`.
  - Update sender's `ConversationParticipant.lastReadAt = now()` (read your own).
- [ ] Server Action `markRead({ conversationId })`.
- [ ] No UI yet; write Vitest unit tests for these three actions covering permission and dedupe paths.

## Files created/changed
- Prisma migration with chat tables.
- `lib/chat.ts` (the three Server Actions exposed via importable functions for tests).
- `app/api/conversations/route.ts` (Route Handler wrapping `openOrFindConversation` for client fetch).
- `tests/chat.test.ts`.

## Done when
- All Vitest cases green: open by candidate without application → rejected; open by candidate with application → succeeds and adds 1 candidate + N company participants.
- Sending two messages updates `lastMessageAt`.
- Sending HTML in the body strips it server-side.
