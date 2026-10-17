# Animation: `animate()`

## Contents
- Targets (`TargetsParam`)
- Animatable properties
- Values
- Tween params (`TweenParamsOptions`), animation-level or per property
- Playback params (`TimerOptions` + `AnimationOptions`)
- Keyframes
- Callbacks (each receives the animation)
- Methods (all return `this` unless noted)
- Properties
- Examples

Checked against: `animation/animation.d.ts`, `timer/timer.d.ts`, `core/clock.d.ts`, `types/index.d.ts` (`AnimationParams`, `TweenKeyValue`, `TweenParamsOptions`, `TimerOptions`, `DurationKeyframes`, `PercentageKeyframes`, `FunctionValue`, `TargetsParam`), `core/consts.d.ts`; defaults from `src/core/globals.js`, behavior from `src/animation/animation.js`.

```ts
const { animate } = anime;
animate(targets: TargetsParam, parameters: AnimationParams): JSAnimation
```
`JSAnimation extends Timer extends Clock`. Starts automatically (`autoplay: true`).

## Targets (`TargetsParam`)
CSS selector string, `HTMLElement`/`SVGElement`, `NodeList`, plain JS object, or an array mixing these. Selectors resolve against the active `createScope` root. A JS object target animates its numeric properties (`animate(obj, { value: 100 })`).

## Animatable properties
- CSS properties in camelCase or kebab (`opacity`, `backgroundColor`, `'border-radius'`).
- Individual transforms: `translateX translateY translateZ rotate rotateX rotateY rotateZ scale scaleX scaleY scaleZ skew skewX skewY perspective`; shorthands `x`, `y`, `z` map to `translateX/Y/Z`. Composed independently, so `x` and `rotate` can run in separate animations.
- CSS variables: `'--my-var': 200`.
- SVG/DOM attributes (`r`, `cx`, `d`, `points`, `stroke-dashoffset`...) and `innerHTML`/text-ish props.
- Any numeric property of a JS object.

## Values
| Form | Example | Meaning |
|---|---|---|
| number | `x: 100` | Uses the property's default unit (px for translate, deg for rotate). |
| unit string | `x: '10rem'`, `width: '50%'` | Unit converted from current value. |
| relative | `x: '+=100'`, `'-=2rem'`, `'*=2'` | Relative to current value. |
| color | `'#f0a'`, `'rgb(...)'`, `'rgba(...)'`, `'hsl(...)'`, `'hsla(...)'`, `'var(--c)'` | Interpolated as RGBA. |
| `[from, to]` | `opacity: [0, 1]` | Exactly 2 non-object items = from/to. |
| `[a, b, c, ...]` | `y: [0, -40, 0]` | 3+ non-object items = keyframes (first pair is from/to). |
| object | `x: { to: 100, from: 0, duration: 500, delay: 100, ease: 'inOut(3)', modifier, composition }` | Per-property tween params. |
| keyframe array | `x: [{ to: 100, duration: 300 }, { to: 0, ease: 'in' }]` | Property keyframes. |
| function | `x: (target, index, targets) => index * 50` | Evaluated per target. Third arg is the **targets array**. May return any value form, including `{ to, ... }`. |

`fromTo` appears in `TweenValues` in the .d.ts but `src/animation/animation.js` never reads it: use `from` + `to`, or `[from, to]`.

## Tween params (`TweenParamsOptions`), animation-level or per property
| Param | Type | Default |
|---|---|---|
| `duration` | number or FunctionValue | `1000` (ms) |
| `delay` | number or FunctionValue | `0` (applied to first keyframe only) |
| `ease` | `EasingParam` or FunctionValue | `'out(2)'` |
| `composition` | `'replace' \| 'none' \| 'blend'` | `'replace'`; auto `'none'` when 1000+ targets and unset |
| `modifier` | `(v: number) => number \| string` | identity. Not function-based. E.g. `utils.round(0)` |

Composition: `replace` cuts overlapping tweens on the same property; `none` lets them overlap (fastest); `blend` adds values together (additive).

## Playback params (`TimerOptions` + `AnimationOptions`)
| Param | Type | Default |
|---|---|---|
| `id` | string or number | auto |
| `loop` | number or boolean | `0` (repeats; `true`/`Infinity` = forever) |
| `loopDelay` | number | `0` (between loops only) |
| `alternate` | boolean | `false` |
| `reversed` | boolean | `false` |
| `autoplay` | boolean or `ScrollObserver` | `true`; pass `onScroll({...})` to scroll-link |
| `frameRate` | number | `240` (max) |
| `playbackRate` | number | `1` |
| `playbackEase` | EasingParam | none; eases the whole playback across keyframes |
| `keyframes` | `DurationKeyframes \| PercentageKeyframes` | none |

## Keyframes
Default keyframe duration = total `duration` / number of keyframes.
```js
// Duration-based: array of param objects
animate('.box', { keyframes: [ { y: -40, ease: 'out', duration: 400 }, { x: 80, scale: .5, duration: 800 }, { y: 0 } ], loop: true });
// Percentage-based: keys are percentages of duration
animate('.box', { keyframes: { '0%': { x: 0 }, '50%': { x: 100, ease: 'out' }, '100%': { x: 0 } }, duration: 2000 });
```

## Callbacks (each receives the animation)
`onBegin` (after delay), `onBeforeUpdate`, `onUpdate`, `onRender` (values rendered), `onLoop`, `onPause`, `onComplete`. Default `noop`.

## Methods (all return `this` unless noted)
| Method | Notes |
|---|---|
| `play()` | Always forward. |
| `reverse()` | Always backward. |
| `pause()` / `resume()` | `resume` keeps current direction. |
| `restart()` | From start. |
| `alternate()` | Flip current direction. |
| `seek(time, muteCallbacks?, internalRender?)` | Time in ms. |
| `stretch(newDuration)` | Rescales tweens. |
| `refresh()` | Re-evaluates function-based values (e.g. after resize). |
| `cancel()` | Stop and remove from engine; values stay. |
| `revert()` | Cancel and restore original values (use for cleanup). |
| `complete(muteCallbacks?)` | Jump to end, fire `onComplete`. |
| `reset(softReset?)`, `init(internalRender?)`, `resetTime()` | Low level. |
| `then(cb?)` | Returns `Promise`; `await anim` works. |

## Properties
`id`, `targets` (array), `duration` (total incl. loops), `iterationDuration`, `iterationCount`, `currentTime`, `iterationCurrentTime`, `progress` (0..1), `iterationProgress` (0..1), `currentIteration`, `reversed`, `paused`, `began`, `completed`, `cancelled`, `backwards`, `speed` (playback rate), `fps` (frame rate), `deltaTime`, `parent` (timeline or null). Settable: `currentTime`, `iterationCurrentTime`, `progress`, `iterationProgress`, `currentIteration`, `reversed`, `cancelled`, `speed`, `fps`, and callback fields.

## Examples
```js
const { animate, stagger, utils } = anime;

const anim = animate('.square', {
  x: '15rem',
  rotate: { to: 360, ease: 'inOut(3)' },
  opacity: [0, 1],
  backgroundColor: '#FF4B4B',
  delay: stagger(100),
  duration: 800,
  loop: 2,
  alternate: true,
  onComplete: self => console.log(self.progress), // 1
});

anim.then(() => console.log('done'));

const [ $count ] = utils.$('.count');
const counter = { value: 0 };
animate(counter, { value: 1000, modifier: utils.round(0), onUpdate: () => ($count.textContent = counter.value) });

animate('.dot', { x: (t, i, all) => (i - all.length / 2) * 20, y: [0, -30, 0], ease: 'inOutSine' });
```
