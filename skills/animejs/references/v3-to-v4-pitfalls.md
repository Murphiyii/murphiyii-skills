# V3 to V4 pitfalls (anime.js 4.5.0)

Checked against: `index.d.ts`, `types/index.d.ts`, `animation/animation.d.ts`, `timer/timer.d.ts`, `timeline/timeline.d.ts`, `utils/target.d.ts`, `utils/stagger.d.ts`, `utils/chainable.d.ts`, `utils/random.d.ts`, `svg/motionpath.d.ts`, `svg/drawable.d.ts`, `easings/spring/index.d.ts`, `easings/steps/index.d.ts`, `easings/cubic-bezier/index.d.ts`, `engine/engine.d.ts` (all under the 4.5.0 package's `dist/modules/`, not shipped here: grep `assets/anime.umd.js`), plus `src/` for runtime behavior.

Source: the official wiki "Migrating from v3 to v4", corrected where 4.5.0 differs (marked **4.5.0**). Models default to V3 syntax. If any left-column pattern appears in code you write or review, it is a bug.

## Imports and entry point

| V3 (wrong) | V4 (correct) | Notes |
|---|---|---|
| `<script src=".../anime.min.js">` then `anime({...})` | `<script src="https://cdn.jsdelivr.net/npm/animejs@4.5.0/dist/bundles/anime.umd.min.js">`, then `const { animate, stagger } = anime;` | The V4 global is a plain object, not a function: calling `anime(...)` throws. Adapters are not in the bundle. |
| `import anime from 'animejs'` (or `'animejs/lib/anime.es.js'`) | `const { animate, createTimeline, stagger } = anime;` | This setup loads the UMD bundle, so there is nothing to import. |

## Animation call and params

| V3 (wrong) | V4 (correct) | Notes |
|---|---|---|
| `anime({ targets: '.el', translateX: 250 })` | `animate('.el', { x: 250 })` | Targets are the mandatory first argument. `translateX` still works; `x`/`y`/`z` are shorthands. |
| `easing: 'easeOutQuad'` | `ease: 'outQuad'` | Param renamed, `ease` prefix dropped from all names. |
| `easing: 'easeInOutSine'` | `ease: 'inOutSine'` | Same rule for every Penner ease. |
| `easing: 'easeOutElastic(1, .5)'` | `ease: 'outElastic(1, .5)'` | Parameterized `eases` strings still parse. |
| default easing `'easeOutElastic(1, .5)'` | default `ease: 'out(2)'` | Global default lives in `engine.defaults.ease`. |
| `easing: 'spring(1, 80, 10, 0)'` | `ease: spring({ mass: 1, stiffness: 80, damping: 10, velocity: 0 })` | **4.5.0**: the guide says `createSpring()`, which is deprecated (console.warn). Use `spring()`. A spring overrides `duration` with its settling duration. |
| `easing: 'steps(5)'` | `const { steps } = anime;`, then `ease: steps(5)` | **4.5.0**: string forms `'steps('`, `'linear('`, `'cubicBezier('`, `'irregular('` warn and fall back to no easing. |
| `easing: 'cubicBezier(.5, 0, .5, 1)'` | `ease: cubicBezier(.5, 0, .5, 1)` | A function from the global (`const { cubicBezier } = anime;`). |
| `easing: () => t => 1 - Math.sqrt(1 - t * t)` | `ease: t => 1 - Math.sqrt(1 - t * t)` | Pass the easing function directly, no wrapper. |
| `direction: 'reverse'` | `reversed: true` | |
| `direction: 'alternate'` | `alternate: true` | Combine with `loop`. |
| `loop: 1` (meant: play once) | omit `loop` (default `0`) | V4 `loop` = number of repeats. `loop: 1` plays twice. `loop: true` or `Infinity` = forever. |
| `endDelay: 1000` | `loopDelay: 1000` | Not 1:1. `loopDelay` only waits between loops, never after the last iteration. |
| `round: 100` | `modifier: utils.round(2)` | `utils.round(decimals)`, not a multiplier. `round: 1` becomes `utils.round(0)`. |
| `opacity: { value: .5, duration: 250 }` | `opacity: { to: .5, duration: 250 }` | `value` renamed `to`. Also `from`. (The wiki example changes `.5` to `1` by mistake.) |
| `opacity: [{ value: .5 }, { value: 1 }]` | `opacity: [{ to: .5 }, { to: 1 }]` | Property keyframes use `to`. |
| `translateX: (el, i, l) => l * 10` (third arg = count) | `x: (target, index, targets) => targets.length * 10` | **4.5.0**: third argument is the targets array, not a number. |
| `anim.progress` in `0..100` | `anim.progress` in `0..1` | Getter clamps to `[0, 1]`; also settable. |

## Callbacks and promises

| V3 (wrong) | V4 (correct) | Notes |
|---|---|---|
| `begin: fn` | `onBegin: fn` | Fires after `delay` completes (V3 fired immediately). |
| `update: fn` | `onUpdate: fn` | All callbacks receive the instance as first argument. |
| `complete: fn` | `onComplete: fn` | |
| `loopBegin` / `loopComplete` | `onLoop` | Single callback. |
| `change: fn` | `onRender: fn` | Fires whenever values render. |
| `changeBegin` / `changeComplete` | (removed) | Use `onBegin`/`onComplete` or `onRender`. |
| `anim.finished.then(fn)` | `anim.then(fn)` or `await anim` | `then()` returns a Promise. The wiki's `finishedPromise.then = () => ...` is an assignment, do not copy it. |
| (no equivalent) | `onBeforeUpdate`, `onPause` | V4 only. |

## Playback controls

| V3 behavior | V4 behavior |
|---|---|
| `play()` resumes in current direction | `play()` always plays forward. Use `resume()` to continue in the current direction. |
| `reverse()` toggles direction | `reverse()` always plays backward. Use `alternate()` to flip direction. |
| `anim.tick(t)` / custom rAF | `engine.useDefaultMainLoop = false;` then call `engine.update()` in your loop. |
| `anime.running` | removed |
| `anime.suspendWhenDocumentHidden = true` | `engine.pauseOnDocumentHidden = true` (default `true`) |
| `anime.remove(t)` / `animation.remove(t)` | `utils.remove(targets, renderable?, propertyName?)` or `tl.remove(targets, prop?)` (`anime.remove` on the UMD global is the same V4 function) |

## Timeline

| V3 (wrong) | V4 (correct) | Notes |
|---|---|---|
| `anime.timeline()` | `createTimeline()` | |
| `anime.timeline({ easing: 'easeOutQuad', duration: 250 })` | `createTimeline({ defaults: { ease: 'outQuad', duration: 250 } })` | Child defaults go in `defaults`. Top-level timeline params are the timeline's own playback. |
| `tl.add({ targets: '.a', x: 100 }, '-=200')` | `tl.add('.a', { x: 100 }, '-=200')` | Targets first, params second, position third. |
| `tl.add({ ... }, 500)` | `tl.add('.a', { ... }, 500)` | Absolute ms still a number. V4 only: `'<'`, `'<<'`, `'<<+=100'`, labels, `stagger()` as position. |
| children `loop` ignored | children `loop` respected | Can mix with timeline `loop`. |

## Stagger

| V3 (wrong) | V4 (correct) |
|---|---|
| `anime.stagger(100)` | `const { stagger } = anime;`, then `delay: stagger(100)` (`anime.stagger(100)` itself is valid V4 with the UMD global) |
| `stagger(100, { direction: 'reverse' })` | `stagger(100, { reversed: true })` |
| `stagger(100, { easing: 'easeInOutQuad' })` | `stagger(100, { ease: 'inOutQuad' })` |

## Utilities

| V3 (wrong) | V4 (correct) | Notes |
|---|---|---|
| `anime.set(t, { x: 10 })` | `utils.set(t, { x: 10 })` | Also on the global as `set` (so `anime.set` is valid V4 with the UMD global; check the V4 params). Returns a JSAnimation. |
| `anime.get(t, 'x')` | `utils.get(t, 'x')` | Also `anime.get`. `utils.get(t, 'x', false)` returns a number; `utils.get(t, 'x', 'rem')` converts. |
| `anime.random(50, 100)` | `utils.random(50, 100)` | Also `anime.random`. Optional 3rd arg: decimal places. |
| `anime.path(el)` returning `{ x, y, angle }` | `const { translateX, translateY, rotate } = createMotionPath(el)` | Spread into `animate(target, { ...createMotionPath('path'), ease: 'linear' })`. |
| `strokeDashoffset: [anime.setDashoffset, 0]` | `animate(createDrawable('path'), { draw: ['0 0', '0 1'] })` | `createDrawable(sel, start=0, end=0)` hides the stroke immediately (draw `'0 0'`). |
| `document.querySelectorAll` everywhere | `utils.$('.sel')` | Returns an array (respects `createScope` root). |

## Things with no V3 equivalent (do not invent V3-style APIs)

`createTimer`, `createAnimatable`, `createDraggable`, `onScroll`, `createScope`, `createLayout`, `splitText`, `scrambleText`, `morphTo`, `waapi.animate`, `engine`, adapters (ESM only, not in the UMD bundle). See the matching reference file.

## Review grep

Flag any of: `anime(`, `anime.<name>` where the name is not on the V4 global (`anime.timeline`, `anime.running`, `anime.path`, `anime.setDashoffset`, `anime.penner`, `anime.speed`, `anime.suspendWhenDocumentHidden`), `import ... from 'animejs`, `targets:`, `easing:`, `direction:`, `endDelay`, `value:` inside a property object, `begin:`, `update:`, `complete:`, `.finished`, `ease: 'ease`, `'steps(`, `'cubicBezier(`, `'spring(`, `createSpring(`, `text.split(`.

False positives (valid V4, do not "fix"): CSS easing strings like `ease: 'ease-in-out'` or `'steps(4, end)'` inside `waapi.animate()`; `easing:` and `fill:` in native `Element.animate()` options; `value:` as a property of a JS object target (`animate(counter, { value: 100 })`); the UMD `anime` global's `anime.animate(...)`, `anime.set(...)`, `anime.stagger(...)`, `anime.utils.random(...)` and any other of its 67 exports (listed in SKILL.md).
