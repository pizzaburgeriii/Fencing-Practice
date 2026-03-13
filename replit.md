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
│   └── fencing-trainer/    # Fencing Decision Trainer React app
├── lib/                    # Shared libraries
│   ├── api-spec/           # OpenAPI spec + Orval codegen config
│   ├── api-client-react/   # Generated React Query hooks
│   ├── api-zod/            # Generated Zod schemas from OpenAPI
│   └── db/                 # Drizzle ORM schema + DB connection
├── scripts/                # Utility scripts (single workspace package)
│   └── src/                # Individual .ts scripts
├── pnpm-workspace.yaml     # pnpm workspace (artifacts/*, lib/*, lib/integrations/*, scripts)
├── tsconfig.base.json      # Shared TS options (composite, bundler resolution, es2022)
├── tsconfig.json           # Root TS project references
└── package.json            # Root package with hoisted devDeps
```

## Fencing Decision Trainer

A training tool for fencers to practice reading and responding to first-person fencing videos.

### Features
- **Practice Mode**: Random video is selected and played. User picks which fencing action it shows (Attack, Parry, Riposte, Lunge, Fleche, Beat Attack, Counter-Attack, Feint). Correct/wrong feedback shown.
- **Upload Library**: Upload video files (MP4/WebM), tag them with the correct action, add optional notes.
- **Stats**: Session accuracy breakdown by action type with charts.

### Fencing Actions Supported
- Attack, Parry, Riposte, Lunge, Fleche, Beat Attack, Counter-Attack, Feint

### API Endpoints
- `GET /api/videos` — list all videos
- `POST /api/videos/upload` — upload a video file (multipart)
- `GET /api/videos/random` — get a random video for practice
- `GET /api/videos/:id` — get a video by ID
- `DELETE /api/videos/:id` — delete a video
- `GET /api/videos/stream/:filename` — stream video file
- `GET /api/actions` — list fencing action types
- `POST /api/sessions` — record a practice attempt
- `GET /api/sessions/stats` — get practice statistics

### DB Schema
- `videos` table: id, title, filename, action_label, notes, created_at
- `practice_sessions` table: id, video_id, chosen_action, correct_action, is_correct, created_at

### Video Files
Video files are stored on disk at `artifacts/api-server/uploads/`. They are streamed via `/api/videos/stream/:filename`.

## TypeScript & Composite Projects

Every package extends `tsconfig.base.json` which sets `composite: true`. The root `tsconfig.json` lists all packages as project references. This means:

- **Always typecheck from the root** — run `pnpm run typecheck` (which runs `tsc --build --emitDeclarationOnly`). This builds the full dependency graph so that cross-package imports resolve correctly. Running `tsc` inside a single package will fail if its dependencies haven't been built yet.
- **`emitDeclarationOnly`** — we only emit `.d.ts` files during typecheck; actual JS bundling is handled by esbuild/tsx/vite...etc, not `tsc`.
- **Project references** — when package A depends on package B, A's `tsconfig.json` must list B in its `references` array. `tsc --build` uses this to determine build order and skip up-to-date packages.

## Root Scripts

- `pnpm run build` — runs `typecheck` first, then recursively runs `build` in all packages that define it
- `pnpm run typecheck` — runs `tsc --build --emitDeclarationOnly` using project references
