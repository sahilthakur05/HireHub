# HireHub — UI Flows

User journeys rendered as Mermaid flowcharts. Each diagram is a separate flow; pages are rectangles, decisions are diamonds, system actions are rounded.

## 1. Sign-up / sign-in / verify

```mermaid
flowchart TD
    Start([User lands on /]) --> Choice{Has account?}
    Choice -- No --> SignUp[/sign-up page/]
    Choice -- Yes --> SignIn[/sign-in page/]

    SignUp --> Method1{Method}
    Method1 -- Google/GitHub --> OAuth([OAuth roundtrip])
    Method1 -- Email/Password --> Form[Fill form + accept Terms]
    Form --> Hash([bcrypt hash + create User])
    Hash --> SendVerify([Send verify email via Resend])
    SendVerify --> VerifyPending[/dashboard with 'Verify' banner/]
    OAuth --> RolePick

    VerifyPending --> ClickLink([User clicks email link])
    ClickLink --> VerifyOk{Token valid?}
    VerifyOk -- No --> VerifyFail[/Token expired page/]
    VerifyOk -- Yes --> RolePick

    SignIn --> Creds{Auth method}
    Creds -- OAuth --> OAuth
    Creds -- Password --> CheckPwd{Password match?}
    CheckPwd -- No --> RateLimit{5 fails in 15m?}
    RateLimit -- Yes --> Blocked[/Locked out/]
    RateLimit -- No --> SignIn
    CheckPwd -- Yes --> Verified{Email verified?}
    Verified -- No --> VerifyPending
    Verified -- Yes --> RolePick

    RolePick{Role?}
    RolePick -- CANDIDATE --> CandWelcome[/welcome wizard step 1/]
    RolePick -- EMPLOYER --> EmpWelcome[/employer/company/new/]
    RolePick -- ADMIN --> AdminHome[/admin/]
```

## 2. Candidate onboarding → first apply

```mermaid
flowchart TD
    A[/welcome step 1: basic info/] --> B[/welcome step 2: resume upload/]
    B --> Parse([pdf-parse → prefill profile])
    Parse --> C[/welcome step 3: review skills/]
    C --> D[/welcome step 4: alert prefs/]
    D --> Done([Mark onboarded → /dashboard])
    Done --> E[/dashboard with completeness widget/]
    E --> Browse[/jobs list/]
    Browse --> Search[Search + filters]
    Search --> Detail[/jobs/slug/]
    Detail --> Save{Action}
    Save -- Save --> Bookmark([SavedJob row])
    Save -- Apply --> ApplyForm[/jobs/slug/apply/]
    ApplyForm --> SelectResume[Pick resume + cover letter]
    SelectResume --> Submit{requireVerified?}
    Submit -- No --> VerifyNudge[/Verify-first page/]
    Submit -- Yes --> Insert([Application row + dedupe check])
    Insert --> EmailEmp([Email employer])
    EmailEmp --> Confirm[/Application confirmed/]
    Confirm --> Track[/applications page → status timeline/]
```

## 3. Employer post job → review applicants

```mermaid
flowchart TD
    Sign[/Sign in as employer/] --> HasCo{Has Company?}
    HasCo -- No --> NewCo[/employer/company/new/]
    NewCo --> EvalVerify([evaluateVerification: domain match?])
    EvalVerify --> CoSet[/employer/company with badge state/]
    HasCo -- Yes --> EmpJobs[/employer/jobs list/]
    CoSet --> EmpJobs

    EmpJobs --> Action{Action}
    Action -- New --> NewJob[/employer/jobs/new/]
    Action -- Edit --> EditJob[/employer/jobs/id/edit/]
    Action -- Close --> CloseAct([updateJob → CLOSED])
    Action -- View applicants --> AppList[/applicants table/]

    NewJob --> Editor[Tiptap rich-text editor]
    Editor --> Validate{Salary, dates, sanitize HTML}
    Validate -- Invalid --> NewJob
    Validate -- Valid --> InsertJob([Job row + tsvector regenerated])
    InsertJob --> EmpJobs

    AppList --> Tab{Tab}
    Tab -- Table --> Table[/Sortable applicant table/]
    Tab -- Kanban --> Kanban[/Drag-and-drop kanban/]
    Table --> Drawer[/Applicant drawer/]
    Drawer --> Download{Download resume?}
    Download -- Yes --> DLCheck{Daily limit ok?}
    DLCheck -- No --> Blocked[/Premium nudge/]
    DLCheck -- Yes --> Signed([Generate 5-min signed URL])

    Kanban --> Drag([Optimistic move])
    Drag --> Save([updateApplicationStatus])
    Save --> Notify([Email candidate via Resend])
```

## 4. Public job discovery (anonymous + authed)

```mermaid
flowchart LR
    Home[/Home page/] --> Hero[Hero search bar]
    Home --> Latest[Latest jobs]
    Home --> Categories[Top tags]
    Hero --> List[/jobs?q=.../]
    Categories --> List
    Latest --> Detail[/jobs/slug/]
    List --> Filters[/Filters drawer or sidebar/]
    Filters --> List
    List --> Detail
    Detail --> JsonLd([JobPosting JSON-LD for SEO])
    Detail --> Similar[Similar jobs]
    Similar --> Detail
    Detail --> Apply[/Apply CTA/]
    Apply --> Auth{Logged in?}
    Auth -- No --> SignIn[/sign-in?next=apply-url/]
    Auth -- Yes --> ApplyForm[/Apply page/]
```

## 5. Chat (candidate ↔ company)

```mermaid
flowchart TD
    Trigger{Open thread from}
    Trigger -- Job page --> OpenA[Click 'Message company']
    Trigger -- Inbox --> OpenB[Click conversation row]
    Trigger -- Applicant card --> OpenC[Click 'Chat with candidate']

    OpenA --> CheckPerm{Applied to a job at this co?}
    CheckPerm -- No --> Locked[/Apply first or upgrade Premium/]
    CheckPerm -- Yes --> OpenOrFind([openOrFindConversation Server Action])
    OpenC --> OpenOrFind
    OpenOrFind --> AddPart([Add candidate + OWNER/ADMIN/RECRUITER/HR as participants])
    AddPart --> Thread[/messages/conversationId/]
    OpenB --> Thread

    Thread --> Sub([Subscribe to private-conversation channel via Pusher])
    Thread --> MarkRead([markRead → lastReadAt = now])
    Thread --> Composer[Composer textarea]
    Composer --> Send{Within rate limit?}
    Send -- No --> Throttled[/Throttled toast/]
    Send -- Yes --> Optim([Optimistic append])
    Optim --> Sanitize([sanitize-html + insert Message])
    Sanitize --> Push([pusher.trigger message:new])
    Push --> Recipients{Recipient online?}
    Recipients -- Yes --> RtUpdate([Live UI update for everyone subscribed])
    Recipients -- No --> Cron([cron unread-message-emails fires after 10min])
    Cron --> Email([Coalesced email via Resend])
```

## 6. Candidate analytics dashboard

```mermaid
flowchart LR
    Dash[/dashboard/] --> Cache{Redis cache hit?}
    Cache -- Yes --> Render[/Render widgets/]
    Cache -- No --> Q1([Query ProfileEvent grouped by day])
    Q1 --> Q2([Query Applications grouped by status])
    Q2 --> Q3([Query SavedJob count])
    Q3 --> Q4([Compute completeness])
    Q4 --> Set([Set Redis key TTL 60s])
    Set --> Render

    Render --> Views[Profile views sparkline]
    Render --> DLs[Resume downloads]
    Render --> Funnel[Application funnel]
    Render --> WhoView[Who viewed you]
    Render --> Skill[Skill demand]
    Render --> Comp[Profile completeness]

    WhoView --> Tier{Plan tier}
    Tier -- FREE --> Blur[Blurred names + count]
    Tier -- PREMIUM --> Full[Full company names]
```

## 7. Premium upgrade

```mermaid
flowchart TD
    Gate{Hit a Premium feature on FREE} --> Redirect[/upgrade/]
    Redirect --> Pricing[/pricing page or upgrade page/]
    Pricing --> Pick{Pick plan}
    Pick -- Candidate Premium --> Checkout1([POST /api/checkout])
    Pick -- Employer Premium --> Checkout2([POST /api/checkout])
    Checkout1 --> Stripe[/Stripe Checkout hosted page/]
    Checkout2 --> Stripe
    Stripe --> Payment{Payment success?}
    Payment -- No --> Cancel[/upgrade with banner/]
    Payment -- Yes --> Webhook([Stripe webhook → /api/webhooks/stripe])
    Webhook --> Update([Update User.plan + planRenewsAt + ids])
    Update --> Notif([Notification: subscription renewed])
    Notif --> Back[/Original gated page now unlocked/]

    Renew{Daily cron downgrade-expired} --> CheckSub{planRenewsAt < now?}
    CheckSub -- Yes --> Downgrade([Set plan = FREE])
    Downgrade --> EmailExp([Email subscription_expired])
```

## 8. Admin moderation

```mermaid
flowchart TD
    AdminIn[/Sign in as ADMIN/] --> Hub[/admin dashboard/]
    Hub --> Users[/admin/users/]
    Hub --> Jobs[/admin/jobs/]
    Hub --> Reports[/admin/reports/]
    Hub --> Audit[/admin/audit/]

    Users --> Pick[Search user]
    Pick --> ActU{Action}
    ActU -- Suspend --> SetSusp([User.suspendedAt = now])
    ActU -- Delete --> HardDel([Cascade delete via Prisma])
    SetSusp --> Log([auditLog row])
    HardDel --> Log

    Jobs --> ActJ{Action}
    ActJ -- Force close --> CloseJ([Job.status = CLOSED])
    ActJ -- Feature toggle --> Feat([Job.featured = !featured])
    CloseJ --> Log
    Feat --> Log

    Reports --> Triage[Pick a flagged item]
    Triage --> Decide{Decide}
    Decide -- Dismiss --> Mark([Mark resolved])
    Decide -- Action user --> Users
    Decide -- Action job --> Jobs
```

## 9. GDPR account-deletion lifecycle

```mermaid
flowchart LR
    User[/account/delete page/] --> Confirm[Type email to confirm]
    Confirm --> SoftDel([Set deletedAt + deleteScheduledFor + signout])
    SoftDel --> EmailC([Confirmation email with cancel link])
    EmailC --> Wait{Within 30 days?}
    Wait -- User clicks cancel --> Restore([Clear deletedAt fields])
    Wait -- Grace expires --> Cron([cron finalize-deletions])
    Cron --> Hard([Hard delete user + cascade])
    Hard --> Audit([AuditLog: SYSTEM deletion])
```

## 10. Cron-jobs map (reference)

```mermaid
flowchart LR
    subgraph "Hourly / sub-hourly"
        C1[*/10 unread-message-emails]
        C2[*/15 instant-alerts]
        C3[*/5 process-exports]
        C4[0 * expire-jobs]
    end
    subgraph "Daily"
        D1[0 8 daily-alerts]
        D2[0 8 expiring-soon]
        D3[0 1 finalize-deletions]
        D4[0 2 rollup-profile-events]
        D5[0 3 cleanup-unverified]
        D6[0 4 downgrade-expired]
        D7[0 5 verify-dns-txt]
    end
    subgraph "Weekly"
        W1[Mon 09 weekly-alerts / digest]
    end
```

## How to read these

- **Rectangles** are pages or screens (something the user sees).
- **Rounded rectangles** are system actions (DB writes, emails, webhooks).
- **Diamonds** are decisions/branches.
- Edges are labeled with the trigger or condition.
- Anywhere you see "v1.1" — that path is deferred per the trimmed scope in `PLAN.md`.
