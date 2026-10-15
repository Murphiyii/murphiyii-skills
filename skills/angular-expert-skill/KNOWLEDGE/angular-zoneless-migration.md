# Angular Zoneless & Migration Knowledge

## Zoneless Change Detection
Remove Zone.js overhead for better performance and debugging.
```typescript
bootstrapApplication(App, {
  providers: [
    provideZonelessChangeDetection()
  ]
});
```
- **Requirements**: Use `OnPush` strategy, Signals, `AsyncPipe`, or `markForCheck`.
- **SSR**: Use `PendingTasks` service to handle async operations during rendering.

## View Transitions
Native browser transitions for route changes.
```typescript
provideRouter(routes, withViewTransitions())
```
- **Customization**: Use CSS `::view-transition-old(root)` and `::view-transition-new(root)`.

## Migrating Animations
Replace `@angular/animations` with CSS animations (`animate.enter`/`animate.leave`) or native Web Animations API.

## Keeping Updated
Use `ng update` to update Angular and dependencies.
- Check [Angular Update Guide](https://update.angular.io/) for detailed instructions.

## Testing Zoneless
- **TestBed Setup**:
  ```typescript
  TestBed.configureTestingModule({
    providers: [provideZonelessChangeDetection()]
  });
  ```
- **Avoid `fixture.detectChanges()`**: Let Angular handle scheduling.
- **Use `await fixture.whenStable()`**: Wait for state synchronization.
- **Debug Mode**: Use `provideCheckNoChangesConfig({exhaustive: true})` to verify no bindings update without notification.
