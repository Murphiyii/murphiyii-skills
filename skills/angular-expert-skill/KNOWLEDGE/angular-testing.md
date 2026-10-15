# Angular Testing Knowledge

## Unit Testing Setup
- **Framework**: Vitest (default in new CLI) or Karma/Jasmine.
- **Command**: `ng test`
- **Configuration**: `angular.json` or `vitest.config.ts`.

## TestBed
The primary utility for testing Angular artifacts.
```typescript
beforeEach(async () => {
  await TestBed.configureTestingModule({
    imports: [MyComponent], // Standalone component
    providers: [provideHttpClient(), provideHttpClientTesting()]
  }).compileComponents();
  
  fixture = TestBed.createComponent(MyComponent);
  component = fixture.componentInstance;
  fixture.detectChanges(); // Trigger initial data binding
});
```

## Component Testing
- **ComponentFixture**: Wrapper around component and its DOM.
- **DebugElement**: Platform-agnostic wrapper around DOM element.
- **Querying**:
  ```typescript
  const h1 = fixture.debugElement.query(By.css('h1'));
  expect(h1.nativeElement.textContent).toContain('Hello');
  ```

## Service Testing
- **Spies**: Use `jasmine.createSpyObj` or `vi.fn()` to mock dependencies.
- **Injection**: `TestBed.inject(MyService)`.

## HTTP Testing
Use `provideHttpClientTesting()` and `HttpTestingController` to mock requests.
```typescript
const httpTesting = TestBed.inject(HttpTestingController);
const req = httpTesting.expectOne('/api/data');
req.flush({ data: 'test' });
```
