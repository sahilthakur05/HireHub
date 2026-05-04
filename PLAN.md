# HireHub — Job Board Platform Build Plan

A full-stack job board with employer/candidate roles, resume uploads, email notifications, search, and an admin panel. Built on the 100% free-tier stack.

> ⚠️ **Next.js version notice**: This repo is on **Next.js 16.2.4** with **React 19.2.4** (not Next 14). Verified against `node_modules/next/package.json`. Notable Next 16 breaking changes that affect this plan:
> - **Middleware → Proxy**: the file is now `proxy.ts` at the project root, exporting a default function (formerly `middleware.ts` / `middleware`). Functionality is the same; the name changed in Next 16 to better reflect its purpose.
> - Always consult `node_modules/next/dist/docs/01-app/` before writing framework code, and heed deprecation notices.

---

## 1. Tech Stack (all free-tier)

| Layer | Choice | Why / Free tier |
|---|---|---|
| Framework | Next.js 16 (App Router) + Server Actions | Already installed |
| Language | TypeScript | Already configured |
| Styling | Tailwind CSS v4 + shadcn/ui | Tailwind installed; add shadcn components on demand |
| Auth | **Auth.js v5** (formerly NextAuth) — Google + GitHub OAuth + Credentials | Free, OSS |
| Database | **Neon Postgres** (free 512 MB) + **Prisma ORM** | Serverless Postgres, branching |
| Caching / Rate-limit | **Upstash Redis** (free 10k req/day) | Sessions, rate limit, search cache |
| File storage | **Cloudinary** (free 25 GB) | Resume PDFs + employer logos + avatars |
| Email | **Resend** (free 3 000/month) + **React Email** templates | Replaces Nodemailer; better DX |
| Payments (optional) | Stripe test mode | For featured/sponsored job posts |
| Background jobs | **Vercel Cron** + Route Handlers | Free, no extra service |
| Deployment | Vercel hobby | Free; zero-config |
| Search | Postgres full-text search (`tsvector`) | Free, no Algolia needed |
| Rich text editor | **Tiptap** (free, OSS) | For job descriptions |
| Client state | **Zustand** + **TanStack Query** (default); Redux Toolkit available if needed | See §1.1 |

### 1.1 State Management Strategy

In Next 16 App Router most "state" is one of four things — match the right tool to each, and only reach for Redux if the lighter options don't fit.

| State type | Examples | Tool |
|---|---|---|
| **Server data** (authoritative) | Job list, applications, profile | RSC + Server Actions; `revalidatePath` after mutation. No client store needed. |
| **Server data on the client** (cached + invalidated) | Inbox messages, notification feed, kanban applicants | **TanStack Query** — handles fetching, caching, optimistic updates, refetching on focus. |
| **Ephemeral UI** | Modals, drawers, form-step index, theme | `useState` / `useReducer` co-located in the component. |
| **Cross-page shared client state** | Global unread counter, realtime presence, optimistic chat queue | **Zustand** — tiny, ergonomic, no provider gymnastics. |

**When to add Redux Toolkit instead:**
- The team already standardizes on RTK and time-travel debugging is a hard requirement.
- You need RTK Query's deep middleware (entity adapters, lifecycle subscriptions) for a complex feature like the kanban.
- You expect 5+ slices that genuinely interact (cross-slice selectors, complex transitions). For HireHub today, this is unlikely.

**Decision for v1**: ship with **Zustand + TanStack Query**. If a feature (most likely chat or kanban) outgrows them, migrate that slice to **Redux Toolkit** rather than rewriting everything. Don't add Redux to the bundle until something forces it.

**Server-Component caveats** (whichever client store you pick):
- Stores live in client components only — never `import`ed from a Server Component.
- Wrap with a small `<Providers>` client component in `app/providers.tsx` mounted from `app/layout.tsx`.
- Hydrate from server data via TanStack Query's `dehydrate`/`HydrationBoundary` (or props for Zustand) — don't fetch the same data twice.
- All writes still go through Server Actions; the client store mirrors the result, it never owns the truth.

```bash
# Default
npm i zustand @tanstack/react-query @tanstack/react-query-devtools

# Only if/when we adopt Redux for a specific slice
npm i @reduxjs/toolkit react-redux
```

---

## 2. Roles & Permissions (RBAC)

Roles are enforced in **`proxy.ts`** (Next 16's renamed middleware) + Server Actions. **Plan tier** (FREE / PREMIUM) is orthogonal to role — a user has both.

### Roles
- **CANDIDATE** — browse jobs, apply, manage profile + resume, save jobs, view application status.
- **EMPLOYER** — create company profile, post/edit/close jobs, view applicants, message candidates.
- **ADMIN** — moderate jobs/users, view platform stats, manage reports, suspend accounts.

### Plan Tiers (`PlanTier` enum on User)
- **FREE** — default for everyone on signup.
- **PREMIUM** — paid (Stripe subscription) — unlocks gated features below.

Role is stored on the `User` row; gated via:
- `proxy.ts` — route-level guard (redirect/forbid). *(Next 16 renamed `middleware.ts` → `proxy.ts`.)*
- Server Action helper `requireRole(role)` — throws on unauthorized writes.
- Server Action helper `requirePremium()` — throws / redirects to `/upgrade` if `plan !== PREMIUM` or subscription expired.
- UI conditional rendering (purely cosmetic; never trust the client).

### Premium feature matrix

| Capability | FREE Candidate | PREMIUM Candidate | FREE Employer | PREMIUM Employer |
|---|---|---|---|---|
| Browse + apply | ✅ | ✅ | — | — |
| Saved jobs | 10 max | Unlimited | — | — |
| Resumes stored | 1 | 5 | — | — |
| AI cover-letter assist | ❌ | ✅ | — | — |
| Application priority badge | ❌ | ✅ (employer sees ⭐) | — | — |
| Weekly job-match digest | ❌ | ✅ | — | — |
| Chat with applied companies | ✅ | ✅ | — | — |
| Cold-message companies (no application) | ❌ | ✅ (5/day) | — | — |
| Active job posts | — | — | 1 | 10 |
| Featured job placement | — | — | ❌ | ✅ (auto) |
| Applicant kanban + bulk actions | — | — | View only | ✅ |
| Resume downloads / month | — | — | 5 | Unlimited |
| Cold-message candidates | — | — | ❌ | ✅ (50/day) |
| Receive cold messages from candidates | — | — | ❌ | ✅ |
| Company branding (logo, banner, colors) | — | — | Logo only | Full |
| Analytics dashboard | — | — | Basic | Advanced |
| Priority support | — | ✅ | — | ✅ |

### Subscription handling
- Stripe Checkout → webhook `/api/webhooks/stripe` updates `User.plan`, `User.planRenewsAt`, `User.stripeCustomerId`, `User.stripeSubscriptionId`.
- On `customer.subscription.deleted` or past-due → downgrade to FREE.
- Daily cron `downgrade-expired` as a safety net for missed webhooks.
- Billing portal link via Stripe Customer Portal (no custom UI needed).

---

## 3. Feature List

### 3.1 Authentication
- Auth.js v5 with **Google** + **GitHub** OAuth providers.
- Email/password (Credentials provider) with bcrypt + email verification link via Resend.
- Magic-link sign-in (optional, Resend).
- Password reset flow (token in DB, expires 1 h).
- Session: JWT strategy (stateless) — Redis used only for refresh-token denylist if we add one.
- Role assignment on first sign-in: candidate by default, employer via "I'm hiring" checkbox at signup.

### 3.2 Candidate Features
- Profile page — name, headline, bio, skills (tags), location, portfolio links.
- **Resume upload** — PDF only, ≤ 5 MB, virus-scan via Cloudinary's auto-moderation, signed URLs for downloads.
- Apply to a job — choose from uploaded resumes + cover letter.
- Saved jobs (bookmark).
- Application status tracker: Applied → Under Review → Interview → Offer / Rejected.
- Email alerts on status changes.
- Job recommendations based on skills (simple Postgres query — no ML needed).

### 3.3 Candidate Analytics Dashboard

A `/dashboard` view for candidates with engagement and reach metrics.

**Cards / widgets**
- **Profile views** — total, last 7d, last 30d, sparkline; segmented by viewer role (Recruiter / HR / Hiring Manager / Other).
- **Resume downloads** — count + table of `{company, role of downloader, downloaded_at}`.
- **Search appearances** — how many times the profile appeared in recruiter search results (Premium also shows the matching keywords).
- **Who viewed you** — feed of `{company, viewer role, viewed_at}`.
  - FREE: total count + blurred names ("Acme Corp + 4 others viewed you").
  - PREMIUM: full identities + viewer role + link to company.
- **Application funnel** — Applied → Reviewing → Interview → Offer / Rejected with conversion %.
- **Avg time-to-response** per company you applied to.
- **Profile completeness** — score 0–100 with a checklist (resume, skills, headline, location, portfolio).
- **Skill demand** — count of open jobs matching your top 5 skills (clickable → pre-filtered search).
- **Salary benchmark** — median min/max for your skills + location (computed from `Job.salaryMin/Max`).
- **Saved-jobs activity** — count, latest saved.
- **Job-alert performance** (Premium) — opens/clicks of weekly digest.

**Premium-only widgets**
- Full identities in "Who viewed you".
- Search-keyword breakdown ("you appeared for: react, typescript, remote").
- Peer comparison — "Your profile is viewed 2.3× more / less than avg candidate with similar skills".
- Weekly stats email digest.

**Tracking — how the data is collected**

Every relevant action writes a row to a single `ProfileEvent` table (cheap, append-only):

```prisma
model ProfileEvent {
  id            String       @id @default(cuid())
  candidateId   String       // who is being viewed/downloaded
  candidate     CandidateProfile @relation(fields: [candidateId], references: [id], onDelete: Cascade)
  actorUserId   String?      // null = anonymous / not signed in
  actorCompanyId String?     // company the actor belongs to (denormalized)
  actorRole     ActorRole?   // RECRUITER | HR | HIRING_MANAGER | OTHER
  type          EventType
  meta          Json?        // search query, job context, etc.
  createdAt     DateTime     @default(now())

  @@index([candidateId, createdAt(sort: Desc)])
  @@index([candidateId, type, createdAt])
}

enum EventType {
  PROFILE_VIEW
  RESUME_DOWNLOAD
  SEARCH_IMPRESSION
  CONTACT_CLICK
}

enum ActorRole { RECRUITER HR HIRING_MANAGER OTHER }
```

**Where events fire**
- `PROFILE_VIEW` — server component for `/candidates/[id]` records once per `(candidate, actor, day)` (dedupe via Redis `SETNX` with 24h TTL to avoid bot/refresh inflation).
- `RESUME_DOWNLOAD` — the signed-URL issuing endpoint logs the event before redirecting to Cloudinary.
- `SEARCH_IMPRESSION` — recruiter search Server Action batch-inserts impressions for the page of results (only for results actually rendered).
- `CONTACT_CLICK` — clicks on email/portfolio/website links (small `/api/track` POST from client).

**Aggregation strategy**
- For dashboard reads, run aggregate queries with date-bucketing (`date_trunc('day', created_at)`) — fast enough up to ~100k events per candidate with the indexes above.
- Cache the dashboard payload in Upstash Redis for 60s per candidate.
- Nightly cron rolls events older than 90 days into a `ProfileEventDaily` summary table and prunes raw rows (keeps the table lean).

**Anti-abuse**
- Rate-limit `/api/track` (60 req/min/IP via Upstash).
- Don't count self-views (`actorUserId === candidate.userId`).
- Don't count admin views.
- Bot detection: drop events when User-Agent matches a known crawler list.

### 3.4 Messaging / Chat (Candidate ↔ Company)

Direct chat between a candidate and any member of a company (recruiter, HR, hiring manager, owner). Threads are scoped to a `(candidate, company)` pair so multiple company members can collaborate inside one conversation.

**Capabilities**
- 1:1 and many-to-1 — candidate talks to a company; multiple company members can reply on the same thread.
- Thread can be opened from: a job page ("Message company"), an applicant card (employer side), or a company profile.
- Optional context: a thread can reference a `Job` and/or `Application` to give recipients context.
- Read receipts, typing indicator, unread badge in nav.
- File attachments (PDFs, images) via Cloudinary signed upload — same pipeline as resumes.
- Message reactions (emoji) — optional/v2.
- Mute / archive / block.
- Report message → admin moderation queue.

**Who can start a chat**
- Employer side: any verified company member can open a thread with any candidate.
- Candidate side: can message a company only if (a) they applied to one of its jobs, OR (b) the company is **PREMIUM** and accepts cold inbound, OR (c) the candidate is **PREMIUM** (cold-outreach unlock).
- Rate-limit cold messages: 5/day FREE candidate Premium, 50/day Premium employer.

**Data model**

```prisma
model CompanyMember {
  id        String       @id @default(cuid())
  companyId String
  userId    String
  role      CompanyRole  @default(MEMBER)
  invitedBy String?
  joinedAt  DateTime     @default(now())
  company   Company @relation(fields: [companyId], references: [id], onDelete: Cascade)
  user      User    @relation(fields: [userId], references: [id], onDelete: Cascade)
  @@unique([companyId, userId])
}

enum CompanyRole { OWNER ADMIN RECRUITER HR HIRING_MANAGER MEMBER }

model Conversation {
  id            String   @id @default(cuid())
  candidateId   String   // User.id of the candidate
  companyId     String
  jobId         String?  // optional context
  applicationId String?  // optional context
  lastMessageAt DateTime @default(now())
  status        ConvStatus @default(OPEN)
  createdAt     DateTime @default(now())

  candidate     User     @relation("CandidateConvs", fields: [candidateId], references: [id], onDelete: Cascade)
  company       Company  @relation(fields: [companyId], references: [id], onDelete: Cascade)
  messages      Message[]
  participants  ConversationParticipant[]

  @@unique([candidateId, companyId, jobId]) // one thread per (candidate, company, job)
  @@index([companyId, lastMessageAt(sort: Desc)])
  @@index([candidateId, lastMessageAt(sort: Desc)])
}

enum ConvStatus { OPEN ARCHIVED BLOCKED }

// Tracks who has joined the thread + per-user read state
model ConversationParticipant {
  id              String   @id @default(cuid())
  conversationId  String
  userId          String
  side            Side     // CANDIDATE | COMPANY
  lastReadAt      DateTime?
  muted           Boolean  @default(false)
  conversation    Conversation @relation(fields: [conversationId], references: [id], onDelete: Cascade)
  user            User         @relation(fields: [userId], references: [id], onDelete: Cascade)
  @@unique([conversationId, userId])
}

enum Side { CANDIDATE COMPANY }

model Message {
  id             String   @id @default(cuid())
  conversationId String
  senderId       String
  body           String   // sanitized text/markdown
  attachments    Attachment[]
  createdAt      DateTime @default(now())
  editedAt       DateTime?
  deletedAt      DateTime?

  conversation   Conversation @relation(fields: [conversationId], references: [id], onDelete: Cascade)
  sender         User         @relation(fields: [senderId], references: [id], onDelete: Cascade)

  @@index([conversationId, createdAt])
}

model Attachment {
  id        String   @id @default(cuid())
  messageId String
  url       String
  filename  String
  mime      String
  size      Int
  message   Message  @relation(fields: [messageId], references: [id], onDelete: Cascade)
}
```

**Realtime transport**
- Use **Pusher Channels** (free tier: 100 concurrent / 200k msgs/day) **or** Upstash Redis pub/sub + Server-Sent Events from a Route Handler.
- Channel naming: `private-conversation-{conversationId}`; auth endpoint `/api/realtime/auth` checks the user is a participant.
- Server Action `sendMessage` writes to DB → publishes to channel → recipients update via subscription.
- Fallback polling every 15 s if socket disconnects.

**UI**
- `/messages` — split-pane inbox (threads list left, active thread right).
- Unread badge in main nav (driven by SSE/WebSocket-pushed counter).
- Company-side inbox shows candidate name, applied-to job tag, last message snippet.
- Candidate-side inbox shows company logo + name, job tag, last message snippet.
- Thread header: company avatar + names of company members currently in the thread; click to view their company-role.
- Composer: textarea, attach button, Cmd+Enter to send, optimistic message append.

**Notifications**
- In-app: real-time push + persistent unread count.
- Email (Resend): if recipient hasn't read message within 10 minutes AND is offline, send "New message from {sender}" with deep link. Coalesce multiple unread messages into one email per hour.
- Browser push (v2, optional).

**Moderation & abuse**
- Server-side text sanitization (`sanitize-html`) before storing.
- Rate-limit `sendMessage` per user (30/min, 500/day) via Upstash.
- Automatic flagging of messages matching a profanity/spam list → routed to admin queue, sender suspended after N flags.
- "Block" creates a `ConvStatus = BLOCKED` and prevents new threads `(candidate, company)`.
- Cold-outreach gate (above) prevents spam from random companies.

**Permission rules (server-enforced)**
- Only `ConversationParticipant` rows may read messages or send.
- A new company member auto-joins ongoing threads only if their `CompanyRole` ∈ {OWNER, ADMIN, RECRUITER, HR}; HIRING_MANAGER joins only when invited by another member.
- Admins can read threads only via the moderation queue (audit-logged).

**Routes (App Router)**

```
app/(messages)/
  messages/page.tsx                  # inbox
  messages/[conversationId]/page.tsx # thread view
app/api/
  conversations/route.ts             # POST: open/find thread
  conversations/[id]/messages/route.ts  # GET (paginated) / POST (send)
  conversations/[id]/read/route.ts   # POST: update lastReadAt
  realtime/auth/route.ts             # Pusher private-channel auth
  upload/attachment/route.ts         # signed Cloudinary upload
```

### 3.5 Employer Features
- Company profile — logo, description, website, size, industry.
- **Post a job** — Tiptap rich-text editor, salary range, location (remote/hybrid/onsite), tags, expires_at.
- Manage postings — edit, close, repost, view stats (views, applicants).
- Applicant pipeline — kanban board (Applied / Reviewing / Interview / Offer / Rejected), drag-and-drop with optimistic UI.
- Download candidate resume (signed Cloudinary URL).
- Bulk actions on applicants.
- (Optional) Pay $X via Stripe to feature a job for 30 days.

### 3.6 Public / Discovery
- Home page — featured jobs, latest jobs, popular categories.
- **Search & filters**:
  - Keyword (Postgres `to_tsvector` with weighted ranking on title > company > description).
  - Filters: location, remote/hybrid/onsite, employment type, salary range, experience level, tags.
  - Sort: relevance | newest | salary.
  - Server-rendered for SEO; URL params drive state (shareable links).
- Individual job page — full description, company info, "Apply" button, similar jobs, structured data (`JobPosting` JSON-LD) for Google Jobs.
- Company directory + per-company page.
- `sitemap.xml` + `robots.txt` generated dynamically from active jobs.

### 3.7 Admin Panel
- Dashboard — totals, signups today, applications today, active jobs, MRR (if Stripe).
- Moderation queue — flagged jobs, reported users.
- User management — search, change role, suspend, delete.
- Job management — bulk close expired, force-feature, delete spam.
- Audit log — who did what when (admin actions only).

### 3.8 Notifications & Email
React Email templates rendered via Resend:
- Welcome / verify email
- Password reset
- New application received (to employer)
- Application status changed (to candidate)
- Job match digest (weekly cron — see §3.7)
- Job expiring soon (to employer, 3 days before `expires_at`)

### 3.9 Cron Jobs (Vercel Cron)
Defined in `vercel.json` → hits Route Handlers under `app/api/cron/*` protected by a shared secret.

| Schedule | Job | Purpose |
|---|---|---|
| `0 * * * *` | `expire-jobs` | Mark jobs past `expires_at` as CLOSED |
| `0 9 * * 1` | `weekly-digest` | Email candidates matching jobs from last 7 days |
| `0 8 * * *` | `expiring-soon` | Notify employers 3 days before expiry |
| `0 3 * * *` | `cleanup` | Purge unverified accounts > 7 days |
| `0 4 * * *` | `downgrade-expired` | Downgrade users whose `planRenewsAt < now()` to FREE |
| `0 2 * * *` | `rollup-profile-events` | Aggregate `ProfileEvent` > 90d into `ProfileEventDaily`, prune raw rows |
| `*/10 * * * *` | `unread-message-emails` | Email recipients with unread messages older than 10 min, coalesced per hour |
| `*/15 * * * *` | `instant-alerts` | Send saved-search instant alerts for new matching jobs |
| `0 8 * * *` | `daily-alerts` | Send daily saved-search digests |
| `0 9 * * 1` | `weekly-alerts` | Send weekly saved-search digests (also covers job-match digest) |
| `0 5 * * *` | `verify-dns-txt` | Re-check pending company DNS TXT verifications |
| `0 1 * * *` | `finalize-deletions` | Hard-delete users whose 30-day grace expired |
| `*/5 * * * *` | `process-exports` | Build pending GDPR data exports → email signed URL |

### 3.10 Caching & Rate Limiting (Upstash Redis)
- Cache home-page job feed (TTL 60 s) — invalidate on new post.
- Cache search results keyed by query hash (TTL 5 min).
- Rate-limit auth endpoints (5 attempts / 15 min / IP).
- Rate-limit job applications (10 / hour / user) to prevent spam.
- Use `@upstash/ratelimit` sliding-window algorithm.

### 3.11 Onboarding Wizard
First-run flow after signup; prevents the "empty profile" cliff.
- Step 1 — choose role (Candidate / Employer).
- Step 2 — basic info (name, location).
- Step 3 (Candidate) — upload resume → auto-parse → pre-fill skills, experience, education.
- Step 3 (Employer) — company name + website → auto-pull `og:image`/favicon as logo, prefill description from meta tags.
- Step 4 — preferences (job alerts opt-in, weekly digest, locale).
- Persisted as `User.onboardingStep`; `proxy.ts` redirects to `/welcome/[step]` until complete.
- Skippable but a banner nags until 100% profile completeness.

### 3.12 Resume Parser
- PDF → structured fields using `pdf-parse` (free, OSS) + heuristic regex for sections (Skills, Experience, Education, Contact).
- Optional: Claude API call (`claude-haiku-4-5-20251001`) with prompt-cached system instructions for higher accuracy on messy PDFs — gated behind a feature flag for cost control.
- Pre-fills the candidate profile; user reviews/edits before save.
- Stored extracted text on `Resume.extractedText` for full-text search of candidates by recruiters.

### 3.13 Email Verification + 2FA
- **Email verification required** for any write action (post job, apply, send message). 24h-expiring token via Resend.
- **2FA (TOTP)** optional for candidates, required for employers before first job post.
  - `otplib` to generate/verify; QR via `qrcode` package.
  - Recovery codes (10 single-use codes shown once, hashed in DB).
- Schema: `User.totpSecret` (encrypted), `User.recoveryCodes` (hashed array), `User.twoFactorEnabled`.
- Login flow: email+password → if 2FA enabled, prompt for TOTP before issuing session.

### 3.14 Company Verification
- Verified ✓ badge on company profile + job listings.
- Two paths:
  1. **Domain match** — owner email matches company website domain (e.g. `jane@acme.com` + `acme.com` website) → auto-verify.
  2. **DNS TXT** — for non-corp emails, ask owner to add `hirehub-verify=<token>` TXT record; daily cron checks.
- `Company.verifiedAt: DateTime?` field; unverified companies can post but are flagged in moderation.
- Verified companies rank higher in search.

### 3.15 Saved Searches & Job Alerts
- Candidate saves a search (keyword + filters) → choose frequency (instant / daily / weekly).
- Schema:
  ```prisma
  model SavedSearch {
    id        String   @id @default(cuid())
    userId    String
    name      String
    query     Json     // serialized filter object
    frequency AlertFrequency @default(WEEKLY)
    lastSentAt DateTime?
    createdAt DateTime @default(now())
    user User @relation(fields: [userId], references: [id], onDelete: Cascade)
    @@index([frequency, lastSentAt])
  }
  enum AlertFrequency { OFF INSTANT DAILY WEEKLY }
  ```
- Crons: `instant-alerts` (every 15 min), `daily-alerts` (08:00 UTC), `weekly-alerts` (Mon 09:00).
- Each alert email = matching jobs since `lastSentAt`, single Resend send.

### 3.16 Notifications Inbox (in-app)
- Bell icon in nav with unread count.
- `Notification` table: `{userId, type, title, body, link, readAt, createdAt}`.
- Types: `PROFILE_VIEW`, `RESUME_DOWNLOAD`, `NEW_MESSAGE`, `APP_STATUS_CHANGE`, `JOB_MATCH`, `COMPANY_VERIFIED`, `SUBSCRIPTION_RENEWED`, etc.
- Realtime push via the same Pusher/SSE channel as messaging.
- Dropdown shows last 10; full page at `/notifications`.
- Bulk "Mark all read", per-type mute settings on `/account/notifications`.

### 3.17 SEO
- Per-job dynamic metadata + Open Graph image (Next `ImageResponse`).
- `JobPosting` schema.org JSON-LD on every job page → Google Jobs eligibility.
- Server-rendered job listings (no client-only fetching for SEO routes).
- Dynamic `sitemap.ts` + `robots.ts` (Next 16 file conventions).

---

## 4. Database Schema (Prisma)

Core models — refine during implementation:

```prisma
model User {
  id            String    @id @default(cuid())
  email         String    @unique
  emailVerified DateTime?
  passwordHash  String?
  name          String?
  image         String?
  role          Role      @default(CANDIDATE)
  locale        String    @default("en")

  // Plan / subscription
  plan                 PlanTier  @default(FREE)
  planRenewsAt         DateTime?
  stripeCustomerId     String?   @unique
  stripeSubscriptionId String?   @unique

  // Onboarding
  onboardingStep       Int       @default(0)   // 0 = not started; N = up to step N done
  onboardedAt          DateTime?

  // 2FA
  twoFactorEnabled     Boolean   @default(false)
  totpSecret           String?   // encrypted at rest
  recoveryCodes        String[]  // bcrypt-hashed

  // Soft delete (GDPR grace period)
  deletedAt            DateTime?
  deleteScheduledFor   DateTime?

  // Compliance
  termsAcceptedAt      DateTime?
  termsVersion         String?

  createdAt     DateTime  @default(now())

  candidate     CandidateProfile?
  company       Company?
  applications  Application[]
  savedJobs     SavedJob[]
  savedSearches SavedSearch[]
  notifications Notification[]
  accounts      Account[] // OAuth links (Auth.js)
  sessions      Session[]
}

enum Role { CANDIDATE EMPLOYER ADMIN }
enum PlanTier { FREE PREMIUM }

model Notification {
  id        String   @id @default(cuid())
  userId    String
  type      NotificationType
  title     String
  body      String?
  link      String?
  readAt    DateTime?
  createdAt DateTime @default(now())
  user      User @relation(fields: [userId], references: [id], onDelete: Cascade)
  @@index([userId, readAt, createdAt(sort: Desc)])
}

enum NotificationType {
  PROFILE_VIEW
  RESUME_DOWNLOAD
  NEW_MESSAGE
  APP_STATUS_CHANGE
  JOB_MATCH
  COMPANY_VERIFIED
  SUBSCRIPTION_RENEWED
  SUBSCRIPTION_EXPIRING
  SYSTEM
}

model CandidateProfile {
  id        String   @id @default(cuid())
  userId    String   @unique
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  headline  String?
  bio       String?
  location  String?
  skills    String[] // Postgres text[]
  resumes   Resume[]
}

model Resume {
  id            String   @id @default(cuid())
  candidateId   String
  candidate     CandidateProfile @relation(fields: [candidateId], references: [id], onDelete: Cascade)
  url           String   // Cloudinary public_id
  filename      String
  extractedText String?  // pdf-parse output, used for recruiter full-text search
  parsedFields  Json?    // {skills, experience, education, contact}
  createdAt     DateTime @default(now())
  @@index([candidateId, createdAt(sort: Desc)])
}

model Company {
  id          String   @id @default(cuid())
  ownerId     String   @unique
  owner       User     @relation(fields: [ownerId], references: [id], onDelete: Cascade)
  name        String
  slug        String   @unique
  logoUrl     String?
  bannerUrl   String?
  website     String?
  description String?
  size        String?
  industry    String?

  // Verification
  verifiedAt        DateTime?
  verificationToken String?   // for DNS TXT path
  verificationMethod VerificationMethod?

  jobs        Job[]
  members     CompanyMember[]
  @@index([verifiedAt])
}

enum VerificationMethod { DOMAIN_EMAIL DNS_TXT MANUAL }

model Job {
  id            String   @id @default(cuid())
  companyId     String
  company       Company  @relation(fields: [companyId], references: [id], onDelete: Cascade)
  title         String
  slug          String   @unique
  descriptionHtml String  // sanitized Tiptap output
  location      String
  remote        RemoteType
  employmentType EmploymentType
  experience    Experience
  salaryMin     Int?
  salaryMax     Int?
  currency      String   @default("USD")
  tags          String[]
  status        JobStatus @default(OPEN)
  featured      Boolean  @default(false)
  featuredUntil DateTime?
  expiresAt     DateTime
  createdAt     DateTime @default(now())
  applications  Application[]
  savedBy       SavedJob[]

  // Full-text search
  searchVector  Unsupported("tsvector")?
  @@index([searchVector], type: Gin)
  @@index([status, createdAt(sort: Desc)])
}

enum JobStatus { OPEN CLOSED DRAFT }
enum RemoteType { REMOTE HYBRID ONSITE }
enum EmploymentType { FULL_TIME PART_TIME CONTRACT INTERNSHIP }
enum Experience { JUNIOR MID SENIOR LEAD }

model Application {
  id          String   @id @default(cuid())
  jobId       String
  job         Job      @relation(fields: [jobId], references: [id], onDelete: Cascade)
  userId      String
  user        User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  resumeUrl   String
  coverLetter String?
  status      AppStatus @default(APPLIED)
  createdAt   DateTime @default(now())
  @@unique([jobId, userId])
}

enum AppStatus { APPLIED REVIEWING INTERVIEW OFFER REJECTED }

model SavedJob {
  userId String
  jobId  String
  createdAt DateTime @default(now())
  user User @relation(fields: [userId], references: [id], onDelete: Cascade)
  job  Job  @relation(fields: [jobId], references: [id], onDelete: Cascade)
  @@id([userId, jobId])
}

// Auth.js required tables
model Account { /* … per Auth.js Prisma adapter */ }
model Session { /* … */ }
model VerificationToken { /* … */ }

model AuditLog {
  id        String   @id @default(cuid())
  actorId   String
  action    String
  target    String?
  ip        String?
  userAgent String?
  meta      Json?
  createdAt DateTime @default(now())
  @@index([actorId, createdAt(sort: Desc)])
  @@index([action, createdAt(sort: Desc)])
}

model SavedSearch {
  id         String   @id @default(cuid())
  userId     String
  name       String
  query      Json
  frequency  AlertFrequency @default(WEEKLY)
  lastSentAt DateTime?
  createdAt  DateTime @default(now())
  user       User @relation(fields: [userId], references: [id], onDelete: Cascade)
  @@index([frequency, lastSentAt])
}

enum AlertFrequency { OFF INSTANT DAILY WEEKLY }

model DataExportRequest {
  id          String   @id @default(cuid())
  userId      String
  status      ExportStatus @default(PENDING)
  downloadUrl String?  // signed Cloudinary URL, expires 7d
  createdAt   DateTime @default(now())
  completedAt DateTime?
  user        User @relation(fields: [userId], references: [id], onDelete: Cascade)
}

enum ExportStatus { PENDING PROCESSING READY FAILED }
```

A SQL migration adds the `tsvector` trigger:
```sql
ALTER TABLE "Job" ADD COLUMN search_vector tsvector
  GENERATED ALWAYS AS (
    setweight(to_tsvector('english', coalesce(title,'')), 'A') ||
    setweight(to_tsvector('english', coalesce(array_to_string(tags, ' '),'')), 'B') ||
    setweight(to_tsvector('english', coalesce(description_html,'')), 'C')
  ) STORED;
```

---

## 5. App Router Structure

```
app/
  (marketing)/
    page.tsx                  # home
    jobs/page.tsx             # browse + search
    jobs/[slug]/page.tsx      # job detail (SSG-ish + revalidate)
    companies/[slug]/page.tsx
  (auth)/
    sign-in/page.tsx
    sign-up/page.tsx
    verify/page.tsx
    reset/page.tsx
    2fa/page.tsx                       # TOTP challenge
  (onboarding)/
    welcome/page.tsx                   # entry, picks role
    welcome/[step]/page.tsx            # 1..4 wizard
  (account)/
    account/page.tsx
    account/security/page.tsx          # password, 2FA, sessions
    account/notifications/page.tsx     # per-type preferences
    account/export/page.tsx            # GDPR export
    account/delete/page.tsx            # GDPR delete
  notifications/page.tsx               # full notifications inbox
  legal/terms/page.tsx
  legal/privacy/page.tsx
  legal/cookies/page.tsx
  legal/acceptable-use/page.tsx
  status/page.tsx                      # public health page
  (candidate)/
    dashboard/page.tsx              # analytics: views, downloads, funnel, etc.
    dashboard/viewers/page.tsx      # full "who viewed you" list (Premium)
    applications/page.tsx
    profile/page.tsx
    resumes/page.tsx
  (employer)/
    employer/jobs/page.tsx
    employer/jobs/new/page.tsx
    employer/jobs/[id]/edit/page.tsx
    employer/jobs/[id]/applicants/page.tsx
    employer/company/page.tsx
  (admin)/
    admin/page.tsx
    admin/users/page.tsx
    admin/jobs/page.tsx
    admin/reports/page.tsx
  (messages)/
    messages/page.tsx                  # inbox
    messages/[conversationId]/page.tsx # thread view
  (billing)/
    pricing/page.tsx              # public pricing page
    upgrade/page.tsx              # gated CTA target
    billing/page.tsx              # manage subscription (links to Stripe portal)
  api/
    auth/[...nextauth]/route.ts
    upload/resume/route.ts        # signed Cloudinary upload
    checkout/route.ts             # create Stripe Checkout session
    portal/route.ts               # create Stripe Customer Portal session
    cron/expire-jobs/route.ts
    cron/weekly-digest/route.ts
    cron/downgrade-expired/route.ts
    cron/rollup-profile-events/route.ts
    cron/unread-message-emails/route.ts
    cron/instant-alerts/route.ts
    cron/daily-alerts/route.ts
    cron/weekly-alerts/route.ts
    cron/verify-dns-txt/route.ts
    cron/finalize-deletions/route.ts
    cron/process-exports/route.ts
    conversations/route.ts
    conversations/[id]/messages/route.ts
    conversations/[id]/read/route.ts
    realtime/auth/route.ts
    upload/attachment/route.ts
    track/route.ts
    webhooks/stripe/route.ts
  sitemap.ts
  robots.ts
  layout.tsx
proxy.ts                          # role/auth guards (Next 16 renamed middleware → proxy)
```

---

## 6. Implementation Roadmap

Build in vertical slices so each milestone is shippable.

### Milestone 1 — Foundations (week 1)
- Set up Prisma + Neon, run first migration.
- Configure Auth.js v5 with Google + GitHub + Credentials.
- Add shadcn/ui base, layout, theme toggle.
- Role-aware `proxy.ts` (Next 16 middleware replacement).
- Sentry, PostHog, `next-intl` scaffolding installed.
- Legal pages stubbed; Terms-accept checkbox on signup.
- Smoke test: sign in, see role-specific dashboard.

### Milestone 2 — Onboarding, verification, job posting (week 2)
- Onboarding wizard (`/welcome/[step]`) for both roles.
- Email verification flow + 2FA (TOTP) setup.
- Company verification (domain-email auto-verify + DNS TXT cron).
- Employer flow: company onboarding → create job (Tiptap).
- Public `/jobs` list with server-side search + filters.
- Job detail page + JSON-LD.
- `sitemap.ts`.

### Milestone 3 — Apply flow + resume parser (week 3)
- Cloudinary signed-upload route for resumes.
- `pdf-parse` extraction → pre-fill profile during onboarding.
- Application form, server action, dedupe.
- Employer applicant list → kanban with status updates.
- Resend emails on apply + status change.
- Notifications inbox (bell + `/notifications`).

### Milestone 4 — Saved searches, alerts, polish (week 4)
- Saved jobs + saved searches with instant/daily/weekly alerts.
- Job recommendations widget.
- Upstash rate limit on auth + apply + messages.
- Vercel cron: expire-jobs, weekly-digest, alert crons.
- SEO pass (OG images, meta).

### Milestone 5 — Admin + compliance (week 5)
- Admin panel, audit log.
- Reports/moderation.
- GDPR: `/account/export`, `/account/delete` with 30-day grace + finalize cron.
- Cookie consent banner.
- Accessibility audit (axe + manual keyboard pass).
- Mobile responsiveness audit; PWA manifest + service worker.

### Milestone 6 — Messaging (week 6, parallel-friendly)
- Schema: `CompanyMember`, `Conversation`, `ConversationParticipant`, `Message`, `Attachment`.
- `/messages` inbox + thread UI with optimistic send.
- Pusher (or Upstash pub/sub + SSE) realtime delivery.
- Read receipts, unread badge, attachments via Cloudinary.
- Cold-outreach gating + per-tier rate limits.
- Resend "you have unread messages" coalesced email cron.
- Moderation hooks (report, block, profanity flag).

### Milestone 7 — Premium / Billing + launch ops (week 7)
- Status page (`/status`) pinging Neon, Upstash, Cloudinary, Resend, Stripe.
- Backup runbook + restore drill.
- CI quality gates: typecheck, lint, vitest, playwright, Lighthouse mobile ≥ 90.
- Stripe products + prices (Candidate Premium, Employer Premium).
- `/pricing` and `/upgrade` pages.
- Checkout + Customer Portal routes.
- Webhook → update `User.plan` / `planRenewsAt`.
- `requirePremium()` server helper + UI gates that match §2 matrix.
- Enforce per-tier limits: saved-jobs cap, resume slots, active job posts, monthly resume downloads (Redis counter).
- `downgrade-expired` cron.

---

## 7. Required Environment Variables

```env
DATABASE_URL=postgres://...neon...
DIRECT_URL=postgres://...neon...                  # for Prisma migrations

AUTH_SECRET=
AUTH_GOOGLE_ID=
AUTH_GOOGLE_SECRET=
AUTH_GITHUB_ID=
AUTH_GITHUB_SECRET=

UPSTASH_REDIS_REST_URL=
UPSTASH_REDIS_REST_TOKEN=

CLOUDINARY_CLOUD_NAME=
CLOUDINARY_API_KEY=
CLOUDINARY_API_SECRET=

RESEND_API_KEY=
EMAIL_FROM="HireHub <noreply@hirehub.app>"

CRON_SECRET=                                       # shared with Vercel cron headers

# Stripe (required for Premium)
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=
STRIPE_PRICE_CANDIDATE_PREMIUM=     # price_xxx for candidate plan
STRIPE_PRICE_EMPLOYER_PREMIUM=      # price_xxx for employer plan
NEXT_PUBLIC_APP_URL=                # used for Checkout success/cancel URLs

# Realtime (if using Pusher)
PUSHER_APP_ID=
PUSHER_KEY=
PUSHER_SECRET=
PUSHER_CLUSTER=
NEXT_PUBLIC_PUSHER_KEY=
NEXT_PUBLIC_PUSHER_CLUSTER=

# Observability
NEXT_PUBLIC_SENTRY_DSN=
SENTRY_AUTH_TOKEN=
NEXT_PUBLIC_POSTHOG_KEY=
NEXT_PUBLIC_POSTHOG_HOST=

# 2FA
TOTP_ENCRYPTION_KEY=                # 32-byte key for AES-GCM of TOTP secrets

# Compliance
TERMS_VERSION=2026-05-04             # bump when terms change to force re-consent
```

---

## 8. Packages to Install

```bash
# Core data + auth
npm i prisma @prisma/client @auth/prisma-adapter next-auth@beta bcryptjs zod

# UI
npm i @radix-ui/react-* lucide-react class-variance-authority tailwind-merge
npx shadcn@latest init

# Editor
npm i @tiptap/react @tiptap/starter-kit @tiptap/extension-link

# Storage + email
npm i cloudinary next-cloudinary resend react-email @react-email/components

# Cache + rate limit
npm i @upstash/redis @upstash/ratelimit

# Realtime chat
npm i pusher pusher-js   # OR rely on Upstash pub/sub + SSE (no extra package)
npm i sanitize-html
npm i -D @types/sanitize-html

# Resume parser + 2FA
npm i pdf-parse otplib qrcode
npm i -D @types/qrcode

# Observability + analytics
npm i @sentry/nextjs posthog-js posthog-node

# i18n + a11y + cookie consent
npm i next-intl vanilla-cookieconsent
npm i -D @axe-core/playwright

# PWA
npm i @serwist/next serwist

# Client state (default stack)
npm i zustand @tanstack/react-query @tanstack/react-query-devtools

# Optional: Redux Toolkit — install only when a slice outgrows Zustand
# npm i @reduxjs/toolkit react-redux

# Billing (Premium)
npm i stripe @stripe/stripe-js

# Dev
npm i -D @types/bcryptjs
```

---

## 9. Security Checklist
- [ ] All Server Actions wrapped in `requireRole(...)` or `requireAuth()`.
- [ ] Zod-validate every input on the server (never trust the client form).
- [ ] Sanitize Tiptap HTML server-side (`sanitize-html` or `DOMPurify` via JSDOM).
- [ ] Resume URLs are **signed** Cloudinary URLs scoped to the requesting user.
- [ ] Rate limit auth, apply, and password-reset endpoints.
- [ ] CSRF: Auth.js handles it for auth; Server Actions are origin-checked.
- [ ] Cron routes check `Authorization: Bearer ${CRON_SECRET}`.
- [ ] Email-verification required before applying to jobs.
- [ ] No secrets in `NEXT_PUBLIC_*` except the Stripe publishable key.

---

## 10. Operational & Compliance Must-Haves

These are launch-blockers, not nice-to-haves.

### 10.1 Error Monitoring — Sentry
- `@sentry/nextjs` (free 5k events/month).
- Auto-capture Server Action errors, Route Handler 5xx, client unhandled rejections.
- Source maps uploaded on deploy via Sentry Vercel integration.
- PII scrubbing on (no email/IP in payloads).

### 10.2 Product Analytics — PostHog
- Self-host or free cloud.
- Track funnel: `signup → profile_completed → job_viewed → applied → application_status_change`.
- Feature flags via PostHog (use for AI features, premium experiments).
- Session replay disabled by default for privacy; opt-in for support.

### 10.3 Status Page
- Static `status.hirehub.app` with `BetterStack`/`Instatus` free tier, or a hand-rolled `/status` page that pings Neon, Upstash, Cloudinary, Resend, Stripe.
- Webhook from Sentry creates incidents automatically on error spikes.

### 10.4 Backups & Disaster Recovery
- Neon: Point-in-Time Restore enabled (free tier supports 24h).
- Weekly logical dump to Cloudinary (encrypted) via cron — keeps 4 weeks.
- Cloudinary: enabled "Backup originals" so deletes are recoverable for 30 days.
- Document a one-page **Restore Drill** runbook; test quarterly.

### 10.5 Legal Pages (required)
- `/legal/terms` — Terms of Service.
- `/legal/privacy` — Privacy Policy (covers GDPR, CCPA).
- `/legal/cookies` — Cookie Policy + a banner (only required if EU/UK traffic; use `vanilla-cookieconsent`).
- `/legal/acceptable-use` — for employers (no discrimination, no fake jobs).
- `/legal/dpa` — Data Processing Addendum (downloadable PDF for enterprise customers).
- All linked from footer; signup form has explicit "I accept Terms + Privacy" checkbox (logged with timestamp + IP).

### 10.6 GDPR / Data Rights
- `/account/export` — Server Action zips JSON of all user data + uploaded files into a Cloudinary signed URL emailed to the user.
- `/account/delete` — soft-delete with 30-day grace, then hard cascade (Prisma `onDelete: Cascade`).
- Audit log entries for every export/delete request.
- Data Processing Records page for admin (who has data, retention policy per table).

### 10.7 Accessibility (WCAG 2.2 AA)
- All shadcn/ui primitives are accessible by default — preserve semantics, don't disable focus rings.
- Manual checklist on apply form, signup, chat, search:
  - Keyboard-only navigation works.
  - All inputs have labels (no placeholder-only).
  - Color contrast ≥ 4.5:1 (Tailwind config audited).
  - `aria-live` for toasts and chat updates.
  - Skip-to-main link on every page.
- CI check: `axe-core` via `@axe-core/playwright` on critical flows.

### 10.8 i18n Scaffolding
- Install `next-intl` from day 1; wrap all user-facing strings in `t('...')`.
- Default locale `en`; structure supports adding `es`, `de`, `fr`, `hi` later without refactor.
- Locale stored on `User.locale`; URL pattern `/[locale]/...` (defer activation until 2nd locale ships).

### 10.9 Mobile-First UX
- Tailwind breakpoints: design at 360px first, scale up.
- Apply form: single-column, large tap targets, file picker uses native `<input capture>`.
- Chat: full-screen on mobile, sticky composer.
- PWA manifest + service worker (`@serwist/next`) for installability and offline shell.
- Lighthouse mobile score ≥ 90 gate in CI.

### 10.10 CI/CD Quality Gates
- GitHub Actions: typecheck, lint, unit tests (`vitest`), e2e (`playwright`), Lighthouse mobile.
- Required to pass before Vercel preview merges to `main`.
- Dependabot for weekly dep PRs.
- Branch protection: linear history, signed commits.

---

## 11. Open Questions / Decisions
1. **Premium pricing** — what monthly $ for Candidate Premium and Employer Premium? Annual discount?
2. Free trial on Premium (e.g., 7 days), or freemium-only?
3. **Magic links** or just OAuth + password?
4. Initial launch locales — English-only, or include another at v1?
5. Realtime transport — Pusher (simpler) vs Upstash pub/sub + SSE (one less vendor)?
6. Should Featured-job placement be a **separate one-off purchase** or bundled into Employer Premium only?
7. Resume parser — heuristics-only (free) or Claude Haiku with prompt caching (paid but better)?
8. 2FA — required for all employers from day 1, or only after first job post?
9. State management — stay on Zustand + TanStack Query (default), or commit to Redux Toolkit upfront because team familiarity?
