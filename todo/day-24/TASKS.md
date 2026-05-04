# Day 24 — Chat inbox & thread UI (polling)

## Goal
Working chat UI with optimistic send, no realtime yet — polling refresh every 5s when window is focused.

## Depends on
Day 23.

## Tasks
- [ ] Build `/messages/page.tsx` (Server Component):
  - List conversations for current user, sorted by `lastMessageAt` desc.
  - Show counterparty (company logo+name for candidate side; candidate avatar+name for company side), last message snippet, unread badge.
- [ ] Build `/messages/[conversationId]/page.tsx`:
  - Fetch first page of messages (latest 50 desc, then reverse for display).
  - Mark read on mount via Server Action.
  - Composer at the bottom: textarea, Cmd+Enter to send.
- [ ] Use **TanStack Query** for the message list with `refetchInterval: 5000` while tab is focused, and optimistic updates on send.
- [ ] Show typing-indicator-shaped placeholder cells; real typing indicator deferred to Day 25.
- [ ] Header bell badge: unread count = sum of conversations with `lastMessageAt > participant.lastReadAt`. Use **Zustand** for the cross-page badge so multiple tabs stay roughly in sync.
- [ ] "Message company" CTA on a job page (if logged-in candidate who applied) → `openOrFindConversation` then redirect to thread.

## Files created/changed
- `app/(messages)/messages/page.tsx`
- `app/(messages)/messages/[conversationId]/page.tsx`
- `components/chat/composer.tsx`
- `components/chat/message-list.tsx`
- `lib/stores/unread.ts` (Zustand)

## Done when
- Sending a message appears instantly (optimistic), then settles after server confirms.
- Switching tabs and back updates the list within 5s.
- Bell badge increments when a new message arrives in another tab (within 5s).
