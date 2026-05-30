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

---

## 🚀 Improvement Proposals

### First-Principles Analysis
- **The "bolt.new for robotics" framing is accurate but hides the key difficulty**: bolt.new generates web code in a well-defined, text-based domain with established patterns; robotics hardware spans thousands of incompatible components, evolving datasheets, and physical compatibility constraints that are not captured in any single structured API.
- **The BOM and the firmware are interdependent artifacts**: if the LLM selects a particular motor driver, the firmware must use that driver's specific register map and timing; a system that generates both must maintain internal consistency across outputs — this is a hard constraint-satisfaction problem, not just text generation.
- **Hardware availability and pricing are volatile**: Octopart/Digi-Key stock and prices change daily; a BOM generated today may be unbuildable next week; the system needs real-time availability checks, not cached prices, before the BOM is presented as actionable.
- **The wiring diagram is the highest-value and hardest output to get right**: pin mapping errors in a wiring diagram can destroy components or create fire hazards; auto-generated SVG schematics must be validated against known-safe pinout databases, not just hallucinated from training data.

### Key Risks & Assumptions
- **Assumes the LLM has accurate knowledge of component pinouts and register maps** — this is false for long-tail components; the model will hallucinate plausible-looking but incorrect pin assignments; the firmware and wiring outputs require ground-truth component databases, not LLM memory.
- **The Octopart and Digi-Key APIs have rate limits, authentication requirements, and terms of service restrictions** — building a production system on top of them requires API agreements that may not be available at the hobbyist/startup tier.
- **No mention of a feedback loop for incorrect BOMs or firmware** — when a user reports that the generated firmware doesn't compile or the BOM is missing a component, there is no mechanism to improve; without this loop, the system's accuracy is bounded by its training data forever.
- **ROS2 scaffolding generation requires knowledge of the specific hardware interfaces selected** — generating a generic ROS2 skeleton is trivial; generating one that correctly interfaces with the selected sensors and actuators requires component-specific knowledge that the LLM may not reliably possess.

### Concrete Improvement Ideas
1. **Build a curated, structured component database as the foundational layer** — before any LLM integration, create a database of 500 common robotics components (sensors, motors, microcontrollers) with verified pinouts, datasheets, and compatibility rules; the LLM selects from this database, not from training memory; this eliminates hallucinated pinouts (highest safety impact).
2. **Add real-time stock and price verification** — after BOM generation, verify each part number against the Octopart API and flag unavailable or end-of-life components before presenting to the user; offer substitution suggestions for unavailable parts.
3. **Implement a human-in-the-loop review step for wiring diagrams** — add a "verify this wiring" step where the user confirms the diagram before firmware is generated; add prominent warnings that AI-generated schematics must be reviewed by someone with electronics experience before powering hardware.
4. **Add a "validate firmware compiles" CI step** — for each firmware output (Arduino/MicroPython), run an actual compilation check (Arduino CLI or MicroPython cross-compiler) before delivering to the user; catch syntax errors and missing libraries automatically.
5. **Start with a constrained component catalog (Arduino Nano + common shields)** — limit v1 to a well-known platform with excellent community documentation; this drastically reduces hallucination risk and lets you build a track record of correct outputs before expanding to exotic components.
6. **Build community-contributed correction workflows** — let users submit corrections to generated BOMs and firmware; store verified corrections as few-shot examples for future generations; this creates a flywheel where each user interaction improves quality for the next.
