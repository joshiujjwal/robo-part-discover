# RoboPartDiscover 🤖

> **bolt.new for robotics** — describe your robot project and instantly get a hardware BOM, firmware code, wiring diagrams, and a full software stack.

![Status](https://img.shields.io/badge/status-🚧%20Early%20Development-orange)
![Stack](https://img.shields.io/badge/stack-TypeScript%20%7C%20Node.js%20%7C%20React-blue)
![License](https://img.shields.io/badge/license-MIT-green)

---

## What It Does

You type: *"Build me a 4-wheeled line-following robot with obstacle avoidance and WiFi telemetry."*

RoboPartDiscover returns:

| Output | Description |
|---|---|
| 🛒 **BOM** | Itemised hardware list with part numbers, suppliers, and prices |
| ⚡ **Firmware** | Arduino / MicroPython / Rust-embedded starter code |
| 🔌 **Wiring Diagram** | Auto-generated SVG schematic (pin-by-pin) |
| 📦 **Software Stack** | ROS2 / custom middleware scaffolded and ready to clone |
| 📖 **Build Guide** | Step-by-step assembly instructions |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React 18 + TypeScript + Vite |
| Backend API | Node.js + Express + TypeScript |
| AI Engine | OpenAI GPT-4o / Anthropic Claude (structured outputs) |
| BOM Data | Octopart API + Digi-Key API |
| Wiring Render | KiCanvas / SVG generation |
| Database | PostgreSQL + Prisma ORM |
| Auth | Clerk |
| Deployment | Docker + Railway |

---

## Getting Started

```bash
# Clone
git clone https://github.com/joshiujjwal/robo-part-discover.git
cd robo-part-discover

# Install dependencies
npm install

# Set up environment
cp .env.example .env
# Edit .env with your API keys

# Run database migrations
npm run db:migrate

# Start dev servers (frontend + backend)
npm run dev
```

### Run Tests

```bash
npm test              # all tests
npm run test:watch    # watch mode
npm run test:coverage # with coverage report
```

### Lint & Format

```bash
npm run lint
npm run format
```

---

## Project Structure

```
robo-part-discover/
├── src/
│   ├── api/          # Express routes, middleware, validation
│   ├── ai/           # LLM prompt engineering, structured output parsing
│   ├── bom/          # BOM generation, supplier lookups, pricing
│   ├── firmware/     # Firmware template generation (Arduino, MicroPython, Rust)
│   ├── wiring/       # Wiring diagram SVG generation
│   └── ui/           # React frontend (Vite)
├── tests/            # Mirror of src/ — unit + integration tests
├── docs/
│   ├── spec.md       # Feature specification
│   └── adr/          # Architecture Decision Records
├── .github/
│   ├── copilot-instructions.md
│   └── instructions/
├── CLAUDE.md         # AI agent context
├── AGENTS.md         # OpenAI Codex instructions
└── TODO.md           # Evidence-gated task breakdown
```

---

## Contributing

1. Read `TODO.md` and pick a task in the current phase
2. **Write failing tests first** (red phase) — no exceptions
3. Implement until tests pass (green phase)
4. Run `npm run lint && npm test` — both must be clean
5. Open a PR with: what you changed, test output screenshot, any open questions
6. Keep PRs small and focused — one feature/fix per PR

> PRs without passing tests or evidence of manual testing will not be merged.
