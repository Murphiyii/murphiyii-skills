# Multi-page structure (MPA)

One HTML file per route, with plain `<a href>` navigation and no client-side router.
These rules extend [SKILL.md](SKILL.md).

## Folder layout
Replace `index.html` and `src/main.js` with:
```
index.html
pages/<route>.html
src/pages/<route>.js   entry module for one page, wiring only
```

## Rules
1. Each page loads exactly one entry module (`src/pages/<route>.js`) that creates the
   store, registers the components the page uses and wires them. Beyond wiring, it only
   handles the back/forward cache rehydrate in rule 4 (shared code for it goes in
   `services/`) and the heading update in rule 7.
2. Each page imports only the modules it needs.
3. The shared page shell (header, nav, footer) is a Web Component imported by each page.
4. The store resets on every navigation. State that must survive a page change goes
   through an `api/` repository and is rehydrated when the page entry module creates the
   store. The repository uses `localStorage` (durable) by default, and `sessionStorage`
   (per-tab) only for state that must end with the tab. Transient values may travel in
   the URL query string.
   A page restored from the back/forward cache keeps its old store, so the entry module
   re-reads durable state on `pageshow` when `event.persisted` is true and on `storage`
   events. Otherwise a stale page can overwrite newer data on its next save.
5. Every page links the same `base.css` (first, it sets the layer order) and `tokens.css`.
   Page-specific CSS lives next to its entry module.
6. Page HTML files use relative paths for links, imports and assets, so the site works
   from any folder. Components build their hrefs from a `routes` map in `config.js` that
   resolves each path with `new URL(path, import.meta.url)`, so a link is correct from
   any page depth.
7. Each page has its own `<title>`, meta description and a single `<h1>`. When the
   heading depends on data, the page keeps a static `<h1>` placeholder and its entry
   module updates the `<h1>` and `document.title` from a store subscription.
