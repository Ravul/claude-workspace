# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a collection of browser-based games — each is a **single self-contained HTML file** with no build step, no dependencies, and no external assets. Everything (HTML, CSS, JS) lives in one file per game.

## Running Games

Open any `.html` file directly in a browser:
```
open shooter.html
open tictactoe.html
```

No server, no install, no build process.

## Architecture

### Single-File Game Pattern

Each game follows the same structure inside one `.html` file:

```
<style>        — canvas centering, cursor, background
<canvas>       — the rendering surface
<script>
  CONSTANTS    — sizes, colors, config objects
  INPUT        — keyboard/mouse state (event listeners at top level)
  ENTITIES     — ES6 classes (Player, Enemy, Bullet, Particle, etc.)
  STATE        — single global object holding all game state
  UPDATE fns   — one function per entity type, called each frame with dt
  DRAW fns     — one function per entity type, pure rendering
  GAME LOOP    — requestAnimationFrame, dispatches update+draw by STATE.screen
```

### Rendering

- All rendering uses **HTML5 Canvas 2D API** — no WebGL, no libraries.
- Sprites are drawn programmatically with Canvas primitives (arcs, rects, lines).
- Draw order: background → world entities → particles → HUD → screen overlays.
- Screen shake is applied via `ctx.save()` / `ctx.translate(shake.x, shake.y)` / `ctx.restore()` so the HUD (drawn after restore) is always shake-stable.

### Game Loop Convention (`shooter.html`)

```js
// dt is capped at 50ms to prevent physics explosions after tab backgrounding
const dt = Math.min((ts - lastTime) / 1000, 0.05);
```

All movement, timers, and cooldowns are **dt-based** (seconds). No `setInterval`.

### State Machine

`STATE.screen` drives everything — `'MENU' | 'PLAYING' | 'LEVEL_COMPLETE' | 'GAME_OVER'`. The game loop reads this and dispatches to the correct update + draw functions. Transitions are handled by simple functions (`startGame()`, `advanceLevel()`, `goMenu()`).

### Enemy System (`shooter.html`)

Enemy behavior is defined in `ENEMY_CONFIG` and dispatched by type in `updateEnemies()`:
- **basic** — beelines toward player
- **rusher** — pause-aim → straight-line charge cycle
- **tank** — slow walk + periodic AoE slam (3s timer)

Wave composition scales with level via `buildWave(level)` which returns a shuffled queue + spawn interval.

## Git / GitHub

- Remote: `https://github.com/Ravul/claude-workspace`
- Branch: `main`
- Commit and push after every meaningful change. Use descriptive commit messages (what changed and why, not just "update file").
