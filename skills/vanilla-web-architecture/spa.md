# Single-page structure (SPA)

One HTML file, with client-side routing between page components. These rules extend
[SKILL.md](SKILL.md).

## Folder layout
Add to the base layout:
```
src/router.js        route table and navigation
src/pages/<route>.js + <route>.css   one page Web Component per route, styled like any component
```

## Rules
1. `main.js` wires: it creates the store and services, rehydrates durable state, awaits
   the initial content load, then starts the router. Page logic lives in the pages.
2. `router.js` owns a route table mapping each path to a page component loader, and
   renders the matching page into one outlet element. It includes a not-found route.
3. Routing uses the hash (`<a href="#/route">`), which needs no server configuration.
   Use the History API only when the user confirms the server falls back to `index.html`.
4. Each route loads its page with a dynamic `import()`, so the first load carries only the
   current page.
5. Page components are ordinary Web Components. They subscribe to the store in
   `connectedCallback` and unsubscribe in `disconnectedCallback`.
6. The store lives for the whole session, so state carries across routes. State that must
   survive a reload goes through an `api/` wrapper and is rehydrated when `main.js` creates
   the store.
7. On each route change, the router sets `document.title` from the route table's
   `title(params)` function (which may read loaded data through a service) and moves focus
   to the new page's main heading, so screen readers announce the navigation.
8. The router assigns `store` and `services` as properties on each page before attaching
   it. Route parameters reach pages as attributes.
