# Draggable: `createDraggable()`

Checked against: `draggable/draggable.d.ts`, `types/index.d.ts` (`DraggableParams`, `DraggableAxisParam`, `DraggableCursorParams`, `DraggableDragThresholdParams`); defaults from `src/draggable/draggable.js` (`constructor` and `refresh()`).

```ts
const { createDraggable } = anime;
createDraggable(target: TargetsParam, parameters?: DraggableParams): Draggable
```
Drags one element (first match) with velocity-based release, container bounds, snapping and auto-scroll. Target can also be a plain JS object (wrapped in a proxy), e.g. to drive a carousel value without moving a DOM node.

## Parameters
Any parameter typed `| ((draggable) => value)` can be a function, re-evaluated on `refresh()`.

| Param | Type | Default |
|---|---|---|
| `trigger` | DOMTargetSelector | the target itself (drag handle) |
| `container` | selector/element, or `[top, right, bottom, left]` px bounds, or fn | `document.body` (window scroll) |
| `x` / `y` | `boolean` or `DraggableAxisParam` | `true`; `false` locks the axis |
| `DraggableAxisParam` | `{ mapTo?: string, modifier?, composition?, snap? }` | `mapTo` defaults to `'translateX'` / `'translateY'` |
| `modifier` | `(v) => number\|string` | none |
| `snap` | number (increment) or `number[]` (snap points) or fn | `0` (off); per axis via `x: { snap }` |
| `containerPadding` | number or `[t, r, b, l]` or fn | `0` |
| `containerFriction` | 0..1 | `0.8` (resistance when dragged past bounds) |
| `releaseContainerFriction` | 0..1 | same as `containerFriction` |
| `dragSpeed` | number | `1` (negative inverts) |
| `dragThreshold` | number or `{ mouse, touch }` | `{ mouse: 3, touch: 7 }` px before drag starts |
| `scrollSpeed` | number | `1.5` (auto-scroll speed near container edges) |
| `scrollThreshold` | number | `20` px |
| `minVelocity` | number | `0` |
| `maxVelocity` | number | `50` |
| `velocityMultiplier` | number | `1` |
| `releaseMass` | number | `1` |
| `releaseStiffness` | number | `80` |
| `releaseDamping` | number | `20` (docs page says 10; src uses 20) |
| `releaseEase` | EasingParam | `eases.outQuint`; a `spring()` here overrides releaseMass/Stiffness/Damping and its `velocity` is replaced by the drag velocity |
| `cursor` | `boolean` or `{ onHover, onGrab }` or fn | `{ onHover: 'grab', onGrab: 'grabbing' }`; `false` disables |

## Callbacks (each receives the draggable)
`onGrab`, `onDrag`, `onUpdate` (position changed), `onRelease`, `onSnap`, `onSettle` (motion stopped after release), `onResize` (container/target resized), `onAfterResize`.

## Methods
| Method | Returns | Notes |
|---|---|---|
| `setX(x, muteUpdateCallback?)` / `setY(y, muteUpdateCallback?)` | this | Set position without animation. |
| `animateInView(duration?, gap?, ease?)` | this | Animate back inside container. Defaults: 350ms, gap 0, `inOutQuad`. |
| `scrollInView(duration?, gap?, ease?)` | this | Scroll container so the target is visible. Defaults: 350ms, gap 0, `inOutQuad`. |
| `stop()` | this | Stop release motion. |
| `reset()` | this | Stop, set x/y to 0, clear drag state. |
| `enable()` / `disable()` | this | |
| `refresh()` | void | Recompute function params and bounds. |
| `revert()` | this | Remove listeners and inline styles (cleanup). |
| `update()`, `updateBoundingValues()`, `updateScrollCoords()`, `computeVelocity(dx, dy)`, `isOutOfBounds(bounds, x, y)` | | Low level. |

## Properties
Get/set: `x`, `y`, `progressX`, `progressY` (0..1 within container). Read: `velocity`, `angle`, `deltaX`, `deltaY`, `destX`, `destY`, `grabbed`, `dragged`, `released`, `enabled`, `$target`, `$trigger`, `$container`, `containerBounds`, `snapX`, `snapY`, `animate` (internal `AnimatableObject`), `parameters`.

## Examples
```js
const { createDraggable, spring, utils } = anime;

createDraggable('.square', {
  container: '.grid',
  containerPadding: 10,
  snap: 50,
  releaseEase: spring({ bounce: .4 }),
  onSettle: self => console.log(self.x, self.y),
});

// Horizontal slider: only x, snapped to card positions
const slider = createDraggable('.track', {
  container: '.viewport',
  y: false,
  x: { snap: [0, -300, -600, -900] },
  onSnap: self => console.log('snapped', self.progressX),
});

// Map drag to a different property
createDraggable('.knob', { x: false, y: { mapTo: 'rotate' } });
```

Cleanup: `draggable.revert()`, or create inside `createScope().add()` and call `scope.revert()`.
