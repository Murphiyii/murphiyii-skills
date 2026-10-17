# Timeline: `createTimeline()`

Checked against: `timeline/timeline.d.ts`, `timeline/position.d.ts`, `timer/timer.d.ts`, `types/index.d.ts` (`TimelineParams`, `TimelineOptions`, `DefaultsParams`, `TimelinePosition`, `TimelineAnimationPosition`, `TimerParams`, `AnimationParams`); behavior from `src/timeline/timeline.js` and `src/timeline/position.js`.

```ts
const { createTimeline } = anime;
createTimeline(parameters?: TimelineParams): Timeline   // Timeline extends Timer
```

## Parameters (`TimelineParams` = TimerOptions & TimelineOptions & callbacks)
| Param | Type | Default | Notes |
|---|---|---|---|
| `defaults` | `DefaultsParams` | global defaults | Inherited by every child (`ease`, `duration`, `delay`, `loop`, `alternate`, `composition`, `modifier`, callbacks...). |
| `playbackEase` | EasingParam | none | Eases the whole timeline playhead. |
| `composition` | boolean | `true` | `false` skips tween composition between children (faster for huge timelines). |
| `id`, `delay`, `loop`, `loopDelay`, `alternate`, `reversed`, `autoplay`, `frameRate`, `playbackRate` | as Timer | | `autoplay` accepts `onScroll({...})`. |
| `onBegin onBeforeUpdate onUpdate onRender onLoop onPause onComplete` | `Callback<Timeline>` | noop | |

Timeline `duration` is computed from children (starts at 0); do not pass it.

## Methods (all chainable, return `this`)
| Signature | Notes |
|---|---|
| `add(targets, animationParams, position?)` | Adds an animation. `position` may be a `TimelinePosition` or a `stagger()` function to offset each target. |
| `add(timerParams, position?)` | Adds a Timer (object with timer params, no targets). |
| `set(targets, params, position?)` | Instant set (duration ~0, composition `replace`). |
| `call(callback, position?)` | Calls `callback(timeline)` at position. |
| `label(name, position?)` | Named position. |
| `sync(synced?, position?)` | Embeds an existing Animation, Timer, Timeline, `WAAPIAnimation`, or native `Animation`; it is paused and driven by the timeline. |
| `remove(targets, propertyName?)` | Removes tweens of targets. |
| `stretch(newDuration)` | Rescales children and labels. |
| `refresh()` | Re-evaluates children function-based values. |
| `revert()` | Cancel and restore all values. |
| `then(cb?)` | Promise. |
| `init()` | Inherited from Timer; renders initial state (examples call `.init()` after building a paused timeline). |
| Playback: `play reverse pause resume restart alternate seek cancel complete reset` | Same as Timer. |

Properties: `labels` (Record<string, number>), `defaults`, `composition`, `duration`, `iterationDuration`, plus all Timer properties.

## Position syntax (`TimelinePosition`)
| Value | Meaning |
|---|---|
| omitted | End of the timeline (after the last child). |
| `500` | Absolute ms. |
| `'+=100'` | 100ms after timeline end (gap). |
| `'-=100'` | 100ms before timeline end (overlap). |
| `'*=.5'` | Timeline end multiplied by 0.5. |
| `'<'` | End of the previous child. |
| `'<<'` | Start of the previous child. |
| `'<<+=250'`, `'<<-=250'`, `'<+=100'`, `'<-=100'` | Relative to previous child's start or end. |
| `'myLabel'`, `'myLabel+=100'` | Label, optionally offset. |
| `stagger(100)` / `stagger(100, { start: '<<' })` | Only on `add()`: per-target offsets (each target becomes its own child). |

## Examples
```js
const { createTimeline, stagger } = anime;

const tl = createTimeline({
  defaults: { ease: 'inOutExpo', duration: 600 },
  loop: true,
  alternate: true,
  loopDelay: 300,
})
.label('start')
.add('.square', { x: '15rem' })
.add('.circle', { x: '15rem', scale: [1, 1.5] }, '-=300')
.add('.triangle', { x: '15rem', rotate: 120 }, '<<')
.set('.label', { opacity: 1 }, 'start')
.call(self => console.log('mid', self.currentTime), '<')
.add('.dot', { y: -20 }, stagger(50, { from: 'center' }));

// Embed an existing animation or WAAPI animation
const { animate, waapi } = anime;
const spin = animate('.gear', { rotate: 360, autoplay: false });
const fade = waapi.animate('.panel', { opacity: [0, 1] });
createTimeline().sync(spin).sync(fade, '<<');
```

Gotchas: children inherit only from `defaults`, not from top-level timeline params. A child's own `loop` is respected. Labels must be created before use.
