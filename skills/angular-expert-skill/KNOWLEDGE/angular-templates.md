# Angular Templates & Directives Knowledge

## Deferrable Views (@defer)
Deferrable views allow you to lazy load components and their dependencies.

### Syntax
```html
@defer (on viewport) {
  <large-component />
} @placeholder (minimum 500ms) {
  <p>Placeholder</p>
} @loading (after 100ms; minimum 1s) {
  <img src="loading.gif" />
} @error {
  <p>Failed to load</p>
}
```

### Triggers
- `on idle`: Default. Loads when browser is idle.
- `on viewport`: Loads when element enters viewport.
- `on interaction`: Loads on click/keydown.
- `on hover`: Loads on mouseover.
- `on immediate`: Loads immediately after non-deferred content.
- `on timer(500ms)`: Loads after delay.
- `when condition`: Loads when condition is true.

## Expression Syntax
- **Supported**: Standard JS operators, ternary, nullish coalescing.
- **Unsupported**: Bitwise, destructuring, `new`, `typeof`, `void`.
- **Pipes**: `{{ value | pipe }}`
- **Optional Chaining**: `obj?.prop` (returns null if undefined).

## Attribute Directives
Directives that change the appearance or behavior of an element.

### Creation
```typescript
@Directive({ selector: '[appHighlight]' })
export class HighlightDirective {
  private el = inject(ElementRef);
  
  constructor() {
    this.el.nativeElement.style.backgroundColor = 'yellow';
  }
}
```

### Handling Events
Use `host` property in decorator:
```typescript
@Directive({
  selector: '[appHighlight]',
  host: {
    '(mouseenter)': 'onMouseEnter()',
    '(mouseleave)': 'onMouseLeave()'
  }
})
class HighlightDirective { ... }
```

## Structural Directives
Directives that change the DOM layout by adding/removing elements (`*ngIf`, `*ngFor`).

### Shorthand Syntax
`*directive="exp"` expands to `<ng-template [directive]="exp">`.

### Creating Custom Structural Directive
Inject `TemplateRef` and `ViewContainerRef`.
```typescript
@Directive({ selector: '[appSelect]' })
export class SelectDirective {
  private templateRef = inject(TemplateRef);
  private viewContainer = inject(ViewContainerRef);
  
  // Logic to createEmbeddedView
}
```
