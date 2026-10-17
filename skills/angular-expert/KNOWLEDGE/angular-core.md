# Angular Core Concepts Knowledge

## Coding Style Guide
### Naming
- Separate words in file names with hyphens (`user-profile.ts`).
- Use `.spec.ts` for tests.
- Match file names to TypeScript identifiers.
- Use the same name for component files (`user-profile.ts`, `user-profile.html`, `user-profile.css`).

### Project Structure
- All code in `src`.
- Bootstrap in `src/main.ts`.
- Group related files (component, template, styles) in the same directory.
- Organize by feature areas (e.g., `movie-reel/show-times/`), not by type.
- One concept per file.

## Dependency Injection
### Prefer `inject` function
Prefer `inject()` over constructor injection for readability, type inference, and ES2022+ compatibility.
```typescript
private service = inject(MyService);
```

## Components
### Selectors
- Use custom element names with hyphens (`user-profile`).
- Use attribute selectors for directives or extending native elements (`button[yt-upload]`).
- Do NOT use `ng` prefix.

### Properties
- Group Angular properties (inputs, outputs, queries) before methods.
- Use `readonly` for properties initialized by Angular (`input()`, `output()`).
- Use `protected` for members used only in templates.

### Templates
- Avoid complex logic; use computed signals instead.
- Prefer `class` and `style` bindings over `ngClass` and `ngStyle`.
- Name event handlers for actions (`saveUserData()`), not events (`handleClick()`).

### Lifecycle
- Keep hooks simple; call named methods.
- Implement lifecycle interfaces (`implements OnInit`).

## Styling
- **Emulated (Default)**: Styles scoped to component.
- **ShadowDom**: Uses native Shadow DOM.
- **None**: Global styles.
- Avoid `::ng-deep`.
