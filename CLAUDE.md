# Orbit

A simple browser gravity game. You launch a moon into orbit around one or more fixed planets. Keep it moving for 15 seconds without hitting a planet or leaving the screen and you pass the level. It's pass/fail only, with no score.

The whole game is one self-contained file, `index.html`, with inline CSS and JS, a canvas, and no dependencies or build step. It's hosted with GitHub Pages from `main` / root at davelangford.github.io/orbit.

Inspired by an old Flash game called "Orbit" (launch a comet, survive the timer) that's no longer playable anywhere.

## How it plays

- Press inside the level's launch zone (dashed blue circle; presses within ~24px snap to its edge), drag back slingshot-style, and release to launch. The press point is where the moon starts. Velocity = (press − release) × `LAUNCH_K`, capped at `MAX_V`.
- While aiming, a dotted trajectory preview shows the next `PREVIEW_T` seconds of sim time. It stops where the path hits a planet or leaves the screen.
- A cancel spot (✕, bottom-centre, about 70px up to clear phone gesture bars) appears while you drag. Releasing on it cancels the launch.
- After a failed attempt, the previous launch's press point (solid dot), release point (hollow ring) and the line between them show as a faint orange ghost guide. The ghost clears on a new level.
- **Fail:** the moon hits a planet ("Crashed into a planet") or fully leaves the visible screen ("Flew off the screen"). After about 1.3s you're back aiming at the same level, with unlimited retries.
- **Pass:** survive `TARGET` seconds. Shows "PASS", and you tap for the next level. The moon keeps flying behind the overlay.
- **Timer:** a large, faint countdown number in the centre background (the ✓ on pass). In the last 5 seconds the background and the number pulse on each tick, harder as it nears 0.
- **Levels:** each has a launch zone, so you can't just park next to one planet. 1–6 are hand-made layouts in `LAYOUTS` (planets + zone) (1 planet, then 2, 2 equal, a triangle of 3, centre + 2 small, a square of 4). From level 7 they're randomly generated with spacing checks, and the planet count grows up to 6.
- The HUD shows only the level number. The current level is saved in `localStorage` (`orbit.level`) when you advance, so a reload resumes there. A "↺ restart" button in the HUD (with confirm) resets to level 1.

## Key constants (top of the script)

| Constant | Value | Meaning |
|---|---|---|
| `TARGET` | 15 | seconds to pass a level (real time) |
| `SPEED` | 1.25 | sim speed-up: same orbits, moon moves faster |
| `PREVIEW_T` | 10 | seconds of sim time shown in the aim preview (~8s real) |
| `LAUNCH_K` | 2.2 | drag distance → launch speed |
| `MAX_V` | 700 | launch speed cap |
| `K` | 3200 | planet gravity = K × radius² |
| `MOON_R` | 7 | moon radius (world units) |

**World coordinates:** about −500..500 on the shorter screen axis, centred. `S = min(W,H)/1000` converts to screen pixels. So phones in portrait have less horizontal room, and a given drag in pixels gives a bigger launch on a small screen.

## Important: the preview must match the flight exactly

Flight and preview both use the same fixed-step integrator: `physStep()` with `H_STEP = 1/600`, semi-implicit Euler. Flight builds up `dt × SPEED` in `simAcc` and runs whole fixed steps, checking planet hits and off-screen after every step, the same way the preview does. Launch velocity comes from the same formula as `launchVel()`.

An earlier version used frame-dependent substeps for the flight and a different step for the preview, and with multiple planets the two drifted apart visibly. **Don't reintroduce variable-step physics.** Any change to the physics must go through `physStep()` so both stay identical.

## Mobile

Pointer events handle mouse and touch. Zoom, scrolling, long-press menus and tap highlights are all disabled (viewport meta, `touch-action:none`, a `touchmove` preventDefault, contextmenu blocked). `devicePixelRatio` is capped at 2.

## Design decisions already tried and rejected

- Score, best score, bonus and lives: removed. The owner wants pass/fail only.
- A timer bar along the top: not visible enough.
- A timer line running around the screen border: felt wrong.
- A progress ring around the moon: disliked. Keep the big background number.
- The off-screen arrow indicator: removed, because leaving the screen is now a fail.

## Working with Dave

- Prefers blunt recommendations, not hedging. Keep replies short and plain, not overly structured or enthusiastic.
- Favours shipping over polish. Make the change, say in a line what changed, and flag any side effects (e.g. "this makes it easier").
- Small iterative tweaks are the norm. Most changes are a constant or a few lines.
