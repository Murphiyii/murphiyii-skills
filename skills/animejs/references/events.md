# Events: `onScroll()` (ScrollObserver)

Checked against: `events/scroll.d.ts` (`ScrollObserver`, `onScroll`, `scrollContainers`), `types/index.d.ts` (`ScrollObserverParams`, `ScrollThresholdValue`, `ScrollThresholdParam`, `ScrollThresholdCallback`, `ScrollObserverAxisCallback`); defaults from `src/events/scroll.js`.

```ts
const { onScroll } = anime;
onScroll(parameters?: ScrollObserverParams): ScrollObserver
```
Triggers or scrubs a Timer, Animation, Timeline, or `waapi.animate()` animation on scroll. Pass it as `autoplay`, or call `observer.link(tickable)`.

```js
animate('.square', { x: '15rem', autoplay: onScroll({ /* params */ }) });
```

## Params
| Param | Type | Default | Notes |
|---|---|---|---|
| `container` | TargetsParam | `document.body` (window) | Scrolling element. |
| `target` | TargetsParam | first DOM target of the linked animation | Element whose position is observed. |
| `axis` | `'x' \| 'y'` or fn | `'y'` | |
| `enter` | threshold | `'end start'` | |
| `leave` | threshold | `'start end'` | |
| `sync` | see below | `'play pause'` | |
| `repeat` | boolean or fn | `true` | `false`: revert after first completion. |
| `debug` | boolean | `false` | Shows threshold markers. |
| `id` | string or number | auto | |

## Thresholds
String `'<container> <target>'`: first value is the position in the container (viewport), second is the position on the target. Default enter `'end start'` = container bottom meets target top.
- Keywords: `top`/`left`/`start` (0), `bottom`/`right`/`end` (100%), `center` (50%), `min`/`max` (clamped scroll limits).
- Numbers and percentages: `'80% 20%'`, `'50 -25'` (px).
- Relative: `'bottom-=50 top'`, `'top+=60 bottom'`, `'center+=1em top-=100%'`.
- Object form: `{ container: 'bottom', target: 'top' }`.
- Function form: `() => \`bottom-=${offset} top\`` (re-evaluated on `refresh()`).

## Sync modes (`sync`)
| Value | Behavior |
|---|---|
| `'play pause'` (default) | Method names to call on enter / leave. |
| `'play pause reverse reset'` | 4 names: enterForward, leaveForward, enterBackward, leaveBackward. Valid names: Timer methods (`play`, `pause`, `resume`, `reverse`, `restart`, `reset`, `complete`, `alternate`, ...). |
| `true` or `1` | Scrub: progress follows scroll exactly. |
| number `0..1` (e.g. `.25`) | Smoothed scrub (lower = smoother/laggier). |
| ease name/function (e.g. `'inOutCirc'`) | Scrub with eased progress. |

Use `ease: 'linear'` on the animation when scrubbing so scroll maps linearly.

## Callbacks (each receives the ScrollObserver)
`onEnter`, `onLeave`, `onEnterForward`, `onLeaveForward`, `onEnterBackward`, `onLeaveBackward`, `onUpdate` (every scroll while active), `onSyncComplete`, `onResize`.

## Methods
`link(tickable | WAAPIAnimation)` (one linked object; returns this), `refresh()` (recompute function thresholds/bounds), `revert()` (disable, remove listeners and debug), `removeDebug()`, `updateBounds()`.

## Properties
`progress` (0..1), `scroll`, `velocity`, `backward`, `isInView`, `began`, `completed`, `linked`, `target`, `container`, `horizontal`, `repeat`, `enter`, `leave`, `sync`, `offset`, `offsetStart`, `offsetEnd`, `distance`, `id`. Access from an animation: `anim._autoplay` (the observer passed as `autoplay`).

## Examples
```js
const { animate, createTimeline, createTimer, onScroll, stagger, utils } = anime;

// Play on enter, pause on leave (default sync 'play pause', repeats on every re-enter).
// One observer has ONE target: give each card its own observer.
utils.$('.card').forEach($card =>
  animate($card, { opacity: [0, 1], y: [40, 0], autoplay: onScroll({ target: $card, enter: 'bottom-=100 top' }) })
);

// Scrub a timeline while a section scrolls through the viewport
createTimeline({ autoplay: onScroll({ target: '.section', enter: 'bottom top', leave: 'top bottom', sync: true }) })
  .add('.title', { x: ['-50vw', 0], ease: 'linear' })
  .add('.image', { scale: [.8, 1], ease: 'linear' }, '<<');

// Inside a custom scroll container, smoothed
animate('.square', {
  x: '15rem', rotate: '1turn', ease: 'linear', delay: stagger(100),
  autoplay: onScroll({ container: '.scroll-container', sync: .25, debug: true }),
});

// Callbacks only (Timer)
createTimer({ autoplay: onScroll({ target: '.footer', onEnter: () => loadMore() }) });
```

Gotchas: one ScrollObserver observes a single `target` element (default: the linked animation's first DOM target), so `animate('.card', { autoplay: onScroll() })` triggers every card from the first card's position; loop over elements for per-element triggers. Thresholds order is container first, target second. A `sync` scrub ignores the animation's own timing for progress; keep `ease: 'linear'` unless you want the curve. Call `revert()` (or use a scope) on teardown.
