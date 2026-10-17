# Timer: `createTimer()`

Checked against: `timer/timer.d.ts`, `core/clock.d.ts`, `types/index.d.ts` (`TimerParams`, `TimerOptions`, `TickableCallbacks`); defaults from `src/timer/timer.js`.

```ts
const { createTimer } = anime;
createTimer(parameters?: TimerParams): Timer   // Timer extends Clock
```
A Timer is an animation without targets: a scheduled, controllable clock for callbacks (game loops, counters, polling, syncing non-anime work). `JSAnimation` and `Timeline` extend `Timer`, so every method below exists on them too.

## Params
| Param | Type | Default |
|---|---|---|
| `duration` | number | **`Infinity`** (unlike animations, which default to 1000) |
| `delay` | number | `0` |
| `loop` | number or boolean | `0` |
| `loopDelay` | number | `0` |
| `alternate` | boolean | `false` |
| `reversed` | boolean | `false` |
| `autoplay` | boolean or `ScrollObserver` | `true` |
| `frameRate` | number | `240` (lower it to throttle `onUpdate`) |
| `playbackRate` | number | `1` |
| `priority` | number | internal ordering |
| `id` | string or number | auto |
| `onBegin onBeforeUpdate onUpdate onLoop onPause onComplete` | `Callback<Timer>` | noop |

Timers have no `onRender` (that is on `JSAnimation`/`Timeline`).

## Methods (return `this` unless noted)
`play()`, `reverse()`, `pause()`, `resume()`, `restart()`, `alternate()`, `seek(time, muteCallbacks?, internalRender?)`, `stretch(newDuration)`, `cancel()`, `revert()`, `complete(muteCallbacks?)`, `reset(softReset?)`, `init(internalRender?)`, `resetTime()`, `then(cb?)` returns `Promise`.

## Properties
`id`, `parent`, `duration`, `iterationDuration`, `iterationCount`, `currentTime`, `iterationCurrentTime`, `progress` (0..1), `iterationProgress`, `currentIteration`, `reversed`, `paused`, `began`, `completed`, `cancelled`, `backwards`, `deltaTime` (ms since last frame), `speed`, `fps`.

## Examples
```js
const { createTimer, utils } = anime;

const [ $time ] = utils.$('.time');

// Clock display updated 30 times per second, runs forever
const clock = createTimer({
  frameRate: 30,
  onUpdate: self => { $time.textContent = (self.currentTime / 1000).toFixed(1); },
});

// 3 loops of 1s with a callback each loop
createTimer({
  duration: 1000,
  loop: 2,
  onLoop: self => console.log('loop', self.currentIteration),
  onComplete: () => console.log('done'),
});

clock.pause();
clock.seek(5000);
clock.resume();
```

Related: `utils.sync(cb)` runs `cb` on the next engine tick via a 1-frame Timer. For a custom render loop (Three.js etc.), set `engine.useDefaultMainLoop = false` and call `engine.update()` in your loop.
