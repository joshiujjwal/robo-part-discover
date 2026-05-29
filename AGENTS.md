# AGENTS.md — RoboPartDiscover

Instructions for OpenAI Codex and other code-generation agents.

---

## Setup

```bash
npm install
cp .env.example .env      # fill in required keys before running anything
docker compose up -d      # start Postgres
npm run db:migrate        # apply migrations
npm test                  # verify baseline is green
```

Required env vars (see `.env.example` for full list):
- `OPENAI_API_KEY` — GPT-4o for robot spec parsing
- `ANTHROPIC_API_KEY` — Claude fallback
- `OCTOPART_API_KEY` — BOM part lookup
- `DIGIKEY_CLIENT_ID` / `DIGIKEY_CLIENT_SECRET` — BOM fallback
- `CLERK_SECRET_KEY` — auth
- `DATABASE_URL` — Postgres connection string

---

## Code Style

### TypeScript
- Strict mode: `"strict": true` in tsconfig — no exceptions
- Zero `any` — use `unknown` + Zod narrowing for external data
- Prefer `interface` over `type` for object shapes; use `type` for unions/intersections
- Name booleans with `is`/`has`/`can` prefix: `isLoading`, `hasError`, `canExport`
- Functions: prefer named function declarations at module level, arrow functions for callbacks
- Imports: named imports only (no `import * as`); group order: Node builtins → npm packages → internal

### Naming
- Files: `camelCase.ts` for modules, `PascalCase.tsx` for React components
- Classes/interfaces/types: `PascalCase`
- Functions/variables: `camelCase`
- Constants: `SCREAMING_SNAKE_CASE` for module-level immutable values
- Test files: `<subject>.test.ts` or `<subject>.integration.test.ts`

### Express Routes
- Routes are thin — validate input, call service, return response
- All business logic in service modules under `src/ai/`, `src/bom/`, etc.
- Use `express-async-errors` — no try/catch in route handlers
- Return consistent shape: `{ data: T }` on success, `{ error: { code, message } }` on failure

### React Components
- Function components with named exports only
- Props interface named `<ComponentName>Props`
- Co-locate component + its test: `BOMTable.tsx` and `BOMTable.test.tsx` in same directory
- No inline styles — Tailwind classes only
- Loading/error/empty states required for every data-fetching component

---

## Testing

**TDD is mandatory.** Workflow per feature:

1. Write a failing test that describes the behaviour (`// FAILING` comment on first commit)
2. Run `npm test` — confirm it fails for the right reason
3. Implement the minimum code to make it pass
4. Run `npm test` again — confirm green
5. Refactor if needed, keeping tests green

```bash
npm test                      # run all tests once
npm run test:watch            # watch mode during development
npm run test:coverage         # coverage report (target: 90%+ on core modules)
npx vitest run tests/ai/      # run only AI module tests
```

### Test Rules
- Unit tests: mock ALL external dependencies (`vi.mock()`)
- Integration tests (`.integration.test.ts`): may hit real APIs — guard with `if (!process.env.OCTOPART_API_KEY) test.skip(...)`
- No `test.only` or `describe.only` committed to main
- Each test must have a clear, specific description: `"returns error when MCU is unsupported"` not `"handles errors"`
- Fixtures in `tests/fixtures/` — reuse across tests, never duplicate fixture data

---

## PR Instructions

Every PR must include:

1. **What changed** — one paragraph description
2. **Test evidence** — paste `npm test` output showing all tests pass
3. **Manual test evidence** — if UI or API changed: screenshot or `curl` output
4. **Checklist:**
   - [ ] `npm test` passes
   - [ ] `npm run lint` passes
   - [ ] `npm run build` passes (no TypeScript errors)
   - [ ] New tests written for new behaviour
   - [ ] `TODO.md` checkbox updated
   - [ ] `CLAUDE.md` updated if non-obvious convention discovered

### PR Size
- One feature or fix per PR
- Max ~400 lines changed (excluding generated files, migrations)
- If larger, split into stack: model → service → route → UI

### Do Not
- Do not merge PRs with failing tests
- Do not remove or skip existing tests
- Do not use `@ts-ignore` or `@ts-expect-error` without a comment explaining why
- Do not commit `.env` files or secrets
- Do not modify `docs/spec.md` data models without updating Zod schemas and Prisma schema to match
