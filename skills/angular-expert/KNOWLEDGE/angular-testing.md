# Angular Testing Knowledge

## Unit Testing Setup
- **Framework**: Vitest (default for new projects since Angular v21) or Karma/Jasmine.
- **Vitest prerequisites**: New projects include `vitest` and `jsdom`. An existing Karma project installs them with `npm install --save-dev vitest jsdom`, then follows the Angular "Migrating from Karma to Vitest" guide.
- **Karma/Jasmine prerequisites**: `ng new <name> --test-runner=karma`, or in an existing project `npm install --save-dev karma karma-chrome-launcher karma-coverage karma-jasmine karma-jasmine-html-reporter jasmine-core @types/jasmine` plus the guide's `angular.json` setup.
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
- **Spies**: Use `jasmine.createSpyObj` (Karma/Jasmine) or `vi.fn()` (Vitest) to mock dependencies.
- **Injection**: `TestBed.inject(MyService)`.

## HTTP Testing
Use `provideHttpClientTesting()` and `HttpTestingController` to mock requests.
```typescript
const httpTesting = TestBed.inject(HttpTestingController);
const req = httpTesting.expectOne('/api/data');
req.flush({ data: 'test' });
```
