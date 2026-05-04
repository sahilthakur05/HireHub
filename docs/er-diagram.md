# HireHub — Entity-Relationship Diagram

All Mermaid `erDiagram` blocks render natively on GitHub. The schema is split into 5 logical groups so each is readable; cross-group FKs are annotated under each diagram.

## 1. Identity & Auth

```mermaid
erDiagram
    User ||--o{ Account             : "has OAuth links"
    User ||--o{ Session             : "has sessions"
    User ||--o| CandidateProfile    : "if role=CANDIDATE"
    User ||--o| Company             : "if role=EMPLOYER (owner)"
    User ||--o{ CompanyMember       : "may belong to companies"
    VerificationToken }o--|| User   : "by email"

    User {
        string id PK
        string email UK
        datetime emailVerified
        string passwordHash
        string name
        string image
        Role role "CANDIDATE | EMPLOYER | ADMIN"
        PlanTier plan "FREE | PREMIUM"
        datetime planRenewsAt
        string stripeCustomerId UK
        string stripeSubscriptionId UK
        int onboardingStep
        boolean twoFactorEnabled
        string totpSecret "encrypted"
        string_array recoveryCodes "hashed"
        datetime suspendedAt
        datetime deletedAt
        datetime deleteScheduledFor
        datetime termsAcceptedAt
        string termsVersion
        string locale
        datetime createdAt
    }

    Account {
        string id PK
        string userId FK
        string provider "google | github | credentials"
        string providerAccountId
        string access_token
        string refresh_token
        int expires_at
    }

    Session {
        string id PK
        string sessionToken UK
        string userId FK
        datetime expires
    }

    VerificationToken {
        string identifier "email"
        string token UK
        datetime expires
    }
```

## 2. Candidate side

```mermaid
erDiagram
    User ||--o| CandidateProfile     : has
    CandidateProfile ||--o{ Resume   : owns
    User ||--o{ Application          : submits
    User ||--o{ SavedJob             : bookmarks
    User ||--o{ SavedSearch          : creates
    Application }o--|| Job           : targets
    SavedJob }o--|| Job              : refers
    Resume ||--o{ Application        : "snapshot used in"

    CandidateProfile {
        string id PK
        string userId FK,UK
        string headline
        string bio
        string location
        string_array skills "lowercased"
        json portfolioLinks
    }

    Resume {
        string id PK
        string candidateId FK
        string url "Cloudinary public_id"
        string filename
        string extractedText "for FTS"
        json parsedFields "skills, experience, education"
        datetime createdAt
    }

    Application {
        string id PK
        string jobId FK
        string userId FK
        string resumeUrl "snapshotted"
        string coverLetter
        AppStatus status "APPLIED | REVIEWING | INTERVIEW | OFFER | REJECTED"
        datetime createdAt
    }

    SavedJob {
        string userId PK,FK
        string jobId PK,FK
        datetime createdAt
    }

    SavedSearch {
        string id PK
        string userId FK
        string name
        json query "filters serialized"
        AlertFrequency frequency "OFF | INSTANT | DAILY | WEEKLY"
        datetime lastSentAt
        datetime createdAt
    }
```

Cross-group: `Application.userId → User.id`, `SavedJob.userId → User.id`.

## 3. Employer side

```mermaid
erDiagram
    User ||--o| Company            : "owns (1:1)"
    Company ||--o{ CompanyMember   : has
    User ||--o{ CompanyMember      : "joins via"
    Company ||--o{ Job             : posts
    Job ||--o{ Application         : receives

    Company {
        string id PK
        string ownerId FK,UK
        string name
        string slug UK
        string logoUrl
        string bannerUrl
        string website
        string description
        string size
        string industry
        datetime verifiedAt
        VerificationMethod verificationMethod "DOMAIN_EMAIL | DNS_TXT | MANUAL"
        string verificationToken
    }

    CompanyMember {
        string id PK
        string companyId FK
        string userId FK
        CompanyRole role "OWNER | ADMIN | RECRUITER | HR | HIRING_MANAGER | MEMBER"
        string invitedBy
        datetime joinedAt
    }

    Job {
        string id PK
        string companyId FK
        string title
        string slug UK
        string descriptionHtml "sanitized"
        string location
        RemoteType remote "REMOTE | HYBRID | ONSITE"
        EmploymentType employmentType "FULL_TIME | PART_TIME | CONTRACT | INTERNSHIP"
        Experience experience "JUNIOR | MID | SENIOR | LEAD"
        int salaryMin
        int salaryMax
        string currency
        string_array tags
        JobStatus status "OPEN | CLOSED | DRAFT"
        boolean featured
        datetime featuredUntil
        datetime expiresAt
        int viewsCount
        tsvector searchVector "GIN-indexed"
        datetime createdAt
    }
```

## 4. Messaging / Chat

```mermaid
erDiagram
    User ||--o{ Conversation              : "candidate side"
    Company ||--o{ Conversation           : "company side"
    Conversation ||--o{ ConversationParticipant : has
    User ||--o{ ConversationParticipant   : "joined as"
    Conversation ||--o{ Message           : contains
    User ||--o{ Message                   : sends
    Message ||--o{ Attachment             : "has (v1.1)"
    Job ||--o{ Conversation               : "optional context"
    Application ||--o{ Conversation       : "optional context"

    Conversation {
        string id PK
        string candidateId FK "User.id"
        string companyId FK
        string jobId FK "nullable"
        string applicationId FK "nullable"
        ConvStatus status "OPEN | ARCHIVED | BLOCKED"
        datetime lastMessageAt
        datetime createdAt
    }

    ConversationParticipant {
        string id PK
        string conversationId FK
        string userId FK
        Side side "CANDIDATE | COMPANY"
        datetime lastReadAt
        boolean muted
    }

    Message {
        string id PK
        string conversationId FK
        string senderId FK "User.id"
        string body "sanitized text"
        datetime createdAt
        datetime editedAt
        datetime deletedAt
    }

    Attachment {
        string id PK
        string messageId FK
        string url "Cloudinary"
        string filename
        string mime
        int size
    }
```

Unique: `Conversation` is unique on `(candidateId, companyId, jobId)` — one thread per candidate-company-job tuple.

## 5. Cross-cutting (events, notifications, audit, GDPR)

```mermaid
erDiagram
    User ||--o{ Notification         : receives
    User ||--o{ AuditLog             : "actor of"
    User ||--o{ DataExportRequest    : requests
    CandidateProfile ||--o{ ProfileEvent : "subject of"
    User ||--o{ ProfileEvent         : "actor (nullable)"
    Company ||--o{ ProfileEvent      : "actor company (nullable)"

    Notification {
        string id PK
        string userId FK
        NotificationType type "PROFILE_VIEW | RESUME_DOWNLOAD | NEW_MESSAGE | APP_STATUS_CHANGE | JOB_MATCH | COMPANY_VERIFIED | SUBSCRIPTION_RENEWED | SUBSCRIPTION_EXPIRING | SYSTEM"
        string title
        string body
        string link
        datetime readAt
        datetime createdAt
    }

    ProfileEvent {
        string id PK
        string candidateId FK
        string actorUserId FK "nullable"
        string actorCompanyId FK "denormalized"
        ActorRole actorRole "RECRUITER | HR | HIRING_MANAGER | OTHER"
        EventType type "PROFILE_VIEW | RESUME_DOWNLOAD | SEARCH_IMPRESSION | CONTACT_CLICK"
        json meta
        datetime createdAt
    }

    AuditLog {
        string id PK
        string actorId FK
        string action
        string target
        string ip
        string userAgent
        json meta
        datetime createdAt
    }

    DataExportRequest {
        string id PK
        string userId FK
        ExportStatus status "PENDING | PROCESSING | READY | FAILED"
        string downloadUrl "signed Cloudinary URL"
        datetime createdAt
        datetime completedAt
    }
```

## 6. Combined high-level overview

This is the same data, abstracted to entity-level relationships only.

```mermaid
erDiagram
    User ||--o| CandidateProfile : has
    User ||--o| Company          : owns
    User ||--o{ CompanyMember    : joins
    Company ||--o{ CompanyMember : has
    Company ||--o{ Job           : posts
    Job ||--o{ Application       : receives
    User ||--o{ Application      : submits
    CandidateProfile ||--o{ Resume : owns
    User ||--o{ SavedJob         : bookmarks
    Job ||--o{ SavedJob          : referenced_by
    User ||--o{ SavedSearch      : creates
    User ||--o{ Conversation     : "as candidate"
    Company ||--o{ Conversation  : "as company"
    Conversation ||--o{ Message  : contains
    Conversation ||--o{ ConversationParticipant : has
    User ||--o{ ConversationParticipant : "joined as"
    Job ||--o{ Conversation      : "context"
    User ||--o{ Notification     : receives
    User ||--o{ AuditLog         : actor
    CandidateProfile ||--o{ ProfileEvent : tracked_in
    User ||--o{ DataExportRequest : requests
```

## Index notes (not visible in diagrams)

- `Job.searchVector` — GIN index for full-text search.
- `Job(status, createdAt DESC)` — list pagination.
- `ProfileEvent(candidateId, createdAt DESC)` — dashboard reads.
- `Notification(userId, readAt, createdAt DESC)` — bell badge query.
- `Conversation(candidateId, lastMessageAt DESC)` and `(companyId, lastMessageAt DESC)` — inbox lists.
- `Message(conversationId, createdAt)` — thread paging.
- `AuditLog(actorId, createdAt DESC)` and `(action, createdAt DESC)` — admin search.
