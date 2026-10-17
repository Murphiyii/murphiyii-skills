# Animatable: `createAnimatable()`

Checked against: `animatable/animatable.d.ts`, `types/index.d.ts` (`AnimatableParams`, `AnimatablePropertyParamsOptions`, `AnimatableObject`, `AnimatableProperty`, `AnimatablePropertySetter`, `AnimatablePropertyGetter`); behavior from `src/animatable/animatable.js`.

```ts
const { createAnimatable } = anime;
createAnimatable(targets: TargetsParam, parameters: AnimatableParams): AnimatableObject
// AnimatableObject = Animatable & Record<string, AnimatableProperty>
```
Use for values that change continuously from input (cursor follow, pointer parallax, sliders). Each declared property becomes a getter/setter function that retargets a running tween instead of creating a new animation per event. Much cheaper than calling `animate()` in a `mousemove` handler.

## Params
Property keys declare which properties are animatable. Value forms:
| Form | Meaning |
|---|---|
| `x: 500` | Number = transition duration (ms) for that property. |
| `x: { duration, ease, unit, modifier, composition }` | Per-property params. `unit` e.g. `'rem'`, `'rad'`. |
| `x: 0` | Duration 0 = set instantly (use with your own smoothing). |

Top-level params (apply to all properties unless overridden): `duration` (default 1000), `ease` (default `'out(2)'`, prefer `out` types), `modifier`. `unit` works only per property (`x: { unit: 'rem' }`): a top-level `unit` key is treated as an animatable property named `unit`. A top-level `composition` is ignored (always overwritten with `replace`); set `composition` per property instead. Callbacks: `onBegin`, `onUpdate`, `onRender`, `onComplete`, `onPause`, etc. (fired by an internal callbacks animation).

## Property functions (`AnimatableProperty`)
```ts
animatable.x(to: number | number[], duration?: number, ease?: EasingParam): AnimatableObject  // setter, chainable
animatable.x(): number | number[]                                                           // getter, current value
```
The setter animates from the current value to `to`; optional `duration` and `ease` override for this call.

## Instance members
`targets` (array), `animations` (`Record<string, JSAnimation>`, one per property), `callbacks` (`JSAnimation | null`), `revert()` (stops, restores values, disables property functions).

## Examples
```js
const { createAnimatable, utils } = anime;

const [ $area ] = utils.$('.area');
const follower = createAnimatable('.circle', {
  x: 500,                         // 500ms on x
  y: { duration: 800, ease: 'out(4)' },
  ease: 'out(3)',
});

$area.addEventListener('pointermove', e => {
  const { left, top, width, height } = $area.getBoundingClientRect();
  const hw = width / 2, hh = height / 2;
  follower.x(utils.clamp(e.clientX - left - hw, -hw, hw));
  follower.y(utils.clamp(e.clientY - top - hh, -hh, hh));
});

// Read current value
console.log(follower.x());

// Radians unit
const dial = createAnimatable('.dial', { rotate: { unit: 'rad' }, ease: 'linear' });
dial.rotate(Math.PI / 2, 300);
```

Gotchas: only properties declared at creation get functions. Values passed to setters are numbers (unit comes from the `unit` param). Call `revert()` on cleanup (or create it inside `createScope().add()`).
