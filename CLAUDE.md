# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Initial setup (install deps, generate Prisma client, run migrations)
npm run setup

# Development server (with Turbopack)
npm run dev

# Run tests
npm test

# Run a single test file
npx vitest run src/lib/__tests__/file-system.test.ts

# Run tests in watch mode
npx vitest

# Lint
npm run lint

# Build for production
npm run build

# Reset database
npm run db:reset
```

## Code Style

Use comments sparingly. Only comment complex code.

## Architecture

This is an AI-powered React component generator with live preview. Users describe components in chat, Claude generates code, and a live preview renders in an iframe.

### Core Flow

1. **Chat Interface** (`src/components/chat/`) - User sends message describing desired component
2. **API Route** (`src/app/api/chat/route.ts`) - Streams response from Claude using Vercel AI SDK
3. **AI Tools** - Claude uses `str_replace_editor` and `file_manager` tools to create/modify files in the virtual filesystem
4. **Virtual File System** (`src/lib/file-system.ts`) - In-memory filesystem that stores all generated code (nothing written to disk)
5. **Preview** (`src/components/preview/PreviewFrame.tsx`) - Transforms JSX via Babel in browser, creates blob URLs with import maps, renders in sandboxed iframe

### Key Patterns

**Virtual File System**: All files exist only in memory via `VirtualFileSystem` class. Serialized to JSON for persistence in database. Context provider (`FileSystemContext`) distributes filesystem state to components.

**AI Tool Integration**: Two tools available to Claude:
- `str_replace_editor`: view/create/edit files (view, create, str_replace, insert commands)
- `file_manager`: rename/delete files

**Preview Rendering**: JSX is transformed client-side using `@babel/standalone`. Files become blob URLs mapped via ES module import maps. Third-party packages resolve to esm.sh CDN.

**Authentication**: JWT-based sessions (`jose`), passwords hashed with bcrypt. Optional - app works without auth for anonymous users.

### Directory Structure

- `src/app/` - Next.js App Router pages and API routes
- `src/components/` - React components (chat, editor, preview, auth, ui)
- `src/lib/` - Core utilities:
  - `file-system.ts` - VirtualFileSystem class
  - `transform/jsx-transformer.ts` - Babel transforms and import map generation
  - `tools/` - AI tool definitions
  - `contexts/` - React contexts (FileSystem, Chat)
  - `prompts/generation.tsx` - System prompt for Claude
- `prisma/` - Database schema (SQLite)

### Tech Stack

- Next.js 15 with App Router and Turbopack
- React 19
- TypeScript
- Tailwind CSS v4
- Prisma with SQLite
- Vercel AI SDK (`ai` package) with Anthropic provider
- Vitest for testing
