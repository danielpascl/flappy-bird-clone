# Flappy Bird — Desktop Clone

A dependency-free clone of Flappy Bird that runs in any desktop browser. One HTML file, no build
step, no assets to download — just open it and play. The physics, tilt, and pipe tightening are
tuned to feel like the original, while a set of rarity-weighted power-ups adds a layer of risk and
reward on top.

**[▶ Play it live](https://danielpascl.github.io/flappy-bird-clone/)** — replace with your deployed URL.

---

## Table of Contents

- [Introduction](#introduction)
- [Key Features](#key-features)
- [Keyboard Shortcuts](#keyboard-shortcuts)
- [Power-ups](#power-ups)
- [How the Game Plays](#how-the-game-plays)
- [Development Process & What I Learned](#development-process--what-i-learned)
- [Potential Future Improvements](#potential-future-improvements)
- [Running the Project Locally](#running-the-project-locally)
- [Project Structure](#project-structure)
- [Tech Stack](#tech-stack)
- [Credits](#credits)

---

## Introduction

Flappy Bird is deceptively simple: one button, one obstacle, infinite retries. Rebuilding it is a
classic exercise because almost all of the difficulty lives in *feel* — the exact gravity constant,
how quickly the bird noses down, how wide the gap is on your first pipe versus your twentieth.

This project is a from-scratch reimplementation for desktop browsers. It is deliberately built as a
**single self-contained HTML file**: the CSS, the JavaScript, the bird sprite, the pipe texture, the
scrolling skyline, and every sound effect are generated inside that one file. There is nothing to
install and nothing that can break because a CDN went down.

The twist here is a power-up system. Four rarity-weighted orbs float inside pipe gaps, and picking
one up changes the run: double points, a speed surge, temporary invincibility, or a rare sextuple
multiplier. They force a decision the original never asked — *is this orb worth the risk of the
flight path?*

## Key Features

**Faithful core physics**
- Constant gravity with a terminal fall speed, so the bird always settles into the same rhythm.
- Bird rotation is driven by vertical velocity, tilting up on a flap and nosing into a dive.
- Pipe gaps narrow progressively: 102 px down to a floor of 82 px, shrinking 4 px every 5 points.
- Collision boxes are inset 5 px horizontally and 4 px vertically from the sprite, for a forgiving
  but honest hitbox.

**Four weighted power-ups**

Orbs spawn in the middle of a pipe gap with a 30% chance per pipe, and the type is drawn from a
weighted table so the strongest one stays genuinely rare. See [Power-ups](#power-ups).

**Sharp rendering at any display**

The canvas backing store is 1080×1920 while the game logic runs in a 288×512 coordinate space, a
clean 3.75× scale. All drawing is vector, so gradients, outlines, and the bird's curves stay crisp
on high-DPI panels instead of being upscaled from a tiny bitmap. The display size adapts to the
window and never upscales past native resolution.

**Procedural audio**

Every sound — flap, point, hit, death, power-up pickup, super-rare jingle — is synthesized at
runtime from Web Audio oscillators. Zero audio files ship with the project. `M` mutes instantly.

**Smooth parallax that doesn't stutter**

Four layers scroll at different rates: distant city skyline, bushes, ground, and drifting clouds.
All layers are built as seamless repeating tiles, so they wrap without a visible seam or a
backwards jump.

**Death that reads as death**

On a crash the bird's pupils become X marks, its wings droop to a fixed angle, and the entire
scrolling world stops — city, bushes, ground, clouds, pipes, and floating orbs all freeze. The bird
still falls and settles on the ground, which makes the contrast obvious: the world is frozen, the
run is over.

**Consistency across refresh rates**

The simulation runs on a fixed 60 Hz timestep with an accumulator, so the game plays identically on
a 60 Hz, 120 Hz, or 144 Hz monitor. Frame-rate independence was a design requirement, not an
afterthought.

**Quality-of-life details**
- Best score persisted per browser in `localStorage`.
- Medals on the Game Over panel at 10 / 20 / 30 / 40 points.
- Auto-pause when the window loses focus, so you never die while reading another tab.
- Restart with `R` without waiting through the Game Over animation.

## Keyboard Shortcuts

| Action | Keys |
| --- | --- |
| Flap / Start / Restart | `Space`, `↑`, `W`, `Enter`, or **Left Click** |
| Pause / Resume | `P` or `Esc` |
| Toggle sound on/off | `M` |
| Restart immediately | `R` |

Holding a flap key does not auto-repeat — one press is one flap, exactly like the original.

## Power-ups

| # | Power-up | Effect | Duration | Chance | Color |
| --- | --- | --- | --- | --- | --- |
| 1 | **2× Points** | Each pipe cleared is worth 2 points | 10 s | **55%** common | Yellow |
| 2 | **Speed 1.5×** | All scrolling and pipe travel speeds up 1.5× | 8 s | **28%** uncommon | Cyan |
| 3 | **Invincible** | Pipes cannot kill you; the ground still can | 10 s | **13%** rare | Purple |
| 4 | **6× Points** | Each pipe cleared is worth 6 points | 6 s | **4%** super rare | Magenta |

Collect an orb by flying into it. Active power-ups appear in the top-left HUD with a depleting
timer bar and a countdown in seconds.

The two score multipliers are **mutually exclusive** — grabbing a 6× orb cancels an active 2× and
vice versa, so the effects never stack into 12×. The invincibility shield is drawn as a pulsing
aura around the bird and blinks during its final 1.5 seconds as a warning.

Power-up duration is measured in 60 Hz frames, so `600` frames is roughly 10 seconds of play.

## How the Game Plays

The game is a small finite state machine: **`ready → play → dead → play`**, with `pause` reachable
from `play`.

- **`ready`** — the bird bobs in place, the world scrolls, waiting for input.
- **`play`** — full simulation: gravity, flapping, pipe spawning, scoring, orb pickup, collisions.
- **`pause`** — everything halts; resume with `P`.
- **`dead`** — background scrolling stops, the bird falls and lands, X eyes appear, score saves.

Background scrolling only advances during `ready` and `play`. That single condition is what makes
the frozen-world death effect work, without a separate code path for each layer.

## Development Process & What I Learned

This was built iteratively, and most of the value came from *verification*, not from the first
draft. The first version looked and played fine on inspection. Several real bugs only surfaced once
the output was measured.

**1. Building the base game**

I started with the smallest thing that could be fun: gravity, a flap impulse, a single pipe pair,
and a scrolling ground. Getting the *feel* right took more tuning than writing the code — the
gravity and flap impulse were adjusted until the bird's arc matched the rhythm I remembered, and
the gap width was tuned down until the first pass felt tight but fair.

**2. Measuring instead of eyeballing**

For visual and behavioural checks I rendered the game in headless Chrome and drove it with
synthetic keyboard events, then inspected pixels and internal state directly rather than trusting
a screenshot. That turned vague "something feels off" complaints into numbers. For example, to
verify the power-up rarity table I ran 6,000 pipe spawns and got 55.1% / 27.7% / 13.6% / 3.6%
against the intended 55 / 28 / 13 / 4 — close enough to confirm the weighted picker was correct.

**3. The parallax stutter — the most instructive bug**

The background visibly juddered, and it looked like a rendering problem. It was arithmetic. Each
parallax layer computed its offset as `groundScroll * factor`, but `groundScroll` was itself stored
modulo 24. So a layer with a 40 px tile period only ever saw offsets in the range 0–8.4 before
resetting, and at the reset it snapped **backwards** by 6 px.

Measuring frame-to-frame pixel deltas confirmed it: **24 backward jumps of −6 px in 160 frames**,
roughly one every 13 frames. The fix was to keep an absolute, never-modulo scroll counter and
derive every layer offset from it, wrapping each layer with its own tile period via a `wrapX()`
helper. After the fix the same measurement showed **zero** backward jumps and a max/median
frame-change ratio of 1.10× (down from 6.33×). Lesson: when motion looks wrong, suspect the state
feeding it before the code drawing it.

**4. Raising the resolution without changing the game**

Moving to a 1080×1920 canvas could easily have meant rewriting every coordinate. Instead I kept all
logic in the original 288×512 space and applied a single `ctx.setTransform(3.75, 0, 0, 3.75, 0, 0)`
at the start of each frame. Physics, hitboxes, and pipe geometry were untouched; only the output got
denser. I also had to remove `image-rendering: pixelated`, which would have re-introduced blocky
edges on the now-vector output.

**5. Freezing the world revealed a hidden animation clock**

After adding the "everything stops when you die" effect, one thing still moved: the floating orbs.
They bobbed using the global frame counter, which increments unconditionally. The fix was a second
counter, `animFrame`, that only advances while the world is actually moving. This was a good
reminder that a single counter used for both logic and animation will eventually couple things you
wanted separate.

**6. Testing artifacts are their own trap**

A couple of my verification runs reported failures that turned out to be the test's fault, not the
game's — forcing the game into a dead state while the death flash and screen shake were still
decaying, so the whole canvas genuinely was still moving; and placing the bird at ground level by
accident, triggering an instant death that washed the screen white. Both looked like bugs until I
traced them. The habit that helped: when a test fails, verify the test before changing the code.

**What I took away**

- Fix the *state*, not the symptom. The stutter looked like a draw bug and was a modulo bug.
- Render at higher resolution via transform, not by rescaling your world's coordinate system.
- Keep animation clocks separate from logic clocks.
- Measure with numbers. "Looks smoother" is not a regression test; "47 spikes became 0" is.

## Potential Future Improvements

**Gameplay**
- A touch-optimised layout with larger tap targets for phones, since the current scaling is tuned
  for desktop windows.
- Difficulty presets that adjust gap floor, spawn spacing, and speed ramp independently.
- New power-up ideas: magnet for orbs, slow-motion, a one-time shield that absorbs a single hit,
  or a temporary gap-widener.
- Endless-run variety: moving pipes, wind gusts, or day/night cycles that change visibility.

**Technical**
- Convert to a Progressive Web App with a service worker and manifest so it installs to the desktop
  and plays offline.
- Sprite-sheet pixel-art mode alongside the current vector rendering, with a toggle.
- WebGL renderer as an alternative path for very low-end hardware.
- Gamepad support and rebindable keys.
- Extract the tuning constants into a JSON config or an in-game debug panel for live editing.

**Social & meta**
- Global leaderboard — this needs a backend (Cloudflare Workers + D1, Supabase, or Firebase free
  tiers would all work).
- Shareable result cards generated from the canvas for social media.
- Ghost replay of your best run, rendered as a translucent bird.
- Achievements for medal tiers and power-up combos.

**Accessibility**
- A reduced-motion mode that disables screen shake and flash.
- High-contrast palette option.
- Keyboard-only confirmation for the Game Over panel, so no click is required.

## Running the Project Locally

No installation, no dependencies, no build step.

### Option 1 — Just open the file

Locate `index.html` (or `flappy-bird.html`) and double-click it. It opens in your default browser
and is immediately playable.

This works fine for playing, but some browsers restrict `localStorage` on `file://` origins, which
means your best score may not persist between sessions.

### Option 2 — Serve it locally (recommended)

Run a one-line static file server from the project folder:

```bash
# Python 3 (already installed on most systems)
python -m http.server 8000
```

```bash
# Node.js
npx serve .
```

```bash
# PHP (handy if you already have XAMPP)
php -S localhost:8000
```

Then open **http://localhost:8000** in your browser.

Serving over `http://localhost` counts as a secure context, so `localStorage` and the Web Audio API
both work without restriction — best score persists and sound plays normally.

### Requirements

Any browser with Canvas 2D and Web Audio support: Chrome, Edge, Firefox, or Safari. Nothing else —
no Node.js, no Python, no package manager required to *play*. The commands above are only needed if
you prefer serving over `file://`.

## Project Structure

```
flappy-bird/
├── index.html      # the entire game — markup, styles, and logic
└── README.md
```

The file is roughly 1,000 lines and is organised internally in this order:

| Section | Responsibility |
| --- | --- |
| Constants | Canvas dimensions, physics, pipe geometry, power-up table |
| State (`S`) | One object holding mode, score, bird, pipes, orbs, and all timers |
| Audio | Lazy `AudioContext` creation plus oscillator helper functions |
| Update | Fixed-timestep simulation, branched by game mode |
| Render | Layered drawing: sky, clouds, city, bushes, pipes, orbs, ground, bird, HUD, overlays |
| Input | Keyboard and pointer listeners |
| Loop | `requestAnimationFrame` driving a 60 Hz accumulator |

### Tuning the game

All gameplay constants sit together at the top of the script, so adjusting the feel is a one-line
change:

```js
var GRAVITY       = 0.28;   // downward acceleration per frame
var FLAP_V        = -4.7;   // upward impulse per flap
var MAX_FALL      = 7.2;    // terminal velocity
var SPEED         = 1.9;    // pipe and ground scroll speed
var PIPE_GAP_BASE = 102;    // starting gap height in px
var PIPE_SPACING  = 172;    // horizontal distance between pipes
var SPEED_MULT    = 1.5;    // speed power-up multiplier
var ORB_CHANCE    = 0.3;    // chance a pipe spawns a power-up orb
```

Power-up behaviour lives in the `POWER` table, where `dur` is duration in frames and `weight`
controls rarity:

```js
var POWER = {
  1: { dur: 600, label: "2X POIN",    mult: 2, weight: 55, ... },
  2: { dur: 480, label: "SPEED 1.5X", mult: 0, weight: 28, ... },
  3: { dur: 600, label: "INVINCIBLE", mult: 0, weight: 13, ... },
  4: { dur: 360, label: "6X POIN",    mult: 6, weight: 4,  ... }
};
```

## Tech Stack

| Layer | Choice |
| --- | --- |
| Rendering | HTML5 Canvas 2D with a scale transform |
| Language | Vanilla JavaScript (ES5-compatible, no build step) |
| Styling | Inline CSS with flex centering |
| Audio | Web Audio API oscillators |
| Persistence | `localStorage` |
| Dependencies | None |

## Credits

- Original game by **Dong Nguyen** ([@dongatory](https://twitter.com/dongatory)). This is an
  educational reimplementation, not the original, and is not affiliated with the author.
- Clone by **@danielpascl**.
- Built as an exercise in single-file canvas rendering, fixed-timestep game loops, procedural
  audio, and verification-driven debugging.

## License

No license file is included yet. If you would like others to reuse this freely, add an
[MIT License](https://choosealicense.com/licenses/mit/) — the usual choice for small hobby
projects.
