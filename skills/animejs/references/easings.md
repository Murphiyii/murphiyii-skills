# Easings

Checked against: `easings/index.d.ts`, `easings/eases/index.d.ts`, `easings/eases/parser.d.ts`, `easings/spring/index.d.ts`, `easings/cubic-bezier/index.d.ts`, `easings/steps/index.d.ts`, `easings/linear/index.d.ts`, `easings/irregular/index.d.ts`, `types/index.d.ts` (`EasingParam`, `EaseStringParamNames`, `EasingFunction`, `SpringParams`, `PowerEasing`, `BackEasing`, `ElasticEasing`); behavior from `src/easings/**`.

`ease` accepts: a built-in name string, an easing function `(t: number) => number`, or a `Spring`. Default `'out(2)'`. Used by `ease`, `playbackEase`, `stagger({ ease })`, `onScroll({ sync })`, draggable `releaseEase`, layout `ease`.

From the `anime` global: `const { eases, spring, cubicBezier, steps, linear, irregular } = anime;`, or through the `easings` namespace (`anime.easings.spring(...)`).

## Built-in names (strings, or functions on `eases`)
Types: `in`, `out`, `inOut`, `outIn`. Families: `Quad Cubic Quart Quint Sine Circ Expo Bounce Back Elastic`.
- Power: `'in'`, `'out'`, `'inOut'`, `'outIn'`; parameter = power, default `1.68` in src (the .d.ts name hint says `1.675`): `'out(3)'`, `'inOut(2)'`.
- Named: `'inQuad' 'outQuad' 'inOutQuad' 'outInQuad'`, same pattern for every family: `'outExpo'`, `'inOutSine'`, `'outBounce'`...
- Back: `'outBack'`, `'outBack(overshoot = 1.7)'` e.g. `'inOutBack(2)'`.
- Elastic: `'outElastic(amplitude = 1, period = .3)'` e.g. `'outElastic(1, .5)'`.
- `'linear'` (and `'none'`).
- Function form: `eases.outQuad`, `eases.out(3)`, `eases.outElastic(1, .5)`.

No `ease` prefix (V3 `'easeOutQuad'` is wrong).

## Easing generators (take them from the `anime` global and call them; string forms are removed)
| Function | Signature | Defaults | Notes |
|---|---|---|---|
| `cubicBezier` | `cubicBezier(mX1?, mY1?, mX2?, mY2?)` | `.5, 0, .5, 1` | Same as CSS cubic-bezier. |
| `steps` | `steps(steps?, fromStart?)` | `10`, `false` | |
| `linear` | `linear(...args: (string \| number)[])` | | CSS `linear()` style points: `linear(0, '0.5 50%', 1)`. Not the same as the `'linear'` string. |
| `irregular` | `irregular(length?, randomness?)` | `10`, `1` | Random stepped pacing. |

**4.5.0**: `ease: 'steps(5)'`, `'cubicBezier(...)'`, `'linear(...)'`, `'irregular(...)'` log a warning and fall back to no easing. Use the functions.

## Spring
```ts
spring(parameters?: SpringParams): Spring
createSpring(parameters?: SpringParams): Spring   // deprecated in 4.5.0 (console.warn), use spring()
```
Two param modes:
- Perceived: `bounce` (-1..1, default `.5`), `duration` (perceived ms, 10..10000, default `628`). Used when either is provided.
- Physics: `mass` (default `1`), `stiffness` (`100`), `damping` (`10`), `velocity` (`0`).
- `onComplete(animation)`: fires when the perceived duration is reached (before the spring fully settles).

A spring **overrides the animation's `duration`** with its computed settling duration (`spring.settlingDuration`). Instance setters (`bounce`, `duration`, `stiffness`, `damping`, `mass`, `velocity`) recompute it. `spring.ease` is the raw easing function.

Discrepancy: `SpringParams` JSDoc in `types/index.d.ts` says bounce and duration default to 0; `src/easings/spring/index.js` uses `.5` and `628`.

## Examples
```js
const { animate, eases, spring, cubicBezier, steps, stagger } = anime;

animate('.a', { x: 200, ease: 'outExpo' });
animate('.b', { x: 200, ease: 'inOut(3)' });
animate('.c', { x: 200, ease: spring({ bounce: .35, duration: 500 }) });
animate('.d', { x: 200, ease: spring({ stiffness: 90, damping: 14 }) });
animate('.e', { x: 200, ease: cubicBezier(.7, .1, .5, .9) });
animate('.f', { x: 200, ease: steps(5) });
animate('.g', { x: 200, ease: t => t * t * t });             // custom
animate('.h', { x: 200, rotate: { to: 360, ease: 'outBack(2)' } });   // per property
animate('.i', { x: [0, 200, 0], ease: 'inOut', playbackEase: 'outIn(5)' }); // whole playback
animate('.j', { y: -20, delay: stagger(100, { ease: 'out(3)' }) });

const { engine } = anime;
engine.defaults.ease = 'outElastic(1, .5)'; // change global default
```
In `waapi.animate()`, native CSS easing strings (`'ease-in-out'`, `'cubic-bezier(...)'`, `'steps(6, start)'`, `'linear(...)'`) pass through unchanged, while anime.js easing names, functions and `spring()` are converted to a CSS `linear()` string. `waapi.convertEase(fn, samples = 100)` returns that string for native `Element.animate()`.
