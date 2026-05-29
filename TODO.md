# RoboPartDiscover — Task Breakdown

## How to Use This File

Work one task at a time. Before each task:
1. **Write failing tests FIRST** (red phase) — commit them
2. **Implement until tests pass** (green phase) — commit
3. **Review your own diff** before opening a PR
4. **Update CLAUDE.md / AGENTS.md** if you discovered a non-obvious convention
5. **Update Lessons Learned** at the bottom when something surprised you

Gate rule: **Do not start the next phase until all tasks in the current phase have passing tests and have been reviewed.**

---

## Phase 0: Foundation ⬜

- [ ] Init monorepo with `npm workspaces` — `packages/api` and `packages/ui`
- [ ] Configure TypeScript (`tsconfig.json`) for both packages with strict mode
- [ ] Set up ESLint + Prettier with `@typescript-eslint` rules
- [ ] Add Vitest for backend tests; add Vitest + React Testing Library for frontend
- [ ] Write a smoke test that passes (proves the test harness works)
- [ ] Set up GitHub Actions CI: install → lint → test on push/PR
- [ ] Add `.env.example` with all required keys (no values)
- [ ] Set up Docker Compose for local Postgres
- [ ] Init Prisma with a stub `User` model, run first migration
- [ ] Review all AI config files (CLAUDE.md, AGENTS.md, copilot-instructions.md) and adjust

**Evidence gate:** CI is green, `npm test` passes locally, Prisma migration applied ✅

---

## Phase 1: AI Core — Robot Description → Structured Output ⬜

- [ ] Write spec for AI prompt contract in `docs/spec.md` (input/output schema)
- [ ] Define `RobotSpec` TypeScript type (microcontroller, sensors, actuators, connectivity, power)
- [ ] Write failing tests for `parseRobotDescription()` with 3 fixture inputs
- [ ] Implement `parseRobotDescription()` using OpenAI structured outputs (JSON mode)
- [ ] Write failing tests for `validateRobotSpec()` — boundary cases (missing fields, unsupported MCU)
- [ ] Implement `validateRobotSpec()` with Zod schema
- [ ] Add Claude fallback path (if OpenAI fails, retry with Anthropic)
- [ ] Integration test: end-to-end prompt → parsed spec for 3 robot archetypes
- [ ] Manual test: run 5 real prompts and review quality — document results in PR

**Evidence gate:** Unit tests green, integration tests green, manual test results in PR description ✅

---

## Phase 2: BOM Generation ⬜

- [ ] Write failing tests for `generateBOM(robotSpec)` with mocked supplier responses
- [ ] Implement `generateBOM()` — maps spec fields to component categories
- [ ] Integrate Octopart API — search by keyword, return top 3 matches per component
- [ ] Write failing tests for `enrichBOMWithPricing()` — verifies price/stock fields populated
- [ ] Implement `enrichBOMWithPricing()` with Octopart pricing endpoint
- [ ] Add Digi-Key API fallback for parts not found on Octopart
- [ ] Write failing tests for `exportBOM()` — CSV and JSON output formats
- [ ] Implement `exportBOM()` for both formats
- [ ] Manual test: generate BOM for a line-following robot — verify part numbers are real

**Evidence gate:** All BOM unit + integration tests green, manual BOM spot-checked against real supplier sites ✅

---

## Phase 3: Firmware Generation ⬜

- [ ] Write failing tests for `selectFirmwareTarget(robotSpec)` — maps MCU to platform (Arduino/MicroPython/Rust)
- [ ] Implement `selectFirmwareTarget()` with MCU→platform lookup table
- [ ] Create firmware template system: Handlebars templates per platform + sensor combo
- [ ] Write failing tests for `generateFirmware(robotSpec)` — verifies output compiles (syntax check)
- [ ] Implement `generateFirmware()` for Arduino (C++) — motor control + sensor read stubs
- [ ] Implement `generateFirmware()` for MicroPython — same feature set
- [ ] Implement `generateFirmware()` for Rust (embassy-rs) — advanced path, mark experimental
- [ ] Add LLM-assisted custom logic generation for non-template scenarios
- [ ] Manual test: flash generated firmware to a physical Arduino Uno — document result

**Evidence gate:** Firmware tests green, generated Arduino code compiles with `arduino-cli`, manual flash documented ✅

---

## Phase 4: Wiring Diagram Generation ⬜

- [ ] Define `WiringGraph` data structure: nodes (components) + edges (connections with pin labels)
- [ ] Write failing tests for `buildWiringGraph(robotSpec, bom)` — verifies all components connected
- [ ] Implement `buildWiringGraph()` — rules engine: MCU pins → sensor/actuator pins
- [ ] Write failing tests for `renderWiringDiagram(graph)` — outputs valid SVG
- [ ] Implement `renderWiringDiagram()` using D3 force layout + custom SVG templates per component
- [ ] Add component SVG sprites library (MCU, motor driver, servo, ultrasonic, IR sensor, etc.)
- [ ] Write tests for SVG accessibility (title, desc elements present)
- [ ] Manual review: render wiring diagram for 3 robot types — share screenshots in PR

**Evidence gate:** Wiring tests green, SVGs render correctly in browser, manual screenshots in PR ✅

---

## Phase 5: API Layer ⬜

- [ ] Write failing tests for `POST /api/v1/generate` — full pipeline: description → all outputs
- [ ] Implement `/api/v1/generate` endpoint with streaming SSE response (show progress)
- [ ] Write failing tests for `GET /api/v1/projects/:id` — retrieves saved project
- [ ] Implement project persistence: save robot spec + outputs to Postgres via Prisma
- [ ] Add rate limiting middleware (10 req/hour per IP, 100/hour per user)
- [ ] Add request validation with Zod — reject malformed inputs early
- [ ] Write integration tests for auth-protected routes (Clerk JWT middleware)
- [ ] Implement auth middleware and wire to protected routes
- [ ] Load test: 10 concurrent `/generate` requests — verify no race conditions

**Evidence gate:** All API tests green, auth tested manually, load test results documented ✅

---

## Phase 6: React UI ⬜

- [ ] Scaffold Vite + React 18 + TypeScript app in `packages/ui`
- [ ] Write failing component tests for `<PromptInput />` — submits on Enter, shows loading state
- [ ] Implement `<PromptInput />` with controlled input + submit handler
- [ ] Write failing tests for `<BOMTable />` — renders rows, sortable columns, CSV export button
- [ ] Implement `<BOMTable />` with TanStack Table
- [ ] Write failing tests for `<FirmwareViewer />` — code syntax highlighting, copy button
- [ ] Implement `<FirmwareViewer />` using Shiki (server-side syntax highlighting)
- [ ] Write failing tests for `<WiringDiagram />` — renders SVG, zoom/pan works
- [ ] Implement `<WiringDiagram />` with SVG pan/zoom (panzoom library)
- [ ] Add streaming UI: SSE consumer shows live progress per pipeline stage
- [ ] Manual test: full user journey — prompt → BOM → firmware → wiring in browser

**Evidence gate:** All component tests green, full user journey tested manually, screenshots in PR ✅

---

## Phase 7: Polish & Harden ⬜

- [ ] Add OpenTelemetry tracing to API — trace each pipeline stage
- [ ] Add Sentry for frontend + backend error capture
- [ ] Write E2E tests with Playwright: full user journey (2–3 scenarios)
- [ ] Add `robots.txt` and security headers (helmet.js)
- [ ] Database connection pooling + query timeout tuning
- [ ] Add retry logic + exponential backoff for all LLM/supplier API calls
- [ ] Audit and fix all TypeScript `any` usages — must be zero
- [ ] Review and harden Zod schemas — all external data validated at boundary

**Evidence gate:** E2E tests green, zero TypeScript errors, Sentry reporting in staging ✅

---

## Phase 8: Ship ⬜

- [ ] Write `Dockerfile` for API (multi-stage, non-root user)
- [ ] Write `docker-compose.prod.yml` for full stack
- [ ] Deploy to Railway — configure env vars, Postgres addon
- [ ] Set up custom domain + TLS
- [ ] Configure production CI/CD: push to `main` → deploy to Railway
- [ ] Write `CHANGELOG.md` for v0.1.0
- [ ] Announce on X / HN / r/robotics

**Evidence gate:** App live at production URL, smoke test passes, v0.1.0 tag on GitHub ✅

---

## Parking Lot 🅿️

> Ideas to revisit later — not blocking current phases

- KiCad `.kicad_sch` export (proper EDA format, not just SVG)
- Marketplace: share and remix robot projects
- Parts affiliate links (Digi-Key, Adafruit referral)
- ROS2 package scaffolding
- PCB layout generation
- 3D-printable enclosure STL generation via text-to-CAD API
- Mobile app (React Native) with AR overlay on physical parts

---

## Lessons Learned 📝

> Add entries here when something non-obvious is discovered during development

- *(none yet)*
