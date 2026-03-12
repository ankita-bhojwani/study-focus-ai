# Workspace

## Overview

pnpm workspace monorepo using TypeScript. Each package manages its own dependencies.

## Stack

- **Monorepo tool**: pnpm workspaces
- **Node.js version**: 24
- **Package manager**: pnpm
- **TypeScript version**: 5.9
- **API framework**: Express 5
- **Database**: PostgreSQL + Drizzle ORM
- **Validation**: Zod (`zod/v4`), `drizzle-zod`
- **API codegen**: Orval (from OpenAPI spec)
- **Build**: esbuild (CJS bundle)

## Structure

```text
artifacts-monorepo/
├── artifacts/              # Deployable applications
│   ├── api-server/         # Express API server
│   └── study-planner/      # React + Vite study planner frontend
├── lib/                    # Shared libraries
│   ├── api-spec/           # OpenAPI spec + Orval codegen config
│   ├── api-client-react/   # Generated React Query hooks
│   ├── api-zod/            # Generated Zod schemas from OpenAPI
│   └── db/                 # Drizzle ORM schema + DB connection
├── scripts/                # Utility scripts (single workspace package)
│   └── src/                # Individual .ts scripts
├── pnpm-workspace.yaml     # pnpm workspace
├── tsconfig.base.json      # Shared TS options
├── tsconfig.json           # Root TS project references
└── package.json            # Root package with hoisted devDeps
```

## Project: AI Study Planner

A full-stack AI-powered study planner web app.

### Features
1. **Dashboard** - Overview of study stats, streak, upcoming tasks, recent sessions
2. **Tasks** - CRUD study tasks with priority, due date, estimated time
3. **Study Timer** - Pomodoro-style timer (25/5/15 min) with session auto-save
4. **AI Schedule** - Generates personalized 7-day study schedules based on subjects and availability
5. **Analytics** - Bar charts, pie charts, completion rates, streaks, subject breakdowns

### Database Schema
- `tasks` - Study tasks (title, subject, status, priority, due_date, estimated_minutes, completed_at)
- `study_sessions` - Session logs (subject, task_id, duration_minutes, notes, started_at, ended_at)

### API Routes (all under `/api`)
- `GET/POST /tasks` - List/create tasks
- `GET/PATCH/DELETE /tasks/:id` - Get/update/delete task
- `GET/POST /sessions` - List/log sessions
- `POST /schedule/generate` - AI schedule generation
- `GET /analytics/summary` - Full analytics summary
- `GET /analytics/daily` - Daily study data (30 days)

## TypeScript & Composite Projects

Every package extends `tsconfig.base.json` which sets `composite: true`. The root `tsconfig.json` lists all packages as project references.

- **Always typecheck from the root** — run `pnpm run typecheck`
- **`emitDeclarationOnly`** — only emit `.d.ts` files during typecheck

## Root Scripts

- `pnpm run build` — runs `typecheck` first, then recursively runs `build` in all packages
- `pnpm run typecheck` — runs `tsc --build --emitDeclarationOnly` using project references

## Packages

### `artifacts/study-planner` (`@workspace/study-planner`)

React + Vite frontend. Routes: `/`, `/tasks`, `/timer`, `/schedule`, `/analytics`.

Frontend deps: `framer-motion`, `recharts`, `react-hook-form`, `@hookform/resolvers`, `date-fns`, `wouter`, `@tanstack/react-query`

### `artifacts/api-server` (`@workspace/api-server`)

Express 5 API server. Routes live in `src/routes/` and use `@workspace/api-zod` for validation and `@workspace/db` for persistence.

- `pnpm --filter @workspace/api-server run dev` — run the dev server

### `lib/db` (`@workspace/db`)

Database layer using Drizzle ORM with PostgreSQL.

- `src/schema/tasks.ts` — tasks table
- `src/schema/sessions.ts` — study_sessions table
- `pnpm --filter @workspace/db run push` — push schema to DB

### `lib/api-spec` (`@workspace/api-spec`)

OpenAPI 3.1 spec + Orval codegen config.

- Run codegen: `pnpm --filter @workspace/api-spec run codegen`
