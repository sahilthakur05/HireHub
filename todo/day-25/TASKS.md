# Day 25 — Realtime chat + unread-message email cron

## Goal
Replace 5s polling with realtime push. Send a coalesced email if a message goes unread > 10 minutes.

## Depends on
Day 24.

## Tasks
- [ ] Decide between Pusher and Upstash pub/sub + SSE. **Default: Pusher** (less custom code).
- [ ] `npm i pusher pusher-js`.
- [ ] Add Pusher env vars (PLAN.md §7).
- [ ] Build `app/api/realtime/auth/route.ts` — verify the user is a participant of `private-conversation-{id}` before signing.
- [ ] In `sendMessage` Server Action, after DB insert, `pusher.trigger('private-conversation-{id}', 'message:new', payload)`.
- [ ] In the message-list client component, subscribe to that channel; append messages received. Unsubscribe on unmount.
- [ ] Bell badge: subscribe to a per-user channel `private-user-{userId}` for new-message notifications across all conversations.
- [ ] Add typing indicator: client-only event via Pusher `client-typing` debounced; show "X is typing…" for 3s.
- [ ] Build `app/api/cron/unread-message-emails/route.ts`:
  - Find `Message` rows where `createdAt < now() - 10min` and `lastReadAt < message.createdAt` for the recipient participant, AND no email sent in past hour for that recipient.
  - Coalesce per recipient → one email listing senders + counts.
  - Track sends in a small table or in-memory Redis key.
- [ ] Add to `vercel.json` cron: `*/10 * * * *`.

## Files created/changed
- `app/api/realtime/auth/route.ts`
- `lib/pusher.ts`
- `app/api/cron/unread-message-emails/route.ts`
- `emails/UnreadMessages.tsx`
- Updated chat components to subscribe.

## Done when
- Two browsers logged in as different participants see messages in real time.
- Pusher dashboard shows messages flowing on private channels only.
- Closing one browser, sending a message, waiting 10 minutes → email arrives, never again within the same hour for that thread.
