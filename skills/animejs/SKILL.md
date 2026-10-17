---
name: animejs
description: Anime.js V4 expert reference. Use whenever code imports or mentions anime.js / animejs (any version), or the user asks to write, debug, review, refactor, or migrate JavaScript animations with anime.js, including V3 to V4 migration, timelines, stagger, scroll-triggered or scroll-synced animation (onScroll), SVG line drawing, morphing and motion paths, draggable elements, text splitting and scramble effects, layout/FLIP transitions, animatables (cursor follow), springs and easings, WAAPI, Three.js animation through anime adapters, and React/Vue cleanup with createScope. Prevents V3 syntax such as anime({ targets }), easing 'easeOutQuad', anime.timeline, and direction 'alternate'.
---

# Anime.js V4

Target version: **anime.js 4.5.0**. Projects load the UMD bundle from the CDN:

```html
<script src="https://cdn.jsdelivr.net/npm/animejs@4.5.0/dist/bundles/anime.umd.min.js"></script>
```

Always write the URL with the `@4.5.0` pin. Without it jsDelivr serves the npm `latest` tag, which moves to a new major version silently (a `5.0.0-beta` already exists). If a page loads another version, treat anything below as possibly stale.

Source of truth, in order: `assets/anime.umd.js` in this skill folder (the unminified build of that same 4.5.0 bundle: JSDoc typedefs for every params object, defaults, and the implementation), then the docs site. The file is about 430 KB: Grep it for the bare name of a typedef, function, or parameter (`AnimationParams`, `createTimeline =`, `loopDelay`), never Read it whole. The `references/` files condense it. Their "Checked against" lines and `src/...` citations name the 4.5.0 `.d.ts` and source files they were verified against; those files are not shipped here, so grep the symbol name in the bundle instead (line numbers will not match).

## When to use
- Writing any animation with anime.js, or choosing between its APIs.
- Debugging anime.js code (nothing moves, wrong easing, callbacks not firing, leaks after unmount).
- Reviewing or migrating code: anything containing `anime(`, `targets:`, `easing:`, `anime.timeline` is V3 and must be converted. Read `references/v3-to-v4-pitfalls.md` first.

## Imports

The bundle defines one global, `anime`, a plain namespace object (not a function). Destructure what you need, in a classic `<script>` that runs after the bundle tag:

```js
const { animate, createTimeline, stagger, utils } = anime;
```

Never write `import ... from 'animejs'` or subpaths like `'animejs/text'`: there is no module to import. The examples in `references/` are already written with `const { ... } = anime;` (except `adapters.md`, which only works with ESM).

The global holds all 67 root exports: `animate`, `createTimer`, `createTimeline`, `createAnimatable`, `createDraggable`, `createScope`, `onScroll`, `scrollContainers`, `engine`, `createLayout`, `stagger`, the utils helpers also at top level (`$`, `get`, `set`, `remove`, `random`, `randomPick`, `shuffle`, `createSeededRandom`, `clamp`, `round`, `snap`, `wrap`, `lerp`, `damp`, `mapRange`, `sync`, `keepTime`, `cleanInlineStyles`, ...), easings (`eases`, `spring`, `createSpring` (deprecated), `cubicBezier`, `steps`, `linear`, `irregular`), SVG (`createDrawable`, `morphTo`, `createMotionPath`), text (`splitText`, `scrambleText`, `split` (deprecated)), `waapi`, the classes (`JSAnimation`, `Timer`, `Timeline`, `Animatable`, `Draggable`, `Scope`, `ScrollObserver`, `AutoLayout`, `TextSplitter`, `Spring`, `WAAPIAnimation`), the namespaces `utils`, `easings`, `svg`, `text`, plus `globals`.

**Not in the bundle: the adapters** (`registerAdapter`, `threeAdapter`, `getInstances`, `commitChanges`). They exist only as ESM subpaths (`animejs/adapters`, `animejs/adapters/three`), so Three.js targets and custom adapters cannot be used with this setup.

## Core rules and gotchas (V3 traps first)

1. **Targets are the first argument**: `animate('.el', { x: 100 })`. Never `anime({ targets: '.el', ... })`, never a `targets` key.
2. **`ease`, not `easing`, with no `ease` prefix**: `ease: 'outQuad'`, `'inOutSine'`, `'out(3)'`, `'outElastic(1, .5)'`. Default is `'out(2)'`.
3. **Easing generators are functions, not strings**: `ease: steps(5)`, `cubicBezier(.7,.1,.5,.9)`, `linear(0, '0.5 50%', 1)`, `irregular(10)`. In 4.5.0 the string forms warn and fall back to no easing.
4. **Springs: `spring({ bounce, duration })` or `spring({ stiffness, damping, mass, velocity })`.** `createSpring` is deprecated (warns). A spring replaces the animation `duration` with its settling duration.
5. **Direction**: `alternate: true` and/or `reversed: true`. There is no `direction` param.
6. **`loop` counts repeats**: `loop: 1` plays twice; `loop: true` is infinite. `endDelay` is gone, `loopDelay` only waits between loops.
7. **Callbacks use `on*`**: `onBegin` (after delay), `onUpdate`, `onRender`, `onLoop`, `onComplete`, `onPause`, `onBeforeUpdate`. Promise: `await animation` or `animation.then(cb)`, not `.finished`.
8. **Timelines**: `createTimeline({ defaults: { ease, duration } })` then `.add(targets, params, position)`. Child defaults go in `defaults`. Positions: `'-=200'`, `'+=100'`, `'<'`, `'<<'`, `'<<+=100'`, labels, or `stagger()`.
9. **Object syntax uses `to`/`from`**: `opacity: { from: 0, to: 1, duration: 300 }`. `[0, 1]` is from/to; 3+ plain values are keyframes.
10. **Function values get `(target, index, targets)`**: the third argument is the targets array (`targets.length`), not a count.
11. **Helpers live in `utils` and at the top level of the global**: `utils.set`, `utils.get`, `utils.remove`, `utils.random`, `utils.$`; `modifier: utils.round(2)` replaces `round: 100`. With the UMD bundle `anime.set(...)`, `anime.random(...)` and `anime.stagger(100)` are also valid V4 (the same functions); only `anime.<name>` where the name is not one of the 67 exports (`anime.timeline`, `anime.running`, `anime.path`, `anime.setDashoffset`, `anime.penner`, `anime.speed`) is V3. `stagger(100, { from: 'center', reversed: true, ease: 'out(2)' })`.
12. **Playback semantics changed**: `play()` always goes forward, `reverse()` always backward, `resume()` continues the current direction, `alternate()` flips it. `progress` is 0..1.
13. **Transforms are individual properties**: `x`, `y`, `z`, `rotate`, `scale`, `skewX`... and compose across separate animations. Overlapping animations on the same property replace each other (`composition: 'replace'`); use `'blend'` for additive or `'none'` to overlap.
14. **Always clean up in components**: build inside `createScope({ root }).add(() => { ... })` and call `scope.revert()` on unmount. `revert()` (not `pause()`) restores styles and removes listeners on every instance type.
15. **Scroll**: `autoplay: onScroll({ ... })`. Threshold strings are `'<container> <target>'` (default enter `'end start'`, leave `'start end'`). `sync: true` scrubs; use `ease: 'linear'` on scrubbed animations.
16. **SVG drawing**: `animate(createDrawable('path'), { draw: ['0 0', '0 1'] })`. `createDrawable` hides the stroke immediately (default `'0 0'`). Morph with `d: morphTo('#other')` (`points` for polygons). Motion path: spread `...createMotionPath('#path')`.
17. **Text**: `splitText(el, { chars: true })` (words split by default; only the first matching element is split, so loop for several). For `lines: true`, animate inside `splitter.addEffect(self => animate(self.lines, ...))` because lines re-split after font load and resize. `text.split()` is deprecated.
18. **Timers default to infinite duration** (`createTimer({ onUpdate })` runs forever); animations default to 1000ms.
19. **WAAPI**: `waapi.animate(targets, params)` for hardware-accelerated CSS-only animations; only `onComplete` exists. Use lowercase `alternate`/`reversed` (the typedefs capitalize them by mistake).
20. **Do not guess**: if a parameter or method is not in the matching reference file, grep `assets/anime.umd.js` for it before using it.

## Task to reference map

Read only the file(s) the task needs.

| Task | Read |
|---|---|
| Migrating V3 code, reviewing for V3 leaks | `references/v3-to-v4-pitfalls.md` |
| Basic tweens, values, units, keyframes, colors, CSS vars, JS objects, controls, promises | `references/animation.md` |
| Sequencing, overlaps, labels, positions, syncing animations | `references/timeline.md` |
| Callback-only clocks, game loops, counters, throttled updates | `references/timer.md` |
| Cursor follow, pointer parallax, high-frequency value updates | `references/animatable.md` |
| Drag, sliders, carousels, snapping, throw/release physics | `references/draggable.md` |
| Layout/FLIP transitions, reorder, grid/list switch, enter/leave, shared element | `references/layout.md` |
| React/Vue/Svelte/Angular cleanup, scoped selectors, media queries, reduced motion | `references/scope.md` |
| Scroll-triggered, scroll-synced (scrub), parallax, sticky sections | `references/events.md` |
| SVG line drawing, path morphing, motion along a path | `references/svg.md` |
| Text splitting (chars/words/lines), reveal effects, scramble/decode text | `references/text.md` |
| Stagger (grids, from center, ranges, jitter), `utils.*` helpers, modifiers | `references/utilities.md` |
| Easing names, custom curves, springs, steps | `references/easings.md` |
| WAAPI, off-main-thread CSS animation, `convertEase` | `references/waapi.md` |
| Global defaults, seconds time unit, global speed, external render loop | `references/engine.md` |
| Three.js meshes, cameras, lights, materials, instanced meshes, custom target adapters (not available with the UMD bundle: say so first) | `references/adapters.md` |
| Copy-ready verified patterns | `references/recipes.md` |

## Canonical example

```js
const { animate, createTimeline, createScope, onScroll, stagger, spring } = anime;

const scope = createScope({ root: '#app', defaults: { duration: 600 } }).add(() => {
  animate('.card', {
    y: [40, 0],
    opacity: [0, 1],
    delay: stagger(80, { from: 'center' }),
    ease: spring({ bounce: .3 }),
    autoplay: onScroll({ enter: 'bottom-=80 top' }), // one observer: all cards start when the FIRST .card enters (see events.md)
  });

  createTimeline({ loop: true, alternate: true, defaults: { ease: 'inOutQuad' } })
    .add('.dot', { x: '12rem' })
    .add('.dot', { scale: 1.5 }, '<<')
    .call(() => console.log('cycle'));
});

// later, e.g. on unmount
scope.revert();
```

## Working method
1. Identify the module(s) from the map above and read those reference files.
2. Write V4 code that destructures from the `anime` global. Re-check every parameter name against the reference (or grep `assets/anime.umd.js`).
3. Scan your output with the review grep at the bottom of `references/v3-to-v4-pitfalls.md`.
