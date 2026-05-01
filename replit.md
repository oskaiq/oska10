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

## Key Commands

- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- `pnpm --filter @workspace/api-server run dev` — run API server locally

See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details.

## Artifacts

- **api-server** (`artifacts/api-server`) — Express + WebSocket server. Routes under `/api`; WebSocket under `/ws`. Hosts the Mafia game engine (`src/lib/game.ts`) and websocket dispatcher (`src/lib/wsServer.ts`). Rooms live in-memory; only completed games persist for stats.
- **mafia** (`artifacts/mafia`) — React + Vite frontend for "ليلة المافيا" (Arabic Mafia). Mounted at `/`. Uses TanStack Query + orval-generated hooks for REST and a custom `useGameSocket` hook for realtime gameplay (chat, voting, night actions).
- **mockup-sandbox** — design canvas (not used for Mafia gameplay).

### Mafia game flow

1. Player creates room (REST `POST /api/rooms`) or joins by code.
2. Frontend opens WebSocket to `${BASE_URL}ws?code=X&name=Y`. Auth is by code+name.
3. Phases: `lobby` → `night` (30s) → `day` (60s) → `ended`. Mafia count: ≤6 → 1, ≤9 → 2, else 3. Doctor + Detective always present.
4. Roles: mafia / doctor / detective / citizen. Mafia coordinates kill, doctor saves, detective inspects.
5. On end, game record + per-player rows insert into `gamesTable` + `gamePlayersTable` (`lib/db/src/schema/games.ts`). Stats endpoints (`/api/stats/*`) read from these.

### Seeding

- `pnpm --filter @workspace/scripts run seed-mafia` inserts 3 sample completed games for the leaderboard. Idempotent (skips if any game already exists).
