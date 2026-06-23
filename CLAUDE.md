# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Conway's Game of Life as a single, self-contained `index.html`. No build step, no dependencies, no tests — all HTML, CSS, and JavaScript live in that one file. To run it, open `index.html` in a browser (e.g. `open index.html`) or serve the directory statically (`python3 -m http.server`).

## Architecture

The simulation lives in a single IIFE in the `<script>` block of `index.html`. Key design points:

- **Grid state** is held in two flat `Uint8Array` buffers (`cur` / `nxt`) indexed as `y * cols + x` via `idx()`. `step()` writes the next generation into `nxt`, then swaps the two buffers — no per-tick allocation.
- **Toroidal topology**: edges wrap around (neighbor lookups in `step()` wrap x and y modulo the grid bounds), so the world has no borders.
- **Grid dimensions are derived from the viewport.** `resize()` computes `cols`/`rows` from `window.innerWidth/innerHeight` divided by `CELL`, reallocates the buffers, and resets the generation. Resizing the window therefore clears the board.
- **Render loop** is a single `requestAnimationFrame` loop (`loop()`) that advances and redraws only when `running` and at most every `TICK_MS`. Rendering is full-canvas redraw to a `{ alpha: false }` 2D context, scaled by `devicePixelRatio`.
- **Input**: pointer events paint cells (drag to draw; the first cell toggled sets whether the stroke draws or erases). Keyboard shortcuts: `space` play/pause, `n` step (only while paused), `r` randomize, `c` clear.

## Tunable constants

At the top of the IIFE: `CELL` (cell size in px), `TICK_MS` (sim speed), `RAND_DENSITY` (fraction alive on randomize), `HEADER_H` (must match the CSS `header` height and the canvas `top` offset). Colors are CSS custom properties in `:root` (`--fg` is the live-cell color, also hardcoded as `#ff2a2a` in `draw()`).
