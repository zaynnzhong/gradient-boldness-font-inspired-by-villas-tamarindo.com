---
name: hand-writing-like-stroke
description: Use when building animated SVG underlines that look hand-drawn, organic brush-stroke lines beneath text, or wiggling/squirming line animations on hover
---

# Hand-Writing-Like Stroke

## Overview

An SVG underline that looks like a hand-drawn brush stroke — always visible, with a gentle organic wobble. On hover, the line wiggles in place (扭来扭去) using animated phase-shifting noise.

## When to Use

- Adding an organic underline beneath buttons, links, or headings
- Creating hand-drawn/brush-stroke line effects
- Building hover interactions where a line wiggles or undulates
- Any decorative stroke that should feel organic, not mechanical

## Core Pattern

### 1. HTML Structure

Text wrapped in a container with an inline SVG underneath:

```html
<button class="button-underline" id="btn">
  <span class="text-wrap" id="textWrap">YOUR TEXT</span>
  <svg class="underline-svg" viewBox="0 0 200 6" fill="none"
       xmlns="http://www.w3.org/2000/svg" preserveAspectRatio="none">
  </svg>
</button>
```

### 2. CSS

```css
.button-underline svg {
  display: block;
  width: 100%;
  height: auto;
  margin-top: 2px;
}

.button-underline svg path {
  stroke: currentColor;
  stroke-width: 1.3;
  stroke-linecap: round;
  vector-effect: non-scaling-stroke;
  fill: none;
}
```

Key: `vector-effect: non-scaling-stroke` keeps stroke width consistent regardless of SVG scaling.

### 3. Generating the Organic Path (Catmull-Rom Splines)

Use low-frequency sine noise for smooth brush-stroke curves, then convert through Catmull-Rom → Cubic Bezier:

```js
const SVG_W = 200;
const SVG_H = 6;
const POINTS = 30;
const STEP = SVG_W / POINTS;
const MID_Y = SVG_H / 2;
const WOBBLE = 1.0;

function noise(i, phase) {
  // Low-frequency only — smooth brush feel, not jagged
  return Math.sin(i * 0.5 + 1.7 + phase) * 0.6
       + Math.sin(i * 1.3 + 3.2 + phase * 0.8) * 0.4;
}

function buildPathD(phase) {
  const pts = [];
  for (let i = 0; i <= POINTS; i++) {
    pts.push({ x: i * STEP, y: MID_Y + noise(i, phase) * WOBBLE });
  }

  let d = `M${pts[0].x.toFixed(2)} ${pts[0].y.toFixed(2)}`;
  for (let i = 0; i < pts.length - 1; i++) {
    const p0 = pts[Math.max(i - 1, 0)];
    const p1 = pts[i];
    const p2 = pts[i + 1];
    const p3 = pts[Math.min(i + 2, pts.length - 1)];
    const cp1x = p1.x + (p2.x - p0.x) / 6;
    const cp1y = p1.y + (p2.y - p0.y) / 6;
    const cp2x = p2.x - (p3.x - p1.x) / 6;
    const cp2y = p2.y - (p3.y - p1.y) / 6;
    d += ` C${cp1x.toFixed(2)} ${cp1y.toFixed(2)} ${cp2x.toFixed(2)} ${cp2y.toFixed(2)} ${p2.x.toFixed(2)} ${p2.y.toFixed(2)}`;
  }
  return d;
}
```

### 4. Wiggle Animation on Hover

Shift the noise phase each frame to make the line squirm in place:

```js
let svgPath = null;
let wiggleRafId = null;
let wigglePhase = 0;

function startWiggle() {
  if (wiggleRafId) return;
  function tick() {
    wigglePhase += 0.08;
    if (svgPath) svgPath.setAttribute("d", buildPathD(wigglePhase));
    wiggleRafId = requestAnimationFrame(tick);
  }
  wiggleRafId = requestAnimationFrame(tick);
}

function stopWiggle() {
  if (wiggleRafId) {
    cancelAnimationFrame(wiggleRafId);
    wiggleRafId = null;
  }
  // Settle back to static shape
  if (svgPath) svgPath.setAttribute("d", buildPathD(0));
}
```

### 5. Match SVG Width to Text

Prevent the line extending past the text by setting SVG width from measured text:

```js
function buildUnderlineSvg() {
  const textW = textWrap.getBoundingClientRect().width;
  svgEl.style.width = textW + "px";
  svgEl.setAttribute("viewBox", `0 0 ${SVG_W} ${SVG_H}`);
  svgEl.innerHTML = `<path d="${buildPathD(0)}"></path>`;
  svgPath = svgEl.querySelector("path");
}
requestAnimationFrame(buildUnderlineSvg);
```

## Quick Reference

| Parameter | Value | Effect |
|-----------|-------|--------|
| `WOBBLE` | 0.5–1.0 | Amplitude of line waviness |
| `POINTS` | 20–40 | Path resolution (more = smoother) |
| `noise` frequencies | 0.3–1.5 | Lower = gentler curves, higher = more bumps |
| `wigglePhase += N` | 0.05–0.12 | Wiggle speed (higher = faster) |
| `stroke-width` | 1.0–2.0 | Line thickness |

## Tuning the Feel

- **More rugged (崎岖)**: Increase `WOBBLE` to 2.0+, add higher-frequency noise terms (`i * 7.8`, `i * 14.1`)
- **Smoother brush stroke (笔触)**: Keep only low frequencies (`i * 0.5`, `i * 1.3`), `WOBBLE` around 1.0
- **Subtle wiggle**: `phase += 0.04`, low `WOBBLE`
- **Energetic wiggle**: `phase += 0.12`, higher `WOBBLE`

## Common Mistakes

- **Line extends past text**: Set `svgEl.style.width` to measured text width, don't rely on `width: 100%` alone
- **Jagged/choppy line**: Too many high-frequency noise terms — use only 2–3 low-frequency sines for brush-stroke feel
- **Wiggle looks mechanical**: Vary the phase multipliers across noise terms (e.g., `phase * 0.8`, `phase * 1.2`) so waves shift at different rates
- **Line disappears on hover**: Keep line always visible; animate the path `d` attribute, not opacity/visibility
