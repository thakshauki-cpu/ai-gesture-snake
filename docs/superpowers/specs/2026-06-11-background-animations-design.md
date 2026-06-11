# Background Animations Design

**Date:** 2026-06-11
**Status:** Approved

## Goal

Add dramatic, alien-space SVG background animations to the full viewport using GSAP. The effect should feel immersive — rich nebula color washes, a deep layered starfield, and occasional shooting stars — while never interfering with gameplay or page interaction.

## Approach

Option A: Full-viewport SVG layer animated entirely with GSAP. No Canvas API. SVG elements are declarative and easy to maintain; GSAP provides precise control over timing, sequencing, and chaining.

## HTML Structure

A single `<svg id="bg-canvas">` is inserted as the **first child of `<body>`**, before `.console`.

CSS on the SVG:
```css
position: fixed;
inset: 0;
width: 100%;
height: 100%;
z-index: 0;
pointer-events: none;
```

`.console` gets `position: relative; z-index: 1` so all page content floats above the background.

## SVG Elements

### Stars (`#stars`)
- ~120 `<circle>` elements, randomised x/y positions across the viewport
- Radii: 0.5–2.5px
- Split into three sub-groups: `slow`, `mid`, `fast` for parallax depth
- Colors: mostly white/pale-cyan, with a few amber and purple outliers for alien flavor

### Nebulas (`#nebulas`)
- 5–6 `<ellipse>` elements filled via `<radialGradient>`
- Colors: deep cyan, violet, magenta
- Opacity: 0.06–0.15 — atmospheric washes, not solid shapes
- Scattered across the viewport at varied angles

### Shooting Stars (`#shooters`)
- Pool of 4 reusable `<line>` elements
- Cyan or white stroke, `stroke-dasharray` to render as a streak
- Positioned off-screen; animated on demand

## GSAP Animations

### Star Drift (parallax)
Three GSAP timelines, one per depth layer. Each star drifts downward and slightly sideways, then reverses (`yoyo: true`).

| Layer | Y drift | Duration  |
|-------|---------|-----------|
| slow  | +5px    | 20–30s    |
| mid   | +10px   | 10–15s    |
| fast  | +20px   | 5–8s      |

Stars within each layer are staggered so movement is never synchronised.

### Star Twinkle
Each star has an independent `gsap.to()` on `opacity` (0.3 → 1.0), duration 2–5s randomised, `repeat: -1`, `yoyo: true`.

### Nebula Pulse
Each ellipse oscillates:
- `opacity`: 0.06 ↔ 0.18
- `scale`: 1 ↔ 1.15
- `rotate`: ±15°
- Duration: 8–14s, `ease: "sine.inOut"`, `repeat: -1`, `yoyo: true`

### Shooting Stars
A recurring GSAP timeline fires every 3–6s (randomised interval). Each firing:
1. Picks one of the 4 `<line>` elements from the pool
2. Positions it off-screen at the top-right
3. Animates diagonally across the viewport in 0.6s, `ease: "power2.in"`
4. Fades opacity 1 → 0 at the end
5. Resets position for reuse

## Files Changed

| File         | Change                                                      |
|--------------|-------------------------------------------------------------|
| `index.html` | Add `<svg id="bg-canvas">` as first child of `<body>`; add GSAP CDN script tag |
| `style.css`  | Add fixed positioning for `#bg-canvas`; add `z-index: 1` to `.console` |
| `bg.js`      | New file — all star/nebula/shooter generation and GSAP animation logic |

## Constraints

- `pointer-events: none` on the SVG at all times — must never block game or UI interaction
- Respects `prefers-reduced-motion`: all GSAP animations paused when the media query matches
- GSAP loaded from CDN (same pattern as MediaPipe); no build step required
- No changes to `script.js` or `gesture-control.js`
