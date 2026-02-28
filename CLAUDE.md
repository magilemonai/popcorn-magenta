# GREG'S ADVENTURE

A comedy platformer where Greg, a 43-year-old accountant, is accidentally sucked into a video game. He is NOT happy about it. An unhinged Narrator tries to make it epic. Chaos ensues.

## Project Structure

Single-file HTML game: `index.html` (~1700 lines). No build tools, no dependencies. Open in a browser and play.

- **Canvas**: 800x500, scales to fit window
- **Tile size**: 32x32px
- **Player**: 20x30px hitbox

## Code Architecture

The game is organized into clearly labeled sections within one `<script>` tag:

| Section | Line ~  | Purpose |
|---------|---------|---------|
| Constants | 32 | Tile size, gravity, speed, jump force |
| Input | 40 | Keyboard state + just-pressed tracking (`keys`, `jp`) |
| Sound | 50 | Web Audio API synthesized effects (no audio files) |
| Camera | 124 | Smooth follow + screen shake |
| Particles | 139 | Simple particle emitter for juice |
| Dialogue | 165 | Queue-based typewriter system (`say()` function) |
| Game State | 234 | All global state variables |
| Title Screen | 266 | Animated title with duck + comedy subtitle |
| Draw Helpers | 342 | `drawDuck()` and `drawGreg()` sprite renderers |
| Level Data | 428 | `LEVELS[]` array — maps, triggers, dialogues |
| Level Loading | 654 | `startLevel()` — parses ASCII map into tiles/entities |
| Collision | 731 | Tile-based AABB: `solid()`, `oneWay()`, `movePlayer()` |
| Entity Update | 856 | Coins, enemies, ducks, traps, coffee, goals |
| Boss Update | 942 | Spreadsheet boss AI, projectiles, phase transitions |
| Triggers | 1043 | Position-based dialogue triggers (fire once per level) |
| Rendering | 1074 | Background, tiles, platform labels, entities, boss, HUD |
| Ending & Credits | 1385 | Office scene, timed credit cards, duck wink |
| Glitch Effects | 1537 | Level 3 visual chaos — scanlines, fake errors, screen tear |
| Fade | 1613 | Fade-to-black transitions between levels |
| Main Loop | 1630 | `requestAnimationFrame` loop — update then render |

## Level Format

Each level in `LEVELS[]` is an object with:
- `name` / `sub` — chapter title and subtitle
- `bg` — gradient colors `[top, bottom]`
- `tileCol` / `tileSide` — ground tile colors
- `map` — array of strings (ASCII tilemap)
- `triggers` — `[{x, y, id}]` dialogue trigger positions
- `dialogues` — `{id: [[speaker, text], ...]}` conversation scripts
- `platLabels` — (optional) floating text labels on platforms
- `isBoss` — (optional) enables boss update logic

### Map Characters
- `' '` = air
- `'#'` = solid block
- `'-'` = one-way platform (jump through from below)
- `'P'` = player spawn
- `'G'` = goal/exit
- `'c'` = coin (Exposure Buck)
- `'d'` = duck (Quackthar)
- `'e'` = patrol enemy
- `'t'` = trap platform (shakes then falls)
- `'B'` = boss spawn

## Game Levels

1. **The Onboarding** — Tutorial. Greg argues with every mechanic. Quackthar the Destroyer (a duck) does nothing.
2. **The Workplace** — Office enemies with lanyards. Printers are "Paper Elementals."
3. **The Narrator Snaps** — Magenta chaos. Fake error dialogs, screen glitches, too many enemies. Platforms labeled "DEFINITELY REAL" and "FREE PIZZA →".
4. **Annual Performance Review** (boss) — Sentient spreadsheet with `=SUM(RAGE)`. Throws pie charts and ACTION ITEMS. Coffee powerup triggers FULLY CAFFEINATED mode.

## Key Design Decisions

- **No game over**: When you lose all 3 "Will To Live" hearts, the Narrator brings you back with a rotating set of excuses. This IS the joke.
- **Dialogue is queue-based**: Call `say('N', 'text')` or `say('G', 'text')` to queue narrator/Greg lines. They play sequentially with typewriter effect.
- **Speakers**: `'N'` = Narrator (gold text, purple box), `'G'` = Greg (white text, dark box).
- **Triggers fire once**: The `triggered` map tracks which dialogue triggers have fired per level.
- **All sound is synthesized**: `playSound('jump')`, `playSound('quack')`, etc. No audio files needed.
- **Comedy comes first**: Every system exists to serve a joke. The platforming is a vehicle for the writing.

## Controls

- Arrow keys or WASD: Move
- Space or Up/W: Jump (press again mid-air for double jump)
- Space/Enter: Advance dialogue

## How to Add Content

### New level
Add an entry to the `LEVELS[]` array following the existing format. Design the ASCII map, set up triggers and dialogues.

### New enemy type
1. Add spawn character to `startLevel()` parser
2. Add update logic in `updateEnts()`
3. Add rendering in `drawEnts()`

### New dialogue
Add entries to a level's `dialogues` map. Trigger them by adding a position trigger, or call `say()` directly from game logic (e.g., boss phase changes, death events).

### New sound effect
Add a case to the `playSound()` switch statement using Web Audio API oscillators.

## Testing

Open `index.html` in a browser. There are no automated tests — this is a single-file game. Playtest each level after changes. Key things to verify:
- Player can't clip through walls (collision)
- All triggers fire at the right position
- Boss phases transition correctly
- Dialogue doesn't overlap or get stuck
- Level transitions fade properly
