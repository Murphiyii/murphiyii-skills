---
name: vanilla-web-architecture
description: Architecture for maintainable no-framework web apps (plain HTML, CSS, JS; Web Components, one-way store, multi-page). Use when building or extending such an app.
---

Plain HTML 5, CSS 3 and JavaScript only: no frameworks, no build step, no npm runtime
dependencies. ES modules do not load from `file://`, so develop and test through any
static server. If the user wants to open the app by double-clicking `index.html`, tell
them in one sentence that it needs a static server and give the command
(`python -m http.server`). Keep the modules: classic scripts would drop the import
structure this skill depends on.

Multi-page app (one HTML file per route): also read [mpa.md](mpa.md).
Single-page app (client-side routing): also read [spa.md](spa.md).
A single-screen app uses the layout below alone; add mpa.md or spa.md when a second
page appears.

## Core pattern
Component-Based Architecture with a small Flux/Redux-lite store. State flows one way:
event -> action -> reducer -> store -> subscribed components re-render.

## Rules
1. Use native ES modules (`<script type="module">`). One responsibility per file.
2. Each UI piece is a Web Component (custom element) owning its markup, CSS and behavior
   in one folder. Code that two or more components repeat (shadow-root setup, stylesheet
   link, subscribe/unsubscribe, a DOM helper) lives in `components/base/`.
3. A component composes its own child components. Siblings communicate through the store,
   or through `CustomEvent`/`EventTarget` when the message is not state. Components read
   state from the store, dispatch actions, and reach data only through services.
4. `store/` exports a `createStore(reducer, initialState)` factory returning
   `getState()`, `dispatch(action)` and `subscribe(listener)`. The reducer updates state
   immutably. The entry module creates the store and the services once and assigns them
   as `store` and `services` properties to each top-level component, and each component
   passes them to its children. The base class renders once the element is connected and
   both `services` and `store` are set, in whatever order that happens.
5. Data access lives in `api/` wrappers (Repository pattern), use-case logic in
   `services/` (Service Layer), so swapping the data source never touches components.
   Durable state is written back by a `persistOnChange(store, repository)` subscriber in
   `services/`, which the entry module starts after rehydrating. It takes the repository
   as an argument so importing it never touches `localStorage` and tests can pass a fake.
   The reducer stays pure.
6. Singletons (config, logger) are module-level exports.
7. CSS: each component adds a `<link rel="stylesheet">` to its own shadow root for its
   sibling `.css` file, with the href resolved by `new URL('./x.css', import.meta.url).href`
   because a bare relative href resolves against the page. Design tokens are CSS custom
   properties in one `tokens.css` (they inherit into shadow roots). Styling lives in CSS
   files. Page-level CSS (`tokens.css`, `base.css`, plus `layouts.css` or `utilities.css`
   once a second use exists) goes in cascade layers, one `<link>` per file in the HTML and
   no `@import`. `base.css` is linked first and opens with
   `@layer reset, tokens, base, layouts, utilities;`. Page-level CSS, layered or not, never
   reaches into shadow roots, so component CSS stays unlayered, and CSS that two or more
   components share (a shadow-root reset, a visually-hidden helper) lives in
   `components/base/` and is linked like a sibling stylesheet.
8. Rendering: update only the nodes that changed, and leave a focused input in place.
   Write dynamic values with `textContent` or DOM APIs. Never use `innerHTML` with user
   or remote data.
9. Services and reducers are pure or take their dependencies as arguments. Cover them
   with plain `*.test.js` files run by `node --test`, a dev-only tool.

## Folder layout
```
index.html
package.json    {"type":"module"} and the test script, no dependencies
src/
  main.js       entry module, wiring only
  config.js     singletons (config, logger) as module-level exports
  components/base/            shared component code
  components/<name>/<name>.js + <name>.css
  store/        createStore, reducers, actions
  services/     use-case logic
  api/          fetch/localStorage wrappers
  styles/tokens.css  base.css (layer order, page-level reset)
```

## Scope
Keep to the layers above. Add an abstraction once code has a second use, and a library or
build tool only with the user's approval.

When unsure, choose the simplest option that keeps the folder boundaries, and state the
tradeoff in one sentence.

## Done when
Every rule above holds for each changed file. Check each rule against the changes and fix
every violation.
