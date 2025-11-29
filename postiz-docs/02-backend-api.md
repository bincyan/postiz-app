# Backend API Architecture

## Overview

The Postiz backend is built with **NestJS**, a progressive Node.js framework. It provides both private authenticated APIs for the web application and a public API for third-party integrations.

## Architecture

### Tech Stack
- **Framework**: NestJS 10.x
- **Runtime**: Node.js 22.12.0+
- **Database**: PostgreSQL with Prisma ORM
- **Queue**: Redis + BullMQ
- **Auth**: JWT (JSON Web Tokens)
- **API Docs**: Swagger/OpenAPI
- **Rate Limiting**: @nestjs/throttler
- **Monitoring**: Sentry

### Application Structure

```
apps/backend/src/
├── api/
│   └── routes/           # Main API controllers
├── public-api/
│   └── routes/           # Public API v1
├── services/
│   └── auth/             # Authentication services
│       └── permissions/  # CASL-based authorization
├── app.module.ts         # Root application module
└── main.ts              # Bootstrap & configuration
```

---

## API Controllers

The backend exposes 22 main controllers, each handling a specific domain:

### 1. Authentication & Users

#### `auth.controller.ts`
**Endpoints**: `/auth/*`

**Functionality**:
- User registration and login
- OAuth callback handling
- Password reset
- Email verification
- Session management
- Token refresh

**Key Endpoints**:
- `POST /auth/register` - User registration
- `POST /auth/login` - User login
- `POST /auth/forgot-password` - Request password reset
- `POST /auth/reset-password` - Reset password with token
- `GET /auth/callback/:provider` - OAuth callback
- `POST /auth/refresh` - Refresh JWT token

#### `users.controller.ts`
**Endpoints**: `/users/*`

**Functionality**:
- User profile management
- User preferences
- Avatar upload
- Timezone settings

---

### 2. Social Media Integration

#### `integrations.controller.ts`
**Endpoints**: `/integrations/*`

**Functionality**:
- Connect social media accounts
- Manage connected integrations
- OAuth flow initiation
- Token refresh
- Integration settings
- Account grouping
- Custom customer names

**Key Endpoints**:
- `GET /integrations` - List all available platforms
- `POST /integrations/connect` - Initiate OAuth connection
- `GET /integrations/:id` - Get integration details
- `PUT /integrations/:id/group` - Update integration group
- `DELETE /integrations/:id` - Disconnect integration
- `POST /integrations/:id/refresh` - Refresh access token
- `GET /integrations/customers` - Get customer list

**Supported Platforms**:
- X (Twitter)
- Instagram
- LinkedIn
- Facebook
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

### 3. Post Management

#### `posts.controller.ts`
**Endpoints**: `/posts/*`

**Functionality**:
- Create and schedule posts
- Edit draft posts
- Delete posts
- Post comments
- Tag management
- AI content generation
- Post statistics
- Marketplace integration

**Key Endpoints**:
- `GET /posts` - List posts (with filters)
- `POST /posts` - Create new post
- `GET /posts/:id` - Get post details
- `PUT /posts/:id` - Update post
- `DELETE /posts/:id` - Delete post
- `POST /posts/:id/comments` - Add comment
- `GET /posts/:id/statistics` - Get post analytics
- `GET /posts/tags` - List tags
- `POST /posts/tags` - Create tag
- `POST /posts/generate` - AI-powered post generation

**Post Scheduling**:
- Immediate publishing
- Scheduled future posts
- Recurring posts
- Draft management

---

### 4. Media Management

#### `media.controller.ts`
**Endpoints**: `/media/*`

**Functionality**:
- File upload (images, videos, PDFs)
- Media library management
- Image processing
- Video transcoding
- File storage (S3 or local)

**Key Endpoints**:
- `POST /media/upload` - Upload file
- `GET /media` - List media files
- `DELETE /media/:id` - Delete media
- `GET /media/:id` - Get media URL

**Supported Formats**:
- Images: PNG, JPG, GIF, WebP
- Videos: MP4, MOV, AVI
- Documents: PDF
- Maximum file size: Configurable

---

### 5. Analytics

#### `analytics.controller.ts`
**Endpoints**: `/analytics/*`

**Functionality**:
- Social media metrics
- Engagement analytics
- Follower growth
- Post performance
- Platform-specific insights

**Key Endpoints**:
- `GET /analytics/overview` - Overall statistics
- `GET /analytics/integration/:id` - Platform-specific analytics
- `GET /analytics/posts` - Post performance metrics
- `GET /analytics/engagement` - Engagement trends

**Metrics Provided**:
- Impressions
- Likes
- Comments
- Shares
- Clicks
- Follower growth
- Engagement rate

---

### 6. AI & Automation

#### `copilot.controller.ts`
**Endpoints**: `/copilot/*`

**Functionality**:
- AI content generation
- Post suggestions
- Hashtag recommendations
- Content optimization
- Image generation

**AI Capabilities**:
- OpenAI GPT integration
- LangChain workflows
- Custom AI agents
- Content personalization

#### `autopost.controller.ts`
**Endpoints**: `/autopost/*`

**Functionality**:
- RSS feed monitoring
- Auto-posting rules
- Content curation
- Scheduling automation

---

### 7. Marketplace

#### `marketplace.controller.ts`
**Endpoints**: `/marketplace/*`

**Functionality**:
- Buy/sell social media posts
- Browse available posts
- Post bidding
- Transaction management

**Key Endpoints**:
- `GET /marketplace/posts` - Browse marketplace
- `POST /marketplace/offer` - Make an offer
- `PUT /marketplace/accept/:id` - Accept offer
- `GET /marketplace/purchases` - View purchases

#### `messages.controller.ts`
**Endpoints**: `/messages/*`

**Functionality**:
- Marketplace messaging
- Buyer-seller communication
- Message threads

---

### 8. Billing & Subscriptions

#### `billing.controller.ts`
**Endpoints**: `/billing/*`

**Functionality**:
- Subscription management
- Plan upgrades/downgrades
- Usage tracking
- Payment history

**Key Endpoints**:
- `GET /billing/subscription` - Current subscription
- `POST /billing/upgrade` - Upgrade plan
- `POST /billing/cancel` - Cancel subscription
- `GET /billing/invoices` - Payment history

#### `stripe.controller.ts`
**Endpoints**: `/stripe/*`

**Functionality**:
- Stripe webhook handling
- Payment processing
- Subscription events

---

### 9. Organization Management

#### `agencies.controller.ts`
**Endpoints**: `/agencies/*`

**Functionality**:
- Multi-organization support
- Team management
- Organization settings
- Member invitations

**Key Endpoints**:
- `GET /agencies` - List organizations
- `POST /agencies` - Create organization
- `POST /agencies/:id/invite` - Invite member
- `PUT /agencies/:id/settings` - Update settings

---

### 10. Settings & Configuration

#### `settings.controller.ts`
**Endpoints**: `/settings/*`

**Functionality**:
- User preferences
- Organization settings
- Notification preferences
- API key management

#### `notifications.controller.ts`
**Endpoints**: `/notifications/*`

**Functionality**:
- In-app notifications
- Email notifications
- Notification preferences
- Mark as read

---

### 11. Advanced Features

#### `sets.controller.ts`
**Endpoints**: `/sets/*`

**Functionality**:
- Predefined posting schedules
- Content templates
- Bulk operations

#### `signature.controller.ts`
**Endpoints**: `/signature/*`

**Functionality**:
- Post signatures
- Branding elements
- Watermarks

#### `webhooks.controller.ts`
**Endpoints**: `/webhooks/*`

**Functionality**:
- Webhook configuration
- Event subscriptions
- Webhook delivery logs

#### `third-party.controller.ts`
**Endpoints**: `/third-party/*`

**Functionality**:
- Third-party integrations
- External service connections
- API bridges

---

## Public API (`/api/v1/public`)

### Purpose
Enables third-party developers to integrate with Postiz programmatically.

### Authentication
Uses API keys for authentication:
```
Authorization: Bearer YOUR_API_KEY
```

### Available Endpoints

#### Public Integrations Controller
**Base Path**: `/api/v1/public/integrations`

**Endpoints**:
- `GET /api/v1/public/integrations` - List connected integrations
- `POST /api/v1/public/integrations/post` - Create and schedule post
- `GET /api/v1/public/integrations/:id/posts` - Get posts for integration

### Rate Limiting
- Default: 100 requests per minute per API key
- Configurable per organization tier

---

## Authentication & Authorization

### JWT Authentication

**Token Structure**:
```json
{
  "userId": "uuid",
  "orgId": "uuid",
  "email": "user@example.com",
  "iat": 1234567890,
  "exp": 1234567890
}
```

**Token Lifetime**:
- Access Token: 1 hour
- Refresh Token: 30 days

### Authorization (CASL)

**Permission System**:
- Resource-based permissions
- Action-based controls (read, create, update, delete)
- Organization-scoped access

**Roles**:
- Owner: Full access
- Admin: Manage members and content
- User: Create and manage own posts
- Viewer: Read-only access

---

## Data Validation

### DTOs (Data Transfer Objects)

Located in `libraries/nestjs-libraries/src/dtos/`

**Validation Stack**:
- `class-validator` - Decorator-based validation
- `class-transformer` - Type transformation
- Custom validators for complex rules

**Example DTO**:
```typescript
export class CreatePostDto {
  @IsString()
  @IsNotEmpty()
  content: string;

  @IsArray()
  @ValidateNested({ each: true })
  integrations: IntegrationDto[];

  @IsDateString()
  @IsOptional()
  scheduledAt?: string;

  @IsArray()
  @IsOptional()
  media?: string[];
}
```

---

## Queue System (BullMQ)

### Job Types

1. **Post Publishing**
   - Scheduled post publication
   - Retry logic for failures
   - Platform-specific formatting

2. **Media Processing**
   - Image optimization
   - Video transcoding
   - Thumbnail generation

3. **Analytics Sync**
   - Periodic metrics fetching
   - Data aggregation

4. **Notifications**
   - Email sending
   - In-app notifications

### Queue Configuration

```typescript
{
  defaultJobOptions: {
    attempts: 3,
    backoff: {
      type: 'exponential',
      delay: 2000
    }
  }
}
```

---

## Error Handling

### HTTP Status Codes

- `200` - Success
- `201` - Created
- `400` - Bad Request (validation error)
- `401` - Unauthorized (missing/invalid token)
- `403` - Forbidden (insufficient permissions)
- `404` - Not Found
- `429` - Too Many Requests (rate limited)
- `500` - Internal Server Error

### Error Response Format

```json
{
  "statusCode": 400,
  "message": "Validation failed",
  "errors": [
    {
      "field": "email",
      "message": "Invalid email format"
    }
  ]
}
```

---

## Database Access Layer

### Prisma Services

Each entity has a dedicated service:

```
libraries/nestjs-libraries/src/database/prisma/
├── posts/
│   └── posts.service.ts
├── integrations/
│   └── integration.service.ts
├── users/
│   └── users.service.ts
└── ...
```

**Service Pattern**:
```typescript
@Injectable()
export class PostsService {
  constructor(private prisma: DatabaseService) {}

  async createPost(data: CreatePostDto) {
    return this.prisma.post.create({ data });
  }

  async getPosts(orgId: string, filters: GetPostsDto) {
    return this.prisma.post.findMany({
      where: { orgId, ...filters },
      include: { integration: true, media: true }
    });
  }
}
```

---

## Social Platform Integration

### Integration Manager

Located: `libraries/nestjs-libraries/src/integrations/`

**Provider Pattern**:
Each social platform has a provider class implementing:
- OAuth flow
- Post publishing
- Media upload
- Analytics fetching
- Profile information

**Example Platforms**:
- `twitter.provider.ts` - X/Twitter
- `instagram.provider.ts` - Instagram
- `linkedin.provider.ts` - LinkedIn
- `facebook.provider.ts` - Facebook
- `tiktok.provider.ts` - TikTok
- ... and 15+ more

### Provider Interface

```typescript
interface SocialProvider {
  authenticate(code: string): Promise<AuthToken>;
  publishPost(post: Post): Promise<PublishResult>;
  uploadMedia(file: Buffer): Promise<MediaId>;
  getAnalytics(period: DateRange): Promise<Analytics>;
  refreshToken(token: RefreshToken): Promise<AuthToken>;
}
```

---

## API Documentation (Swagger)

### Access
- Development: `http://localhost:3000/api`
- Production: `https://api.postiz.com/api`

### Features
- Interactive API explorer
- Request/response schemas
- Authentication testing
- Code generation

---

## Performance Optimizations

### Caching Strategy
- Redis caching for frequently accessed data
- Cache invalidation on updates
- TTL-based expiration

### Database Optimization
- Connection pooling
- Query optimization with indexes
- Eager loading relationships
- Pagination for large datasets

### Rate Limiting
- Per-endpoint limits
- User/organization-based quotas
- Distributed rate limiting (Redis)

---

## Monitoring & Logging

### Sentry Integration
- Error tracking
- Performance monitoring
- Release tracking

### Logging
- Structured logging (JSON)
- Log levels: debug, info, warn, error
- Request/response logging
- Database query logging (development)

---

## Environment Configuration

### Required Variables

```env
# Database
DATABASE_URL=postgresql://user:password@localhost:5432/postiz

# Redis
REDIS_URL=redis://localhost:6379

# JWT
JWT_SECRET=your-secret-key

# OAuth Credentials (per platform)
TWITTER_CLIENT_ID=...
TWITTER_CLIENT_SECRET=...
INSTAGRAM_CLIENT_ID=...
# ... etc

# AWS S3 (optional)
AWS_ACCESS_KEY_ID=...
AWS_SECRET_ACCESS_KEY=...
S3_BUCKET=...

# Email (Resend)
RESEND_API_KEY=...

# Monitoring
SENTRY_DSN=...
```

---

## API Best Practices

### Request Format
- Use JSON for request bodies
- Include `Content-Type: application/json` header
- Use query parameters for filters

### Response Format
- Consistent JSON structure
- ISO 8601 dates
- Pagination metadata

### Security
- Always use HTTPS in production
- Validate all inputs
- Sanitize user content
- Rate limit all endpoints
- Use parameterized queries (Prisma)

---

## Next Steps

- See `03-frontend-architecture.md` for frontend integration
- See `04-database-models.md` for data structures
- See `05-api-capabilities.md` for usage examples
