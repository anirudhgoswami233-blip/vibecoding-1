# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Doodle Garden: a single-file, dependency-free symmetry drawing app (`doodle-garden/index.html`). HTML, CSS and JS all live in that one file. There is no build step, package manager, linter or test suite.

## Running

Open `doodle-garden/index.html` directly in a browser. To serve it locally: `python -m http.server` from the repo root, then visit `/doodle-garden/`.

The repo is published to GitHub as `vibecoding-1`. `GITHUB_SETUP.md` has the Windows git/`gh` workflow (branch, push, `gh pr create`, optional GitHub Pages at `/doodle-garden/`). `Build-and-Publish-Guide.pdf` is a companion guide.

## Architecture

All state is module-level in the inline `<script>`:

- **Strokes are stored, not just painted.** `strokes` is an array of `{pts, n, w, pal, mirror, t0}`. Symmetry count, brush size, palette and mirror are captured per stroke at `pointerdown`, so changing the controls only affects later strokes.
- **Rendering is replayable.** `drawSeg` draws one segment of a stroke, and `redraw()` clears the canvas and replays every stroke. Live drawing calls `drawSeg` incrementally on `pointermove`. Undo (`strokes.pop()` + `redraw`), Clear, and window resize all rely on `redraw()`. Any new feature that alters appearance has to be reflected in the stroke data so replay stays correct.
- **Symmetry** is done in `seg()`. Points are taken relative to the canvas centre, rotated `n` times, and optionally reflected (the `[1,-1]` factor `f`).
- **Colour** comes from `palettes[name](t)`, where `t = st.t0 + i*2`. `t0` is the global `hue` counter, bumped by 40 on each stroke, so colour depends on the point index within the stroke. Adding a palette means adding an entry to `palettes` and an `<option>` in `#pal`.
- **HiDPI:** `resize()` sets the canvas backing size to CSS size × `devicePixelRatio` and applies `setTransform`. Drawing code uses CSS pixels (`clientWidth`/`clientHeight`).
- **Save PNG** composites the canvas onto a `#0b0d1a` background, because the canvas itself is transparent. Keep that colour in sync with `--bg` in the CSS.
