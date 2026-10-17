# SVG: `createDrawable()`, `morphTo()`, `createMotionPath()`

Checked against: `svg/index.d.ts`, `svg/drawable.d.ts`, `svg/morphto.d.ts`, `svg/motionpath.d.ts`, `svg/helpers.d.ts`, `types/index.d.ts` (`DrawableSVGGeometry`, `FunctionValue`, `TargetsParam`); defaults from `src/svg/*.js`; usage from `examples/svg-line-drawing`, `tests/suites/svg.test.js`.

```ts
const { createDrawable, morphTo, createMotionPath } = anime;  // or the `svg` namespace: anime.svg.createDrawable(...)
createDrawable(selector: TargetsParam, start?: number = 0, end?: number = 0): DrawableSVGGeometry[]
morphTo(path2: TargetsParam, precision?: number = .33): FunctionValue
createMotionPath(path: TargetsParam, offset?: number = 0): { translateX: FunctionValue, translateY: FunctionValue, rotate: FunctionValue }
```

## Line drawing: `createDrawable`
Returns proxies of SVG geometry elements (`path`, `line`, `circle`, `rect`, `polygon`, `polyline`, `ellipse`) that expose an animatable `draw` property: `'<start> <end>'`, each 0..1 along the path length. Internally sets `pathLength`, `stroke-dasharray` and `stroke-dashoffset`.
- **Creating the drawable immediately applies `draw: '<start> <end>'` (default `'0 0'`, i.e. invisible).** Pass `createDrawable(el, 0, 1)` to start fully drawn. This initial set only happens when the element's `pathLength` is not already `1000`, so creating a second drawable on the same element does not reset it.
- Elements need a `stroke` and usually `fill="none"`.
- `draw` values: `'0 1'` full, `'0 .5'` first half, `'.5 .5'` nothing at midpoint, `'1 1'` erased at the end. Values outside 0..1 are allowed for offsets.
- Works with `vector-effect: non-scaling-stroke` (scale factor handled).

```js
const { animate, createDrawable, stagger } = anime;

// Draw in, then erase forward, looping
animate(createDrawable('.logo path'), {
  draw: ['0 0', '0 1', '1 1'],
  ease: 'inOutQuad',
  duration: 2000,
  delay: stagger(100),
  loop: true,
});

// Start fully drawn and erase
animate(createDrawable('#line', 0, 1), { draw: '1 1' });
```

## Morphing: `morphTo`
Returns a function-based value. Animate the shape attribute of the source element: `d` for `<path>`, `points` for `<polygon>`/`<polyline>`. Source must be `path`, `polygon` or `polyline` (other tags throw). `path2` is the shape to morph to (selector or element). `precision` (0..1, default `.33`) controls resampling; `0` means no extra points (use only when both shapes already have matching point counts).

```js
const { animate, createTimeline, morphTo } = anime;

animate('#shape-a', { d: morphTo('#shape-b'), duration: 800, ease: 'inOutQuad', alternate: true, loop: true });

// Chain morphs: each reads the previous end state
createTimeline({ loop: true })
  .add('#poly', { points: morphTo('#poly-star') })
  .add('#poly', { points: morphTo('#poly-square') });

// Keyframes
animate('#blob', { d: [{ to: morphTo('#blob-2') }, { to: morphTo('#blob-3') }] });
```
`revert()` on the animation restores the original `d`/`points`.

## Motion path: `createMotionPath`
Returns `translateX`, `translateY`, `rotate` function-based values that follow the path. Spread into `animate()`. `offset` (0..1) shifts the start along the path. The translate values are added to the element's own layout position, so where the element sits before animating matters:
- **SVG targets** get raw path coordinates, so they must share the path's coordinate space (same SVG, same `<g>` transforms).
- **HTML targets** get path points mapped through `path.getCTM()`, i.e. into the `<svg>` box's pixel space (viewBox scale and origin are handled). The translate places the element's **top-left corner** on the path. To make the element's center follow the path: wrap the `<svg>` and the element in a `position: relative` parent, give the svg `display: block`, and put the element at `position: absolute; top: 0; left: 0` with a negative half-size margin. Use margin, not a CSS `transform`, for the offset: anime overwrites `transform`. See the recipe "Motion path with an HTML element" in `recipes.md`.
- The CTM is read once, when the animation is built. If the SVG is resized afterwards, recreate the animation.
- `rotate` uses raw path coordinates, not the CTM, so angles are only correct under a uniform scale (not with `preserveAspectRatio="none"` or skewed/non-uniformly scaled ancestors).

```js
const { animate, createMotionPath } = anime;

animate('.car', {
  ...createMotionPath('#track'),
  ease: 'linear',
  duration: 5000,
  loop: true,
});

// Without rotation
const { translateX, translateY } = createMotionPath('#track', .25);
animate('.dot', { translateX, translateY, ease: 'linear', duration: 3000, loop: true });
```

## Other SVG animation
Plain attributes animate directly: `animate('circle', { r: [0, 40], cx: 100, fill: '#f0a', strokeWidth: 4 })`. `getPath(path)` (internal helper, not on the `anime` global or `anime.svg`) resolves a selector to an `SVGGeometryElement`.
