# CLAUDE.md — RoboPartDiscover

AI agent context file. Keep under 200 lines. Update when you discover non-obvious conventions.

---

## Startup Checklist (Run These First)

```bash
npm install                  # install all workspace deps
npm test                     # must be green before ANY changes
npm run lint                 # must be clean before ANY changes
docker compose up -d         # start local Postgres
npm run db:migrate           # apply pending Prisma migrations
```

If tests fail before you touch anything, **stop and investigate** — do not proceed with new work on a broken baseline.

---

## Commands

```bash
npm run dev            # start both packages/api (port 3001) and packages/ui (port 5173)
npm run dev:api        # API only
npm run dev:ui         # UI only
npm test               # vitest run (all workspaces)
npm run test:watch     # vitest watch
npm run test:coverage  # vitest with Istanbul coverage
npm run lint           # eslint across all workspaces
npm run format         # prettier --write
npm run build          # tsc + vite build (production)
npm run db:migrate     # prisma migrate dev
npm run db:studio      # prisma studio (DB GUI at localhost:5555)
npm run db:seed        # seed dev database
```

---

## Directory Map

```
packages/api/src/
  ai/           # LLM orchestration: parseRobotDescription, structured output parsing
  bom/          # BOM pipeline: generateBOM, enrichBOMWithPricing, exportBOM
  firmware/     # Firmware generation: template engine + LLM-assisted custom logic
  wiring/       # Wiring graph builder + SVG renderer
  routes/       # Express route handlers (thin — delegate to service modules)
  middleware/   # Auth (Clerk JWT), rate limiting, validation, error handling
  db/           # Prisma client singleton + query helpers
  types/        # Shared TypeScript interfaces (RobotSpec, BOMItem, etc.)

packages/ui/src/
  components/   # Reusable UI components (BOMTable, FirmwareViewer, WiringDiagram)
  pages/        # Route-level page components
  hooks/        # Custom React hooks (useGenerate, useSSE, useProject)
  api/          # Typed fetch wrappers for backend API
  lib/          # Utilities (formatCurrency, downloadCSV, etc.)

docs/spec.md    # Source of truth for data models and API contracts
docs/adr/       # Architecture decisions — read before proposing changes
```

---

## Conventions

### TypeScript
- Strict mode on everywhere — zero `any` allowed
- All external data (LLM responses, API responses) validated with **Zod** before use
- Export interfaces from `packages/api/src/types/index.ts` — shared by both packages via workspace alias
- Use `satisfies` operator for config objects to get narrowing without widening

### Testing
- Test files live in `tests/` mirroring `src/` — e.g. `tests/ai/parseRobotDescription.test.ts`
- **Write the test file first**, commit it (red), then implement
- Mock external APIs with `vi.mock()` in unit tests — never hit real APIs in unit tests
- Integration tests (files ending in `.integration.test.ts`) may hit real APIs; skip in CI if env var not set
- Fixture data lives in `tests/fixtures/` — robot description strings + expected parsed outputs

### LLM Calls
- All LLM calls go through `packages/api/src/ai/llmClient.ts` — never call OpenAI/Anthropic SDK directly from other modules
- Always use structured output mode (JSON schema) — never parse free-text LLM responses with regex
- Wrap every call with `withRetry(fn, { maxAttempts: 3, backoff: 'exponential' })`
- Log prompt token count and completion token count to stdout in dev mode

### BOM / Supplier APIs
- Octopart and Digi-Key calls go through `packages/api/src/bom/supplierClient.ts`
- Cache responses in Postgres for 24 hours to avoid hammering rate limits
- Never expose raw supplier API keys to the frontend

### React / UI
- Components are function components with named exports (no default exports)
- State management: React Query for server state, `useState`/`useReducer` for local UI state — no Redux
- Tailwind CSS for styling — no CSS modules, no styled-components
- All user-visible text goes through `i18n` (even if English-only for now — future-proofing)

### Wiring Diagrams
- SVG component sprites live in `packages/api/src/wiring/sprites/` as `.svg` strings
- Each sprite must have a `<title>` and `<desc>` for accessibility
- Pin positions are defined in `packages/api/src/wiring/pinLayouts.ts` — one entry per supported MCU/component

---

## Workflow

1. `git pull && npm install && npm test` — start clean
2. Read `TODO.md` — find the next unchecked item in the current phase
3. Write failing test in `tests/` (red commit)
4. Implement in `src/` until tests pass (green commit)
5. `npm run lint` — fix anything flagged
6. `npm run build` — confirm no TypeScript errors
7. Open PR with: test output, manual evidence if applicable
8. After merge, update `TODO.md` checkbox and `Lessons Learned` if applicable

---

## Non-Obvious Gotchas

- Prisma client must be instantiated as a singleton — see `src/db/client.ts`. Instantiating per-request causes connection pool exhaustion.
- The SSE endpoint `/api/v1/generate` holds the connection open for up to 30s — set `res.setTimeout(35000)` to avoid Express default 30s timeout killing it early.
- Octopart's free tier enforces a **per-minute** rate limit, not just hourly. Implement a token bucket, not just a counter.
- Vitest and Jest use different mock reset semantics — read `vi.clearAllMocks()` vs `vi.resetAllMocks()` docs before writing mock teardown.
- KiCanvas (wiring SVG rendering) requires a Web Worker in the browser — cannot be used SSR. Render SVGs on the server and hydrate as static on the client.
