# CLAUDE.md — Rutina-YAH Codebase Guide

## Project Overview

**Mi Rutina 🌸** is a personalized gym routine tracker PWA (Progressive Web App) for a Spanish-speaking user. It is a **zero-dependency, single-file HTML/CSS/JS app** designed for women's fitness — specifically fat loss and post-meniscus rehabilitation.

- No build step, no package manager, no framework
- All code lives in 3 files: `index.html`, `manifest.json`, `sw.js`
- Deployed as static files on any web server

---

## File Structure

```
Rutina-YAH/
├── index.html      # Entire app: HTML + embedded CSS + embedded JS (~1135 lines)
├── manifest.json   # PWA manifest (name, icons, theme colors)
├── sw.js           # Service Worker for offline caching
└── CLAUDE.md       # This file
```

### index.html Sections (in order)

| Lines (approx) | Content |
|---|---|
| 1–30 | `<head>` with meta tags, PWA links, font imports |
| 31–320 | `<style>` — full CSS with custom properties |
| 321–600 | `<body>` HTML — header, tabs, exercise cards, modals |
| 601–1135 | `<script>` — all application logic |

---

## Data Architecture

### No Backend — LocalStorage Only

All state is persisted to `localStorage` with the prefix `fba_*`.

| Key | Type | Description |
|---|---|---|
| `fba_done_v2` | JSON object | `{ [dayIdx]: [exIdx, ...] }` — completed exercises |
| `fba_reps_v1` | JSON object | `{ "dayIdx-exIdx": setNumber }` — current set per exercise |
| `fba_streak_v1` | JSON object | `{ count, lastDate }` — consecutive completion streak |
| `fba_weights_v1` | JSON object | `{ "exercise_name": [{ date, lbs }, ...] }` — weight history (max 30 entries) |

### Core Data Model — `DAYS` Array

Hardcoded in `index.html`. Each entry represents one training day (Mon–Fri):

```javascript
{
  name: 'Lunes',                  // Spanish day name
  emoji: '🍑',                    // Display emoji
  focus: 'Glúteos + Isquios',     // Target muscle group
  cardio: 'string',               // Low-impact cardio recommendation
  metabolic: false,               // true = full-body metabolic day
  warning: 'string | null',       // Optional warning note
  exercises: [
    {
      name: 'string',             // Exercise name (used as weight log key)
      sets: 3,                    // Number of sets
      reps: '12-15',              // Rep range string
      rest: 60,                   // Rest time in seconds
      note: 'string',             // Muscle group note
      equip: 'string',            // Equipment (e.g. "Mancuernas")
      gif: 'URL',                 // Giphy URL for demonstration
      core: false                 // true = categorized as core exercise
    }
  ]
}
```

---

## JavaScript Conventions

### State Variables (module-level)

```javascript
let doneMap = {};       // { dayIdx: Set<exIdx> }
let repState = {};      // { "d-e": currentSet }
let streak = {};        // { count, lastDate }
let weightLog = {};     // { exerciseName: [{date, lbs}] }
let timerInterval;      // active setInterval handle
let tDur, tRem;         // timer: total duration, remaining seconds
let tDay, tEx, tSets;   // timer context: which day/exercise/set count
let wakeLock = null;    // WakeLockSentinel handle
```

### Function Naming Patterns

| Pattern | Purpose | Example |
|---|---|---|
| `load[X]()` | Parse localStorage → populate state | `loadDone()`, `loadStreak()` |
| `save[X]()` | Serialize state → localStorage | `saveDone()`, `saveWeights()` |
| `render[X]()` | Build HTML string → set innerHTML | `renderDay(idx)` |
| `open[X]()` | Show modal/overlay | `openTimer(dur, day, ex, sets)` |
| `close[X]()` | Hide modal/overlay | `closeTimer()` |
| `update[X]()` | Partial DOM update (no full re-render) | `updateRing()`, `updateDots(day, ex)` |

### Key Business Logic Functions

- `toggle(day, exIdx)` — marks an exercise done/undone, triggers celebration check
- `openTimer(duration, day, exIdx, sets)` — starts rest timer, tracks set progress
- `checkCelebration(dayIdx)` — fires celebration modal when all exercises complete
- `confetti()` — 130-particle CSS animation burst
- `beep()` — Web Audio API tone on timer end

---

## Styling Conventions

### CSS Custom Properties (`:root`)

```css
--pink: #e91e8c;
--pink-light: #fce4ec;
--purple: #9c27b0;
--blue: #2196f3;
--green: #4caf50;
--radius: 16px;
--shadow: 0 2px 12px rgba(0,0,0,.08);
```

- Mobile-first responsive design
- Flexbox and CSS Grid for layout
- All animations use `@keyframes` (no JS animation libraries)
- UI feedback via CSS class toggling (`.open`, `.done`, `.active`)

---

## PWA / Web APIs Used

| API | Usage |
|---|---|
| Service Worker | Offline caching via `sw.js` (cache name: `rutina-v1`) |
| Wake Lock | Prevents screen sleep during workouts |
| Web Notifications | Timer completion alerts |
| Vibration | Haptic feedback on timer end |
| Web Audio | `AudioContext` beep sound on timer completion |

### Service Worker Cache Strategy

Network-first for same-origin requests; falls back to cached versions. Cached assets: `./`, `./index.html`, `./manifest.json`.

---

## Development Workflow

### Making Changes

1. Edit `index.html` directly — CSS, JS, and HTML are all embedded
2. No build step required — open in browser or serve statically
3. Test offline behavior by toggling DevTools Network → Offline

### Testing Manually

- Open `index.html` in a browser (or via `python3 -m http.server`)
- Test each day tab, exercise completion, timer, weight logging, streak
- Test PWA install prompt and offline mode
- Check mobile view (DevTools device toolbar)

### Adding Exercises

Add a new object to the appropriate day's `exercises` array in the `DAYS` constant:

```javascript
{
  name: 'Exercise Name',
  sets: 3,
  reps: '10-12',
  rest: 60,
  note: 'Músculo objetivo',
  equip: 'Equipment needed',
  gif: 'https://media.giphy.com/...', // Find on giphy.com
  core: false
}
```

### Adding a New Training Day

Append a new object to the `DAYS` array and add a corresponding tab button in the HTML tabs section.

---

## Language & Tone

The app UI is entirely in **Spanish**. When modifying UI text, exercise names, or notifications:
- Keep all user-facing strings in Spanish
- Maintain the encouraging, feminine tone (e.g., "¡Lo lograste! 🎉")
- Preserve emoji use in headings and labels

---

## Git Conventions

- Branch pattern: `claude/<description>-<sessionId>`
- Commit messages in Spanish or English, describing the feature/fix clearly
- Merges from Claude-assisted branches go through PRs to `master` (or `main`)
- Push with: `git push -u origin <branch-name>`

---

## Common Pitfalls

1. **LocalStorage key versioning** — when changing the schema of stored data, increment the version suffix (`v1` → `v2`) to avoid stale data conflicts
2. **Exercise name as weight log key** — exercise names in `DAYS` must match exactly what gets stored in `weightLog`; changing a name breaks existing weight history
3. **AudioContext requires user gesture** — `beep()` only works after a user interaction (browser policy); the app already handles this
4. **GIF URLs** — use direct Giphy media URLs (`media.giphy.com`), not share links
5. **Timer state** — `tDay`, `tEx`, `tSets`, `tDur`, `tRem` must all be consistent; bugs arise if timer state is partially reset
