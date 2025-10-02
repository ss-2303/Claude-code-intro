# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development
- `npm run dev` - Start development server with Turbopack (opens at http://localhost:3000)
- `npm run dev:daemon` - Start dev server in background, logs to logs.txt
- `npm run build` - Build production bundle
- `npm run start` - Start production server
- `npm run lint` - Run ESLint
- `npm run test` - Run Vitest tests

### Database
- `npm run setup` - Install dependencies, generate Prisma client, and run migrations
- `npm run db:reset` - Reset database (force migrate reset)

### Single Test
- `npm run test -- <test-file-pattern>` - Run specific test files

## Architecture

### Core Concept
UIGen is an AI-powered React component generator with live preview. Users describe components they want via chat, and the AI generates React components in a virtual file system with real-time preview.

### Key Components

**Virtual File System (`src/lib/file-system.ts`)**
- Core abstraction for managing files in memory without writing to disk
- `VirtualFileSystem` class handles file operations (create, read, update, delete, rename)
- Files are stored as `FileNode` objects with type, name, path, content, and optional children
- Serialization/deserialization for persistence in database

**AI Chat Integration (`src/app/api/chat/route.ts`)**
- Streams responses using Vercel AI SDK
- Uses Anthropic Claude API (falls back to mock provider without API key)
- Two main tools: `str_replace_editor` for file content and `file_manager` for file operations
- Saves conversation and file state to database for authenticated users

**Context Providers**
- `FileSystemContext` - Manages virtual file system state and operations
- `ChatContext` - Handles chat messages and AI streaming

**Database (Prisma + SQLite)**
- `User` - Authentication with email/password
- `Project` - Stores chat messages and virtual file system state as JSON

**Authentication (`src/lib/auth.ts`)**
- JWT-based sessions using `jose` library
- bcrypt for password hashing
- Middleware for protected routes

### File Structure
```
src/
├── app/                 # Next.js App Router
│   ├── [projectId]/     # Dynamic project pages
│   ├── api/chat/        # AI chat endpoint
│   └── page.tsx         # Home page with project routing
├── components/
│   ├── auth/            # Sign in/up forms and dialogs
│   ├── chat/            # Chat interface, messages, markdown
│   ├── editor/          # Code editor and file tree
│   ├── preview/         # Component preview iframe
│   └── ui/              # shadcn/ui components
├── lib/
│   ├── contexts/        # React contexts for state management
│   ├── tools/           # AI tools for file operations
│   ├── transform/       # JSX transformation utilities
│   └── prompts/         # AI generation prompts
└── actions/             # Server actions for database operations
```

### Tech Stack Integration
- **Next.js 15** with App Router and React Server Components
- **Tailwind CSS v4** for styling with shadcn/ui components
- **Monaco Editor** for code editing with syntax highlighting
- **React 19** with new concurrent features
- **Prisma** ORM with SQLite database
- **Vitest** for testing with jsdom environment

### Key Patterns
- All AI-generated components must export default from `/App.jsx`
- Virtual file system uses `@/` import alias for local files
- Mock provider generates static components when no API key is present
- File system state is persisted to database as serialized JSON
- Live preview updates automatically as AI modifies virtual files