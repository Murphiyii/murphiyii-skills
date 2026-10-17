# Utilities: `stagger()` and `utils.*`

Checked against: `utils/index.d.ts`, `utils/stagger.d.ts`, `utils/target.d.ts`, `utils/random.d.ts`, `utils/time.d.ts`, `utils/chainable.d.ts`, `utils/number.d.ts`, `core/helpers.d.ts` (`clamp`, `round`, `snap`, `lerp`, `forEachChildren`, `addChild`, `removeChild`), `core/styles.d.ts` (`cleanInlineStyles`), `core/targets.d.ts` (`registerTargets` = `$`), `types/index.d.ts` (`StaggerParams`, `StaggerFunction`); behavior from `src/utils/*.js`.

All utils are on the global both at the top level (`const { stagger, random, $ } = anime;`) and under the `utils` namespace (`const { utils } = anime; utils.random()`).

## stagger
```ts
stagger(val: number | string | [number, number] | [string, string], params?: StaggerParams): StaggerFunction
```
Use as any function-based value (`delay`, `duration`, property values) or as a timeline `add()` position.
- `stagger(100)`: 0, 100, 200...
- `stagger('1rem')`: unit kept.
- `stagger([0, 500])`: range distributed from first to last value across all targets.

| Param | Type | Default | Notes |
|---|---|---|---|
| `start` | number or string | `0` | Offset added to every value. In a timeline position, can be a position string like `'<<'`, and defaults to the timeline end instead of `0`. |
| `from` | `number \| 'first' \| 'center' \| 'last' \| 'random' \| number[]` | `'first'` | Origin index. With grid, `[fx, fy]` as fractions 0..1. |
| `reversed` | boolean | `false` | |
| `grid` | `[cols, rows]` or `[cols, rows, depth]` or `true` | none | `true` = auto grid from element positions (`getBoundingClientRect`) or `{x, y, z}` of JS targets. |
| `axis` | `'x' \| 'y' \| 'z'` | none | With grid: stagger along one axis only. |
| `ease` | EasingParam | none | Distributes values along a curve. |
| `use` | attribute/property name or `(target, i, total) => number` | index | Custom index, e.g. `'data-line'` after `splitText`. |
| `total` | number | targets length | |
| `modifier` | `(v) => number \| string` | none | |
| `jitter` | number or `[start, end]` | none | Random +/- offset (tuple ramps along the order). |
| `seed` | boolean or number | `false` | Reproducible `jitter` and `from: 'random'`. |

```js
const { animate, stagger } = anime;
animate('.grid .cell', {
  scale: [{ to: .2, ease: 'outSine' }, { to: 1, ease: 'inOutQuad' }],
  delay: stagger(100, { grid: [14, 5], from: 'center' }),
});
animate('.bar', { y: stagger(['-2rem', '2rem'], { ease: 'inOut(3)' }), delay: stagger(50, { from: 'last', jitter: 30 }) });
```

## DOM and value helpers
| Function | Signature | Notes |
|---|---|---|
| `$` | `$(targets): Element[]` | Query (scope-aware), returns an array. `const [ $el ] = utils.$('.x')`. |
| `get` | `get(target, prop): string`; `get(target, prop, unit: string): string`; `get(target, prop, false): number` | Reads current animatable value (incl. transforms like `'x'`, `'rotate'`). |
| `set` | `set(targets, params): JSAnimation` | Instant set. Composition defaults to `'none'` (does not cancel running tweens). Accepts function-based values. |
| `remove` | `remove(targets, renderable?, propertyName?): Target[]` | Stop animating targets (all animations, or only `renderable`, or only one property). |
| `cleanInlineStyles` | `cleanInlineStyles(renderable): renderable` | Removes inline styles set by an animation/timeline. |
| `sync` | `sync(callback?): Timer` | Calls `callback` on the next engine tick (1-frame Timer). |
| `keepTime` | `keepTime(constructor) => (...args) => T` | Wrap a function that returns a Tickable; each call reverts the previous one and restores its progress. |

## Random
| Function | Signature |
|---|---|
| `random` | `random(min = 0, max = 1, decimalLength = 0): number` (inclusive) |
| `createSeededRandom` | `createSeededRandom(seed?, seededMin?, seededMax?, seededDecimalLength?): (min?, max?, decimals?) => number` |
| `randomPick` | `randomPick(items: string \| T[]): string \| T` |
| `shuffle` | `shuffle(items: any[], rnd?): any[]` (mutates) |

## Math (chainable)
Full call returns a value; calling with fewer args returns a function usable as `modifier` and chainable with other utils.

| Function | Full signature | Partial form |
|---|---|---|
| `clamp` | `clamp(v, min, max)` | `clamp(min, max)` |
| `round` | `round(v, decimalLength)` | `round(decimalLength)` |
| `snap` | `snap(v, increment \| number[])` | `snap(increment)` |
| `wrap` | `wrap(v, min, max)` | `wrap(min, max)` |
| `lerp` | `lerp(start, end, factor)` | `lerp(start, end)` |
| `damp` | `damp(start, end, deltaTime, factor)` | `damp(start, end, deltaTime)` |
| `mapRange` | `mapRange(v, inLow, inHigh, outLow, outHigh)` | `mapRange(inLow, inHigh, outLow, outHigh)` |
| `roundPad` | `roundPad(v, decimalLength): string` | `roundPad(decimalLength)` |
| `padStart` / `padEnd` | `padStart(v, totalLength, padString): string` | `padStart(totalLength, padString)` |
| `degToRad` / `radToDeg` | `degToRad(v)` | `degToRad()` |

```js
const { animate, utils } = anime;
animate(counter, { value: 100, modifier: utils.round(0) });
animate('.needle', { rotate: 720, modifier: utils.wrap(0, 360) });
const toPct = utils.clamp(0, 1).mapRange(0, 1, 0, 100).round(1); toPct(0.4567); // 45.7
const smooth = utils.damp(current, target, clock.deltaTime, .1); // frame-rate independent lerp
```
Note: docs pages show conflicting `damp` signatures; `src` is `damp(start, end, deltaTime, factor)`.

Low level (exported, rarely needed): `forEachChildren`, `addChild`, `removeChild` (linked-list helpers).
