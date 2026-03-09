---
name: variable-font-cursor-effect
description: Use when building interactive text effects where font weight changes based on cursor proximity, variable font hover animations, or gradient-smooth-boldness text interactions
---

# Variable Font Cursor Effect

## Overview

A technique for creating smooth, gradient-like font weight changes based on cursor proximity. Characters near the cursor become bold while distant ones stay thin, using variable fonts and `requestAnimationFrame` for fluid animation.

## When to Use

- User wants interactive typography that responds to mouse movement
- Building hero text with hover-based weight animation
- Creating "magnetic" or "spotlight" text effects
- Any gradient-smooth-boldness font interaction

## Core Pattern

### 1. Split Text into Spans (Word-Preserving)

Wrap each character in `<span class="char">`, but group word characters inside a `<span class="word">` with `white-space: nowrap` to prevent mid-word line breaks.

```html
<h1 id="headline">YOUR TEXT HERE</h1>
```

```css
.word {
  display: inline-block;
  white-space: nowrap;
}

.char {
  display: inline-block;
  font-variation-settings: "wght" 120;
}

.space {
  display: inline-block;
  width: 0.24em;
}
```

```js
const words = text.split(" ");
words.forEach((word, i) => {
  const wordSpan = document.createElement("span");
  wordSpan.className = "word";

  for (const letter of word) {
    const span = document.createElement("span");
    span.className = "char";
    span.textContent = letter;
    wordSpan.appendChild(span);
    chars.push(span);
  }

  container.appendChild(wordSpan);

  if (i < words.length - 1) {
    const space = document.createElement("span");
    space.className = "space";
    space.innerHTML = "&nbsp;";
    container.appendChild(space);
  }
});
```

### 2. Elliptical Radius with Smoothstep

Use separate X/Y radii for a natural elliptical influence zone, and smoothstep for organic falloff:

```js
const RADIUS_X = 260;
const RADIUS_Y = 150;

function smoothstep(t) {
  return t * t * (3 - 2 * t);
}

// Normalized elliptical distance
const dx = (mouseX - charCenterX) / RADIUS_X;
const dy = (mouseY - charCenterY) / RADIUS_Y;
const distance = Math.sqrt(dx * dx + dy * dy);

let influence = Math.max(0, Math.min(1, 1 - distance));
influence = smoothstep(influence);

const weight = MIN_WEIGHT + (MAX_WEIGHT - MIN_WEIGHT) * influence;
el.style.fontVariationSettings = `"wght" ${weight}`;
```

### 3. Animation Loop with rAF

- Measure character positions once on `mouseenter` and `resize`
- Use `requestAnimationFrame` loop while cursor is active
- Reset all weights on `mouseleave`

```js
let active = false;
let rafId = null;

element.addEventListener("mouseenter", (e) => {
  active = true;
  measureChars(); // cache getBoundingClientRect() centers
  if (!rafId) rafId = requestAnimationFrame(render);
});

element.addEventListener("mouseleave", () => {
  active = false;
  resetWeights(); // set all back to MIN_WEIGHT
});

window.addEventListener("resize", measureChars);
```

## Quick Reference

| Parameter | Typical Value | Effect |
|-----------|--------------|--------|
| `MIN_WEIGHT` | 100-200 | Resting thinness |
| `MAX_WEIGHT` | 600-900 | Peak boldness near cursor |
| `RADIUS_X` | 200-400 | Horizontal influence range (px) |
| `RADIUS_Y` | 100-250 | Vertical influence range (px) |
| Smoothstep | `t*t*(3-2*t)` | Organic ease-in-out falloff |

## Requirements

- **Variable font** with `wght` axis (e.g., Hanken Grotesk, Inter, Roboto Flex)
- Load with full weight range: `wght@100..900`
- Characters must be `display: inline-block` for `font-variation-settings` to apply per-span

## Common Mistakes

- **Words breaking mid-line**: Wrap each word's chars in a `display: inline-block; white-space: nowrap` container
- **Jerky animation**: Always use `requestAnimationFrame`, never `setInterval`
- **Stale positions**: Re-measure char positions on resize and mouseenter (scroll can shift layout)
- **Linear falloff looks harsh**: Use smoothstep instead of linear interpolation
