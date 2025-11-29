# API Capabilities & Usage Guide

## Overview

This document provides comprehensive examples of what you can do with the Postiz API, including both the private authenticated API and the Public API for third-party integrations.

---

## Authentication

### Private API (Web Application)

**Method**: JWT-based authentication via cookies

```typescript
// Login
POST /api/auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "securepassword"
}

// Response
{
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "name": "John Doe"
  },
  "token": "jwt-token"  // Set in httpOnly cookie
}
```

### Public API

**Method**: API Key in Authorization header

```bash
curl -X GET https://api.postiz.com/api/v1/public/integrations \
  -H "Authorization: Bearer YOUR_API_KEY"
```

**Getting Your API Key**:
1. Log into Postiz dashboard
2. Navigate to Settings → API
3. Generate new API key
4. Copy and store securely

---

## Social Media Integration Management

### 1. List Available Platforms

```bash
GET /integrations

# Response
[
  {
    "identifier": "twitter",
    "name": "X (Twitter)",
    "icon": "https://...",
    "supportsAuth": true,
    "features": ["posts", "media", "analytics"]
  },
  {
    "identifier": "instagram",
    "name": "Instagram",
    "icon": "https://...",
    "supportsAuth": true,
    "features": ["posts", "media", "stories"]
  }
  // ... more platforms
]
```

### 2. Connect Social Account (OAuth Flow)

```typescript
// Step 1: Initiate OAuth
POST /integrations/connect
{
  "providerIdentifier": "twitter"
}

// Response
{
  "authUrl": "https://twitter.com/oauth/authorize?..."
}

// Step 2: User authorizes on platform

// Step 3: OAuth callback (automatic)
GET /auth/callback/twitter?code=...

// Step 4: Integration created
{
  "id": "integration-uuid",
  "name": "John's Twitter",
  "providerIdentifier": "twitter",
  "picture": "https://...",
  "connected": true
}
```

### 3. List Connected Accounts

```bash
GET /integrations/list

# Response
{
  "integrations": [
    {
      "id": "uuid-1",
      "name": "Personal Twitter",
      "providerIdentifier": "twitter",
      "picture": "https://...",
      "disabled": false,
      "refreshNeeded": false,
      "customer": {
        "id": "customer-uuid",
        "name": "Personal Accounts"
      }
    },
    {
      "id": "uuid-2",
      "name": "Brand Instagram",
      "providerIdentifier": "instagram",
      "picture": "https://...",
      "disabled": false
    }
  ]
}
```

### 4. Group Integrations by Customer

```bash
PUT /integrations/{integrationId}/customer-name
{
  "customerName": "Client A Accounts"
}

# Response
{
  "success": true,
  "integration": {
    "id": "uuid",
    "customer": {
      "name": "Client A Accounts"
    }
  }
}
```

### 5. Disconnect Account

```bash
DELETE /integrations/{integrationId}

# Response
{
  "success": true,
  "message": "Integration disconnected"
}
```

---

## Post Creation & Scheduling

### 1. Create Simple Post

```typescript
POST /posts
Content-Type: application/json

{
  "content": "Hello World! 🌍",
  "integrations": [
    "twitter-integration-id",
    "linkedin-integration-id"
  ],
  "publishDate": "2025-01-15T10:00:00Z",
  "state": "QUEUE"  // or "DRAFT"
}

// Response
{
  "id": "post-uuid",
  "content": "Hello World! 🌍",
  "state": "QUEUE",
  "publishDate": "2025-01-15T10:00:00Z",
  "integrations": [
    {
      "id": "twitter-integration-id",
      "name": "Twitter Account",
      "providerIdentifier": "twitter"
    },
    {
      "id": "linkedin-integration-id",
      "name": "LinkedIn Profile",
      "providerIdentifier": "linkedin"
    }
  ]
}
```

### 2. Create Post with Media

```typescript
// Step 1: Upload media
POST /media/upload
Content-Type: multipart/form-data

FormData: {
  file: [image file]
}

// Response
{
  "id": "media-uuid",
  "name": "image.jpg",
  "path": "https://...",
  "thumbnail": "https://...",
  "type": "image"
}

// Step 2: Create post with media
POST /posts
{
  "content": "Check out this image!",
  "integrations": ["twitter-id"],
  "publishDate": "2025-01-15T14:00:00Z",
  "image": "https://media-url",  // Or array for multiple images
  "settings": {
    "media": ["media-uuid"]
  }
}
```

### 3. Create Post with Platform-Specific Settings

```typescript
POST /posts
{
  "content": "Platform-specific post",
  "integrations": ["twitter-id", "instagram-id"],
  "publishDate": "2025-01-15T16:00:00Z",
  "settings": {
    "twitter": {
      "replyToTweetId": "tweet-id"  // Create thread
    },
    "instagram": {
      "location": {
        "name": "New York, NY",
        "id": "location-id"
      },
      "altText": "Image description"
    }
  }
}
```

### 4. Create Thread/Multi-Post

```typescript
// Create parent post
POST /posts
{
  "content": "Thread 1/3: Introduction",
  "integrations": ["twitter-id"],
  "publishDate": "2025-01-15T10:00:00Z"
}

// Response: { "id": "parent-post-id" }

// Create child posts
POST /posts
{
  "content": "Thread 2/3: Details",
  "integrations": ["twitter-id"],
  "publishDate": "2025-01-15T10:01:00Z",
  "parentPostId": "parent-post-id"
}

POST /posts
{
  "content": "Thread 3/3: Conclusion",
  "integrations": ["twitter-id"],
  "publishDate": "2025-01-15T10:02:00Z",
  "parentPostId": "parent-post-id"
}
```

### 5. Create Recurring Post

```typescript
POST /posts
{
  "content": "Daily motivation quote",
  "integrations": ["twitter-id"],
  "publishDate": "2025-01-15T09:00:00Z",
  "intervalInDays": 1  // Repeat every day
}
```

### 6. Save as Draft

```typescript
POST /posts
{
  "content": "Draft post content",
  "integrations": ["twitter-id"],
  "state": "DRAFT",
  "publishDate": "2025-01-20T10:00:00Z"
}
```

### 7. AI-Powered Post Generation

```typescript
POST /posts/generate
{
  "prompt": "Write a post about the benefits of social media scheduling",
  "tone": "professional",
  "platform": "linkedin",
  "length": "medium"
}

// Response
{
  "suggestions": [
    {
      "content": "Social media scheduling is a game-changer...",
      "hashtags": ["#SocialMedia", "#Productivity"],
      "emoji": true
    },
    {
      "content": "Maximize your social media impact...",
      "hashtags": ["#Marketing", "#Automation"],
      "emoji": false
    }
  ]
}
```

---

## Post Management

### 1. List Posts

```bash
GET /posts?state=QUEUE&limit=20&offset=0

# Query Parameters:
# - state: QUEUE, PUBLISHED, ERROR, DRAFT
# - integrationId: Filter by platform
# - startDate: ISO date
# - endDate: ISO date
# - limit: Results per page
# - offset: Pagination offset
# - group: Group identifier

# Response
{
  "posts": [
    {
      "id": "post-uuid",
      "content": "...",
      "state": "QUEUE",
      "publishDate": "2025-01-15T10:00:00Z",
      "integration": {...},
      "tags": [...]
    }
  ],
  "total": 150,
  "hasMore": true
}
```

### 2. Get Post Details

```bash
GET /posts/{postId}

# Response
{
  "id": "post-uuid",
  "content": "...",
  "state": "PUBLISHED",
  "publishDate": "2025-01-15T10:00:00Z",
  "integration": {
    "id": "integration-uuid",
    "name": "Twitter Account",
    "providerIdentifier": "twitter"
  },
  "comments": [
    {
      "id": "comment-uuid",
      "content": "Great post!",
      "user": {
        "name": "Team Member"
      },
      "createdAt": "2025-01-14T15:30:00Z"
    }
  ],
  "tags": [
    {
      "name": "Campaign 2025",
      "color": "#FF5733"
    }
  ]
}
```

### 3. Update Post

```bash
PUT /posts/{postId}
{
  "content": "Updated content",
  "publishDate": "2025-01-16T10:00:00Z"
}
```

### 4. Delete Post

```bash
DELETE /posts/{postId}

# Response
{
  "success": true,
  "message": "Post deleted"
}
```

### 5. Add Comment to Post

```bash
POST /posts/{postId}/comments
{
  "comment": "Please review this before publishing"
}

# Response
{
  "id": "comment-uuid",
  "content": "Please review this before publishing",
  "user": {...},
  "createdAt": "2025-01-14T16:00:00Z"
}
```

---

## Tag Management

### 1. Create Tag

```bash
POST /posts/tags
{
  "name": "Q1 Campaign",
  "color": "#3498db"
}

# Response
{
  "id": "tag-uuid",
  "name": "Q1 Campaign",
  "color": "#3498db"
}
```

### 2. List Tags

```bash
GET /posts/tags

# Response
{
  "tags": [
    {
      "id": "tag-uuid-1",
      "name": "Q1 Campaign",
      "color": "#3498db"
    },
    {
      "id": "tag-uuid-2",
      "name": "Product Launch",
      "color": "#e74c3c"
    }
  ]
}
```

### 3. Tag Post

```bash
PUT /posts/{postId}/tags
{
  "tags": ["tag-uuid-1", "tag-uuid-2"]
}
```

---

## Analytics & Insights

### 1. Get Overview Analytics

```bash
GET /analytics/overview?startDate=2025-01-01&endDate=2025-01-31

# Response
{
  "period": {
    "start": "2025-01-01",
    "end": "2025-01-31"
  },
  "totalPosts": 120,
  "totalImpressions": 45000,
  "totalEngagements": 3500,
  "engagementRate": 7.78,
  "followerGrowth": 250,
  "topPerformingPosts": [
    {
      "postId": "uuid",
      "content": "...",
      "impressions": 5000,
      "engagements": 450
    }
  ],
  "platformBreakdown": {
    "twitter": {
      "posts": 40,
      "impressions": 15000,
      "engagements": 1200
    },
    "linkedin": {
      "posts": 30,
      "impressions": 18000,
      "engagements": 1500
    }
  }
}
```

### 2. Get Platform-Specific Analytics

```bash
GET /analytics/integration/{integrationId}?period=30d

# Response
{
  "integration": {
    "id": "uuid",
    "name": "Twitter Account",
    "providerIdentifier": "twitter"
  },
  "metrics": {
    "followers": 5420,
    "followerGrowth": 150,
    "impressions": 25000,
    "engagements": 2000,
    "clicks": 800,
    "engagementRate": 8.0
  },
  "posts": [
    {
      "id": "post-uuid",
      "content": "...",
      "publishDate": "2025-01-15T10:00:00Z",
      "metrics": {
        "impressions": 1500,
        "likes": 45,
        "retweets": 12,
        "replies": 8
      }
    }
  ],
  "trends": {
    "impressions": [
      {"date": "2025-01-01", "value": 800},
      {"date": "2025-01-02", "value": 950}
    ]
  }
}
```

### 3. Get Post Statistics

```bash
GET /posts/{postId}/statistics

# Response
{
  "postId": "uuid",
  "platform": "twitter",
  "metrics": {
    "impressions": 2500,
    "likes": 85,
    "retweets": 23,
    "replies": 12,
    "clicks": 45,
    "engagementRate": 6.6
  },
  "demographics": {
    "topCountries": [
      {"country": "US", "percentage": 45},
      {"country": "UK", "percentage": 20}
    ],
    "ageGroups": [
      {"range": "25-34", "percentage": 35}
    ]
  }
}
```

---

## Media Library

### 1. Upload Media

```bash
POST /media/upload
Content-Type: multipart/form-data

FormData: {
  file: [file],
  alt: "Image description"
}

# Response
{
  "id": "media-uuid",
  "name": "vacation.jpg",
  "path": "https://cdn.postiz.com/...",
  "thumbnail": "https://cdn.postiz.com/thumb-...",
  "type": "image",
  "fileSize": 245678,
  "alt": "Image description"
}
```

### 2. List Media

```bash
GET /media?type=image&limit=20&offset=0

# Response
{
  "media": [
    {
      "id": "uuid-1",
      "name": "image1.jpg",
      "path": "https://...",
      "thumbnail": "https://...",
      "type": "image",
      "fileSize": 245678,
      "createdAt": "2025-01-10T10:00:00Z"
    }
  ],
  "total": 85,
  "hasMore": true
}
```

### 3. Delete Media

```bash
DELETE /media/{mediaId}

# Response
{
  "success": true
}
```

---

## Team Collaboration

### 1. Invite Team Member

```bash
POST /agencies/{orgId}/invite
{
  "email": "newmember@example.com",
  "role": "USER"  // ADMIN or USER
}

# Response
{
  "success": true,
  "invitation": {
    "email": "newmember@example.com",
    "role": "USER",
    "inviteLink": "https://..."
  }
}
```

### 2. List Team Members

```bash
GET /agencies/{orgId}/members

# Response
{
  "members": [
    {
      "userId": "uuid",
      "name": "John Doe",
      "email": "john@example.com",
      "role": "ADMIN",
      "joinedAt": "2024-01-01T00:00:00Z"
    }
  ]
}
```

### 3. Update Member Role

```bash
PUT /agencies/{orgId}/members/{userId}
{
  "role": "ADMIN"
}
```

### 4. Remove Member

```bash
DELETE /agencies/{orgId}/members/{userId}
```

---

## Webhooks

### 1. Create Webhook

```bash
POST /webhooks
{
  "url": "https://your-server.com/webhook",
  "events": ["post.published", "post.failed", "integration.connected"],
  "secret": "your-webhook-secret"
}

# Response
{
  "id": "webhook-uuid",
  "url": "https://your-server.com/webhook",
  "events": ["post.published", "post.failed", "integration.connected"],
  "enabled": true
}
```

### 2. Webhook Event Format

When an event occurs, Postiz sends:

```json
POST https://your-server.com/webhook
X-Postiz-Signature: sha256=...
Content-Type: application/json

{
  "event": "post.published",
  "timestamp": "2025-01-15T10:00:00Z",
  "data": {
    "postId": "uuid",
    "integrationId": "uuid",
    "platform": "twitter",
    "content": "...",
    "publishedUrl": "https://twitter.com/..."
  }
}
```

### 3. Verify Webhook Signature

```typescript
import crypto from 'crypto';

function verifyWebhook(payload: string, signature: string, secret: string): boolean {
  const hmac = crypto.createHmac('sha256', secret);
  const digest = 'sha256=' + hmac.update(payload).digest('hex');
  return crypto.timingSafeEqual(Buffer.from(signature), Buffer.from(digest));
}
```

---

## Automation

### 1. Create RSS Auto-Post

```bash
POST /autopost
{
  "name": "Blog Auto-Post",
  "url": "https://blog.example.com/feed.xml",
  "integrations": ["twitter-id", "linkedin-id"],
  "template": "New blog post: {title} - {link}",
  "enabled": true
}

# Response
{
  "id": "autopost-uuid",
  "name": "Blog Auto-Post",
  "url": "https://blog.example.com/feed.xml",
  "lastRun": null,
  "enabled": true
}
```

### 2. Create Posting Schedule Set

```bash
POST /sets
{
  "name": "Weekday Morning Posts",
  "times": [
    {"day": 1, "time": "09:00"},
    {"day": 2, "time": "09:00"},
    {"day": 3, "time": "09:00"},
    {"day": 4, "time": "09:00"},
    {"day": 5, "time": "09:00"}
  ],
  "timezone": "America/New_York"
}
```

---

## Public API Examples

### Using Node.js SDK

```bash
npm install @postiz/node
```

```typescript
import { Postiz } from '@postiz/node';

const client = new Postiz({
  apiKey: 'your-api-key'
});

// List integrations
const integrations = await client.integrations.list();

// Create post
const post = await client.posts.create({
  content: 'Hello from API!',
  integrations: ['integration-id'],
  publishDate: new Date('2025-01-15T10:00:00Z')
});

// Get analytics
const analytics = await client.analytics.overview({
  startDate: '2025-01-01',
  endDate: '2025-01-31'
});
```

### Using cURL

```bash
# Create post
curl -X POST https://api.postiz.com/api/v1/public/integrations/post \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "Hello from cURL!",
    "integrationId": "integration-uuid",
    "publishDate": "2025-01-15T10:00:00Z"
  }'
```

---

## Rate Limits

### Private API
- Authenticated users: 1000 requests/hour
- Varies by subscription tier

### Public API
- Standard: 100 requests/minute
- Pro: 500 requests/minute
- Team: 1000 requests/minute
- Ultimate: 5000 requests/minute

### Rate Limit Headers
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1705320000
```

---

## Error Handling

### Error Response Format

```json
{
  "statusCode": 400,
  "message": "Validation failed",
  "errors": [
    {
      "field": "content",
      "message": "Content is required"
    },
    {
      "field": "publishDate",
      "message": "Publish date must be in the future"
    }
  ]
}
```

### Common Error Codes

- `400` - Bad Request (validation error)
- `401` - Unauthorized (invalid API key)
- `403` - Forbidden (insufficient permissions)
- `404` - Not Found
- `429` - Too Many Requests (rate limited)
- `500` - Internal Server Error

---

## Best Practices

### 1. Batch Operations
Instead of creating posts one by one, batch them:

```typescript
const posts = [/* array of post data */];
const results = await Promise.all(
  posts.map(post => client.posts.create(post))
);
```

### 2. Error Handling
Always handle API errors gracefully:

```typescript
try {
  const post = await client.posts.create(data);
} catch (error) {
  if (error.statusCode === 429) {
    // Rate limited - wait and retry
    await sleep(60000);
    return retry();
  }
  // Log error and notify user
  console.error(error);
}
```

### 3. Use Webhooks for Real-Time Updates
Instead of polling for post status:

```typescript
// Set up webhook to receive publish notifications
await client.webhooks.create({
  url: 'https://your-app.com/webhook',
  events: ['post.published', 'post.failed']
});
```

### 4. Optimize Media Upload
- Compress images before upload
- Use appropriate formats (WebP for web)
- Generate thumbnails client-side if possible

---

## Integration Examples

### N8N Workflow

Postiz integrates with N8N for advanced automation:

```json
{
  "nodes": [
    {
      "type": "n8n-nodes-postiz",
      "operation": "createPost",
      "parameters": {
        "content": "={{$json.content}}",
        "integrations": ["twitter-id"],
        "publishDate": "={{$json.scheduledTime}}"
      }
    }
  ]
}
```

### Make.com (Integromat)

Available as Make.com module for visual automation.

### Zapier

Connect Postiz with 5000+ apps via Zapier integration.

---

## Use Cases

### 1. Content Curation Bot
```typescript
// Fetch trending content
const trends = await fetchTrends();

// Generate posts using AI
const suggestions = await client.posts.generate({
  prompt: `Create posts about: ${trends.join(', ')}`,
  count: 5
});

// Schedule posts
for (const suggestion of suggestions) {
  await client.posts.create({
    content: suggestion.content,
    integrations: allIntegrations,
    publishDate: getNextSlot()
  });
}
```

### 2. Cross-Platform Campaign
```typescript
// Create campaign across all platforms
const platforms = ['twitter', 'linkedin', 'facebook', 'instagram'];

for (const platform of platforms) {
  const integration = integrations.find(i => i.providerIdentifier === platform);

  await client.posts.create({
    content: customizeForPlatform(content, platform),
    integrations: [integration.id],
    publishDate: campaignStartDate,
    tags: ['campaign-2025']
  });
}
```

### 3. Analytics Dashboard
```typescript
// Fetch analytics for all platforms
const analytics = await Promise.all(
  integrations.map(integration =>
    client.analytics.integration(integration.id, { period: '30d' })
  )
);

// Generate report
const report = generateReport(analytics);
sendEmailReport(report);
```

---

## Support & Resources

- **API Documentation**: https://docs.postiz.com/public-api
- **SDK on NPM**: https://www.npmjs.com/package/@postiz/node
- **N8N Node**: https://www.npmjs.com/package/n8n-nodes-postiz
- **Community Discord**: https://discord.postiz.com

---

## Conclusion

The Postiz API provides comprehensive capabilities for:
- ✅ Multi-platform social media management
- ✅ Automated content scheduling
- ✅ Team collaboration
- ✅ Analytics and insights
- ✅ AI-powered content generation
- ✅ Webhook integrations
- ✅ Third-party automation

Whether you're building a custom dashboard, automating workflows, or integrating with other services, the Postiz API has you covered.
