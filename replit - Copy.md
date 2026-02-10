# معيار عوائد

## Overview

معيار عوائد - منصة التحليل وصناعة القرار التسويقي. منصة داخلية للذكاء التسويقي توفر أدوات مدعومة بالذكاء الاصطناعي لتوليد أفكار الحملات، إنشاء المحتوى، تحليل الأعمال، وتخطيط الحملات للسوق السعودي والخليجي.

The application is built as a full-stack TypeScript solution with React frontend and Express backend, featuring session-based authentication, PostgreSQL database storage, and OpenAI-powered AI tools.

## User Preferences

Preferred communication style: Simple, everyday language.

## Recent Changes (January 2026)

### Campaign Performance Analyzer (Jan 14)
- New multi-platform campaign performance analysis tool
- Dynamic platform blocks: Users add/remove platforms (Meta, TikTok, Snapchat, Google)
- Platform-specific metrics input with validation
- Automatic KPI derivation: CPA, ROAS, CTR, CPC, CPM, CVR calculated from raw metrics
- Cross-platform performance comparison and summary
- Unified Arabic output: التقييم العام → الملاحظات الرئيسية → التوصية → لماذا هذا القرار
- Database persistence with history tracking
- Integrated with invisible learning layer for consistency

### Flexible Analysis Modes (Jan 14)
- All AI tools now support two modes:
  - **Full Analysis (تحليل كامل)**: URL-based analysis with highest accuracy
  - **Guided Analysis (تحليل اختياري)**: Manual inputs without URL requirement
- Mode selection screen with card-based UI for choosing analysis type
- Transparency badges showing basis of results:
  - "🎯 مبني على تحليل كامل" for full analysis
  - "📋 مبني على معطيات" for guided inputs
- Guided mode always shows "متوسطة" confidence with disclaimer
- UX hint instead of blocking: "للحصول على أعلى دقة، استخدم التحليل الكامل"
- Backend APIs: /api/analyze-guided, /api/brain/content-guided

### UI/UX Improvements (Jan 14)
- Transparent PNG logos for light/dark modes (no white/dark backgrounds)
- Centered logo in header with proper flexbox balancing
- Back button on all pages (except homepage)
- Password change feature via user dropdown menu
- Admin can edit their own account data

### User Management Updates (Jan 14)
- Removed username field - only email and name required
- Admin can edit user emails
- Changed "عضو فريق" to "أخصائي تسويق" (Marketing Specialist)
- Admin section added to dashboard for quick access

### Auth & Roles System
- Database-backed user management with bcrypt password hashing
- Two roles: Admin (full access) and Team/Marketing Specialist (limited access)
- Admin can create/disable Team accounts and control tool visibility
- Tool-level permissions per user
- Default admin: admin@awaed.com (password set via ADMIN_PASSWORD env var)

### Smart Analyzer (محلل القنوات)
- Redesigned for URL-only input (no manual metrics)
- Qualitative analysis: weak/medium/strong rating
- 3-5 observations with executive tone
- Single recommendation with decision reasoning
- Clear disclaimer about estimated analysis

### Unified Output Structure
- All analysis tools follow same format: Rating → Observations → Recommendation
- "لماذا هذا القرار؟" (Why this decision?) section with:
  - Main reason
  - 2-3 evidence points
  - Risks if action not taken

### AI Performance Optimizations
- Response caching with 5-minute TTL for identical inputs
- Reduced max_tokens (1000 fast mode, 1500 normal)
- Fast mode option for quicker responses
- Minimized prompt verbosity

## System Architecture

### Frontend Architecture
- **Framework**: React 18 with TypeScript
- **Routing**: Wouter for lightweight client-side routing
- **State Management**: TanStack React Query for server state
- **Styling**: Tailwind CSS v4 with CSS variables for theming
- **UI Components**: shadcn/ui component library (New York style) with Radix UI primitives
- **Animations**: Framer Motion for UI transitions
- **Build Tool**: Vite with custom plugins for meta images and Replit integrations
- **Language Direction**: RTL (Right-to-Left) for Arabic interface

### Backend Architecture
- **Runtime**: Node.js with Express.js
- **Language**: TypeScript with ESM modules
- **API Design**: RESTful JSON APIs under `/api/*` routes
- **Session Management**: Express-session with MemoryStore
- **Rate Limiting**: Custom in-memory rate limiting for login attempts and AI requests
- **Build**: esbuild for production bundling with selective dependency bundling

### Database Layer
- **Database**: PostgreSQL
- **ORM**: Drizzle ORM with drizzle-zod for schema validation
- **Schema Location**: `shared/schema.ts` for shared type definitions
- **Migrations**: Drizzle Kit with `db:push` command
- **Tables**: users, knowledge_base, ai_request_logs, business_analyses, channel_analyses, campaign_performance_analyses, learning_logs

### Invisible Learning Layer
- **Purpose**: Auto-logs AI tool usage and retrieves similar past cases to improve response consistency
- **Database Table**: `learning_logs` with TTL-based expiration (90 days default)
- **Auto-Logging**: Captures sanitized inputs/outputs from all AI tools (Business Analyzer, Channel Analyzer, Campaign Brain, Content Studio, Performance Analyzer)
- **Retrieval System**: Fetches up to 3 similar historical cases per request to enhance AI context
- **Data Sanitization**: Automatically removes passwords, API keys, emails, phone numbers, and IDs
- **Admin Controls**: Internal API endpoints for learning status, cleanup, and clear-all
- **Environment Variables**:
  - `LEARNING_ENABLED`: Toggle learning system (default: true)
  - `LEARNING_TTL_DAYS`: Log retention period in days (default: 90)
- **Key Files**: `server/ai/learning.ts`, `shared/schema.ts` (learningLogs table)
- **Visibility**: Completely invisible to users - no UI presence, works behind the scenes

### Authentication System
- **Method**: Session-based authentication with email/password
- **Password Security**: bcrypt with 12 rounds
- **Session Storage**: Server-side sessions with secure cookie handling
- **Role System**: Two roles - admin (full access), team (tool-specific access)
- **Tool Permissions**: Array of allowed tool IDs per user
- **Security**: Login rate limiting (5 attempts, 15-minute lockout)

### AI Integration Architecture
- **Provider**: OpenAI API via Replit AI Integrations
- **Model**: gpt-4o
- **Tools**:
  - Business Analyzer (محلل الأعمال): Website/store analysis for ad readiness
  - Smart Analyzer (محلل القنوات): Social media account qualitative analysis
  - Campaign Brain (ملهم الحملات): AI-powered campaign ideation
  - Content Studio (استوديو المحتوى): Arabic marketing content generation
  - Campaign Planner (مخطط الحملات): Budget and KPI planning
  - Performance Analyzer (محلل الأداء): Multi-platform campaign performance analysis with KPI derivation
- **Context System**: "Awaed Brain" system prompt with Saudi market expertise
- **Rate Limiting**: 20 requests per minute per user
- **Caching**: 5-minute TTL for identical inputs

### Project Structure
```
├── client/              # React frontend
│   ├── src/
│   │   ├── components/  # Reusable UI components
│   │   ├── pages/       # Route page components
│   │   │   ├── admin-users.tsx  # User management (admin only)
│   │   │   ├── smart-analyzer.tsx  # Channel analyzer
│   │   │   └── ...
│   │   ├── lib/         # Utilities and contexts
│   │   │   └── auth-context.tsx  # Auth state with canAccessTool
│   │   └── hooks/       # Custom React hooks
├── server/              # Express backend
│   ├── ai/              # AI route handlers and prompts
│   │   ├── prompts.ts   # Unified output structure prompts
│   │   └── routes.ts    # AI endpoints with caching
│   ├── auth-routes.ts   # Database-backed auth with bcrypt
│   └── storage.ts       # User CRUD operations
├── shared/              # Shared types and schemas
│   └── schema.ts        # User roles, tool permissions
└── migrations/          # Database migrations
```

### Key Design Decisions
1. **Monorepo Structure**: Client and server in single repository with shared types
2. **Arabic-First Design**: RTL layout, Arabic fonts (Cairo, Tajawal), Saudi market focus
3. **Dark/Light Theme**: CSS variables-based theming with localStorage persistence
4. **Analysis-First Workflow**: Business Analyzer must run before other AI tools
5. **Tool-Level Permissions**: Admin controls which tools each team member can access
6. **Qualitative Analysis**: Smart Analyzer uses estimated ratings, not API metrics
7. **Unified Output**: All analysis tools follow same structure for consistency

## External Dependencies

### Database
- **PostgreSQL**: Primary database via `DATABASE_URL` environment variable
- **Connection**: node-postgres (pg) with connection pooling

### AI Services
- **OpenAI API**: Accessed via Replit AI Integrations
  - `AI_INTEGRATIONS_OPENAI_API_KEY`: API key
  - `AI_INTEGRATIONS_OPENAI_BASE_URL`: Custom base URL for Replit proxy
- **Model**: gpt-4o for text generation

### Third-Party Libraries
- **bcryptjs**: Password hashing with 12 rounds
- **zod**: Runtime schema validation for API requests
- **date-fns**: Date formatting utilities

### Replit-Specific Integrations
- **@replit/vite-plugin-runtime-error-modal**: Development error overlay
- **@replit/vite-plugin-cartographer**: Development navigation helper
- **@replit/vite-plugin-dev-banner**: Development environment indicator
- **vite-plugin-meta-images**: OpenGraph image URL handling for deployment

## Admin Setup

To create the initial admin account, set these environment variables:
- `ADMIN_EMAIL`: Admin email address (default: admin@awaed.com)
- `ADMIN_PASSWORD`: Admin password (required - no default for security)

Example:
```
ADMIN_EMAIL=admin@awaed.com
ADMIN_PASSWORD=YourSecurePassword123!
```

The admin account is automatically created on first server start if no admin exists.
