# RoboPartDiscover — Feature Specification

**Version:** 0.1.0-draft  
**Author:** joshiujjwal  
**Status:** Draft — awaiting Phase 1 implementation

---

## 1. Problem Statement

Building a robotics prototype today requires hours of research scattered across datasheets, supplier sites, forum posts, and GitHub repos. A maker who wants to build a simple obstacle-avoiding robot must:

1. Research compatible microcontrollers
2. Cross-reference sensor/actuator pin compatibility
3. Price-shop across Digi-Key, Mouser, Adafruit
4. Write boilerplate firmware from scratch
5. Hand-draw a wiring diagram

**RoboPartDiscover collapses this to a single prompt.**

---

## 2. Core User Journey

```
User types natural language description
        ↓
AI parses into structured RobotSpec
        ↓
Parallel pipeline:
  ├── BOM Generator      → itemised parts list + pricing
  ├── Firmware Generator → compilable starter code
  ├── Wiring Generator   → SVG schematic
  └── Build Guide Writer → markdown instructions
        ↓
Results streamed to browser in real-time
        ↓
User can export, fork, or share project
```

---

## 3. Functional Requirements

### 3.1 Input Processing
- [ ] Accept free-text robot descriptions (10–2000 characters)
- [ ] Support follow-up refinement prompts ("make it wireless", "use a Raspberry Pi instead")
- [ ] Detect ambiguous specs and ask clarifying questions before generating
- [ ] Support structured input mode (form-based) as an alternative to free text

### 3.2 AI Parsing (RobotSpec Extraction)
- [ ] Extract microcontroller platform (Arduino Uno/Nano/Mega, ESP32, Raspberry Pi, STM32, etc.)
- [ ] Extract locomotion type (wheeled, tracked, bipedal, hexapod, drone, stationary arm)
- [ ] Extract sensors (ultrasonic, IR, camera, IMU, GPS, encoders, ToF, etc.)
- [ ] Extract actuators (DC motors, servo, stepper, brushless, solenoid)
- [ ] Extract connectivity requirements (WiFi, Bluetooth, LoRa, none)
- [ ] Extract power constraints (battery type/capacity, voltage rails)
- [ ] Infer missing fields from context (e.g. "line-following" → IR sensors implied)
- [ ] Return confidence score per inferred field

### 3.3 BOM Generation
- [ ] Map each RobotSpec field to a component category
- [ ] Search Octopart API for matching parts (top 3 candidates per component)
- [ ] Return for each part: name, manufacturer, part number, supplier, unit price, stock status, datasheet URL
- [ ] Fallback to Digi-Key API for parts not found on Octopart
- [ ] Generate total BOM cost estimate (low / typical / high)
- [ ] Export BOM as CSV and JSON
- [ ] Highlight substitutions when preferred parts are out of stock

### 3.4 Firmware Generation
- [ ] Select target firmware platform from MCU:
  - Arduino (Uno/Nano/Mega/Pro Mini) → C++ (Arduino framework)
  - ESP32 / ESP8266 → C++ (Arduino or ESP-IDF) or MicroPython
  - Raspberry Pi Pico → MicroPython or C SDK
  - STM32 → Rust (embassy-rs) or C (HAL)
- [ ] Generate compilable starter code including:
  - Pin definitions matching generated wiring
  - Sensor read functions with sample values + comments
  - Actuator control functions
  - Main loop with basic behaviour (e.g. stop when obstacle < 20cm)
  - WiFi/Bluetooth init if connectivity required
- [ ] Include `README.md` inside firmware package explaining how to flash
- [ ] Include `platformio.ini` for PlatformIO users

### 3.5 Wiring Diagram
- [ ] Build a component connection graph from RobotSpec + BOM
- [ ] Render as SVG with:
  - Named component boxes with chip silhouette
  - Labeled wire connections (pin number + signal name)
  - Colour-coded wires (power=red, ground=black, signal=blue, PWM=orange)
  - Voltage rail annotations
- [ ] Pan and zoom in browser
- [ ] Export as SVG and PNG
- [ ] Warn when pin count exceeds MCU's available GPIO

### 3.6 Project Persistence
- [ ] Save each generated project with a unique slug
- [ ] Allow registered users to view/edit/fork saved projects
- [ ] Anonymous projects expire after 7 days
- [ ] Public/private visibility toggle

### 3.7 User Accounts
- [ ] Sign up / sign in via Clerk (email + Google OAuth)
- [ ] Dashboard showing saved projects
- [ ] Project sharing via public link

---

## 4. Non-Functional Requirements

- [ ] `/api/v1/generate` responds with first SSE event within 2 seconds
- [ ] Full generation pipeline completes in < 30 seconds for standard 5-component robots
- [ ] API rate limited to 10 requests/hour (anonymous), 100/hour (authenticated)
- [ ] All LLM calls have 15-second timeout + 2 retries with exponential backoff
- [ ] Zero TypeScript `any` — all external data validated with Zod at API boundary
- [ ] All component tests run in < 5 seconds
- [ ] 90%+ unit test coverage on `src/ai/`, `src/bom/`, `src/firmware/`, `src/wiring/`

---

## 5. Data Models

### `RobotSpec`
```typescript
interface RobotSpec {
  id: string;
  rawDescription: string;
  microcontroller: {
    platform: 'arduino-uno' | 'arduino-nano' | 'arduino-mega' | 'esp32' | 'esp8266'
              | 'rpi-pico' | 'rpi4' | 'stm32f4' | 'teensy4';
    quantity: number;
  };
  locomotion: {
    type: 'wheeled-2wd' | 'wheeled-4wd' | 'tracked' | 'bipedal' | 'hexapod' | 'drone' | 'arm' | 'stationary';
    motorCount: number;
  };
  sensors: Array<{
    type: 'ultrasonic' | 'ir-line' | 'ir-obstacle' | 'camera' | 'imu' | 'gps' | 'encoder' | 'tof' | 'temperature';
    quantity: number;
    inferred: boolean;  // true if AI inferred, not explicit in prompt
    confidence: number; // 0–1
  }>;
  actuators: Array<{
    type: 'dc-motor' | 'servo' | 'stepper' | 'brushless' | 'solenoid';
    quantity: number;
  }>;
  connectivity: Array<'wifi' | 'bluetooth' | 'lora' | 'rf433' | 'none'>;
  power: {
    sourceType: 'lipo' | 'nimh' | 'alkaline' | 'usb' | 'wall-adapter';
    nominalVoltage: number;
    capacityMah?: number;
  };
  firmwareTarget: 'arduino-cpp' | 'micropython' | 'rust-embassy' | 'esp-idf';
}
```

### `BOMItem`
```typescript
interface BOMItem {
  id: string;
  specField: string;           // which RobotSpec field this satisfies
  componentCategory: string;
  quantity: number;
  candidates: SupplierPart[];  // top 3, sorted by score
  selectedPartIndex: number;
}

interface SupplierPart {
  name: string;
  manufacturer: string;
  partNumber: string;
  supplier: 'octopart' | 'digikey' | 'adafruit' | 'sparkfun';
  supplierUrl: string;
  unitPriceCents: number;
  stockStatus: 'in-stock' | 'limited' | 'out-of-stock' | 'unknown';
  datasheetUrl?: string;
}
```

### `Project` (Prisma model)
```
Project {
  id          String    @id @default(cuid())
  slug        String    @unique
  userId      String?   // null = anonymous
  title       String
  description String
  robotSpec   Json
  bom         Json
  firmware    Json      // { platform, files: [{name, content}] }
  wiringGraph Json
  isPublic    Boolean   @default(false)
  expiresAt   DateTime? // set for anonymous projects
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt
}
```

---

## 6. API Interface

### `POST /api/v1/generate`
```
Body: { description: string, sessionId?: string }
Response: SSE stream
  event: stage  data: { stage: 'parsing'|'bom'|'firmware'|'wiring', status: 'started'|'done', progress: 0–100 }
  event: result data: { robotSpec, bom, firmware, wiringGraph, buildGuide }
  event: error  data: { code, message }
```

### `GET /api/v1/projects/:slug`
```
Response: { project: Project }
```

### `POST /api/v1/projects/:slug/fork`
```
Auth: required
Response: { project: Project } // new forked project
```

---

## 7. Test Plan

### Unit Tests
- `parseRobotDescription()` — 10 fixture inputs covering: simple, complex, ambiguous, edge cases
- `validateRobotSpec()` — invalid MCU, missing locomotion, empty sensors array
- `generateBOM()` — mocked Octopart responses, verify part selection logic
- `enrichBOMWithPricing()` — mocked pricing data, out-of-stock fallback
- `selectFirmwareTarget()` — every supported MCU type
- `generateFirmware()` — Arduino output for 3 robot archetypes
- `buildWiringGraph()` — pin conflict detection, missing power rail warning
- `renderWiringDiagram()` — valid SVG output, all nodes present

### Integration Tests
- Full pipeline: `"4WD obstacle-avoiding robot with ESP32"` → all outputs generated
- BOM with real Octopart API (in CI with API key secret)
- SSE streaming: verify all stage events arrive in order

### E2E Tests (Playwright)
- New user: type prompt → see streaming progress → view BOM → download CSV
- Returning user: sign in → load saved project → fork it → edit description → regenerate

---

## 8. Open Questions

| # | Question | Owner | Status |
|---|---|---|---|
| 1 | Which LLM gives best structured output for robotics? (GPT-4o vs Claude 3.5 Sonnet) | joshiujjwal | ❓ Open |
| 2 | Does Octopart API free tier allow 10 req/hour per user at scale? | joshiujjwal | ❓ Open |
| 3 | KiCad `.kicad_sch` export — is there a JS library or does it need a sidecar process? | joshiujjwal | ❓ Open |
| 4 | Real-time compile check for Arduino firmware — `arduino-cli` in Docker or WASM? | joshiujjwal | ❓ Open |
| 5 | How to handle robots requiring custom PCBs (e.g. motor driver not off-the-shelf)? | joshiujjwal | ❓ Open |
