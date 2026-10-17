# Recipes (verified patterns)

Recipes are patterns that were run in a browser against anime.js 4.5.0 with zero console errors and a checked end state. Untested snippets belong in the section references, not here.

Target version: anime.js 4.5.0. Re-verify recipes when the CDN version changes.

## Entry format

Copy this block for each recipe:

````md
### <Title>

**Use case:** one or two lines on when to reach for this.

```js
const { ... } = anime;
// complete, runnable code
```

**Verification:** date, version, and what was checked (console clean, end-state values).
````

## Recipes

### Motion path with an HTML element

**Use case:** an HTML element (div, img, icon) follows an SVG path, centered on it and rotating with it. The svg may be scaled by its viewBox.

```html
<style>
  .stage { position: relative; }               /* shared origin for svg and follower */
  .stage > svg { display: block; }             /* no inline baseline gap above/below */
  .follower {
    position: absolute; top: 0; left: 0;       /* sit at the svg's top-left corner */
    width: 40px; height: 40px;
    margin: -20px 0 0 -20px;                   /* minus half size: center on the path */
  }
</style>
<div class="stage">
  <svg width="600" height="240" viewBox="50 0 300 120">
    <path id="track" d="M10 60 C 60 0, 110 120, 160 60 S 260 0, 290 60" fill="none" stroke="#36c"/>
  </svg>
  <div class="follower"></div>
</div>
```

```js
const { animate, createMotionPath } = anime;

animate('.follower', {
  ...createMotionPath('#track'),
  ease: 'linear',
  duration: 3000,
  loop: true,
});
```

**Verification:** 2026-10-16, anime.js 4.5.0. A test page built this markup twice (identity `viewBox="0 0 300 120"` at 300x120, and the scaled, offset viewBox above at 600x240), with `autoplay: false`. At t = 0, 0.5, 1 the follower's `getBoundingClientRect()` center matched `path.getPointAtLength(L * t).matrixTransform(path.getScreenCTM())` exactly (dx = dy = 0) in both, while rotating. Zero console messages.
