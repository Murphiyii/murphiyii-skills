# Scope: `createScope()` (cleanup, frameworks, media queries)

Checked against: `scope/scope.d.ts`, `types/index.d.ts` (`ScopeParams`, `ScopeConstructorCallback`, `ScopeCleanupCallback`, `ScopeMethod`, `ScopedCallback`, `Revertible`, `ReactRef`, `AngularRef`, `DefaultsParams`); behavior from `src/scope/scope.js`; usage from `examples/onscroll-responsive-scope`, `examples/text/hover-effects`.

```ts
const { createScope } = anime;
createScope(params?: ScopeParams): Scope
```
Everything created inside a scope constructor (animations, timers, timelines, animatables, draggables, scroll observers, text splitters, layouts, WAAPI animations, nested scopes) is registered and reverted together by `scope.revert()`. This is the V4 way to clean up in React, Vue, Svelte, Angular, or any component lifecycle.

## Params (`ScopeParams`)
| Param | Type | Default | Notes |
|---|---|---|---|
| `root` | selector, element, `ReactRef` (`{ current }`), or `AngularRef` (`{ nativeElement }`) | `document` | String selectors in `animate()`, `utils.$()` etc. are queried inside root only. |
| `defaults` | `DefaultsParams` | global defaults | Default params for everything created inside. |
| `mediaQueries` | `Record<string, string>` | none | Named queries; results in `scope.matches[name]`. On change, the scope reverts and re-runs its constructors. |

## Methods
| Method | Returns | Notes |
|---|---|---|
| `add(constructor: (scope) => cleanup \| void)` | this | Runs now inside the scope; re-runs on `refresh()`/media query change. Optional returned function runs on revert. |
| `add(name: string, method: (...args) => any)` | this | Registers `scope.methods[name]`; calls run inside the scope (new animations get registered). |
| `addOnce(constructor)` | this | Runs once; not re-run on refresh. |
| `keepTime(cb: (scope) => Tickable)` | Tickable | Recreate a tickable on refresh while preserving its current time/progress. |
| `execute(cb: (scope) => T)` | T | Run `cb` with this scope active (root, defaults). |
| `refresh()` | this | Revert and re-run all constructors. |
| `revert()` | void | Revert everything, remove media query listeners, clear methods and data. |
| `register(revertible)` | void | Manually register a revertible. |

Properties: `root`, `defaults`, `matches`, `methods`, `data` (free storage), `revertibles`, `constructors`, `mediaQueryLists`.

## React
```jsx
import { useEffect, useRef } from 'react';
const { animate, createScope, stagger } = anime;

export function Boxes() {
  const root = useRef(null);
  const scope = useRef(null);

  useEffect(() => {
    scope.current = createScope({ root }).add(self => {
      animate('.box', { x: '10rem', delay: stagger(80), loop: true, alternate: true });
      self.add('spin', () => animate('.box', { rotate: '+=360' }));
    });
    return () => scope.current.revert();   // cleanup on unmount (and StrictMode double-run)
  }, []);

  return (
    <div ref={root}>
      <div className="box" /><div className="box" />
      <button onClick={() => scope.current.methods.spin()}>Spin</button>
    </div>
  );
}
```

## Vue 3
```js
import { onMounted, onBeforeUnmount, ref } from 'vue';
const { animate, createScope } = anime;

const root = ref(null);
let scope;
onMounted(() => {
  scope = createScope({ root: root.value }).add(() => {
    animate('.item', { opacity: [0, 1], y: [20, 0] });
  });
});
onBeforeUnmount(() => scope && scope.revert());
```

## Responsive (media queries)
```js
const { animate, createScope, onScroll } = anime;

createScope({
  mediaQueries: { landscape: '(orientation: landscape)', reduceMotion: '(prefers-reduced-motion)' },
  defaults: { ease: 'out(3)', duration: 500 },
}).add(({ matches }) => {
  if (matches.reduceMotion) return;
  animate('.card', { y: matches.landscape ? -40 : -20, autoplay: onScroll({ sync: true }) });
});
```

Gotchas: `root` is resolved once at creation (`ref.current`, then `ref.nativeElement`, then selector), so create the scope after mount (inside `useEffect`/`onMounted`). In React pass the ref object (`{ root }`); a selector also works but is not component-local. Create animations inside `add()` callbacks so they are registered; ones created outside the scope are not reverted. `revert()` restores original inline styles.
