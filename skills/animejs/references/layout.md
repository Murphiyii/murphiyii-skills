# Layout (FLIP): `createLayout()`

Checked against: `layout/layout.d.ts` (`AutoLayout`, `AutoLayoutParams`, `LayoutAnimationParams`, `LayoutSpecificAnimationParams`, `LayoutStateParams`, `LayoutOptions`), `types/index.d.ts` (`TimerParams`, `DOMTargetSelector`); defaults from `src/layout/layout.js`; usage from `examples/auto-layout/*`.

```ts
const { createLayout } = anime;
createLayout(root: DOMTargetSelector, params?: AutoLayoutParams): AutoLayout
```
Animates layout changes (position, size, reorder, enter/leave, moving between parents) that you make with normal DOM/CSS code. It records a snapshot, you mutate the DOM, then it animates from old to new state (FLIP). New in v4 (not in the 4.0.0 docs).

## Parameters (`AutoLayoutParams` = animation params + options)
| Param | Type | Default |
|---|---|---|
| `children` | selector or array of selectors | `'*'` (all descendants of root) |
| `properties` | `string[]` | extra CSS props to transition. Always included: `opacity fontSize color backgroundColor borderRadius border filter clipPath` |
| `duration` | number or FunctionValue | `350` |
| `delay` | number or FunctionValue | `0` (accepts `stagger()`) |
| `ease` | EasingParam or FunctionValue | `'inOut(3.5)'`; a `spring()` sets duration from the spring |
| `playbackEase` | EasingParam | none |
| `enterFrom` | `LayoutStateParams` | `{ opacity: 0 }`: state new elements animate from |
| `leaveTo` | `LayoutStateParams` | `{ opacity: 0 }`: state removed/hidden elements animate to |
| `swapAt` | `LayoutStateParams` | `{ opacity: 0 }` with ease `'inOut(1.75)'`: midpoint state when an element swaps |
| `id` | string or number | auto |
| Timer params / callbacks | `loop`, `alternate`, `autoplay` (accepts `onScroll()`), `onBegin`, `onUpdate`, `onComplete`, ... | |

`LayoutStateParams` = CSS props (number, string, or function) plus their own `delay`, `duration`, `ease`.

## Methods
| Method | Returns | Notes |
|---|---|---|
| `update(callback(layout), params?)` | `Timeline` | Shortcut: `record()`, run `callback`, then `animate(params)`. Main API. |
| `record()` | this | Snapshot current state (cancels running layout animation). |
| `animate(params?)` | `Timeline` | Animate from the last recorded state to the current DOM. `params` overrides duration/delay/ease/enterFrom/leaveTo/swapAt/callbacks for this run. |
| `revert()` | this | Cancel and clean up. |

Properties: `root`, `params`, `children`, `timeline` (current), `oldState`, `newState`, `animating`, `entering`, `leaving`, `swapping` (arrays of elements), `properties`, `id`.

Element identity across states uses `data-layout-id` (auto-assigned; set it yourself to match elements that are re-created, e.g. re-rendered lists).

## Examples
```js
const { createLayout, spring, stagger, onScroll } = anime;

// Accordion
const accordion = createLayout('.accordion', {
  ease: spring({ bounce: .2, duration: 450 }),
  enterFrom: { opacity: 0, filter: 'blur(5px)' },
  leaveTo: { opacity: 0, filter: 'blur(5px)' },
});
document.addEventListener('click', e => {
  const $btn = e.target.closest('.accordion button');
  if ($btn) accordion.update(() => $btn.parentElement.classList.toggle('is-open'));
});

// Grid/list view switch with staggered delay
const layout = createLayout('#cards', { duration: 750, ease: 'inOutExpo' });
function setView(view, reversed) {
  layout.update(({ root }) => {
    root.classList.remove('tiles', 'list');
    root.classList.add(view);
  }, { delay: stagger([0, 500], { reversed }) });
}

// Removing an item: animate leave, then remove from DOM
layout.update(() => $item.classList.add('is-removed'), {
  onComplete: () => layout.update(() => $item.remove()),
});

// Two-step: record, mutate, animate (scroll-linked)
layout.record();
layout.root.classList.toggle('expanded');
layout.animate({ autoplay: onScroll({ sync: true, enter: 'top top' }) });
```

Gotchas: mutate the DOM inside the `update()` callback (or between `record()` and `animate()`), not before. Hidden elements (`display: none`) count as leaving/entering. For framework re-renders, call `record()` before the state change and `animate()` after the DOM commits (e.g. in a layout effect).
