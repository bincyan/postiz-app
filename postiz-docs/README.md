# Postiz Application Documentation

This directory contains comprehensive documentation for the Postiz application, an open-source AI social media scheduling tool.

## Documentation Overview

This documentation suite provides detailed information about:

1. **Project Structure** - Architecture and organization of the codebase
2. **Backend API** - REST API endpoints, controllers, and services
3. **Frontend Architecture** - Next.js application structure and components
4. **Database Models** - Prisma schema and data relationships
5. **API Capabilities** - What you can do with the Postiz API

## What is Postiz?

Postiz is a comprehensive social media management platform that enables:
- Scheduling posts across multiple social media platforms
- AI-powered content generation
- Analytics and performance tracking
- Team collaboration features
- Marketplace for buying/selling posts
- API automation support

## Supported Platforms

- X (Twitter)
- Instagram
- YouTube
- LinkedIn
- Reddit
- TikTok
- Facebook
- Pinterest
- Threads
- Discord
- Slack
- Mastodon
- Bluesky
- And more...

## Documentation Files

- `01-project-structure.md` - Detailed breakdown of the monorepo structure
- `02-backend-api.md` - Backend API architecture and endpoints
- `03-frontend-architecture.md` - Frontend application structure
- `04-database-models.md` - Database schema and relationships
- `05-api-capabilities.md` - Comprehensive API usage guide

## Tech Stack

### Backend
- **Framework**: NestJS (Node.js)
- **Database**: PostgreSQL with Prisma ORM
- **Queue**: Redis with BullMQ
- **API Documentation**: Swagger/OpenAPI
- **Authentication**: JWT-based authentication
- **Email**: Resend for notifications

### Frontend
- **Framework**: Next.js 14+ (React 18)
- **UI Libraries**: Mantine, TailwindCSS
- **State Management**: Zustand, SWR
- **Forms**: React Hook Form with Yup validation
- **Rich Text**: TipTap editor
- **File Upload**: Uppy

### Infrastructure
- **Monorepo**: NX workspace
- **Package Manager**: pnpm
- **Node Version**: 22.12.0
- **TypeScript**: 5.5.4

## Quick Navigation

- For API integration → See `05-api-capabilities.md`
- For database queries → See `04-database-models.md`
- For backend development → See `02-backend-api.md`
- For frontend development → See `03-frontend-architecture.md`
- For project overview → See `01-project-structure.md`

## Repository Information

- **License**: AGPL-3.0
- **GitHub**: https://github.com/gitroomhq/postiz-app
- **Documentation**: https://docs.postiz.com
- **Public API**: https://docs.postiz.com/public-api

## Getting Started

Refer to the main repository README for installation and setup instructions:
https://docs.postiz.com/quickstart
