# Frontend Architecture

## Overview

The Postiz frontend is built with **Next.js 14+** using the App Router, providing a modern, server-rendered React application with excellent performance and SEO.

## Tech Stack

### Core Framework
- **Next.js**: 14.2.30 (App Router)
- **React**: 18.3.1
- **TypeScript**: 5.5.4
- **Node.js**: 22.12.0+

### UI Libraries
- **Mantine**: 5.10.5 - Component library
- **TailwindCSS**: 4.1.7 - Utility-first CSS
- **@tailwindcss/postcss**: For Next.js integration
- **tailwind-scrollbar**: Custom scrollbars
- **tailwindcss-rtl**: RTL language support

### State Management
- **Zustand**: 5.0.5 - Lightweight state management
- **SWR**: 2.2.5 - Data fetching and caching
- **React Hook Form**: 7.58.1 - Form state management

### Rich Content Editing
- **TipTap**: 3.0.6 - WYSIWYG editor
  - Bold, underline, headings
  - Links and mentions
  - Custom extensions
- **@uiw/react-md-editor**: Markdown editor
- **Emoji Picker React**: Emoji selection

### File Upload
- **Uppy**: 4.4.6 - Modern file uploader
  - Dashboard UI
  - Drag & drop
  - Progress tracking
  - AWS S3 integration
  - Compression support

### Data Visualization
- **Chart.js**: 4.4.1 - Analytics charts
- **React Country Flag**: Country indicators

### AI Integration
- **CopilotKit**: 1.10.6 - AI assistant UI
  - Textarea integration
  - Chat interface
  - Runtime integration

### Utilities
- **dayjs**: Date manipulation
- **lodash**: Utility functions
- **clsx**: Class name composition
- **i18next**: Internationalization
- **react-i18next**: React bindings for i18n
- **accept-language**: Language detection

### Monitoring
- **Sentry**: Error tracking and performance
  - @sentry/nextjs
  - @sentry/react
- **PostHog**: Product analytics

---

## Application Structure

```
apps/frontend/src/
├── app/                          # Next.js App Router
│   ├── (app)/                    # Authenticated app group
│   │   ├── (site)/              # Main application routes
│   │   │   ├── launches/        # Post scheduling
│   │   │   ├── analytics/       # Analytics dashboard
│   │   │   ├── integrations/    # Social account management
│   │   │   ├── media/           # Media library
│   │   │   ├── settings/        # User/org settings
│   │   │   ├── billing/         # Subscription management
│   │   │   ├── agents/          # AI agents
│   │   │   ├── plugs/           # Plugin management
│   │   │   └── third-party/     # Third-party integrations
│   │   ├── (preview)/           # Preview pages
│   │   ├── auth/                # Authentication pages
│   │   └── api/                 # API routes (Next.js)
│   ├── (extension)/             # Browser extension popup
│   ├── layout.tsx               # Root layout
│   ├── page.tsx                 # Landing page
│   └── globals.css              # Global styles
├── components/                   # React components
│   ├── launches/                # Post creation & scheduling
│   ├── analytics/               # Analytics components
│   ├── marketplace/             # Marketplace UI
│   ├── settings/                # Settings forms
│   ├── layout/                  # Layout components
│   ├── ui/                      # Reusable UI components
│   └── ...                      # Feature-specific components
├── middleware.ts                # Next.js middleware
└── instrumentation.ts           # Monitoring setup
```

---

## Routing Architecture

### App Router (Next.js 14+)

#### Route Groups

**`(app)` Group** - Authenticated Routes
- Requires user login
- Shares common layout
- Organization-scoped

**`(site)` Group** - Main Application
- Dashboard and features
- Nested under (app)

**`(extension)` Group** - Browser Extension
- Simplified UI for popup
- Limited functionality

### Key Routes

#### 1. Launches (`/launches`)
**Purpose**: Post creation and scheduling

**Features**:
- Multi-platform post composer
- Rich text editing
- Media attachment
- Scheduling calendar
- Draft management
- AI-powered suggestions

**Pages**:
- `/launches` - Post list/calendar
- `/launches/new` - Create new post
- `/launches/:id` - Edit post
- `/launches/:id/preview` - Preview post

#### 2. Analytics (`/analytics`)
**Purpose**: Performance metrics and insights

**Features**:
- Overview dashboard
- Platform-specific analytics
- Engagement metrics
- Growth trends
- Export data

**Pages**:
- `/analytics` - Overview
- `/analytics/:integrationId` - Platform analytics

#### 3. Integrations (`/integrations`)
**Purpose**: Social media account management

**Features**:
- Connect accounts
- OAuth flow
- Account grouping
- Token refresh
- Disconnect accounts

**Pages**:
- `/integrations` - Connected accounts list
- `/integrations/connect` - Add new account

#### 4. Media (`/media`)
**Purpose**: Media library management

**Features**:
- Upload files
- Browse library
- Search and filter
- Delete media
- Organize folders

#### 5. Settings (`/settings`)
**Purpose**: User and organization configuration

**Pages**:
- `/settings/profile` - User profile
- `/settings/organization` - Org settings
- `/settings/team` - Team management
- `/settings/notifications` - Notification preferences
- `/settings/api` - API key management
- `/settings/billing` - Subscription management

#### 6. Billing (`/billing`)
**Purpose**: Subscription and payment management

**Features**:
- Current plan details
- Upgrade/downgrade
- Payment methods
- Invoice history
- Usage tracking

#### 7. Agents (`/agents`)
**Purpose**: AI agent configuration

**Features**:
- AI workflow management
- Agent training
- Automation rules

#### 8. Third Party (`/third-party`)
**Purpose**: External integrations

**Features**:
- N8N workflows
- Make.com integrations
- Zapier connections
- Custom webhooks

---

## Component Architecture

### Component Organization

```
components/
├── launches/                     # Post Creation
│   ├── calendar-view/           # Calendar UI
│   ├── post-editor/             # Rich text editor
│   ├── media-uploader/          # File upload
│   ├── platform-selector/       # Platform picker
│   └── scheduling/              # Date/time picker
├── analytics/
│   ├── overview-chart/          # Main chart
│   ├── metrics-card/            # Stat cards
│   └── platform-breakdown/      # Per-platform stats
├── marketplace/
│   ├── post-browser/            # Browse posts
│   ├── offer-form/              # Make offers
│   └── transaction-list/        # Purchase history
├── settings/
│   ├── profile-form/            # User profile
│   ├── team-management/         # Invite members
│   └── api-keys/                # API key management
├── layout/
│   ├── sidebar/                 # Navigation sidebar
│   ├── topbar/                  # Header
│   ├── modal/                   # Modal dialogs
│   └── loading/                 # Loading states
└── ui/                          # Reusable Components
    ├── button/
    ├── input/
    ├── select/
    ├── modal/
    ├── tooltip/
    └── ...
```

### Component Patterns

#### 1. Server Components (Default)
Used for:
- Static content
- Data fetching
- SEO-critical pages

```typescript
// Server Component
export default async function AnalyticsPage() {
  const data = await fetchAnalytics();
  return <AnalyticsChart data={data} />;
}
```

#### 2. Client Components
Used for:
- Interactivity
- Browser APIs
- State management

```typescript
'use client';

export function PostEditor() {
  const [content, setContent] = useState('');
  // ... interactive logic
}
```

#### 3. Hybrid Approach
Server component wrapping client components:

```typescript
// Server Component
export default async function LaunchesPage() {
  const initialPosts = await getPosts();

  return (
    <div>
      <PostList initialData={initialPosts} /> {/* Client */}
    </div>
  );
}
```

---

## State Management

### 1. Zustand (Global State)

Used for:
- User session
- UI state (sidebar, modals)
- Temporary data

```typescript
// Example store
import { create } from 'zustand';

interface UserStore {
  user: User | null;
  setUser: (user: User) => void;
  logout: () => void;
}

export const useUserStore = create<UserStore>((set) => ({
  user: null,
  setUser: (user) => set({ user }),
  logout: () => set({ user: null }),
}));
```

### 2. SWR (Server State)

Used for:
- API data fetching
- Caching
- Revalidation
- Optimistic updates

```typescript
import useSWR from 'swr';

function PostList() {
  const { data, error, mutate } = useSWR('/api/posts', fetcher);

  if (error) return <Error />;
  if (!data) return <Loading />;

  return <PostGrid posts={data} />;
}
```

### 3. React Hook Form (Form State)

Used for:
- Form management
- Validation
- Error handling

```typescript
import { useForm } from 'react-hook-form';
import { yupResolver } from '@hookform/resolvers/yup';

function CreatePostForm() {
  const { register, handleSubmit, formState: { errors } } = useForm({
    resolver: yupResolver(postSchema),
  });

  const onSubmit = async (data) => {
    await createPost(data);
  };

  return <form onSubmit={handleSubmit(onSubmit)}>...</form>;
}
```

---

## Key Features Implementation

### 1. Post Editor

**Technology**:
- TipTap (rich text)
- React Hook Form (state)
- Uppy (media upload)

**Features**:
- Multi-platform support
- Character count per platform
- Media preview
- Emoji picker
- Hashtag suggestions
- Mention autocomplete
- AI content generation

**Character Limits**:
```typescript
const PLATFORM_LIMITS = {
  twitter: 280,
  linkedin: 3000,
  instagram: 2200,
  facebook: 63206,
  // ... etc
};
```

### 2. Calendar View

**Technology**:
- Custom calendar component
- Drag & drop (react-dnd)
- Date handling (dayjs)

**Features**:
- Month/week/day views
- Drag to reschedule
- Multi-select
- Quick filters
- Color coding by platform

### 3. Analytics Dashboard

**Technology**:
- Chart.js
- SWR for data fetching
- Date range picker

**Metrics Displayed**:
- Impressions
- Engagement rate
- Follower growth
- Top posts
- Platform comparison

### 4. Media Library

**Technology**:
- Uppy Dashboard
- AWS S3 upload
- Image optimization

**Features**:
- Drag & drop upload
- Multiple file selection
- Progress tracking
- Search and filter
- Folder organization

### 5. AI Assistant (Copilot)

**Technology**:
- CopilotKit
- OpenAI integration
- Streaming responses

**Capabilities**:
- Content generation
- Post improvement
- Hashtag suggestions
- Tone adjustment
- Translation

---

## Styling System

### TailwindCSS Configuration

```javascript
// tailwind.config.js
module.exports = {
  content: ['./src/**/*.{js,ts,jsx,tsx}'],
  theme: {
    extend: {
      colors: {
        primary: '#FF6B6B',
        secondary: '#4ECDC4',
        // ... custom colors
      },
    },
  },
  plugins: [
    require('tailwind-scrollbar'),
    require('tailwindcss-rtl'),
  ],
};
```

### Mantine Theme

Custom theme configuration for consistent component styling:

```typescript
const theme = {
  colorScheme: 'dark',
  primaryColor: 'brand',
  fontFamily: 'Inter, sans-serif',
  // ... custom theme
};
```

### Dark Mode Support

Built-in dark mode with system preference detection.

---

## Internationalization (i18n)

### Languages Supported
- English (default)
- Spanish
- French
- German
- Chinese
- Japanese
- And more...

### Implementation

```typescript
import { useTranslation } from 'react-i18next';

function Component() {
  const { t } = useTranslation();

  return <h1>{t('welcome.title')}</h1>;
}
```

### Language Files

```
public/locales/
├── en/
│   └── common.json
├── es/
│   └── common.json
└── ...
```

---

## Authentication Flow

### 1. Login
```
User enters credentials
  ↓
POST /api/auth/login
  ↓
Receive JWT token
  ↓
Store in httpOnly cookie
  ↓
Redirect to /launches
```

### 2. Protected Routes

**Middleware** (`middleware.ts`):
- Checks JWT token
- Validates user session
- Redirects to /auth if unauthenticated

### 3. OAuth Flow

```
Click "Connect Twitter"
  ↓
Redirect to Twitter OAuth
  ↓
User approves
  ↓
Callback to /auth/callback/twitter
  ↓
Save tokens
  ↓
Redirect back to /integrations
```

---

## API Integration

### Fetch Helper

```typescript
// Custom fetch wrapper
async function apiFetch(endpoint: string, options?: RequestInit) {
  const response = await fetch(`${API_URL}${endpoint}`, {
    ...options,
    credentials: 'include', // Include cookies
    headers: {
      'Content-Type': 'application/json',
      ...options?.headers,
    },
  });

  if (!response.ok) throw new Error(response.statusText);
  return response.json();
}
```

### SWR Integration

```typescript
const fetcher = (url: string) => apiFetch(url);

function usePosts() {
  return useSWR('/posts', fetcher);
}
```

---

## Performance Optimizations

### 1. Code Splitting
- Route-based splitting (automatic)
- Dynamic imports for heavy components
- Lazy loading media

### 2. Image Optimization
- Next.js Image component
- Automatic format selection (WebP)
- Responsive images
- Lazy loading

### 3. Caching Strategy
- SWR cache
- Browser cache headers
- Service worker (optional)

### 4. Bundle Optimization
- Tree shaking
- Minification
- Compression (gzip/brotli)

---

## Error Handling

### Error Boundaries

```typescript
'use client';

export function ErrorBoundary({ children }) {
  return (
    <Sentry.ErrorBoundary fallback={<ErrorUI />}>
      {children}
    </Sentry.ErrorBoundary>
  );
}
```

### Toast Notifications

```typescript
import { showNotification } from '@mantine/notifications';

showNotification({
  title: 'Success',
  message: 'Post created successfully',
  color: 'green',
});
```

---

## Testing

### Testing Stack
- Jest
- React Testing Library
- Playwright (E2E)

### Component Testing

```typescript
import { render, screen } from '@testing-library/react';

describe('PostEditor', () => {
  it('renders editor', () => {
    render(<PostEditor />);
    expect(screen.getByRole('textbox')).toBeInTheDocument();
  });
});
```

---

## Build & Deployment

### Build Process

```bash
# Development
pnpm dev:frontend

# Production build
pnpm build:frontend

# Start production server
pnpm start:prod:frontend
```

### Environment Variables

```env
NEXT_PUBLIC_API_URL=http://localhost:3000
NEXT_PUBLIC_SENTRY_DSN=...
NEXT_PUBLIC_POSTHOG_KEY=...
```

### Docker Deployment

Optimized multi-stage Dockerfile for production.

---

## Browser Compatibility

- Chrome 90+
- Firefox 88+
- Safari 14+
- Edge 90+

---

## Accessibility

- ARIA labels
- Keyboard navigation
- Screen reader support
- Focus management
- Color contrast compliance

---

## Next Steps

- See `04-database-models.md` for data structures
- See `05-api-capabilities.md` for API integration examples
