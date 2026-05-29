# GitHub Copilot Instructions — RoboPartDiscover

## Project

AI-powered robotics project scaffolder. User describes a robot in natural language; the platform generates a hardware BOM, compilable firmware, a wiring diagram SVG, and a full software stack.

**Stack:** TypeScript (strict) · Node.js 20 · Express · React 18 · Vite · Prisma · PostgreSQL · Vitest · Tailwind CSS

---

## Coding Conventions

### TypeScript
- Strict mode only — never use `any`; use `unknown` + Zod for external data
- Prefer `interface` for object shapes, `type` for unions
- All public module APIs must have JSDoc comments
- Use `satisfies` for config objects to preserve literal types

### Node.js / Express
- All routes are async; use `express-async-errors` for propagation
- Route handlers: validate → call service → respond. No business logic in routes.
- SSE endpoints: set `Content-Type: text/event-stream`, flush headers immediately
- Prisma client is a singleton in `src/db/client.ts` — never instantiate elsewhere

### LLM Integration
- All OpenAI/Anthropic calls go through `src/ai/llmClient.ts` only
- Always use structured JSON output mode with explicit Zod schema
- Retry with exponential backoff on rate limit errors (429)

### React
- Named function component exports only — no default exports
- Tailwind classes for all styling — no CSS files
- Use React Query for server state; local state with `useState`
- Streaming data via SSE: use the `useSSE` custom hook in `src/hooks/`

---

## Testing Conventions

- **Write tests before implementation** — TDD is the workflow, not an option
- Test files mirror src structure in `tests/` directory
- Mock all external APIs in unit tests with `vi.mock()`
- Integration tests are in files named `*.integration.test.ts`
- Use descriptive test names that read like requirements: `"generateBOM returns 3 candidates per component"`

---

## Boundaries

- Do **not** refactor code unless explicitly asked
- Do **not** remove or modify existing tests
- Do **not** add new npm dependencies without checking if a built-in alternative exists
- Do **not** call OpenAI/Anthropic SDK directly — always go through `llmClient.ts`
- Do **not** expose supplier API keys in any frontend code or public endpoint
- Do **not** use `console.log` in production code — use the `logger` utility (`src/lib/logger.ts`)
- Do **not** add TODO comments to code — add tasks to `TODO.md` instead
