# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page, dependency-free "Wheel of Fortune"–style game that teaches AI interaction patterns with a Seinfeld theme. Deployed as a static site to GitHub Pages at `https://berdilaovidiu.github.io/wheel-of-patterns/`.

There is **no build step, no package manager, no test suite, and no framework**. The entire app is `index.html` (markup, CSS, and JS inline) plus the `resources/` asset folder.

## Running / developing

Open `index.html` directly in a browser, or serve the folder to avoid file:// restrictions on audio/PDF loading:

```
python3 -m http.server 8000   # then open http://localhost:8000
```

Deployment is automatic: pushing to the default branch publishes via GitHub Pages. There is no CI, lint, or test command.

## Architecture

Everything is in [index.html](index.html):

- **Segment data** is embedded as JSON in `<script id="segmentData" type="application/json">`. Each entry has `name`, `image`, `audio`, and `pdf`, with paths pointing into `resources/<Category - Pattern Name>/`. This is the single source of truth for what appears on the wheel — the number of entries determines the number of wheel segments.
- **The wheel is drawn on a `<canvas>`** (`drawWheel()`), not with DOM elements. Segment slices, labels, the outer ring, and the center hub are all painted with the 2D context. Colors come from a hardcoded `getColor(index)` palette array.
- **Rotation** is tracked in a single `currentRotation` (degrees) variable and applied via CSS `transform: rotate()` on the canvas element during `spin()`'s `requestAnimationFrame` loop (cubic ease-out, ~4s). The canvas is redrawn once; spinning only rotates the element, so all angle math must account for `currentRotation`.
- **Two ways to select a pattern**, both resolving to `showPopup(segment)`:
  1. `spin()` — picks a random target rotation, animates, then computes the winning segment from the final angle under the top pointer.
  2. Direct canvas click — `canvas` click handler reverse-maps click coordinates through `currentRotation` back to a segment index.
- **Angle convention:** segment 0 starts at `-Math.PI/2` (top), and the pointer sits at the top of the wheel. Any code that maps between screen angle, wheel rotation, and segment index must keep this `-90°` offset and the current rotation consistent — this is the trickiest part of the file and the most likely place for off-by-one segment bugs.
- **Tick sound + pointer wobble** are synthesized with the Web Audio API (`playTickSound()`) each time the pointer crosses a segment boundary during a spin; no audio files are used for the ticks. The AudioContext is created/resumed on the first user gesture (browser autoplay policy).
- **Popup** overlays the winning/clicked pattern's image, an audio player (the pattern's `.mp3`), and a "View Document" button linking to its PDF.

## Adding or changing wheel patterns

1. Add the assets under `resources/<Category - Pattern Name>/` (image `.png`, audio `.mp3`, document `.pdf`).
2. Add a corresponding object to the `segmentData` JSON block in `index.html` with matching `name`/`image`/`audio`/`pdf` paths.

Note: several existing folder/file names contain typos (e.g. `Ptompt Templates`, `The Promp Genie`) and inconsistent category prefixes — the JSON paths must match the actual on-disk names exactly, so copy paths from the filesystem rather than assuming a clean convention. If you change how many segments exist, the wheel re-slices automatically, but verify the `getColor` palette still gives adjacent segments distinct colors.

## Analytics

The page includes Google Tag Manager (`GTM-MWKZC63X`) and Hotjar (`hjid: 5052747`) snippets in `<head>`. Keep these intact unless intentionally changing tracking.
