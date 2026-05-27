# 🔥 CodeBurn — AI Project Dossier

> **Purpose of this file:** This is the single source of truth for any AI assistant working on this project. Read this ENTIRELY before writing a single line of code.

---

## 1. What Is This Project?

**CodeBurn** is a fun, open-source, gamified typing activity tracker. It measures:

1. **Active typing time** — the exact wall-clock time your hands are physically on the keyboard
2. **Estimated calories burned** — a playful, keystroke-count-based calorie estimate (not scientifically rigorous — it's for fun)

It runs as **two extensions** sharing a single core:
- A **Chrome/Brave browser extension** (Manifest V3)
- A **VS Code extension**

The signature feature is a **dynamic fire emoji 🔥** that continuously animates and shifts colors based on typing speed:
- 🟢 Green (casual, 0–15 WPM) → 🟠 Orange (flow, 16–50 WPM) → 🔴 Red (rage, 50+ WPM)
- When typing stops, it fades back: 🔴 → 🟠 → 🟢 → Gray (idle)

---

## 2. Project Goals

### Primary Goal
**Get significant GitHub stars, forks, and community traction.** This project is meant to be portfolio-worthy and resume-ready. Every decision — from code quality to README design to commit messages — should reflect this.

### What This Means for You (the AI)
- Write code that a senior engineer would be proud to show in an interview
- The README must be stunning — it's the first thing people see
- Code should be so clean and well-commented that contributors can onboard in minutes
- Every file should feel intentional, not auto-generated slop

---

## 3. Architecture

```
typing-tracker/
├── packages/
│   ├── shared-core/           # Platform-agnostic engine (vanilla TypeScript)
│   │   ├── src/
│   │   │   ├── engine.ts      # Timer, idle detection, session management
│   │   │   ├── calories.ts    # Calorie calculation with WPM multiplier
│   │   │   ├── wpm.ts         # Sliding-window WPM calculator
│   │   │   ├── fire-state.ts  # Fire color/animation state machine
│   │   │   ├── storage.ts     # Abstract storage interface
│   │   │   └── types.ts       # Shared TypeScript types
│   │   └── tests/
│   │
│   ├── chrome-extension/      # Chrome/Brave extension (Manifest V3)
│   │   ├── manifest.json
│   │   ├── src/
│   │   │   ├── background.ts  # Service worker — aggregates data
│   │   │   ├── content.ts     # Content script — keydown listener
│   │   │   ├── popup/         # Extension popup UI
│   │   │   └── storage.ts     # chrome.storage.local adapter
│   │   └── icons/
│   │
│   └── vscode-extension/      # VS Code extension
│       ├── src/
│       │   ├── extension.ts   # Entry point — registers listeners
│       │   ├── status-bar.ts  # Status bar item management
│       │   ├── webview/       # Sidebar panel with full animations
│       │   └── storage.ts     # VS Code globalState adapter
│       └── package.json
│
├── package.json               # Root workspace config (pnpm workspaces)
├── tsconfig.base.json
├── README.md
├── For_AI_Type.md             # This file
└── Product_Concept_Typing.txt # Original concept document
```

### Tech Stack
| Component | Technology |
|---|---|
| Language | TypeScript (strict mode) |
| Shared Core | Vanilla TS, zero dependencies |
| Chrome Extension | Manifest V3, Content Scripts |
| VS Code Extension | VS Code Extension API |
| Build | esbuild |
| Tests | Vitest |
| Package Manager | pnpm (workspaces) |
| Monorepo | pnpm workspaces |

---

## 4. Key Design Decisions (All Resolved)

These decisions were made through discussion. Do NOT revisit or change them without explicit user approval.

| Decision | Choice | Rationale |
|---|---|---|
| **Time tracking method** | Idle-threshold timer (1s default, configurable) | Measures exact hands-on-keyboard time. Timer runs while typing, pauses after 1s of no input. |
| **Calorie model** | Keystroke-count with WPM multiplier | Base: 0.00004 kcal/keystroke. Multiplier: 1.0× (0–15 WPM), 1.2× (16–50 WPM), 1.5× (50+ WPM). Smooth interpolation between bands. |
| **UI metric display** | One metric at a time, flip/toggle | Typing time is primary (shown by default). Calories revealed on click/tap with a 3D flip animation. |
| **Browser scope** | All websites (`<all_urls>`) | Track keystrokes everywhere. |
| **Data persistence** | 7-day rolling history | Daily stats archived at midnight. Oldest day pruned. |
| **VS Code animation** | Option C — status bar + webview sidebar | Status bar: quick glance (`🔥 1h 23m`). Sidebar: full animated panel matching Chrome popup. |
| **Calorie disclaimer** | Yes, casual/fun tone | "🔥 This is a fun project! Calorie estimates are hilariously approximate. Don't cancel your gym membership." |
| **Fire animation** | CSS keyframes with smooth interpolation | Color driven by `filter: hue-rotate()`. No snapping between states — smooth lerp based on exact WPM. |
| **WPM calculation** | 5-second sliding window | Circular buffer of keystroke timestamps. Natural decay when typing stops creates automatic color fade-out. |
| **Privacy** | Never read key content | Only `keydown` event trigger is used. Never read `event.key` or `event.code`. All data local. No network requests. |

---

## 5. Coding Rules — MUST FOLLOW

### 5.1 Comments & Documentation
- **Comment like a professional.** Every function, class, and module gets a JSDoc comment explaining what it does and why.
- **Explain the "why", not the "what".** Don't write `// increment counter` above `counter++`. Do write `// Reset the idle timer — any keystroke means the user is still actively typing`.
- **File headers.** Every source file starts with a brief comment block explaining the file's purpose and its role in the architecture.
- **Inline comments** for non-obvious logic, especially math (calorie formulas, WPM calculations, HSL color mapping).

### 5.2 Code Style & Quality
- **TypeScript strict mode.** No `any` types unless absolutely unavoidable (and then comment why).
- **Descriptive variable names.** `keystrokeTimestamps` not `ksTs`. `idleThresholdMs` not `thresh`.
- **Small, focused functions.** Each function does one thing. If a function is longer than ~30 lines, it probably needs splitting.
- **No magic numbers.** Use named constants. `const IDLE_THRESHOLD_MS = 1000` not a bare `1000`.
- **Error handling.** Never silently swallow errors. At minimum, log them.
- **Immutability preferred.** Use `const` over `let`. Use `readonly` in interfaces where appropriate.

### 5.3 Things You Must NOT Do
- ❌ **Do NOT run `git`, `npm`, `pnpm`, `python`, `pip`, or any package manager commands.** The user will handle all installation, building, and version control.
- ❌ **Do NOT run verification commands** (tests, builds, linting). Write the code; the user verifies.
- ❌ **Do NOT create `.git` directories or initialize repos.**
- ❌ **Do NOT install dependencies.** You can specify what needs to be installed in comments or docs, but never run install commands.
- ❌ **Do NOT use `any` type** in TypeScript without a comment justifying it.
- ❌ **Do NOT leave TODO/FIXME comments** unless they are genuine future work items with a description.
- ❌ **Do NOT write placeholder or stub code.** Every function you write should be fully implemented.
- ❌ **Do NOT remove or modify existing comments** that are unrelated to your changes.

### 5.4 File Organization
- One export per file where practical (keeps imports clean)
- Tests live next to source files or in a `tests/` directory mirroring `src/` structure
- CSS custom properties for all design tokens (colors, spacing, animation timing)
- No hardcoded colors in CSS — everything goes through custom properties

### 5.5 Commit-Ready Code
- Every file you create should be complete and functional — no partial implementations
- Code should be self-documenting enough that the README and comments alone explain everything
- Think about what a contributor opening a PR would need to understand

---

## 6. Privacy — Non-Negotiable

This is a keystroke-adjacent tool. Privacy is paramount:

- **NEVER** read `event.key`, `event.code`, `event.keyCode`, or any property that reveals what character was typed
- **NEVER** read DOM content, page text, or document titles
- **NEVER** make network requests — all data stays in local storage
- **NEVER** store timestamps with enough precision to reconstruct typing rhythm at character level
- The content script listens for `keydown` events ONLY as a trigger signal (timestamp + count)
- This will be verified in code review and documented in `PRIVACY.md`

---

## 7. Current Project State

### What Exists
- `Product_Concept_Typing.txt` — Original concept document
- `README.md` — Project README (to be fully written)
- `For_AI_Type.md` — This dossier file
- `.gitignore` — Git ignore rules
- Implementation plan — Detailed phased plan (in conversation artifacts)

### What Needs to Be Built (Phases)
1. **Phase 1: Shared Core Engine** — types, WPM calculator, engine, calories, fire state, storage interface, unit tests
2. **Phase 2: Chrome/Brave Extension** — manifest, content script, service worker, popup UI with animations
3. **Phase 3: VS Code Extension** — document listener, status bar, webview sidebar panel
4. **Phase 4: Polish & UX** — animations, number effects, weekly history chart, settings
5. **Phase 5: Open Source Release** — README, CONTRIBUTING.md, PRIVACY.md, LICENSE, CI/CD, packaging

### What Has NOT Been Built Yet
- No source code has been written yet
- No packages have been initialized
- No dependencies have been installed

---

## 8. UI/UX Specifications

### Fire Emoji Animation States
| State | WPM Range | Color (HSL Hue) | Animation |
|---|---|---|---|
| Idle | 0 (>1s no input) | Gray | Static, `opacity: 0.5` |
| Casual | 1–15 | 🟢 Green (hue: 120) | Slow gentle glow, 2s pulse cycle |
| Flow | 16–50 | 🟠 Orange (hue: 30) | Medium pulse, 1s cycle + subtle scale |
| Rage | 50+ | 🔴 Red (hue: 0) | Intense shake + pulse, 0.3s cycle |

Colors interpolate smoothly between states — no harsh snapping.

### Popup Layout
- Dark theme with glassmorphism
- Large animated fire emoji as hero element
- **Flippable stat card**: typing time (front) ↔ calories (back) on click
- Secondary stats always visible: keystrokes + WPM
- Session timeline for today
- "View History" → 7-day sparkline chart

### VS Code Status Bar
- Shows one metric at a time: `🔥 1h 23m` (default) → `🔥 3.2 kcal` (on click, 3s then flips back)
- Fire emoji color changes with typing speed
- Click opens webview sidebar panel

---

## 9. Calorie Math Reference

```
Base calorie per keystroke: 0.00004 kcal

WPM Multiplier (smoothly interpolated):
  0–15 WPM  → 1.0×
  16–50 WPM → 1.2×
  50+ WPM   → 1.5×

Example: Typing at 40 WPM for 1 hour (~4800 keystrokes)
  = 4800 × 0.00004 × 1.2
  = 0.2304 kcal

Disclaimer: "🔥 This is a fun project! Calorie estimates are based on
keystroke activity and are hilariously approximate. Don't cancel your
gym membership."
```

---

## 10. Quick Reference for AI Assistants

**Before you write code, verify:**
- [ ] Have you read this entire file?
- [ ] Are you following the coding rules in Section 5?
- [ ] Are you NOT running any commands listed in Section 5.3?
- [ ] Does your code respect the privacy rules in Section 6?
- [ ] Are your comments professional and meaningful?
- [ ] Is every function fully implemented (no stubs)?
- [ ] Are you using named constants instead of magic numbers?
- [ ] Is TypeScript strict mode respected (no `any`)?

**When in doubt:** Ask the user. Don't guess at design decisions.
