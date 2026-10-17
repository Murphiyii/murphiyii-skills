# Engine: `engine`

Checked against: `engine/index.d.ts`, `engine/engine.d.ts` (`Engine`), `core/clock.d.ts` (`Clock`: `speed`, `fps`, `deltaTime`), `core/globals.d.ts` (`globals`, `defaults`), `types/index.d.ts` (`DefaultsParams`); defaults from `src/engine/engine.js`, `src/core/globals.js`.

```ts
const { engine } = anime;
```
The singleton main loop that ticks every Timer, Animation and Timeline (requestAnimationFrame in browsers, setImmediate in Node).

## Properties
| Property | Type | Default | Notes |
|---|---|---|---|
| `defaults` | `DefaultsParams` | see below | Global defaults for all new instances (mutable). |
| `timeUnit` | `'ms' \| 's'` | `'ms'` | `'s'` makes every duration/delay in seconds and rescales `defaults.duration` (1000 becomes 1). |
| `speed` | number | `1` | Global playback rate multiplier. |
| `fps` | number | `240` | Global max frame rate. |
| `precision` | number | `4` | Decimal places of rendered CSS values. |
| `pauseOnDocumentHidden` | boolean | `true` | Pause when the tab is hidden (V3 `anime.suspendWhenDocumentHidden`). |
| `useDefaultMainLoop` | boolean | `true` | Set `false` to drive the engine yourself. |
| `paused` | boolean | | Read state. |
| `deltaTime` | number | | ms since last tick. |

`engine.defaults` (from `src/core/globals.js`): `duration: 1000`, `delay: 0`, `ease: 'out(2)'`, `loop: 0`, `loopDelay: 0`, `alternate: false`, `reversed: false`, `autoplay: true`, `frameRate: 240`, `playbackRate: 1`, `composition: 'replace'`, `persist: false`, `playbackEase: null`, `modifier: identity`, all callbacks `noop`.

Docs typo: some docs pages write `engine.engine.defaults`; the correct path is `engine.defaults`.

## Methods
| Method | Returns | Notes |
|---|---|---|
| `update()` | void | Advance all tickables one frame. Call it yourself when `useDefaultMainLoop = false`. |
| `pause()` | Engine | Pause everything. |
| `resume()` | this | Resume. |
| `wake()` | this | Start the rAF loop if needed (internal). |

The root also exports `globals` (`globals.defaults`, `globals.precision`, `globals.timeScale`, `globals.tickThreshold`); prefer the `engine` API.

## Examples
```js
const { engine, animate } = anime;

// Global defaults
engine.defaults.duration = 500;
engine.defaults.ease = 'inOutQuad';

// Work in seconds
engine.timeUnit = 's';
animate('.box', { x: 100, duration: 1.5 });

// Slow motion / debug
engine.speed = 0.25;

// Drive anime.js from an external render loop (Three.js, game loop)
engine.useDefaultMainLoop = false;
renderer.setAnimationLoop(() => {
  engine.update();
  renderer.render(scene, camera);
});

// Keep animating in background tabs
engine.pauseOnDocumentHidden = false;
```
