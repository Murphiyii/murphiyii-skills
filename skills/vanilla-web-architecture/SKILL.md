---
name: vanilla-web-architecture
description: Architecture for maintainable no-framework web apps (plain HTML, CSS, JS; Web Components, one-way store, multi-page or single-page). Use when building or extending such an app.
allowed-tools: Bash(node --test *)
---

Plain HTML 5, CSS 3 and JavaScript only: no frameworks, no build step, no npm runtime
dependencies. ES modules do not load from `file://`, so develop and test through any
static server. If the user wants to open the app by double-clicking `index.html`, tell
them in one sentence that it needs a static server and give the command
(`python -m http.server`). Keep the modules: classic scripts would drop the import
structure this skill depends on.

Prerequisites: Node 20 or later for `node --test` (stable from v20.0.0), Python 3 for
`python -m http.server`, and the `package.json` script `"test": "node --test"`.

Multi-page app (one HTML file per route): also read [mpa.md](mpa.md).
Single-page app (client-side routing): also read [spa.md](spa.md).
If the user has not said which, default to a multi-page app.
A single-screen app uses the folder layout below alone; add mpa.md or spa.md when a
second page appears.

## Core pattern
Component-Based Architecture with a small Flux/Redux-lite store. State flows one way:
event -> action -> reducer -> store -> subscribed components re-render.

## Rules
1. Use native ES modules (`<script type="module">`). One responsibility per file.
2. Each UI piece is a Web Component (custom element) owning its markup (`<name>.html`),
   CSS and behavior in one folder. Code that two or more components repeat (template
   loading, shadow-root setup, stylesheet link, subscribe/unsubscribe, a DOM helper)
   lives in `components/base/`.
3. A component composes its own child components. Siblings communicate through the store,
   or through `CustomEvent`/`EventTarget` when the message is not state. Components read
   state from the store, dispatch actions, and reach data only through services.
4. `store/` exports a `createStore(reducer, initialState)` factory returning
   `getState()`, `dispatch(action)` and `subscribe(listener)`. The reducer updates state
   immutably. The entry module creates the store and the services once and assigns them
   as `store` and `services` properties to each top-level component, and each component
   passes them to its children. The base class renders once the element is connected and
   both `services` and `store` are set, in whatever order that happens.
5. Data access lives in `api/` repositories (Repository pattern), use-case logic in
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
8. Markup lives in HTML files so it can be edited by hand. Static markup never lives in
   JS strings or `createElement` chains. A component's structure is its `<name>.html`
   template, which the base class fetches once per component, caches and clones into the
   shadow root. Page content (headings, text, sections) is written in the page HTML as
   the element's children and shown through a `<slot>` in the template. Repeated items
   clone a nested `<template>` from the component's template file.
9. Rendering: JS only writes dynamic values. Find nodes by `data-ref` attributes, update
   only the nodes that changed, and leave a focused input in place. Write values with
   `textContent` or DOM properties. Parsing the component's own same-origin template
   file is the only `innerHTML` use. Never use `innerHTML` with user or remote data.
10. Services and reducers are pure or take their dependencies as arguments. Cover them
    with plain `*.test.js` files run by `node --test`, a dev-only tool.

## Folder layout
```
index.html
package.json    {"type":"module"} and the test script, no dependencies
src/
  main.js       entry module, wiring only
  config.js     singletons (config, logger) as module-level exports
  components/base/            shared component code
  components/<name>/<name>.js + <name>.html + <name>.css
  store/        createStore, reducers, actions
  services/     use-case logic
  api/          fetch/localStorage repositories
  styles/tokens.css  base.css (layer order, page-level reset)
```

## Wiring example
Rules 4, 5, 8 and 9 in code (reducer, service, repository and component CSS omitted):
```js
// components/base/base-element.js
const templates = new Map();

function loadTemplate(href) {
  if (!templates.has(href)) {
    templates.set(href, fetch(href).then(async (response) => {
      if (!response.ok) throw new Error(`Template ${href}: ${response.status}`);
      const template = document.createElement('template');
      template.innerHTML = await response.text();
      return template;
    }));
  }
  return templates.get(href);
}

export class BaseElement extends HTMLElement {
  #store;
  #services;
  #unsubscribe;
  #ready;
  get store() { return this.#store; }
  set store(value) { this.#store = value; this.#start(); }
  get services() { return this.#services; }
  set services(value) { this.#services = value; this.#start(); }
  connectedCallback() { this.#start(); }
  disconnectedCallback() { this.#unsubscribe?.(); this.#unsubscribe = null; }
  async #start() {
    if (!this.isConnected || !this.#store || !this.#services || this.#unsubscribe) return;
    this.#ready ??= loadTemplate(this.constructor.template).then((template) => {
      this.root = this.attachShadow({ mode: 'open' });
      this.root.append(template.content.cloneNode(true));
    });
    await this.#ready;
    // Re-check: the element may have been removed, or another call started it, during the fetch.
    if (!this.isConnected || this.#unsubscribe) return;
    this.render();
    this.#unsubscribe = this.#store.subscribe(() => this.render());
  }
}
```
```html
<!-- components/todo-count/todo-count.html -->
<p><span data-ref="count"></span> todos</p>
```
```js
// components/todo-count/todo-count.js
import { BaseElement } from '../base/base-element.js';

class TodoCount extends BaseElement {
  static template = new URL('./todo-count.html', import.meta.url).href;
  render() {
    this.count ??= this.root.querySelector('[data-ref="count"]');
    this.count.textContent = this.store.getState().todos.length;
  }
}
customElements.define('todo-count', TodoCount);

// main.js
import { createStore } from './store/store.js';
import { reducer, initialState } from './store/reducer.js';
import { todoRepository } from './api/todo-repository.js';
import { createTodoService } from './services/todo-service.js';
import { persistOnChange } from './services/persist-on-change.js';
import './components/todo-count/todo-count.js';

const services = { todos: createTodoService(todoRepository) };
const store = createStore(reducer, { ...initialState, ...todoRepository.load() });
persistOnChange(store, todoRepository);

const app = document.querySelector('todo-count');
app.services = services;
app.store = store;
```

## Scope
Keep to the layers above. Add an abstraction once code has a second use, and a library or
build tool only with the user's approval.

When unsure, choose the simplest option that keeps the folder boundaries, and state the
tradeoff in one sentence.

## Done when
Every rule above holds for each changed file. Check each rule against the changes and fix
every violation. Run `node --test` and fix any failures.
