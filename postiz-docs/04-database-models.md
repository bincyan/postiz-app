# Database Models

## Overview

Postiz uses **PostgreSQL** as its primary database with **Prisma** as the ORM. The schema is well-structured with proper relationships, indexes, and constraints.

## Database Configuration

**ORM**: Prisma 6.5.0
**Database**: PostgreSQL
**Schema Location**: `libraries/nestjs-libraries/src/database/prisma/schema.prisma`
**Migrations**: Managed via `prisma db push`

---

## Core Models

### 1. User

**Purpose**: Represents individual users of the platform

**Fields**:
```prisma
model User {
  id                    String       @id @default(uuid())
  email                 String
  password              String?      # Null for OAuth users
  providerName          Provider     # LOCAL, GITHUB, GOOGLE, etc.
  name                  String?
  lastName              String?
  isSuperAdmin          Boolean      @default(false)
  bio                   String?
  audience              Int          @default(0)
  pictureId             String?
  picture               Media?       # Profile picture
  providerId            String?      # OAuth provider ID
  timezone              Int
  createdAt             DateTime     @default(now())
  updatedAt             DateTime     @updatedAt
  lastReadNotifications DateTime     @default(now())
  inviteId              String?
  activated             Boolean      @default(true)
  marketplace           Boolean      @default(true)
  account               String?
  connectedAccount      Boolean      @default(false)
  lastOnline            DateTime     @default(now())
  ip                    String?
  agent                 String?      # User agent
}
```

**Relationships**:
- `organizations` → Multiple organizations via UserOrganization
- `picture` → Media (profile picture)
- `comments` → Comments made by user
- `items` → ItemUser (user items/features)
- `groupBuyer` → MessagesGroup (as buyer)
- `groupSeller` → MessagesGroup (as seller)
- `orderBuyer` → Orders (as buyer)
- `orderSeller` → Orders (as seller)
- `agencies` → SocialMediaAgency

**Indexes**:
- `email + providerName` (unique)
- `lastReadNotifications`
- `inviteId`
- `account`
- `lastOnline`
- `pictureId`

---

### 2. Organization

**Purpose**: Represents a team/organization (multi-tenancy)

**Fields**:
```prisma
model Organization {
  id           String    @id @default(uuid())
  name         String
  description  String?
  apiKey       String?   # For Public API access
  paymentId    String?
  allowTrial   Boolean   @default(false)
  isTrailing   Boolean   @default(false)
  createdAt    DateTime  @default(now())
  updatedAt    DateTime  @updatedAt
}
```

**Relationships**:
- `users` → UserOrganization (members)
- `media` → Media library
- `subscription` → Subscription (one-to-one)
- `Integration` → Connected social accounts
- `post` → Posts created
- `submittedPost` → Posts submitted for org
- `Comments` → Comments on posts
- `notifications` → Notifications
- `usedCodes` → Promo codes used
- `credits` → AI credits
- `customers` → Customer groups
- `webhooks` → Webhook configurations
- `tags` → Post tags
- `signatures` → Post signatures
- `autoPost` → Auto-posting rules
- `sets` → Posting schedule sets
- `thirdParty` → Third-party integrations
- `errors` → Error logs

---

### 3. UserOrganization (Join Table)

**Purpose**: Links users to organizations with roles

**Fields**:
```prisma
model UserOrganization {
  id             String       @id @default(uuid())
  userId         String
  organizationId String
  disabled       Boolean      @default(false)
  role           Role         @default(USER)  # SUPERADMIN, ADMIN, USER
  createdAt      DateTime     @default(now())
  updatedAt      DateTime     @updatedAt
}
```

**Unique Constraint**: `userId + organizationId`

---

### 4. Integration

**Purpose**: Represents connected social media accounts

**Fields**:
```prisma
model Integration {
  id                    String     @id @default(cuid())
  internalId            String     # Platform-specific ID
  organizationId        String
  name                  String     # Display name
  picture               String?    # Profile picture URL
  providerIdentifier    String     # e.g., "twitter", "instagram"
  type                  String     # Account type
  token                 String     # OAuth access token
  disabled              Boolean    @default(false)
  tokenExpiration       DateTime?
  refreshToken          String?
  profile               String?    # JSON profile data
  postingTimes          String     @default("[{\"time\":120}, {\"time\":400}, {\"time\":700}]")
  customInstanceDetails String?    # For self-hosted platforms
  customerId            String?    # Customer grouping
  rootInternalId        String?    # Parent integration
  additionalSettings    String?    @default("[]")
  inBetweenSteps        Boolean    @default(false)
  refreshNeeded         Boolean    @default(false)
  deletedAt             DateTime?
  createdAt             DateTime   @default(now())
  updatedAt             DateTime?  @updatedAt
}
```

**Relationships**:
- `organization` → Organization
- `posts` → Posts published via this integration
- `customer` → Customer (grouping)
- `plugs` → Plugs (extensions)
- `webhooks` → IntegrationsWebhooks

**Indexes**:
- `organizationId + internalId` (unique)
- `providerIdentifier`
- `customerId`
- `refreshNeeded`
- `disabled`

**Supported Providers**:
- X (Twitter)
- Instagram
- Facebook
- LinkedIn
- TikTok
- YouTube
- Pinterest
- Reddit
- Discord
- Slack
- Mastodon
- Bluesky
- Threads
- And more...

---

### 5. Post

**Purpose**: Represents scheduled or published social media posts

**Fields**:
```prisma
model Post {
  id                         String    @id @default(cuid())
  state                      State     @default(QUEUE)  # QUEUE, PUBLISHED, ERROR, DRAFT
  publishDate                DateTime
  organizationId             String
  integrationId              String
  content                    String    # Post content
  group                      String    # Grouping identifier
  title                      String?
  description                String?
  parentPostId               String?   # For threads
  releaseId                  String?
  releaseURL                 String?
  settings                   String?   # JSON settings
  image                      String?   # Media URLs
  submittedForOrderId        String?   # Marketplace order
  submittedForOrganizationId String?
  approvedSubmitForOrder     APPROVED_SUBMIT_FOR_ORDER @default(NO)
  lastMessageId              String?
  intervalInDays             Int?      # For recurring posts
  error                      String?   # Error message if failed
  createdAt                  DateTime  @default(now())
  updatedAt                  DateTime  @updatedAt
  deletedAt                  DateTime?
}
```

**Relationships**:
- `organization` → Organization
- `integration` → Integration (platform)
- `parentPost` → Post (for threads)
- `childrenPost` → Post[] (thread replies)
- `submittedForOrder` → Orders (marketplace)
- `submittedForOrganization` → Organization
- `comments` → Comments[]
- `tags` → TagsPosts[]
- `errors` → Errors[]

**Indexes**:
- `group`
- `publishDate`
- `state`
- `organizationId`
- `integrationId`
- `parentPostId`

**Post States**:
- `QUEUE` - Scheduled, waiting to publish
- `PUBLISHED` - Successfully published
- `ERROR` - Failed to publish
- `DRAFT` - Saved draft

---

### 6. Media

**Purpose**: File storage for images, videos, and documents

**Fields**:
```prisma
model Media {
  id                 String    @id @default(uuid())
  name               String
  path               String    # S3 key or local path
  organizationId     String
  thumbnail          String?   # Thumbnail URL
  thumbnailTimestamp Int?      # For video thumbnails
  alt                String?   # Alt text for accessibility
  fileSize           Int       @default(0)
  type               String    @default("image")  # image, video, pdf
  createdAt          DateTime  @default(now())
  updatedAt          DateTime  @updatedAt
  deletedAt          DateTime?
}
```

**Relationships**:
- `organization` → Organization
- `userPicture` → User[] (profile pictures)
- `agencies` → SocialMediaAgency[]

**Supported Types**:
- Images: PNG, JPG, GIF, WebP
- Videos: MP4, MOV, AVI
- Documents: PDF

**Indexes**:
- `name`
- `organizationId`
- `type`

---

### 7. Subscription

**Purpose**: Manages organization billing and plan tiers

**Fields**:
```prisma
model Subscription {
  id               String           @id @default(cuid())
  organizationId   String           @unique
  subscriptionTier SubscriptionTier # STANDARD, PRO, TEAM, ULTIMATE
  identifier       String?          # Stripe subscription ID
  cancelAt         DateTime?
  period           Period           # MONTHLY, YEARLY
  totalChannels    Int              # Number of integrations allowed
  isLifetime       Boolean          @default(false)
  createdAt        DateTime         @default(now())
  updatedAt        DateTime         @updatedAt
  deletedAt        DateTime?
}
```

**Subscription Tiers**:
- `STANDARD` - Basic plan
- `PRO` - Professional plan
- `TEAM` - Team plan
- `ULTIMATE` - Enterprise plan

**Periods**:
- `MONTHLY`
- `YEARLY`

---

### 8. Tags

**Purpose**: Categorize and organize posts

**Fields**:
```prisma
model Tags {
  id           String      @id @default(uuid())
  name         String
  color        String      # Hex color code
  orgId        String
  deletedAt    DateTime?
  createdAt    DateTime    @default(now())
  updatedAt    DateTime    @updatedAt
}
```

**Relationships**:
- `organization` → Organization
- `posts` → TagsPosts[] (many-to-many via join table)

---

### 9. Comments

**Purpose**: Internal comments on posts for team collaboration

**Fields**:
```prisma
model Comments {
  id             String       @id @default(uuid())
  content        String
  organizationId String
  postId         String
  userId         String
  createdAt      DateTime     @default(now())
  updatedAt      DateTime     @updatedAt
  deletedAt      DateTime?
}
```

**Relationships**:
- `organization` → Organization
- `post` → Post
- `user` → User (author)

---

### 10. Notifications

**Purpose**: In-app notifications for users

**Fields**:
```prisma
model Notifications {
  id             String       @id @default(uuid())
  organizationId String
  content        String       # Notification message
  link           String?      # Optional link
  createdAt      DateTime     @default(now())
  updatedAt      DateTime     @updatedAt
  deletedAt      DateTime?
}
```

---

## Marketplace Models

### 11. MessagesGroup

**Purpose**: Chat threads between buyers and sellers

**Fields**:
```prisma
model MessagesGroup {
  id                  String   @id @default(uuid())
  buyerOrganizationId String
  buyerId             String
  sellerId            String
  createdAt           DateTime @default(now())
  updatedAt           DateTime @updatedAt
}
```

**Relationships**:
- `buyerOrganization` → Organization
- `buyer` → User
- `seller` → User
- `messages` → Messages[]
- `orders` → Orders[]

---

### 12. Orders

**Purpose**: Marketplace purchase orders

**Fields**:
```prisma
model Orders {
  id             String      @id @default(uuid())
  buyerId        String
  sellerId       String
  status         OrderStatus # PENDING, ACCEPTED, CANCELED, COMPLETED
  messageGroupId String
  captureId      String?     # Payment capture ID
  createdAt      DateTime    @default(now())
  updatedAt      DateTime    @updatedAt
}
```

**Order Statuses**:
- `PENDING` - Awaiting acceptance
- `ACCEPTED` - Seller accepted
- `CANCELED` - Order canceled
- `COMPLETED` - Order fulfilled

---

### 13. OrderItems

**Purpose**: Individual items within an order

**Fields**:
```prisma
model OrderItems {
  id            String      @id @default(uuid())
  orderId       String
  integrationId String
  quantity      Int
  price         Int         # In cents
  description   String?
  createdAt     DateTime    @default(now())
  updatedAt     DateTime    @updatedAt
}
```

---

## Advanced Features Models

### 14. AutoPost

**Purpose**: Automated posting from RSS feeds or rules

**Fields**:
```prisma
model AutoPost {
  id             String       @id @default(uuid())
  organizationId String
  name           String
  url            String       # RSS feed URL
  integrations   String       # JSON array of integration IDs
  template       String       # Post template
  enabled        Boolean      @default(true)
  lastRun        DateTime?
  createdAt      DateTime     @default(now())
  updatedAt      DateTime     @updatedAt
  deletedAt      DateTime?
}
```

---

### 15. Webhooks

**Purpose**: Outgoing webhook configurations

**Fields**:
```prisma
model Webhooks {
  id             String       @id @default(uuid())
  organizationId String
  url            String       # Webhook endpoint
  events         String       # JSON array of event types
  secret         String?      # Webhook signature secret
  enabled        Boolean      @default(true)
  createdAt      DateTime     @default(now())
  updatedAt      DateTime     @updatedAt
  deletedAt      DateTime?
}
```

**Event Types**:
- `post.published`
- `post.failed`
- `integration.connected`
- `integration.disconnected`
- `subscription.updated`

---

### 16. ThirdParty

**Purpose**: External service integrations (N8N, Make.com, etc.)

**Fields**:
```prisma
model ThirdParty {
  id             String       @id @default(uuid())
  organizationId String
  provider       String       # n8n, make, zapier
  apiKey         String
  webhookUrl     String?
  config         String?      # JSON configuration
  enabled        Boolean      @default(true)
  createdAt      DateTime     @default(now())
  updatedAt      DateTime     @updatedAt
  deletedAt      DateTime?
}
```

---

### 17. Signatures

**Purpose**: Post signatures/watermarks

**Fields**:
```prisma
model Signatures {
  id             String       @id @default(uuid())
  organizationId String
  content        String       # Signature text/HTML
  autoAdd        Boolean      # Auto-append to posts
  createdAt      DateTime     @default(now())
  updatedAt      DateTime     @updatedAt
  deletedAt      DateTime?
}
```

---

### 18. Sets

**Purpose**: Predefined posting schedules

**Fields**:
```prisma
model Sets {
  id             String       @id @default(uuid())
  organizationId String
  name           String
  times          String       # JSON array of times
  timezone       String
  createdAt      DateTime     @default(now())
  updatedAt      DateTime     @updatedAt
  deletedAt      DateTime?
}
```

**Example Times Format**:
```json
[
  {"day": 1, "time": "09:00"},
  {"day": 3, "time": "14:00"},
  {"day": 5, "time": "18:00"}
]
```

---

### 19. Credits

**Purpose**: AI feature credits (image generation, etc.)

**Fields**:
```prisma
model Credits {
  id             String       @id @default(uuid())
  organizationId String
  credits        Int          # Remaining credits
  type           String       @default("ai_images")
  createdAt      DateTime     @default(now())
  updatedAt      DateTime     @updatedAt
}
```

**Credit Types**:
- `ai_images` - AI image generation
- `ai_content` - AI content generation
- `ai_analysis` - AI analytics

---

### 20. Errors

**Purpose**: Error logging and debugging

**Fields**:
```prisma
model Errors {
  id             String       @id @default(uuid())
  organizationId String
  postId         String?
  integrationId  String?
  error          String       # Error message
  stackTrace     String?      # Full stack trace
  context        String?      # JSON context data
  resolved       Boolean      @default(false)
  createdAt      DateTime     @default(now())
  updatedAt      DateTime     @updatedAt
}
```

---

### 21. Customer

**Purpose**: Grouping integrations by customer/client

**Fields**:
```prisma
model Customer {
  id           String        @id @default(uuid())
  name         String
  orgId        String
  createdAt    DateTime      @default(now())
  updatedAt    DateTime      @updatedAt
  deletedAt    DateTime?
}
```

**Unique Constraint**: `orgId + name + deletedAt`

---

## Enumerations

### Provider
```prisma
enum Provider {
  LOCAL       # Email/password
  GITHUB      # GitHub OAuth
  GOOGLE      # Google OAuth
  FARCASTER   # Farcaster
  WALLET      # Web3 wallet
  GENERIC     # Generic OAuth
}
```

### Role
```prisma
enum Role {
  SUPERADMIN  # Platform admin
  ADMIN       # Organization admin
  USER        # Regular user
}
```

### State (Post)
```prisma
enum State {
  QUEUE       # Scheduled
  PUBLISHED   # Successfully posted
  ERROR       # Failed to post
  DRAFT       # Saved draft
}
```

### SubscriptionTier
```prisma
enum SubscriptionTier {
  STANDARD    # Basic features
  PRO         # Advanced features
  TEAM        # Team collaboration
  ULTIMATE    # All features
}
```

### Period
```prisma
enum Period {
  MONTHLY
  YEARLY
}
```

### OrderStatus
```prisma
enum OrderStatus {
  PENDING
  ACCEPTED
  CANCELED
  COMPLETED
}
```

---

## Database Relationships Diagram

```
Organization
├── Users (many-to-many via UserOrganization)
├── Integrations (one-to-many)
├── Posts (one-to-many)
├── Media (one-to-many)
├── Subscription (one-to-one)
├── Tags (one-to-many)
├── Comments (one-to-many)
├── Notifications (one-to-many)
├── Webhooks (one-to-many)
├── AutoPost (one-to-many)
├── Credits (one-to-many)
└── Signatures (one-to-many)

Integration
├── Posts (one-to-many)
└── Customer (many-to-one)

Post
├── Integration (many-to-one)
├── Organization (many-to-one)
├── Comments (one-to-many)
├── Tags (many-to-many via TagsPosts)
├── ParentPost (self-reference for threads)
└── ChildrenPost (self-reference for threads)

User
├── Organizations (many-to-many via UserOrganization)
├── Comments (one-to-many)
├── Picture (many-to-one Media)
└── Marketplace activity (orders, messages)
```

---

## Indexes Strategy

### Primary Indexes
- All `id` fields are UUIDs with primary key index
- Composite unique indexes for multi-column uniqueness

### Performance Indexes
- `organizationId` on most tables (tenant isolation)
- `createdAt`, `updatedAt` for time-based queries
- `deletedAt` for soft-delete filtering
- `state`, `publishDate` on Posts (scheduling queries)
- `providerIdentifier` on Integration (platform filtering)

### Relationship Indexes
- Foreign key columns for join performance
- Composite indexes for common query patterns

---

## Soft Deletes

Most models include `deletedAt` field for soft deletion:
- Allows data recovery
- Maintains referential integrity
- Filtered in queries via `WHERE deletedAt IS NULL`

---

## Data Validation

### Prisma Level
- Type checking
- Required fields
- Unique constraints
- Foreign key constraints

### Application Level (DTOs)
- Business logic validation
- Custom validators
- Complex rules

---

## Database Operations

### Common Queries

```typescript
// Find posts for organization
await prisma.post.findMany({
  where: {
    organizationId: 'org-id',
    deletedAt: null,
    state: 'QUEUE',
  },
  include: {
    integration: true,
    tags: true,
  },
  orderBy: {
    publishDate: 'asc',
  },
});

// Get organization with subscription
await prisma.organization.findUnique({
  where: { id: 'org-id' },
  include: {
    subscription: true,
    users: {
      include: {
        user: true,
      },
    },
  },
});
```

---

## Migration Strategy

### Development
```bash
pnpm prisma-db-push  # Push schema changes
```

### Production
- Use Prisma Migrate for production
- Version-controlled migrations
- Rollback capability

---

## Performance Considerations

1. **Connection Pooling**: Prisma manages connection pool
2. **Query Optimization**: Selective field inclusion
3. **Eager Loading**: Use `include` to prevent N+1 queries
4. **Pagination**: Implement cursor-based pagination for large datasets
5. **Caching**: Redis cache for frequently accessed data

---

## Next Steps

- See `05-api-capabilities.md` for API usage examples
- See `02-backend-api.md` for data access patterns
