# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A Space Invaders clone implemented entirely in a single self-contained file, [index.html](index.html). There is no build system, package manager, bundler, or test suite — HTML, CSS, and JavaScript all live in this one file, and audio is synthesized at runtime via the Web Audio API (no external image or sound assets).

## Running the game

Since the file uses no ES modules or fetch calls, it can be opened directly:

```bash
open index.html
```

For a more browser-realistic environment (e.g. to avoid `file://` restrictions), serve it locally:

```bash
python3 -m http.server 8934
# then visit http://localhost:8934/index.html
```

There is no lint, build, or test command — verify changes by loading the page in a browser and playing through the golden path (start → move/shoot → kill aliens → take damage → wave clear or game over → restart).

## Architecture

Everything is an IIFE at the bottom of `index.html` that runs after the DOM is parsed. Key pieces, in the order they appear in the script:

- **Input**: a single `keys` object keyed by lowercased `e.key`, populated by `keydown`/`keyup` listeners on `window`. Movement/shooting checks in `update()` read from this object directly rather than reacting to events.
- **Audio engine**: synthesized sound only — `beep()` (oscillator-based tones) and `noiseBurst()` (generated noise buffer) are low-level primitives; the `sfx` object composes them into named effects (`shoot`, `alienExplode`, `playerExplode`, `waveClear`, `gameOver`). The classic descending 4-note alien-march beat is driven separately by `alienStepBeep()`, called once per alien grid step. `ensureAudio()` lazily creates the `AudioContext` on the Start button click (required by browser autoplay policy) — don't call audio functions before this fires.
- **Game state**: plain module-scoped variables/arrays (`score`, `lives`, `wave`, `player`, `bullets`, `enemyBullets`, `aliens`, `particles`) rather than a class or state machine. `resetGame()` reinitializes all of it; `spawnAliens()` rebuilds the alien grid and scales `alienSpeed` with `wave`.
- **Update/draw split**: `update()` mutates state (movement, shooting, collisions via `rectsOverlap()`, wave/life transitions) and `draw()` is a pure render pass (`drawStars`, `drawAlien`, `drawPlayer`, `drawBullets`, `drawParticles`) with no side effects on state. `loop()` calls both via `requestAnimationFrame` and is gated by the `running` flag.
- **Overlay flow**: the `#overlay` DOM element (not canvas-drawn) handles the start screen and game-over screen, toggled via the `.hidden` class. `gameOver()` stops the rAF loop, plays the game-over jingle, and repopulates the overlay text/button before showing it again.

When adding new game mechanics, follow the existing pattern: add state variables near the top of the IIFE, mutate them in `update()`, and render them in a dedicated `draw*()` function called from `draw()`. When adding new sounds, add a primitive-composed entry to `sfx` rather than inlining oscillator code at the call site.
