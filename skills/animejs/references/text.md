# Text: `splitText()`, `scrambleText()`

Checked against: `text/index.d.ts`, `text/split.d.ts` (`TextSplitter`, `splitText`, `split`), `text/scramble.d.ts` (`scrambleText`, `ScrambleTextTween`), `types/index.d.ts` (`TextSplitterParams`, `SplitTemplateParams`, `SplitValue`, `SplitFunctionValue`, `ScrambleTextParams`); defaults from `src/text/split.js`, `src/text/scramble.js`; usage from `examples/text/*`.

```ts
const { splitText, scrambleText } = anime;  // or the `text` namespace: anime.text.splitText(...)
splitText(target: Element | NodeList | string | Element[], parameters?: TextSplitterParams): TextSplitter
scrambleText(params?: ScrambleTextParams): FunctionValue<ScrambleTextTween>
```
`text.split()` / `split()` is **deprecated** in 4.5.0 (console.warn). Use `splitText()`.

**Only the first matching element is split** (a selector, NodeList or array resolves to its first element). For several elements, call `splitText()` once per element: `utils.$('p').forEach($p => splitText($p, ...))`.

## splitText parameters
| Param | Type | Default |
|---|---|---|
| `lines` | `boolean \| string \| SplitTemplateParams \| (node) => string` | off |
| `words` | same | **on** (words split when undefined) |
| `chars` | same | off |
| `accessible` | boolean | `true` (keeps an accessible copy, split spans `aria-hidden`) |
| `includeSpaces` | boolean | `false` |
| `debug` | boolean | `false` (outlines lines/words/chars) |

`SplitTemplateParams`: `{ class?: string | false, wrap?: boolean | 'hidden' | 'clip' | 'visible' | 'scroll' | 'auto', clone?: boolean | 'top' | 'right' | 'bottom' | 'left' | 'center' }`.
- `wrap: true` wraps each piece in a span with `overflow: clip` (use for "slide up from mask" reveals).
- `clone: 'bottom'` duplicates each piece inside it, positioned below (for roll-over text effects).
- A string template must contain `{value}`; `{i}` is replaced by the index: `chars: '<span class="c">{value}</span>'`.

Each split element gets `data-line`, `data-word`, `data-char` index attributes, so `stagger(..., { use: 'data-line' })` can group by line.

## TextSplitter
Properties: `lines`, `words`, `chars` (arrays of elements), `$target`, `html` (original), `ready`, `debug`, `includeSpaces`, `accessible`.
Methods: `addEffect(fn: (splitter) => Tickable | (() => void) | void)` (returns this), `revert()` (restore original HTML), `split(clearCache?)`, `refresh()`.

**Lines**: line splitting waits for `document.fonts.ready` and re-splits on resize (ResizeObserver, width change). Animations created directly after `splitText()` with `lines: true` may target nothing or stale elements. Put line animations in `addEffect()`: the effect re-runs after each re-split, the returned animation/timeline is reverted and recreated with its progress kept, and a returned function runs as cleanup.

## Examples
```js
const { animate, createTimeline, splitText, stagger, utils } = anime;

// Chars rise from a mask
const { chars } = splitText('h1', { chars: { wrap: true } });
animate(chars, { y: ['100%', '0%'], duration: 750, ease: 'out(3)', delay: stagger(20) });

// Lines: use addEffect so it survives font load and resize (one splitter per paragraph)
utils.$('p').forEach($p =>
  splitText($p, { lines: { wrap: 'clip' } }).addEffect(self =>
    animate(self.lines, { y: ['100%', '0%'], delay: stagger(80) })
  )
);

// Words grouped by line
splitText('.quote', { lines: true }).addEffect(({ lines, words }) =>
  createTimeline()
    .add(lines, { opacity: [0, 1] }, stagger(100))
    .add(words, { y: [10, 0] }, stagger(100, { use: 'data-line', start: 0 }))
);
```
Cleanup: `splitter.revert()` (or use `createScope`, which registers splitters).

## scrambleText
Returns a function-based value producing a tween object; animate a text property (usually `innerHTML`) with it.

| Param | Type | Default |
|---|---|---|
| `text` | string or fn | element's original text |
| `chars` | string or fn; named sets `'lowercase' 'uppercase' 'numbers' 'symbols' 'braille' 'blocks' 'shades'`, ranges `'A-Z'` | `'a-zA-Z0-9!%#_'` |
| `ease` | EasingParam | `'linear'` |
| `from` | number or `'left' \| 'center' \| 'right' \| 'random' \| 'auto'` | `'auto'` |
| `reversed` | boolean | `false` |
| `cursor` | `boolean \| number \| string` | none; `true` = `'_'` |
| `perturbation` | number | `0` |
| `seed` | number | `0` |
| `override` | `boolean \| string` | `true` (scrambled start); `false` original text; `''` blank; `' '` spaces |
| `revealRate` | number (chars/s) | `60` |
| `settleDuration` | ms | `300` |
| `settleRate` | changes/s | `30` |
| `duration`, `delay`, `revealDelay` | number or fn | auto duration |
| `onChange` | `(text, progress) => void` | noop |

```js
const { animate, createTimeline, scrambleText, utils } = anime;

utils.$('.scramble').forEach($el =>
  $el.addEventListener('pointerenter', () =>
    animate($el, { innerHTML: scrambleText({ cursor: '░▒▓█', perturbation: .2 }) })
  )
);

createTimeline().add('.title', { innerHTML: scrambleText({ text: 'Hello', override: '', from: 'center', duration: 750 }) });
```
