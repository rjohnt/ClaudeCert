# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev              # Start dev server (Turbopack)
npm run build            # Production build
npm run lint             # ESLint
npm run test             # Vitest (jsdom environment)
npx vitest run src/path/to/file.test.ts  # Run a single test file
npm run setup            # Install deps + Prisma generate + migrate
npm run db:reset         # Force reset SQLite database (destructive)
```

## Architecture

UIGen is an AI-powered React component generator built with **Next.js 15 (App Router)**, **TypeScript**, **Prisma/SQLite**, and the **Vercel AI SDK** with Anthropic Claude.

### How it works

Users describe components in natural language via a chat interface. Claude generates/modifies React components using two tools (`str_replace_editor` and `file_manager`) that operate on an in-memory **VirtualFileSystem** — no disk I/O. Generated components are previewed live in an iframe.

### Key flow

1. User sends a message from the chat panel
2. `src/app/api/chat/route.ts` sends the message + current file system state to Claude with tool definitions
3. Claude calls tools to create/edit files in the virtual file system
4. `FileSystemContext` processes tool call results and updates state
5. The preview panel renders the updated `App.jsx`

### State management

Two React contexts coordinate the app:

- **`FileSystemContext`** (`src/lib/contexts/file-system-context.tsx`): Manages the `VirtualFileSystem` instance, handles tool calls from AI, provides file CRUD operations
- **`ChatProvider`** (`src/lib/contexts/chat-context.tsx`): Wraps Vercel AI SDK's `useChat`, sends file state with each request, processes tool results

### AI provider

`src/lib/provider.ts` returns Claude Haiku when `ANTHROPIC_API_KEY` is set in `.env`, otherwise falls back to a `MockLanguageModel` that returns static component code.

### Layout

Three-panel resizable layout in `src/app/main-content.tsx`:
- Left (35%): Chat interface
- Right (65%): Tabbed preview (live render) and code view (file tree + Monaco editor)

### Database

SQLite via Prisma. Two models: `User` (email/password auth with JWT) and `Project` (stores serialized messages and file system data as JSON strings). Schema in `prisma/schema.prisma`.

### Auth

JWT-based with bcrypt password hashing. Server actions in `src/actions/index.ts` handle sign-up/sign-in/sign-out. The app works without authentication; anonymous usage is tracked in localStorage.

### Path alias

`@/*` maps to `./src/*`
