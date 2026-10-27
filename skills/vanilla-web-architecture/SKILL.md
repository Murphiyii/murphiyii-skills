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
3. A component composes its own child components. It imports each child's module (a
   missing import leaves an undefined element that fails silently) and, in `setup()`,
   which the base class runs once after cloning the template, attaches its listeners and
   assigns its `store` and `services` to the children in its template. Siblings
   communicate through the store, or through `CustomEvent`/`EventTarget` when the message
   is not state. Components read state from the store, dispatch actions, and reach data
   only through services.
4. `store/` exports a `createStore(reducer, initialState)` factory returning
   `getState()`, `dispatch(action)` and `subscribe(listener)`. The reducer updates state
   immutably. The entry module creates the store and the services once and assigns them
   as `store` and `services` properties to each top-level component, and each component
   passes them to its children. The base class renders once the element is connected,
   both `services` and `store` are set, in whatever order that happens, and its template
   and stylesheets have loaded.
5. Data access lives in `api/` repositories (Repository pattern), use-case logic in
   `services/` (Service Layer), so swapping the data source never touches components.
   Durable state is written back by a `persistOnChange(store, repository)` subscriber in
   `services/`, which the entry module starts after rehydrating. It takes the repository
   as an argument so importing it never touches `localStorage` and tests can pass a fake.
   The reducer stays pure.
6. Singletons (config, logger) are module-level exports.
7. CSS: the base class adds a `<link rel="stylesheet">` to each shadow root for the
   shared `shadow-reset.css` and for the component's `static stylesheet`, its sibling
   `.css` file, with the href resolved by `new URL('./x.css', import.meta.url).href`
   because a bare relative href resolves against the page. It inserts the template only
   after these stylesheets load (or fail), so markup never shows unstyled, and fetches
   the template in parallel with them. Design tokens are CSS custom properties in one
   `tokens.css` (they inherit into shadow roots). Styling lives in CSS files. Page-level
   CSS (`tokens.css`, `base.css`, plus `layouts.css` or `utilities.css` once a second use
   exists) goes in cascade layers, one `<link>` per file in the HTML and no `@import`. A `<link>` cannot assign a layer, so each file wraps its rules in
   `@layer <name> { }` blocks (`base.css` holds both `reset` and `base`). `base.css` is
   linked first and opens with `@layer reset, tokens, base, layouts, utilities;`. Its
   `base` layer holds `:not(:defined) { visibility: hidden; }`, so page content inside a
   component stays hidden until its module defines it, instead of showing then vanishing.
   Page-level selectors never match inside shadow roots (only inherited properties such
   as custom properties, `color` and `font` cross), so component CSS stays unlayered.
   `shadow-reset.css` in `components/base/` holds CSS that every component shares,
   including `[hidden], :host([hidden]) { display: none !important; }` so component
   `display` rules never override the `hidden` attribute.
8. Markup lives in HTML files so it can be edited by hand. Static markup never lives in
   JS strings or `createElement` chains. A component's structure is its `<name>.html`
   template, which the base class fetches once per component, caches and clones into the
   shadow root. Page content (headings, text, sections) is written in the page HTML as
   the element's children and shown through a `<slot>` in the template. Repeated items
   clone a nested `<template>` from the component's template file with
   `document.importNode(template.content, true)`, which upgrades already-defined child
   components immediately, so setting their `store` and `services` before insertion
   reaches the base class setters. Relative URLs inside a template (`src`, `href`)
   resolve against the page, not the template file, so JS sets them with
   `new URL(path, import.meta.url)` (in an MPA, from the `routes` map). Fixed states
   (loading, empty, not found) are elements already in the HTML that JS shows or hides
   with `hidden`. JS creates elements only for the base class's `<template>` and `<link>`
   elements and, in an SPA, the router's page elements.
9. Rendering: JS only writes dynamic values. The base class collects `[data-ref]` nodes
   into `this.refs`. Keep node references there and never name a component field after a
   built-in element property such as `title`, `hidden` or `id`, which it would silently
   shadow. Update only the nodes that changed, and leave a focused input in place.
   Write values with `textContent` or DOM properties. A component that reads attributes
   lists them in `static observedAttributes`, and the base class re-renders when they
   change. Parsing the component's own same-origin template file is
   the only `innerHTML` use. Never use `innerHTML` with user or remote data.
10. Services and reducers are pure or take their dependencies as arguments. Cover them
    with plain `*.test.js` files run by `node --test`, a dev-only tool.

## Folder layout
```
index.html
package.json    {"type":"module"} and the test script, no dependencies
src/
  main.js       entry module, wiring only
  config.js     singletons (config, logger) as module-level exports
  components/base/            base-element.js, shadow-reset.css, other shared component code
  components/<name>/<name>.js + <name>.html + <name>.css
  store/        createStore, reducers, actions
  services/     use-case logic
  api/          fetch/localStorage repositories
  styles/tokens.css  base.css (layer order, page-level reset)
```

## Wiring example
Rules 4, 5, 7, 8 and 9 in code (reducer, service, repository and CSS omitted):
```js
// components/base/base-element.js
const templates = new Map();
const resetHref = new URL('./shadow-reset.css', import.meta.url).href;

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

function stylesheetLink(href) {
  const link = document.createElement('link');
  link.rel = 'stylesheet';
  link.href = href;
  return link;
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
  attributeChangedCallback() { if (this.#unsubscribe) this.render(); }
  async #prepare() {
    this.root = this.attachShadow({ mode: 'open' });
    const links = [resetHref, this.constructor.stylesheet].filter(Boolean).map(stylesheetLink);
    const styled = links.map((link) => new Promise((resolve) => {
      link.onload = link.onerror = resolve;
    }));
    this.root.append(...links);
    // Insert the markup only once its CSS has loaded, so it never shows unstyled.
    const [template] = await Promise.all([loadTemplate(this.constructor.template), ...styled]);
    this.root.append(document.importNode(template.content, true));
    const refs = this.root.querySelectorAll('[data-ref]');
    this.refs = Object.fromEntries([...refs].map((node) => [node.dataset.ref, node]));
    this.setup?.();
  }
  async #start() {
    if (!this.isConnected || !this.#store || !this.#services || this.#unsubscribe) return;
    this.#ready ??= this.#prepare();
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
  static stylesheet = new URL('./todo-count.css', import.meta.url).href;
  render() {
    this.refs.count.textContent = this.store.getState().todos.length;
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
