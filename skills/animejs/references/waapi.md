# WAAPI: `waapi.animate()`

Checked against: `waapi/index.d.ts`, `waapi/waapi.d.ts` (`WAAPIAnimation`, `waapi` namespace, `easingToLinear` as `convertEase`), `waapi/composition.d.ts`, `types/index.d.ts` (`WAAPIAnimationParams`, `WAAPIAnimationOptions`, `WAAPITweenOptions`, `WAAPIKeyframeValue`, `WAAPIEasingParam`, `WAAPIFunctionValue`); behavior from `src/waapi/waapi.js`.

```ts
const { waapi } = anime;
waapi.animate(targets: DOMTargetsParam, params: WAAPIAnimationParams): WAAPIAnimation
waapi.convertEase(fn: EasingFunction, samples?: number = 100): string   // CSS linear() string
```
`waapi` is a namespace object: there is no standalone `animate` export from `animejs/waapi`. Calling plain `animate()` gives the JS engine, not WAAPI.

A thin layer over native `Element.animate()`: runs off the main thread for `transform`/`opacity`, lighter bundle, but DOM elements only, CSS properties only, and fewer features.

## When to choose WAAPI vs `animate()`
| Use `waapi.animate()` | Use `animate()` |
|---|---|
| Simple CSS transitions that must stay smooth while the main thread is busy | JS objects, SVG attributes, Canvas/WebGL/Three.js, CSS vars needing per-frame logic |
| Hardware-accelerated `transform`, `opacity`, `filter`, `clip-path` | `onUpdate`/`onRender`/`onLoop` callbacks, `modifier`, keyframes with mixed timing |
| Small bundle | Timelines with complex composition, Animatable, Draggable integrations |

## Parameters
| Param | Type | Default |
|---|---|---|
| property values | `WAAPIKeyframeValue`: string, number, array (keyframes), function `(target, i, targets)`, or `{ to, from, duration, delay, ease, composition }` | |
| `duration` | number or fn | `1000` |
| `delay` | number or fn | `0` |
| `ease` | `WAAPIEasingParam` | `'out(2)'` (global default) |
| `loop` | number or boolean | `0` |
| `alternate` | boolean | `false` |
| `reversed` | boolean | `false` |
| `autoplay` | boolean or `ScrollObserver` | `true` |
| `playbackRate` | number | `1` |
| `composition` | native `CompositeOperation`: `'replace' \| 'add' \| 'accumulate'` | `'replace'` |
| `persist` | boolean | `false` (keep animation alive after finish, needed for scroll/timeline sync; forced `true` by `tl.sync()` and `onScroll`) |
| `onComplete` | `Callback<WAAPIAnimation>` | noop. **Only callback available.** |

Discrepancy: `WAAPIAnimationOptions` in `types/index.d.ts` declares `Reversed?` and `Alternate?` (capitalized). `src/waapi/waapi.js` reads lowercase `params.reversed` and `params.alternate`. Write lowercase; capitalized keys are silently ignored at runtime.

Value handling:
- Individual transforms (`x`, `y`, `translateX`, `rotate`, `scale`, ...) work: anime registers `--translateX` etc. CSS custom properties so they compose independently. Where `CSS.registerProperty` is unsupported, src skips the registration and individual transforms do not animate (use `transform` strings instead).
- Unitless numbers get `px` for sizes/translate/`perspective`/`fontSize`/`borderRadius`, `deg` for rotate/skew.
- Arrays are keyframes (`y: [0, -30, 0]`); `{ from, to }` for explicit from.
- `fill` is always `'both'`. On finish (unless `persist: true`) the final styles are committed inline and the native animations are cancelled, so the end state stays without calling `commitStyles()`.
- `cancel()` also commits current styles; `revert()` restores the original inline styles (it resets the whole `transform`, which can affect other WAAPI animations on the same element).

Easing: anime names (`'outExpo'`, `'inOut(3)'`), functions, and `spring()` are converted to a CSS `linear()` string; a spring also sets the duration. Native CSS strings pass through: `'ease'`, `'ease-in-out'`, `'linear(...)'`, `'cubic-bezier(...)'`, `'steps(6, start)'`, `'step-end'`.

## WAAPIAnimation methods (return `this`)
`play()`, `reverse()`, `pause()`, `resume()`, `restart()`, `alternate()`, `seek(time, muteCallbacks?)`, `complete()`, `cancel()`, `revert()` (cancel and restore inline styles), `commitStyles()`, `forEach(cb | methodName)` (iterate native `Animation`s), `then(cb?)` (Promise).

Properties: `targets`, `animations` (native `Animation[]`), `duration`, `currentTime`, `progress`, `speed`, `paused`, `completed`, `reversed`, `persist`, `autoplay`.

## Examples
```js
const { waapi, stagger, spring, onScroll, createTimeline } = anime;

waapi.animate('.square', {
  x: '17rem',
  rotate: { to: 360, ease: 'out(6)' },
  ease: spring({ stiffness: 70 }),
});

waapi.animate('.circle', { y: [0, -30, 0], delay: stagger(75), loop: true, ease: 'inOutSine' });

waapi.animate('.card', { opacity: [0, 1], translate: ['0 20px', '0 0'], autoplay: onScroll() });

// Inside a timeline
const fade = waapi.animate('.panel', { opacity: [0, 1], duration: 400 });
createTimeline().add('.title', { y: [20, 0] }).sync(fade, '<<');

// Use anime easings with native Element.animate() (native option names: easing, fill)
const s = spring({ bounce: .3 });
el.animate({ transform: ['scale(.8)', 'scale(1)'] }, { duration: s.settlingDuration, easing: waapi.convertEase(s.ease), fill: 'forwards' });
```
