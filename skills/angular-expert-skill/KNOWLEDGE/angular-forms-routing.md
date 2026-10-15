# Angular Forms & Routing Knowledge

## Forms
### Reactive Forms
- **FormControl**: Tracks value and validity of individual control.
- **FormGroup**: Tracks value and validity of a group of controls.
- **FormBuilder**: Helper to create forms.

### Validation
- **Built-in**: `Validators.required`, `Validators.minLength(4)`, `Validators.email`.
- **Custom**: Function returning `ValidationErrors | null`.
- **Async**: Function returning `Observable<ValidationErrors | null>`.
- **Cross-field**: Validator on `FormGroup` to compare children.

### Dynamic Forms
Use `FormGroup` and `FormArray` to build forms from metadata (e.g., JSON configuration).

## Routing
### Basics
- **Routes**: Array of route objects (`{ path: 'home', component: HomeComponent }`).
- **RouterOutlet**: `<router-outlet />` placeholder for routed components.
- **RouterLink**: `[routerLink]="['/user', userId]"` for navigation.

### Configuration
- **Parameters**: `path: 'user/:id'`
- **Wildcard**: `path: '**'` (404 page)
- **Redirects**: `redirectTo: '/home'`
- **Page Titles**: `title: 'Home Page'` or `ResolveFn`.

### Lazy Loading
Use `loadComponent` for standalone components.
```typescript
{
  path: 'admin',
  loadComponent: () => import('./admin/admin.component').then(m => m.AdminComponent)
}
```
- **Injection Context**: `loadComponent` runs in route context (can use `inject`).
